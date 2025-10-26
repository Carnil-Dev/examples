# Carnil Next.js App Example

A complete Next.js application demonstrating the integration of Carnil SDK with React components, API routes, and webhook handling.

## Features

- ⚛️ **React Integration** - Full React component integration
- 🚀 **Next.js API Routes** - Server-side payment processing
- 🔔 **Webhook Handling** - Secure webhook verification and processing
- 💳 **Payment Processing** - Complete payment flow implementation
- 👥 **Customer Management** - Customer creation and management
- 🔄 **Subscription Handling** - Recurring billing implementation
- 📱 **Responsive Design** - Mobile-friendly UI components
- 🎨 **Modern UI** - Beautiful and intuitive user interface

## Prerequisites

- Node.js 18+
- Next.js 14+
- Stripe account with API keys
- npm or yarn

## Installation

```bash
# Clone the repository
git clone https://github.com/Carnil-Dev/carnil-sdk.git
cd carnil-sdk/examples/nextjs-app

# Install dependencies
npm install
```

## Configuration

1. Copy the environment variables:

```bash
# Copy environment template
cp env.example .env.local

# Update with your API keys
STRIPE_API_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

2. Update the configuration in `app/api/carnil/route.ts`:

```typescript
export const POST = createCarnilHandler({
  provider: {
    provider: "stripe",
    apiKey: process.env.STRIPE_API_KEY!,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  },
  debug: process.env.NODE_ENV === "development",
});
```

## Project Structure

```
nextjs-app/
├── app/
│   ├── api/
│   │   ├── carnil/
│   │   │   └── route.ts          # Carnil API handler
│   │   └── webhook/
│   │       └── route.ts          # Webhook handler
│   ├── components/
│   │   └── client-carnil-provider.tsx
│   ├── globals.css               # Global styles
│   ├── layout.tsx                # Root layout
│   └── page.tsx                  # Home page
├── env.example                   # Environment variables template
├── next-env.d.ts                 # Next.js type definitions
├── package.json                  # Dependencies and scripts
├── tsconfig.json                 # TypeScript configuration
└── README.md                     # This file
```

## Usage

### Development

```bash
# Start development server
npm run dev

# Open http://localhost:3000
```

### Production

```bash
# Build the application
npm run build

# Start production server
npm start
```

## API Routes

### Carnil API Handler

The main API route handles all Carnil operations:

```typescript
// app/api/carnil/route.ts
import { createCarnilHandler } from "@carnil/next";

export const POST = createCarnilHandler({
  provider: {
    provider: "stripe",
    apiKey: process.env.STRIPE_API_KEY!,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  },
  debug: process.env.NODE_ENV === "development",
});
```

### Webhook Handler

Handles incoming webhooks from payment providers:

```typescript
// app/api/webhook/route.ts
import { createWebhookHandler } from "@carnil/next";

export const POST = createWebhookHandler({
  webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  handlers: {
    "payment.succeeded": async (event) => {
      console.log("Payment succeeded:", event.data);
      // Handle payment success
    },
    "payment.failed": async (event) => {
      console.log("Payment failed:", event.data);
      // Handle payment failure
    },
  },
});
```

## React Components

### Client Carnil Provider

```tsx
// app/components/client-carnil-provider.tsx
"use client";

import { CarnilProvider } from "@carnil/react";

export function ClientCarnilProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <CarnilProvider
      config={{
        provider: {
          provider: "stripe",
          apiKey: process.env.NEXT_PUBLIC_STRIPE_API_KEY!,
          webhookSecret: process.env.NEXT_PUBLIC_WEBHOOK_SECRET!,
        },
        debug: process.env.NODE_ENV === "development",
      }}
    >
      {children}
    </CarnilProvider>
  );
}
```

### Payment Form Component

```tsx
// app/components/payment-form.tsx
"use client";

import { useState } from "react";
import { usePaymentIntent } from "@carnil/react";

export function PaymentForm() {
  const [amount, setAmount] = useState(2000);
  const [loading, setLoading] = useState(false);

  const handlePayment = async () => {
    setLoading(true);
    try {
      const response = await fetch("/api/carnil", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          action: "createPaymentIntent",
          data: { amount, currency: "usd" },
        }),
      });

      const result = await response.json();
      if (result.success) {
        console.log("Payment intent created:", result.data);
      }
    } catch (error) {
      console.error("Payment failed:", error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-4">Payment Form</h2>

      <div className="mb-4">
        <label className="block text-sm font-medium mb-2">Amount</label>
        <input
          type="number"
          value={amount}
          onChange={(e) => setAmount(parseInt(e.target.value))}
          className="w-full p-2 border rounded-md"
          placeholder="Enter amount in cents"
        />
      </div>

      <button
        onClick={handlePayment}
        disabled={loading}
        className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 disabled:opacity-50"
      >
        {loading ? "Processing..." : `Pay $${(amount / 100).toFixed(2)}`}
      </button>
    </div>
  );
}
```

### Customer Dashboard

```tsx
// app/components/customer-dashboard.tsx
"use client";

import { useCustomer, usePaymentMethods } from "@carnil/react";

