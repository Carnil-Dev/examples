# Carnil SaaS Dashboard Example

A complete SaaS dashboard application built with Next.js and Carnil SDK, demonstrating advanced payment features including analytics, compliance, pricing management, and webhook handling.

## Features

- 🚀 **Complete SaaS Dashboard** - Full-featured SaaS application
- 💳 **Payment Processing** - Stripe and Razorpay integration
- 📊 **Analytics Dashboard** - Usage tracking and analytics
- 🛡️ **Compliance Tools** - PCI DSS and GDPR compliance
- 🌍 **Globalization** - Multi-currency and tax management
- 💰 **Pricing Editor** - Visual pricing management with A/B testing
- 🔔 **Webhook System** - Comprehensive webhook handling
- 📱 **Responsive Design** - Mobile-friendly interface
- 🎨 **Modern UI** - Beautiful and intuitive design

## Prerequisites

- Node.js 18+
- Next.js 14+
- Stripe account with API keys
- Razorpay account with API keys
- npm or yarn

## Installation

```bash
# Clone the repository
git clone https://github.com/Carnil-Dev/carnil-sdk.git
cd carnil-sdk/examples/saas-dashboard

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
RAZORPAY_KEY_ID=rzp_test_...
RAZORPAY_KEY_SECRET=your_key_secret
RAZORPAY_WEBHOOK_SECRET=your_webhook_secret
```

2. Update the configuration in `app/api/carnil/route.ts`:

```typescript
export const POST = createCarnilHandler({
  provider: {
    provider: "stripe", // or 'razorpay'
    apiKey: process.env.STRIPE_API_KEY!,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  },
  debug: process.env.NODE_ENV === "development",
});
```

## Project Structure

```
saas-dashboard/
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
│   └── page.tsx                  # Dashboard home
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

## Dashboard Features

### Analytics Dashboard

The analytics dashboard provides comprehensive insights into your payment data:

```tsx
// app/components/analytics-dashboard.tsx
import { useAnalytics } from "@carnil/analytics";

export function AnalyticsDashboard({ customerId }: { customerId: string }) {
  const { analytics, loading, error } = useAnalytics(customerId);

  if (loading) return <div>Loading analytics...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">Total Usage</h3>
        <p className="text-3xl font-bold text-blue-600">
          {analytics?.summary.totalUsage || 0}
        </p>
      </div>

      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">Total Cost</h3>
        <p className="text-3xl font-bold text-green-600">
          ${analytics?.summary.totalCost || 0}
        </p>
      </div>

      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">Daily Average</h3>
        <p className="text-3xl font-bold text-purple-600">
          {analytics?.summary.averageDailyUsage || 0}
        </p>
      </div>

      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">AI Usage</h3>
        <p className="text-3xl font-bold text-orange-600">
          {analytics?.aiUsage?.length || 0}
        </p>
      </div>
    </div>
  );
}
```

### Compliance Dashboard

The compliance dashboard helps you maintain regulatory compliance:

```tsx
// app/components/compliance-dashboard.tsx
import { ComplianceDashboard } from "@carnil/compliance";

export function CompliancePage() {
  return (
    <div className="max-w-6xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">Compliance Dashboard</h1>

      <ComplianceDashboard
        organizationId="org_123"
        showPCIStatus={true}
        showGDPRStatus={true}
        showAuditLogs={true}
        refreshInterval={30000}
      />
    </div>
  );
}
```

### Pricing Editor

The pricing editor allows you to manage your pricing strategies:

```tsx
// app/components/pricing-editor.tsx
import {
  PricingEditor,
  ABTestPanel,
  GrandfatheringPanel,
} from "@carnil/pricing-editor";

export function PricingManagement() {
  const [pricing, setPricing] = useState({
    tiers: [
      {
        id: "basic",
        name: "Basic",
        price: 9,
        currency: "USD",
        interval: "month",
        features: ["Feature 1", "Feature 2"],
      },
      {
        id: "pro",
        name: "Pro",
        price: 29,
        currency: "USD",
        interval: "month",
        features: ["All Basic features", "Feature 3", "Feature 4"],
        popular: true,
      },
    ],
  });

  return (
    <div className="max-w-6xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">Pricing Management</h1>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <PricingEditor
          pricing={pricing}
          onChange={setPricing}
          currency="USD"
          showPreview={true}
          enableABTesting={true}
          enableGrandfathering={true}
        />

        <div className="space-y-6">
          <ABTestPanel
            pricing={pricing}
            onChange={setPricing}
            onTestCreate={(test) => console.log("Test created:", test)}
          />

          <GrandfatheringPanel
            pricing={pricing}
            onChange={setPricing}
            onRuleCreate={(rule) => console.log("Rule created:", rule)}
          />
        </div>
      </div>
    </div>
  );
}
```

### Globalization Features

The globalization features support multi-currency and tax management:

```tsx
// app/components/globalization-dashboard.tsx
import { CurrencyManager, TaxManager } from "@carnil/globalization";

