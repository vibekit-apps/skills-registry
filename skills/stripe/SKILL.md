## ⚠️ MANDATORY: COPY THIS CODE EXACTLY - DO NOT MODIFY
### Step 1: Install packages
\`\`\`json
{
  "stripe": "^14.0.0",
  "@stripe/stripe-js": "^2.0.0",
  "@stripe/react-stripe-js": "^2.0.0"
}
\`\`\`

### Step 2: Stripe server config (copy exactly)
\`\`\`typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});
\`\`\`

### Step 3: Checkout API route (copy exactly)
\`\`\`typescript
// app/api/checkout/route.ts
import { NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe';

export async function POST(req: Request) {
  const { priceId } = await req.json();

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: \`\${process.env.NEXT_PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}\`,
    cancel_url: \`\${process.env.NEXT_PUBLIC_URL}/cancel\`,
  });

  return NextResponse.json({ url: session.url });
}
\`\`\`

### Step 4: Webhook handler (copy exactly)
\`\`\`typescript
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
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object;
      console.log('Checkout completed:', session.id);
      break;
    case 'customer.subscription.deleted':
      const sub = event.data.object;
      console.log('Subscription cancelled:', sub.id);
      break;
  }

  return NextResponse.json({ received: true });
}
\`\`\`

### Step 5: Checkout button (copy exactly)
\`\`\`tsx
// components/checkout-button.tsx
'use client';
import { useState } from 'react';

export function CheckoutButton({ priceId }: { priceId: string }) {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);
    const res = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ priceId }),
    });
    const { url } = await res.json();
    window.location.href = url;
  };

  return (
    <button onClick={handleCheckout} disabled={loading}>
      {loading ? 'Loading...' : 'Subscribe'}
    </button>
  );
}
\`\`\`

### Environment variables needed:
- STRIPE_SECRET_KEY (sk_test_... or sk_live_...)
- STRIPE_WEBHOOK_SECRET (whsec_...)
- NEXT_PUBLIC_URL (your app URL, e.g. https://myapp.vibekit.bot)

### ❌ DO NOT:
- Never expose STRIPE_SECRET_KEY to client
- Never skip webhook signature verification
- Never trust client-side subscription status

### ✅ DO:
- Always verify webhook signatures
- Use test keys (sk_test_) during development
- Store customer IDs in your database
