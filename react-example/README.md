# Carnil React Example

A simple React application demonstrating the integration of Carnil SDK with React components and hooks for building payment applications.

## Features

- ⚛️ **React Integration** - Full React component integration
- 🎣 **Custom Hooks** - Easy-to-use hooks for payment operations
- 💳 **Payment Processing** - Complete payment flow implementation
- 👥 **Customer Management** - Customer creation and management
- 🔄 **Subscription Handling** - Recurring billing implementation
- 📱 **Responsive Design** - Mobile-friendly UI components
- 🎨 **Modern UI** - Beautiful and intuitive design

## Prerequisites

- Node.js 18+
- React 18+
- Stripe account with API keys
- npm or yarn

## Installation

```bash
# Clone the repository
git clone https://github.com/Carnil-Dev/carnil-sdk.git
cd carnil-sdk/examples/react-example

# Install dependencies
npm install
```

## Configuration

1. Copy your Stripe API keys to the environment:

```bash
# Create .env file
echo "REACT_APP_STRIPE_API_KEY=sk_test_..." > .env
echo "REACT_APP_STRIPE_WEBHOOK_SECRET=whsec_..." >> .env
```

2. Update the configuration in `src/App.tsx`:

```tsx
import { CarnilProvider } from "@carnil/react";

function App() {
  return (
    <CarnilProvider
      config={{
        provider: {
          provider: "stripe",
          apiKey: process.env.REACT_APP_STRIPE_API_KEY!,
          webhookSecret: process.env.REACT_APP_STRIPE_WEBHOOK_SECRET!,
        },
        debug: process.env.NODE_ENV === "development",
      }}
    >
      <YourApp />
    </CarnilProvider>
  );
}
```

## Project Structure

```
react-example/
├── public/
│   └── index.html              # HTML template
├── src/
│   ├── App.tsx                 # Main App component
│   ├── App.css                 # App styles
│   └── index.js                # Entry point
├── package.json                # Dependencies and scripts
└── README.md                   # This file
```

## Usage

### Development

```bash
# Start development server
npm start

# Open http://localhost:3000
```

### Production

```bash
# Build the application
npm run build

# Serve the production build
npx serve -s build
```

## React Components

### Payment Form Component

```tsx
// src/components/PaymentForm.tsx
import React, { useState } from "react";
import { usePaymentIntent } from "@carnil/react";

export function PaymentForm() {
  const [amount, setAmount] = useState(2000);
  const [loading, setLoading] = useState(false);
  const [paymentIntent, setPaymentIntent] = useState(null);

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
        setPaymentIntent(result.data);
        console.log("Payment intent created:", result.data);
      }
    } catch (error) {
      console.error("Payment failed:", error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="payment-form">
      <h2>Payment Form</h2>

      <div className="form-group">
        <label htmlFor="amount">Amount (cents)</label>
        <input
          id="amount"
          type="number"
          value={amount}
          onChange={(e) => setAmount(parseInt(e.target.value))}
          placeholder="Enter amount in cents"
        />
      </div>

      <button
        onClick={handlePayment}
        disabled={loading}
        className="btn btn-primary"
      >
        {loading ? "Processing..." : `Pay $${(amount / 100).toFixed(2)}`}
      </button>

      {paymentIntent && (
        <div className="payment-result">
          <h3>Payment Intent Created</h3>
          <p>ID: {paymentIntent.id}</p>
          <p>Amount: ${(paymentIntent.amount / 100).toFixed(2)}</p>
          <p>Status: {paymentIntent.status}</p>
        </div>
      )}
    </div>
  );
}
```

### Customer Dashboard Component

```tsx
// src/components/CustomerDashboard.tsx
import React from "react";
import { useCustomer, usePaymentMethods } from "@carnil/react";

export function CustomerDashboard({ customerId }: { customerId: string }) {
  const {
    customer,
    loading: customerLoading,
    error: customerError,
  } = useCustomer(customerId);
  const {
    paymentMethods,
    loading: methodsLoading,
    error: methodsError,
  } = usePaymentMethods(customerId);

  if (customerLoading || methodsLoading) {
    return <div className="loading">Loading customer data...</div>;
  }

  if (customerError || methodsError) {
    return <div className="error">Error loading customer data</div>;
  }

  return (
    <div className="customer-dashboard">
      <h2>Customer Dashboard</h2>

      <div className="customer-info">
        <h3>Customer Information</h3>
        {customer ? (
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
        ) : (
          <p>No customer data available</p>
        )}
      </div>

      <div className="payment-methods">
        <h3>Payment Methods</h3>
        {paymentMethods.length === 0 ? (
          <p>No payment methods found</p>
        ) : (
          <ul>
            {paymentMethods.map((method) => (
              <li key={method.id} className="payment-method">
                <div>
                  <p>
                    <strong>Type:</strong> {method.type}
                  </p>
                  {method.card && (
                    <p>
                      <strong>Card:</strong> **** **** **** {method.card.last4}
                    </p>
                  )}
                  <p>
                    <strong>Default:</strong> {method.isDefault ? "Yes" : "No"}
                  </p>
                  <p>
                    <strong>Created:</strong>{" "}
                    {method.createdAt.toLocaleDateString()}
                  </p>
                </div>
              </li>
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}
```