export function CustomerDashboard({ customerId }: { customerId: string }) {
  const { customer, loading: customerLoading } = useCustomer(customerId);
  const { paymentMethods, loading: methodsLoading } =
    usePaymentMethods(customerId);

  if (customerLoading || methodsLoading) {
    return <div>Loading...</div>;
  }

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">Customer Dashboard</h1>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {/* Customer Info */}
        <div className="bg-white p-6 rounded-lg shadow-md">
          <h2 className="text-xl font-semibold mb-4">Customer Information</h2>
          {customer && (
            <div>
              <p>
                <strong>Name:</strong> {customer.name || "N/A"}
              </p>
              <p>
                <strong>Email:</strong> {customer.email}
              </p>
              <p>
                <strong>ID:</strong> {customer.id}
              </p>
              <p>
                <strong>Created:</strong>{" "}
                {customer.createdAt.toLocaleDateString()}
              </p>
            </div>
          )}
        </div>

        {/* Payment Methods */}
        <div className="bg-white p-6 rounded-lg shadow-md">
          <h2 className="text-xl font-semibold mb-4">Payment Methods</h2>
          {paymentMethods.length === 0 ? (
            <p>No payment methods found</p>
          ) : (
            <ul className="space-y-2">
              {paymentMethods.map((method) => (
                <li
                  key={method.id}
                  className="flex justify-between items-center"
                >
                  <span>{method.type}</span>
                  {method.isDefault && (
                    <span className="text-green-600 text-sm">Default</span>
                  )}
                </li>
              ))}
            </ul>
          )}
        </div>
      </div>
    </div>
  );
}
```

## Styling

### Global Styles

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto",
    "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans",
    "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}
```

### Tailwind Configuration

```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: "#eff6ff",
          500: "#3b82f6",
          600: "#2563eb",
          700: "#1d4ed8",
        },
      },
    },
  },
  plugins: [],
};
```

## Webhook Testing

### Using Stripe CLI

1. Install Stripe CLI:

   ```bash
   brew install stripe/stripe-cli/stripe
   ```

2. Login to Stripe:

   ```bash
   stripe login
   ```

3. Forward webhooks to your local server:
   ```bash
   stripe listen --forward-to localhost:3000/api/webhook
   ```

### Webhook Events

The application handles the following webhook events:

- `payment_intent.succeeded` - Payment completed successfully
- `payment_intent.payment_failed` - Payment failed
- `customer.subscription.created` - New subscription created
- `customer.subscription.updated` - Subscription updated
- `customer.subscription.deleted` - Subscription cancelled
- `invoice.payment_succeeded` - Invoice payment succeeded
- `invoice.payment_failed` - Invoice payment failed

## Deployment

### Vercel Deployment

1. Connect your repository to Vercel
2. Set environment variables in Vercel dashboard:
   ```
   STRIPE_API_KEY=sk_live_...
   STRIPE_WEBHOOK_SECRET=whsec_...
   NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
   ```
3. Deploy your application

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

## Testing

### Unit Tests

```typescript
// __tests__/api/carnil.test.ts
import { POST } from "@/app/api/carnil/route";
import { NextRequest } from "next/server";

describe("/api/carnil", () => {
  it("should create payment intent", async () => {
    const request = new NextRequest("http://localhost:3000/api/carnil", {
      method: "POST",
      body: JSON.stringify({
        action: "createPaymentIntent",
        data: { amount: 2000, currency: "usd" },
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data.success).toBe(true);
  });
});
```

### Integration Tests

```typescript
// __tests__/integration/payment-flow.test.ts
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { PaymentForm } from "@/app/components/payment-form";

describe("Payment Flow", () => {
  it("should process payment successfully", async () => {
    render(<PaymentForm />);

    const amountInput = screen.getByPlaceholderText("Enter amount in cents");
    const payButton = screen.getByText(/Pay/);

    fireEvent.change(amountInput, { target: { value: "2000" } });
    fireEvent.click(payButton);

    await waitFor(() => {
      expect(screen.getByText("Processing...")).toBeInTheDocument();
    });
  });
});
```

## Security Considerations

1. **API Key Security**: Never expose API keys in client-side code
2. **Webhook Verification**: Always verify webhook signatures
3. **Input Validation**: Validate all user inputs
4. **Rate Limiting**: Implement rate limiting for API endpoints
5. **HTTPS**: Always use HTTPS in production

## Troubleshooting

### Common Issues

1. **API Key Not Found**

   ```
   Error: API key not found
   ```

   Solution: Check your environment variables

2. **Webhook Verification Failed**

   ```
   Error: Invalid webhook signature
   ```

   Solution: Verify your webhook secret

3. **CORS Issues**
   ```
   Error: CORS policy blocked
   ```
   Solution: Configure CORS headers in API routes

### Debug Mode

Enable debug mode for detailed logging:

```typescript
const carnil = new Carnil({
  provider: {
    provider: "stripe",
    apiKey: process.env.STRIPE_API_KEY!,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  },
  debug: process.env.NODE_ENV === "development",
});
```

## Next Steps

After running this example, you can:

1. **Add More Features**: Implement subscriptions, invoices, and refunds
2. **Add Authentication**: Implement user authentication and authorization
3. **Add Analytics**: Implement usage tracking and analytics
4. **Add Compliance**: Implement PCI DSS and GDPR compliance
5. **Add Testing**: Implement comprehensive testing suite

## Resources

- [Carnil Documentation](https://docs.carnil.dev)
- [Next.js Documentation](https://nextjs.org/docs)
- [Stripe Documentation](https://stripe.com/docs)
- [Carnil GitHub Repository](https://github.com/Carnil-Dev/carnil-sdk)
- [Discord Community](https://discord.gg/carnil)

## Support

- 📖 [Documentation](https://docs.carnil.dev)
- 💬 [Discord Community](https://discord.gg/carnil)
- 🐛 [Report Issues](https://github.com/Carnil-Dev/carnil-sdk/issues)
- 📧 [Email Support](mailto:hello@carnil.dev)