export function GlobalizationDashboard() {
  const [currencyManager] = useState(
    new CurrencyManager({
      baseCurrency: "USD",
      exchangeRateProvider: "fixer",
    })
  );

  const [taxManager] = useState(
    new TaxManager({
      taxProvider: "avalara",
      defaultTaxRate: 0.08,
    })
  );

  const [convertedAmount, setConvertedAmount] = useState(0);

  const handleCurrencyConversion = async () => {
    const amount = await currencyManager.convertAmount(100, "USD", "EUR");
    setConvertedAmount(amount);
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">Globalization Dashboard</h1>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div className="bg-white p-6 rounded-lg shadow-md">
          <h2 className="text-xl font-semibold mb-4">Currency Conversion</h2>
          <button
            onClick={handleCurrencyConversion}
            className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700"
          >
            Convert $100 USD to EUR
          </button>
          {convertedAmount > 0 && (
            <p className="mt-4 text-lg">
              Converted amount: €{convertedAmount.toFixed(2)}
            </p>
          )}
        </div>

        <div className="bg-white p-6 rounded-lg shadow-md">
          <h2 className="text-xl font-semibold mb-4">Tax Management</h2>
          <p>Tax calculation features will be available here</p>
        </div>
      </div>
    </div>
  );
}
```

## API Routes

### Enhanced Carnil API Handler

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
  features: {
    analytics: true,
    compliance: true,
    globalization: true,
    pricingEditor: true,
    webhooks: true,
  },
});
```

### Comprehensive Webhook Handler

```typescript
// app/api/webhook/route.ts
import { createWebhookHandler } from "@carnil/next";

export const POST = createWebhookHandler({
  webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  handlers: {
    "payment.succeeded": async (event) => {
      console.log("Payment succeeded:", event.data);
      // Update analytics
      await updateAnalytics(event.data);
      // Send confirmation email
      await sendConfirmationEmail(event.data.customerId);
    },
    "payment.failed": async (event) => {
      console.log("Payment failed:", event.data);
      // Update compliance logs
      await logComplianceEvent(event.data);
      // Send failure notification
      await sendFailureNotification(event.data.customerId);
    },
    "subscription.created": async (event) => {
      console.log("Subscription created:", event.data);
      // Update customer analytics
      await updateCustomerAnalytics(event.data.customerId);
    },
    "invoice.payment_succeeded": async (event) => {
      console.log("Invoice payment succeeded:", event.data);
      // Update revenue analytics
      await updateRevenueAnalytics(event.data);
    },
  },
});
```

## Styling and UI

### Global Styles

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: "Inter", system-ui, sans-serif;
  }
}

@layer components {
  .btn-primary {
    @apply bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 transition-colors;
  }

  .btn-secondary {
    @apply bg-gray-200 text-gray-800 px-4 py-2 rounded-md hover:bg-gray-300 transition-colors;
  }

  .card {
    @apply bg-white rounded-lg shadow-md p-6;
  }

  .dashboard-grid {
    @apply grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6;
  }
}
```

### Responsive Design

The dashboard is fully responsive and works on all device sizes:

```tsx
// app/components/responsive-dashboard.tsx
export function ResponsiveDashboard() {
  return (
    <div className="min-h-screen bg-gray-50">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div className="dashboard-grid">
          <AnalyticsCard />
          <ComplianceCard />
          <PricingCard />
          <GlobalizationCard />
        </div>
      </div>
    </div>
  );
}
```

## Testing

### Component Testing

```typescript
// __tests__/components/analytics-dashboard.test.tsx
import { render, screen } from "@testing-library/react";
import { AnalyticsDashboard } from "@/app/components/analytics-dashboard";

describe("AnalyticsDashboard", () => {
  it("should render analytics dashboard", () => {
    render(<AnalyticsDashboard customerId="cus_123" />);

    expect(screen.getByText("Total Usage")).toBeInTheDocument();
    expect(screen.getByText("Total Cost")).toBeInTheDocument();
  });
});
```

### API Testing

```typescript
// __tests__/api/carnil.test.ts
import { POST } from "@/app/api/carnil/route";
import { NextRequest } from "next/server";

