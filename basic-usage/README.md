# Carnil Basic Usage Example

A simple Node.js example demonstrating basic usage of the Carnil SDK with Stripe provider.

## Features

- 💳 **Payment Processing** - Create payment intents
- 👥 **Customer Management** - Create and manage customers
- 🔄 **Subscription Handling** - Create and manage subscriptions
- 📄 **Invoice Management** - Create and manage invoices
- 💰 **Refund Processing** - Process refunds

## Prerequisites

- Node.js 18+
- Stripe account with API keys
- npm or yarn

## Installation

```bash
# Clone the repository
git clone https://github.com/Carnil-Dev/carnil-sdk.git
cd carnil-sdk/examples/basic-usage

# Install dependencies
npm install
```

## Configuration

1. Copy your Stripe API keys to the environment:

```bash
# Create .env file
echo "STRIPE_API_KEY=sk_test_..." > .env
echo "STRIPE_WEBHOOK_SECRET=whsec_..." >> .env
```

2. Update the configuration in `index.js`:

```javascript
const carnil = new Carnil({
  provider: {
    provider: "stripe",
    apiKey: process.env.STRIPE_API_KEY,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
  },
  debug: true,
});
```

## Usage

### Running the Example

```bash
npm start
```

### Example Operations

The example demonstrates the following operations:

1. **Customer Creation**

   ```javascript
   const customer = await carnil.createCustomer({
     email: "customer@example.com",
     name: "John Doe",
   });
   ```

2. **Payment Intent Creation**

   ```javascript
   const paymentIntent = await carnil.createPaymentIntent({
     amount: 2000, // $20.00
     currency: "usd",
     customerId: customer.data.id,
   });
   ```

3. **Subscription Creation**

   ```javascript
   const subscription = await carnil.createSubscription({
     customerId: customer.data.id,
     priceId: "price_123",
   });
   ```

4. **Invoice Creation**

   ```javascript
   const invoice = await carnil.createInvoice({
     customerId: customer.data.id,
     items: [
       {
         priceId: "price_123",
         quantity: 1,
       },
     ],
   });
   ```

5. **Refund Processing**
   ```javascript
   const refund = await carnil.createRefund({
     paymentId: paymentIntent.data.id,
     amount: 1000, // $10.00
   });
   ```

## Code Structure

```
basic-usage/
├── index.js          # Main example file
├── package.json      # Dependencies and scripts
└── README.md         # This file
```

## Error Handling

The example includes comprehensive error handling:

```javascript
try {
  const result = await carnil.createPaymentIntent({
    amount: 2000,
    currency: "usd",
  });
  console.log("Payment intent created:", result.data.id);
} catch (error) {
  console.error("Error creating payment intent:", error.message);
}
```

## Testing

### Test Mode

Use Stripe test mode for development:

```bash
# Set test API key
export STRIPE_API_KEY=sk_test_...

# Run the example
npm start
```

### Test Cards

Use Stripe test cards for payment testing:

- **Success**: `4242424242424242`
- **Decline**: `4000000000000002`
- **Insufficient Funds**: `4000000000009995`

## Webhook Testing

### Using Stripe CLI

1. Install Stripe CLI:

   ```bash
   # macOS
   brew install stripe/stripe-cli/stripe

   # Windows
   # Download from https://github.com/stripe/stripe-cli/releases
   ```

2. Login to Stripe:

   ```bash
   stripe login
   ```

3. Forward webhooks:
   ```bash
   stripe listen --forward-to localhost:3000/webhooks
   ```

### Webhook Handler

```javascript
app.post(
  "/webhooks",
  express.raw({ type: "application/json" }),
  async (req, res) => {
    try {
      const signature = req.headers["stripe-signature"];
      const payload = req.body;

      const isValid = await carnil.verifyWebhook(
        payload,
        signature,
        process.env.STRIPE_WEBHOOK_SECRET
      );
      if (!isValid) {
        return res.status(400).json({ error: "Invalid signature" });
      }

      const event = await carnil.parseWebhook(
        payload,
        signature,
        process.env.STRIPE_WEBHOOK_SECRET
      );

      switch (event.type) {
        case "payment_intent.succeeded":
          console.log("Payment succeeded:", event.data);
          break;
        case "payment_intent.payment_failed":
          console.log("Payment failed:", event.data);
          break;
        default:
          console.log("Unhandled event type:", event.type);
      }

      res.status(200).json({ received: true });
    } catch (error) {
      console.error("Webhook error:", error);
      res.status(500).json({ error: "Internal server error" });
    }
  }
);
```

## Production Deployment

### Environment Variables

Set production environment variables:

```bash
# Production Stripe keys
export STRIPE_API_KEY=sk_live_...
export STRIPE_WEBHOOK_SECRET=whsec_...
```

### Security Considerations

1. **API Key Security**: Never expose API keys in client-side code
2. **Webhook Verification**: Always verify webhook signatures
3. **Error Handling**: Implement proper error handling and logging
4. **Rate Limiting**: Implement rate limiting for API endpoints

## Troubleshooting

### Common Issues

1. **Invalid API Key**

   ```
   Error: Invalid API key provided
   ```

   Solution: Check your Stripe API key in the environment variables

2. **Webhook Verification Failed**

   ```
   Error: Invalid webhook signature
   ```

   Solution: Verify your webhook secret and signature generation

3. **Payment Failed**
   ```
   Error: Payment failed
   ```
   Solution: Check payment method and customer details

### Debug Mode

Enable debug mode for detailed logging:

```javascript
const carnil = new Carnil({
  provider: {
    provider: "stripe",
    apiKey: process.env.STRIPE_API_KEY,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
  },
  debug: true, // Enable debug logging
});
```

## Next Steps

After running this basic example, you can:

1. **Explore Advanced Features**: Check out the Next.js and React examples
2. **Add More Providers**: Integrate Razorpay or other payment providers
3. **Build Your Application**: Use this as a foundation for your payment system
4. **Add Analytics**: Implement usage tracking and analytics
5. **Add Compliance**: Implement PCI DSS and GDPR compliance features

## Resources

- [Carnil Documentation](https://docs.carnil.dev)
- [Stripe Documentation](https://stripe.com/docs)
- [Carnil GitHub Repository](https://github.com/Carnil-Dev/carnil-sdk)
- [Discord Community](https://discord.gg/carnil)

## Support

- 📖 [Documentation](https://docs.carnil.dev)
- 💬 [Discord Community](https://discord.gg/carnil)
- 🐛 [Report Issues](https://github.com/Carnil-Dev/carnil-sdk/issues)
- 📧 [Email Support](mailto:hello@carnil.dev)