### Subscription Management Component

```tsx
// src/components/SubscriptionManagement.tsx
import React, { useState } from "react";
import { useSubscription } from "@carnil/react";

export function SubscriptionManagement({
  subscriptionId,
}: {
  subscriptionId: string;
}) {
  const { subscription, loading, error, update, cancel } =
    useSubscription(subscriptionId);
  const [updating, setUpdating] = useState(false);
  const [cancelling, setCancelling] = useState(false);

  const handleUpdate = async () => {
    setUpdating(true);
    try {
      await update({
        // Update subscription properties
        metadata: {
          plan: "enterprise",
          updatedAt: new Date().toISOString(),
        },
      });
      console.log("Subscription updated");
    } catch (error) {
      console.error("Failed to update subscription:", error);
    } finally {
      setUpdating(false);
    }
  };

  const handleCancel = async () => {
    setCancelling(true);
    try {
      await cancel();
      console.log("Subscription cancelled");
    } catch (error) {
      console.error("Failed to cancel subscription:", error);
    } finally {
      setCancelling(false);
    }
  };

  if (loading) return <div className="loading">Loading subscription...</div>;
  if (error) return <div className="error">Error: {error}</div>;
  if (!subscription) return <div>Subscription not found</div>;

  return (
    <div className="subscription-management">
      <h2>Subscription Management</h2>

      <div className="subscription-info">
        <h3>Subscription Details</h3>
        <p>
          <strong>ID:</strong> {subscription.id}
        </p>
        <p>
          <strong>Status:</strong> {subscription.status}
        </p>
        <p>
          <strong>Current Period:</strong>{" "}
          {subscription.currentPeriodStart.toLocaleDateString()} -{" "}
          {subscription.currentPeriodEnd.toLocaleDateString()}
        </p>
        <p>
          <strong>Created:</strong>{" "}
          {subscription.createdAt.toLocaleDateString()}
        </p>
      </div>

      <div className="subscription-actions">
        {subscription.status === "active" && (
          <>
            <button
              onClick={handleUpdate}
              disabled={updating}
              className="btn btn-secondary"
            >
              {updating ? "Updating..." : "Update Subscription"}
            </button>

            <button
              onClick={handleCancel}
              disabled={cancelling}
              className="btn btn-danger"
            >
              {cancelling ? "Cancelling..." : "Cancel Subscription"}
            </button>
          </>
        )}
      </div>
    </div>
  );
}
```

## Custom Hooks

### usePaymentFlow Hook

```tsx
// src/hooks/usePaymentFlow.ts
import { useState } from "react";
import { useCustomer, usePaymentMethods } from "@carnil/react";

export function usePaymentFlow(customerId: string) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const { customer } = useCustomer(customerId);
  const { paymentMethods, refetch: refetchPaymentMethods } =
    usePaymentMethods(customerId);

  const createPayment = async (amount: number, currency: string = "usd") => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch("/api/carnil", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          action: "createPaymentIntent",
          data: {
            amount,
            currency,
            customerId,
          },
        }),
      });

      const result = await response.json();
      if (result.success) {
        return result.data;
      } else {
        throw new Error(result.error || "Payment creation failed");
      }
    } catch (err) {
      setError(err instanceof Error ? err.message : "Payment failed");
      throw err;
    } finally {
      setLoading(false);
    }
  };

  const confirmPayment = async (
    paymentIntentId: string,
    paymentMethodId?: string
  ) => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch("/api/carnil", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          action: "confirmPaymentIntent",
          data: {
            paymentIntentId,
            paymentMethodId,
          },
        }),
      });

      const result = await response.json();
      if (result.success) {
        return result.data;
      } else {
        throw new Error(result.error || "Payment confirmation failed");
      }
    } catch (err) {
      setError(
        err instanceof Error ? err.message : "Payment confirmation failed"
      );
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return {
    customer,
    paymentMethods,
    loading,
    error,
    createPayment,
    confirmPayment,
    refetchPaymentMethods,
  };
}
```

### useSubscriptionFlow Hook

```tsx
// src/hooks/useSubscriptionFlow.ts
import { useState } from "react";
import { useCustomer } from "@carnil/react";

export function useSubscriptionFlow(customerId: string) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const { customer } = useCustomer(customerId);

  const createSubscription = async (
    priceId: string,
    paymentMethodId?: string
  ) => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch("/api/carnil", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          action: "createSubscription",
          data: {
            customerId,
            priceId,
            paymentMethodId,
          },
        }),
      });

      const result = await response.json();
      if (result.success) {
        return result.data;
      } else {
        throw new Error(result.error || "Subscription creation failed");
      }
    } catch (err) {
      setError(
        err instanceof Error ? err.message : "Subscription creation failed"
      );
      throw err;
    } finally {
      setLoading(false);
    }
  };

  const cancelSubscription = async (subscriptionId: string) => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch("/api/carnil", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          action: "cancelSubscription",
          data: { subscriptionId },
        }),
      });

      const result = await response.json();
      if (result.success) {
        return result.data;
      } else {
        throw new Error(result.error || "Subscription cancellation failed");
      }
    } catch (err) {
      setError(
        err instanceof Error ? err.message : "Subscription cancellation failed"
      );
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return {
    customer,
    loading,
    error,
    createSubscription,
    cancelSubscription,
  };
}
```