describe("/api/carnil", () => {
  it("should handle analytics requests", async () => {
    const request = new NextRequest("http://localhost:3000/api/carnil", {
      method: "POST",
      body: JSON.stringify({
        action: "trackUsage",
        data: {
          customerId: "cus_123",
          featureId: "api_calls",
          usage: 100,
        },
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data.success).toBe(true);
  });
});
```

## Deployment

### Vercel Deployment

1. Connect your repository to Vercel
2. Set environment variables:
   ```
   STRIPE_API_KEY=sk_live_...
   STRIPE_WEBHOOK_SECRET=whsec_...
   RAZORPAY_KEY_ID=rzp_live_...
   RAZORPAY_KEY_SECRET=your_live_key_secret
   RAZORPAY_WEBHOOK_SECRET=your_live_webhook_secret
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

## Security and Compliance

### PCI DSS Compliance

The dashboard includes PCI DSS compliance features:

```tsx
// app/components/pci-compliance.tsx
import { ComplianceManager } from "@carnil/compliance";

export function PCIComplianceStatus() {
  const [complianceStatus, setComplianceStatus] = useState(null);

  useEffect(() => {
    const checkCompliance = async () => {
      const complianceManager = new ComplianceManager({
        pciDss: { enabled: true, level: "level-1" },
      });

      const status = await complianceManager.validatePCIDSS();
      setComplianceStatus(status);
    };

    checkCompliance();
  }, []);

  return (
    <div className="card">
      <h3 className="text-lg font-semibold mb-4">PCI DSS Compliance</h3>
      {complianceStatus?.compliant ? (
        <div className="text-green-600">✅ Compliant</div>
      ) : (
        <div className="text-red-600">❌ Non-compliant</div>
      )}
    </div>
  );
}
```

### GDPR Compliance

GDPR compliance features are also included:

```tsx
// app/components/gdpr-compliance.tsx
import { GDPRManager } from "@carnil/compliance";

export function GDPRComplianceStatus() {
  const [gdprStatus, setGdprStatus] = useState(null);

  useEffect(() => {
    const checkGDPR = async () => {
      const gdprManager = new GDPRManager({
        enabled: true,
        dataRetentionDays: 2555,
      });

      const status = await gdprManager.validateGDPR();
      setGdprStatus(status);
    };

    checkGDPR();
  }, []);

  return (
    <div className="card">
      <h3 className="text-lg font-semibold mb-4">GDPR Compliance</h3>
      {gdprStatus?.compliant ? (
        <div className="text-green-600">✅ Compliant</div>
      ) : (
        <div className="text-red-600">❌ Non-compliant</div>
      )}
    </div>
  );
}
```

## Monitoring and Analytics

### Real-time Monitoring

The dashboard includes real-time monitoring capabilities:

```tsx
// app/components/real-time-monitoring.tsx
import { useAnalytics } from "@carnil/analytics";

export function RealTimeMonitoring() {
  const { analytics, loading } = useAnalytics("customer_123");

  return (
    <div className="card">
      <h3 className="text-lg font-semibold mb-4">Real-time Monitoring</h3>
      {loading ? (
        <div>Loading...</div>
      ) : (
        <div className="space-y-2">
          <div>Current Usage: {analytics?.summary.totalUsage || 0}</div>
          <div>Current Cost: ${analytics?.summary.totalCost || 0}</div>
          <div>Daily Average: {analytics?.summary.averageDailyUsage || 0}</div>
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### Common Issues

1. **Analytics Not Loading**

   ```
   Error: Analytics data not available
   ```

   Solution: Check analytics configuration and data sources

2. **Compliance Check Failed**

   ```
   Error: Compliance validation failed
   ```

   Solution: Verify compliance configuration and requirements

3. **Pricing Editor Not Working**
   ```
   Error: Pricing editor initialization failed
   ```
   Solution: Check pricing editor configuration and dependencies

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
  features: {
    analytics: { debug: true },
    compliance: { debug: true },
    globalization: { debug: true },
    pricingEditor: { debug: true },
  },
});
```

## Next Steps

After running this example, you can:

1. **Customize Analytics**: Add custom metrics and dashboards
2. **Enhance Compliance**: Implement additional compliance features
3. **Add More Providers**: Integrate additional payment providers
4. **Implement Authentication**: Add user authentication and authorization
5. **Add Advanced Features**: Implement advanced pricing strategies and A/B testing

## Resources

- [Carnil Documentation](https://docs.carnil.dev)
- [Next.js Documentation](https://nextjs.org/docs)
- [Stripe Documentation](https://stripe.com/docs)
- [Razorpay Documentation](https://razorpay.com/docs)
- [Carnil GitHub Repository](https://github.com/Carnil-Dev/carnil-sdk)
- [Discord Community](https://discord.gg/carnil)

## Support

- 📖 [Documentation](https://docs.carnil.dev)
- 💬 [Discord Community](https://discord.gg/carnil)
- 🐛 [Report Issues](https://github.com/Carnil-Dev/carnil-sdk/issues)
- 📧 [Email Support](mailto:hello@carnil.dev)
