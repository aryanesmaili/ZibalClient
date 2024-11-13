# Zibal .NET Client Library

A modern .NET client library for integrating with the Zibal payment gateway. This library provides a simple and type-safe way to interact with Zibal's payment services, supporting both standard and advanced payment workflows.

## Features

- Full support for standard and advanced payment operations
- Strongly-typed request/response models
- Async/await pattern support
- Built-in validation for payment parameters
- Support for multiplexing payments (split payments between multiple beneficiaries)
- Lazy payment request handling
- transaction status inquiry

## Disclaimer
### This is ***Not*** an official package by the Zibal Team. this is my implementation of their documentation made to be used by other developers.

## Installation

```bash
dotnet add package ZibalClient  
```

## Quick Start

### Non-DI Projects:
```csharp
// Create an instance of ZibalClient
using var httpClient = new HttpClient();
var ZibalClient = new ZibalClient(httpClient);

// Create a basic payment request
CreateTransactionRequest request = new()
{
    Merchant = "YOUR_MERCHANT_INFO",
    Amount = 10000,  // Amount to pay in Rials
    CallbackURL = "https://your-website.com/payment/callback", // Where Zibal Will Send Transaction Information.
    IsTest = true, // supported... will set merchant as Zibal as per document.
    Description = "Payment for Order #1984",
    Mobile = "09xxxxxxxxx"
};

// Request a new transaction from Zibal.
var response = await ZibalClient.RequestTransactionAsync(request);

// Get the payment URL
var paymentUrl = $"https://gateway.zibal.ir/start/{response.TrackID}"; 
```

### Dependancy Injection:

#### in program.cs:
```csharp
builder.services.AddHttpClient<ZibalClient>();
```
#### Or if you want to specify object lifetime:
```csharp
services.AddHttpClient<ZibalClient>().AddScoped<ZibalClient>();
```
#### In your service (e.g PaymentService):
```csharp
private readonly ZibalClient _zibalClient;

public PaymentService(ZibalClient zibalClient)
{
    _zibalClient = zibalClient;
}
```

## Usage Examples

### Basic Payment Flow

1. **Request a Transaction**
```csharp
var request = new CreateTransactionRequest
{
    Merchant = "YOUR_MERCHANT_INFO",
    Amount = 10000,  // Amount to pay in Rials
    CallbackURL = "https://your-website.com/payment/callback", // Where Zibal Will Send Transaction Information.
    IsTest = true, // supported... will set merchant as Zibal as per document.
    Description = "Payment for Order #1984",
    Mobile = "09xxxxxxxxx"
};

var response = await ZibalClient.RequestTransactionAsync(request);
```

2. **Verify the Transaction**
```csharp
var verifyRequest = new VerifyTransactionRequest
{
    Merchant = "YOUR_MERCHANT_INFO",
    TrackID = response.TrackID
};

var verificationResponse = await ZibalClient.VerifyTransactionAsync(verifyRequest);
```

### Advanced Payment (Multiplexing)

```csharp
var advancedRequest = new CreateAdvancedTransactionRequest
{
    Merchant = "YOUR_MERCHANT_INFO",
    Amount = 10000,
    CallbackURL = "https://your-website.com/payment/callback",
    PercentMode = 0,  // 0 for fixed amounts, 1 for percentages
    FeeMode = 0,      // 0: from transaction, 1: from wallet, 2: paid by client
    MultiplexingInfos = new List<MultiplexingInformation>
    {
        new()
        {
            SubMerchantID = "MERCHANT1",
            Amount = 7000,
            WagePayer = true
        },
        new()
        {
            BankAccount = "SHABA Number",
            Amount = 3000
        }
    }
};

var response = await ZibalClient.RequestTransactionAsync(advancedRequest, isAdvanced: true);
```

### Transaction Status Inquiry

```csharp
var inquiryRequest = new InquiryTransactionRequest
{
    Merchant = "YOUR_MERCHANT_INFO",
    TrackID = trackId
};

var status = await ZibalClient.GetTransactionStatusAsync(inquiryRequest);
```

### Card Number Restrictions

```csharp
var request = new CreateTransactionRequest
{
    // ... other properties ...
    AllowedCards = new List<string> { "6219861012345678", "6274121234567890" }
};
```

### National Code Validation

```csharp
var request = new CreateTransactionRequest
{
    // ... other properties ...
    NationalCode = "1234567890",
    CheckMobileWithCard = true  // Validates mobile number matches card owner
};
```
## Important Lazy Mode Note:
#### Since Zibal uses `Post` requests and `Json` to inform you about the transaction in `Lazy Mode`, I have also written a class to deserialize that information called `LazyCallbackResponse`.
```csharp
using System.Text.Json;

var transactionInfo = await JsonSerializer.DeserializeAsync<LazyCallbackResponse>(await response.Content.ReadAsStreamAsync());
```

## Error Handling

The library provides detailed error information through response codes and messages. Check the `Result` and `Message` properties in response objects for error details.

## Testing

Set `IsTest = true` in your requests to use Zibal's test environment:

```csharp
var request = new CreateTransactionRequest
{
    IsTest = true,
    // ... other properties ...
};
```

## Response Status Codes

For a complete list of status codes and their meanings, refer to [Zibal's API documentation](https://help.zibal.ir/IPG/API/#status-codes).

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Zibal API Documentation
- Contributors if there will be any!

## Support

For support with this library, please open an issue on GitHub. For Zibal-specific questions, please contact Zibal support directly.