## Styling

### CSS Styles

```css
/* src/App.css */
.App {
  text-align: center;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.payment-form {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  margin: 20px 0;
}

.form-group {
  margin: 15px 0;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin: 5px;
}

.btn-primary {
  background: #007bff;
  color: white;
}

.btn-primary:hover {
  background: #0056b3;
}

.btn-secondary {
  background: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background: #545b62;
}

.btn-danger {
  background: #dc3545;
  color: white;
}

.btn-danger:hover {
  background: #c82333;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.customer-dashboard {
  text-align: left;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin: 20px 0;
}

.customer-info,
.payment-methods {
  margin: 20px 0;
}

.payment-method {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 4px;
  margin: 10px 0;
}

.loading {
  text-align: center;
  padding: 20px;
  color: #666;
}

.error {
  text-align: center;
  padding: 20px;
  color: #dc3545;
  background: #f8d7da;
  border-radius: 4px;
  margin: 20px 0;
}

.payment-result {
  background: #d4edda;
  color: #155724;
  padding: 15px;
  border-radius: 4px;
  margin: 20px 0;
}

.subscription-management {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin: 20px 0;
}

.subscription-info {
  margin: 20px 0;
}

.subscription-actions {
  margin: 20px 0;
}
```

## Error Handling

### Error Boundary Component

```tsx
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from "react";

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false,
  };

  public static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error("Error caught by boundary:", error, errorInfo);
  }

  public render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong.</h2>
          <details style={{ whiteSpace: "pre-wrap" }}>
            {this.state.error && this.state.error.toString()}
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Testing

### Component Testing

```tsx
// src/components/__tests__/PaymentForm.test.tsx
import React from "react";
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { PaymentForm } from "../PaymentForm";

describe("PaymentForm", () => {
  it("should render payment form", () => {
    render(<PaymentForm />);

    expect(screen.getByText("Payment Form")).toBeInTheDocument();
    expect(screen.getByLabelText("Amount (cents)")).toBeInTheDocument();
    expect(screen.getByRole("button", { name: /Pay/ })).toBeInTheDocument();
  });

  it("should handle payment submission", async () => {
    render(<PaymentForm />);

    const amountInput = screen.getByLabelText("Amount (cents)");
    const payButton = screen.getByRole("button", { name: /Pay/ });

    fireEvent.change(amountInput, { target: { value: "3000" } });
    fireEvent.click(payButton);

    await waitFor(() => {
      expect(screen.getByText("Processing...")).toBeInTheDocument();
    });
  });
});
```

## Deployment

### Build for Production

```bash
# Build the application
npm run build

# The build folder contains the production build
```

### Deploy to Netlify

1. Connect your repository to Netlify
2. Set build command: `npm run build`
3. Set publish directory: `build`
4. Set environment variables in Netlify dashboard

### Deploy to Vercel

1. Connect your repository to Vercel
2. Set build command: `npm run build`
3. Set output directory: `build`
4. Set environment variables in Vercel dashboard

## Troubleshooting

### Common Issues

1. **API Key Not Found**

   ```
   Error: API key not found
   ```

   Solution: Check your environment variables

2. **CORS Issues**

   ```
   Error: CORS policy blocked
   ```

   Solution: Configure CORS headers in your API

3. **Component Not Rendering**
   ```
   Error: Component not found
   ```
   Solution: Check import paths and component exports

### Debug Mode

Enable debug mode for detailed logging:

```tsx
<CarnilProvider
  config={{
    provider: {
      provider: "stripe",
      apiKey: process.env.REACT_APP_STRIPE_API_KEY!,
      webhookSecret: process.env.REACT_APP_STRIPE_WEBHOOK_SECRET!,
    },
    debug: process.env.NODE_ENV === "development",
  }}
>
  <YourApp />
</CarnilProvider>
```

## Next Steps

After running this example, you can:

1. **Add More Features**: Implement subscriptions, invoices, and refunds
2. **Add Authentication**: Implement user authentication and authorization
3. **Add Analytics**: Implement usage tracking and analytics
4. **Add Testing**: Implement comprehensive testing suite
5. **Add Styling**: Implement a design system or UI library

## Resources

- [Carnil Documentation](https://docs.carnil.dev)
- [React Documentation](https://reactjs.org/docs)
- [Stripe Documentation](https://stripe.com/docs)
- [Carnil GitHub Repository](https://github.com/Carnil-Dev/carnil-sdk)
- [Discord Community](https://discord.gg/carnil)

## Support

- 📖 [Documentation](https://docs.carnil.dev)
- 💬 [Discord Community](https://discord.gg/carnil)
- 🐛 [Report Issues](https://github.com/Carnil-Dev/carnil-sdk/issues)
- 📧 [Email Support](mailto:hello@carnil.dev)
