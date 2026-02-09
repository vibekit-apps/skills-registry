# Stripe Integration Skill

## Environment Variables
```
STRIPE_SECRET_KEY=sk_live_... or sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_live_... or pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Package
```json
{
  "stripe": "^14.0.0",
  "@stripe/stripe-js": "^2.0.0",
  "@stripe/react-stripe-js": "^2.0.0"
}
```

## Server-Side (Next.js API Routes)

### Initialize Stripe
```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});
```

### Create Checkout Session
```typescript
// app/api/checkout/route.ts
import { NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';

export async function POST(req: Request) {
  const { priceId, successUrl, cancelUrl } = await req.json();

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription', // or 'payment' for one-time
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: successUrl || `${process.env.NEXT_PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: cancelUrl || `${process.env.NEXT_PUBLIC_URL}/cancel`,
  });

  return NextResponse.json({ url: session.url });
}
```

### Webhook Handler
```typescript
// app/api/webhooks/stripe/route.ts
import { NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';
import { headers } from 'next/headers';

export async function POST(req: Request) {
  const body = await req.text();
  const signature = headers().get('stripe-signature')!;

  let event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err: any) {
    console.error('Webhook signature verification failed:', err.message);
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object;
      // Handle successful checkout
      // e.g., provision access, update database
      console.log('Checkout completed:', session.id);
      break;

    case 'customer.subscription.updated':
      const subscription = event.data.object;
      // Handle subscription changes
      console.log('Subscription updated:', subscription.id);
      break;

    case 'customer.subscription.deleted':
      const deletedSub = event.data.object;
      // Handle cancellation
      console.log('Subscription cancelled:', deletedSub.id);
      break;

    case 'invoice.payment_failed':
      const invoice = event.data.object;
      // Handle failed payment
      console.log('Payment failed:', invoice.id);
      break;
  }

  return NextResponse.json({ received: true });
}
```

### Customer Portal
```typescript
// app/api/portal/route.ts
import { NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';

export async function POST(req: Request) {
  const { customerId, returnUrl } = await req.json();

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: returnUrl || process.env.NEXT_PUBLIC_URL,
  });

  return NextResponse.json({ url: session.url });
}
```

## Client-Side (React)

### Stripe Provider
```tsx
// app/providers.tsx
'use client';
import { Elements } from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

export function StripeProvider({ children }: { children: React.ReactNode }) {
  return <Elements stripe={stripePromise}>{children}</Elements>;
}
```

### Checkout Button
```tsx
// components/checkout-button.tsx
'use client';
import { useState } from 'react';

export function CheckoutButton({ priceId }: { priceId: string }) {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);
    try {
      const res = await fetch('/api/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ priceId }),
      });
      const { url } = await res.json();
      window.location.href = url;
    } catch (err) {
      console.error('Checkout error:', err);
    } finally {
      setLoading(false);
    }
  };

  return (
    <button onClick={handleCheckout} disabled={loading}>
      {loading ? 'Loading...' : 'Subscribe'}
    </button>
  );
}
```

### Payment Element (for custom forms)
```tsx
// components/payment-form.tsx
'use client';
import { PaymentElement, useStripe, useElements } from '@stripe/react-stripe-js';
import { useState } from 'react';

export function PaymentForm() {
  const stripe = useStripe();
  const elements = useElements();
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!stripe || !elements) return;

    setLoading(true);
    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/success`,
      },
    });

    if (error) {
      setError(error.message || 'Payment failed');
    }
    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      {error && <p className="text-red-500">{error}</p>}
      <button type="submit" disabled={!stripe || loading}>
        {loading ? 'Processing...' : 'Pay'}
      </button>
    </form>
  );
}
```

## Common Patterns

### Pricing Page with Multiple Tiers
```tsx
const plans = [
  { name: 'Free', price: '$0', priceId: null },
  { name: 'Pro', price: '$9.99/mo', priceId: 'price_xxx' },
  { name: 'Team', price: '$29.99/mo', priceId: 'price_yyy' },
];
```

### Check Subscription Status
```typescript
// Server-side
const subscription = await stripe.subscriptions.retrieve(subscriptionId);
const isActive = subscription.status === 'active';
```

## ❌ DO NOT
- Never expose `STRIPE_SECRET_KEY` to the client
- Never skip webhook signature verification
- Never trust client-side subscription status without server verification
- Never hardcode price IDs in production (use env vars or database)

## ✅ DO
- Always use webhook signature verification
- Store customer IDs in your database
- Use test mode (`sk_test_`) during development
- Set up webhook endpoint in Stripe Dashboard
- Handle all relevant webhook events
