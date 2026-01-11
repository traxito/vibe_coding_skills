---
name: Stripe & Payments
description: Implement Stripe payments, subscriptions, webhooks, and customer management following best practices. Includes subscription tiers, trial periods, upgrade/downgrade flows, invoice handling, and payment security. Use when implementing monetization, billing, or payment processing.
applyTo: "**/*.{ts,tsx,js,jsx}"
---

# Stripe & Payments Best Practices Skill

Complete guide for implementing Stripe payments in SaaS applications with production-ready patterns.

## When to Use This Skill

- Setting up Stripe subscription plans
- Implementing payment flows (one-time or recurring)
- Creating customer portal
- Handling webhooks (payment events)
- Managing subscription upgrades/downgrades
- Implementing trial periods
- Processing refunds and cancellations
- Securing payment endpoints
- Testing payments in development

---

## Stripe Project Setup

### 1. Install Stripe Dependencies

```bash
# Client-side (React/Next.js)
npm install @stripe/stripe-js @stripe/react-stripe-js

# Server-side (Node.js/Cloud Functions)
npm install stripe

# TypeScript types
npm install --save-dev @types/stripe
```

### 2. Environment Variables

```bash
# .env.local (Frontend)
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxx

# .env (Backend/Cloud Functions)
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_ID_FREE=price_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_ID_PRO=price_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_ID_ENTERPRISE=price_xxxxxxxxxxxxxxxxxxxxx
```

### 3. Stripe Dashboard Configuration

**Important Steps:**
1. Create Products and Prices in Stripe Dashboard
2. Set up Customer Portal settings
3. Configure webhook endpoints
4. Enable tax collection (if applicable)
5. Set up email notifications

---

## Stripe Configuration

### Server-side Stripe Client

```typescript
// lib/stripe.ts (Backend/Cloud Functions)
import Stripe from 'stripe';

if (!process.env.STRIPE_SECRET_KEY) {
  throw new Error('STRIPE_SECRET_KEY is not set');
}

// Initialize Stripe with API version
export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2024-11-20.acacia',
  typescript: true,
  appInfo: {
    name: 'YourApp',
    version: '1.0.0',
  },
});

// Price IDs for subscription tiers
export const STRIPE_PRICES = {
  FREE: process.env.STRIPE_PRICE_ID_FREE || '',
  PRO: process.env.STRIPE_PRICE_ID_PRO || '',
  ENTERPRISE: process.env.STRIPE_PRICE_ID_ENTERPRISE || '',
};

// Plan metadata
export const PLANS = {
  free: {
    name: 'Free',
    priceId: STRIPE_PRICES.FREE,
    price: 0,
    currency: 'usd',
    interval: 'month',
    features: ['Basic features', '100 requests/month', 'Email support'],
  },
  pro: {
    name: 'Pro',
    priceId: STRIPE_PRICES.PRO,
    price: 19.99,
    currency: 'usd',
    interval: 'month',
    features: ['All Free features', 'Unlimited requests', 'Priority support', 'Advanced analytics'],
  },
  enterprise: {
    name: 'Enterprise',
    priceId: STRIPE_PRICES.ENTERPRISE,
    price: 99.99,
    currency: 'usd',
    interval: 'month',
    features: ['All Pro features', 'Custom integrations', 'Dedicated support', 'SLA'],
  },
};
```

### Client-side Stripe Configuration

```typescript
// lib/stripe-client.ts (Frontend)
import { loadStripe, Stripe } from '@stripe/stripe-js';

let stripePromise: Promise<Stripe | null>;

export const getStripe = () => {
  if (!stripePromise) {
    const key = import.meta.env.VITE_STRIPE_PUBLISHABLE_KEY;
    if (!key) {
      throw new Error('VITE_STRIPE_PUBLISHABLE_KEY is not set');
    }
    stripePromise = loadStripe(key);
  }
  return stripePromise;
};
```

---

## Subscription Creation

### Create Checkout Session (Cloud Function)

```typescript
// functions/src/stripe/createCheckoutSession.ts
import * as functions from 'firebase-functions';
import { stripe, STRIPE_PRICES } from '../lib/stripe';
import * as admin from 'firebase-admin';

interface CheckoutSessionData {
  priceId: string;
  userId: string;
  successUrl: string;
  cancelUrl: string;
  trialDays?: number;
}

export const createCheckoutSession = functions.https.onCall(
  async (data: CheckoutSessionData, context) => {
    // Verify authentication
    if (!context.auth) {
      throw new functions.https.HttpsError(
        'unauthenticated',
        'User must be authenticated'
      );
    }

    const { priceId, successUrl, cancelUrl, trialDays = 0 } = data;
    const userId = context.auth.uid;

    try {
      // Get or create Stripe customer
      const userDoc = await admin.firestore().collection('users').doc(userId).get();
      const userData = userDoc.data();

      let customerId = userData?.stripeCustomerId;

      if (!customerId) {
        // Create new Stripe customer
        const customer = await stripe.customers.create({
          email: userData?.email || context.auth.token.email,
          metadata: {
            firebaseUID: userId,
          },
        });
        customerId = customer.id;

        // Save customer ID to Firestore
        await admin.firestore().collection('users').doc(userId).update({
          stripeCustomerId: customerId,
        });
      }

      // Create checkout session
      const session = await stripe.checkout.sessions.create({
        customer: customerId,
        mode: 'subscription',
        payment_method_types: ['card'],
        line_items: [
          {
            price: priceId,
            quantity: 1,
          },
        ],
        success_url: successUrl,
        cancel_url: cancelUrl,
        allow_promotion_codes: true,
        billing_address_collection: 'auto',
        subscription_data: trialDays > 0 ? {
          trial_period_days: trialDays,
          metadata: {
            firebaseUID: userId,
          },
        } : {
          metadata: {
            firebaseUID: userId,
          },
        },
        metadata: {
          firebaseUID: userId,
        },
      });

      return { sessionId: session.id, url: session.url };
    } catch (error: any) {
      console.error('Error creating checkout session:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

### Client-side Checkout Component (React)

```tsx
// components/PricingCard.tsx
import { useState } from 'react';
import { getStripe } from '@/lib/stripe-client';
import { httpsCallable } from 'firebase/functions';
import { functions } from '@/lib/firebase';
import { useAuth } from '@/contexts/AuthContext';

interface PricingCardProps {
  plan: {
    name: string;
    price: number;
    priceId: string;
    features: string[];
  };
  trialDays?: number;
}

export default function PricingCard({ plan, trialDays = 0 }: PricingCardProps) {
  const [loading, setLoading] = useState(false);
  const { user } = useAuth();

  const handleSubscribe = async () => {
    if (!user) {
      // Redirect to login
      window.location.href = '/login?redirect=/pricing';
      return;
    }

    setLoading(true);

    try {
      // Create checkout session via Cloud Function
      const createCheckoutSession = httpsCallable(functions, 'createCheckoutSession');

      const { data } = await createCheckoutSession({
        priceId: plan.priceId,
        userId: user.uid,
        successUrl: `${window.location.origin}/dashboard?success=true`,
        cancelUrl: `${window.location.origin}/pricing?canceled=true`,
        trialDays,
      }) as { data: { sessionId: string; url: string } };

      // Redirect to Stripe Checkout
      const stripe = await getStripe();
      if (!stripe) {
        throw new Error('Stripe failed to load');
      }

      const { error } = await stripe.redirectToCheckout({
        sessionId: data.sessionId,
      });

      if (error) {
        console.error('Stripe redirect error:', error);
        alert('Payment failed. Please try again.');
      }
    } catch (error) {
      console.error('Checkout error:', error);
      alert('Something went wrong. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="pricing-card">
      <h3>{plan.name}</h3>
      <p className="price">${plan.price}/month</p>

      {trialDays > 0 && (
        <p className="trial">{trialDays}-day free trial</p>
      )}

      <ul className="features">
        {plan.features.map((feature, index) => (
          <li key={index}>{feature}</li>
        ))}
      </ul>

      <button 
        onClick={handleSubscribe}
        disabled={loading}
        className="subscribe-btn"
      >
        {loading ? 'Loading...' : 'Subscribe Now'}
      </button>
    </div>
  );
}
```

---

## Webhook Handling

### Webhook Endpoint (Cloud Function)

```typescript
// functions/src/stripe/webhooks.ts
import * as functions from 'firebase-functions';
import { stripe } from '../lib/stripe';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

export const stripeWebhook = functions.https.onRequest(async (req, res) => {
  const sig = req.headers['stripe-signature'] as string;
  const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

  let event: Stripe.Event;

  try {
    // Verify webhook signature
    event = stripe.webhooks.constructEvent(req.rawBody, sig, webhookSecret);
  } catch (err: any) {
    console.error('Webhook signature verification failed:', err.message);
    res.status(400).send(`Webhook Error: ${err.message}`);
    return;
  }

  // Handle different event types
  try {
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutSessionCompleted(event.data.object as Stripe.Checkout.Session);
        break;

      case 'customer.subscription.created':
        await handleSubscriptionCreated(event.data.object as Stripe.Subscription);
        break;

      case 'customer.subscription.updated':
        await handleSubscriptionUpdated(event.data.object as Stripe.Subscription);
        break;

      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
        break;

      case 'invoice.payment_succeeded':
        await handleInvoicePaymentSucceeded(event.data.object as Stripe.Invoice);
        break;

      case 'invoice.payment_failed':
        await handleInvoicePaymentFailed(event.data.object as Stripe.Invoice);
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    res.json({ received: true });
  } catch (error: any) {
    console.error('Webhook handler error:', error);
    res.status(500).send(`Webhook handler failed: ${error.message}`);
  }
});

// Handle checkout session completed
async function handleCheckoutSessionCompleted(session: Stripe.Checkout.Session) {
  const userId = session.metadata?.firebaseUID;
  if (!userId) {
    console.error('No firebaseUID in session metadata');
    return;
  }

  const subscriptionId = session.subscription as string;
  const customerId = session.customer as string;

  // Update user document
  await admin.firestore().collection('users').doc(userId).update({
    stripeCustomerId: customerId,
    stripeSubscriptionId: subscriptionId,
    subscriptionStatus: 'active',
    plan: getPlanFromPriceId(session.line_items?.data[0]?.price?.id),
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  console.log(`Checkout completed for user ${userId}`);
}

// Handle subscription created
async function handleSubscriptionCreated(subscription: Stripe.Subscription) {
  const userId = subscription.metadata?.firebaseUID;
  if (!userId) return;

  const priceId = subscription.items.data[0]?.price.id;

  await admin.firestore().collection('users').doc(userId).update({
    stripeSubscriptionId: subscription.id,
    subscriptionStatus: subscription.status,
    plan: getPlanFromPriceId(priceId),
    currentPeriodStart: new Date(subscription.current_period_start * 1000),
    currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  console.log(`Subscription created for user ${userId}`);
}

// Handle subscription updated (upgrade/downgrade)
async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  const userId = subscription.metadata?.firebaseUID;
  if (!userId) return;

  const priceId = subscription.items.data[0]?.price.id;

  await admin.firestore().collection('users').doc(userId).update({
    subscriptionStatus: subscription.status,
    plan: getPlanFromPriceId(priceId),
    currentPeriodStart: new Date(subscription.current_period_start * 1000),
    currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    cancelAtPeriodEnd: subscription.cancel_at_period_end,
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  console.log(`Subscription updated for user ${userId}`);
}

// Handle subscription deleted (cancellation)
async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  const userId = subscription.metadata?.firebaseUID;
  if (!userId) return;

  await admin.firestore().collection('users').doc(userId).update({
    subscriptionStatus: 'canceled',
    plan: 'free',
    stripeSubscriptionId: null,
    canceledAt: admin.firestore.FieldValue.serverTimestamp(),
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  console.log(`Subscription canceled for user ${userId}`);
}

// Handle invoice payment succeeded
async function handleInvoicePaymentSucceeded(invoice: Stripe.Invoice) {
  const userId = invoice.subscription_details?.metadata?.firebaseUID;
  if (!userId) return;

  // Log successful payment
  await admin.firestore().collection('users').doc(userId).collection('payments').add({
    invoiceId: invoice.id,
    amount: invoice.amount_paid / 100,
    currency: invoice.currency,
    status: 'succeeded',
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  console.log(`Payment succeeded for user ${userId}, amount: ${invoice.amount_paid / 100}`);
}

// Handle invoice payment failed
async function handleInvoicePaymentFailed(invoice: Stripe.Invoice) {
  const userId = invoice.subscription_details?.metadata?.firebaseUID;
  if (!userId) return;

  // Update user status
  await admin.firestore().collection('users').doc(userId).update({
    subscriptionStatus: 'past_due',
    paymentFailed: true,
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  // TODO: Send email notification about failed payment

  console.log(`Payment failed for user ${userId}`);
}

// Helper function to map price ID to plan name
function getPlanFromPriceId(priceId?: string): string {
  const { STRIPE_PRICES } = require('../lib/stripe');

  if (priceId === STRIPE_PRICES.PRO) return 'pro';
  if (priceId === STRIPE_PRICES.ENTERPRISE) return 'enterprise';
  return 'free';
}
```

### Test Webhooks Locally

```bash
# Install Stripe CLI
# macOS: brew install stripe/stripe-cli/stripe
# Windows: scoop install stripe
# Linux: Download from stripe.com/docs/stripe-cli

# Login to Stripe
stripe login

# Forward webhooks to local endpoint (PowerShell: use npx)
stripe listen --forward-to localhost:5001/your-project/us-central1/stripeWebhook

# Trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
```

---

## Customer Portal

### Create Portal Session (Cloud Function)

```typescript
// functions/src/stripe/createPortalSession.ts
import * as functions from 'firebase-functions';
import { stripe } from '../lib/stripe';
import * as admin from 'firebase-admin';

export const createPortalSession = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
  }

  const userId = context.auth.uid;
  const { returnUrl } = data;

  try {
    // Get customer ID from Firestore
    const userDoc = await admin.firestore().collection('users').doc(userId).get();
    const customerId = userDoc.data()?.stripeCustomerId;

    if (!customerId) {
      throw new functions.https.HttpsError('not-found', 'No Stripe customer found');
    }

    // Create portal session
    const session = await stripe.billingPortal.sessions.create({
      customer: customerId,
      return_url: returnUrl || `${process.env.APP_URL}/dashboard`,
    });

    return { url: session.url };
  } catch (error: any) {
    console.error('Error creating portal session:', error);
    throw new functions.https.HttpsError('internal', error.message);
  }
});
```

### Client-side Portal Button (React)

```tsx
// components/ManageSubscriptionButton.tsx
import { useState } from 'react';
import { httpsCallable } from 'firebase/functions';
import { functions } from '@/lib/firebase';

export default function ManageSubscriptionButton() {
  const [loading, setLoading] = useState(false);

  const handleManageSubscription = async () => {
    setLoading(true);

    try {
      const createPortalSession = httpsCallable(functions, 'createPortalSession');

      const { data } = await createPortalSession({
        returnUrl: window.location.href,
      }) as { data: { url: string } };

      // Redirect to Customer Portal
      window.location.href = data.url;
    } catch (error) {
      console.error('Error opening customer portal:', error);
      alert('Failed to open billing portal. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <button
      onClick={handleManageSubscription}
      disabled={loading}
      className="manage-subscription-btn"
    >
      {loading ? 'Loading...' : 'Manage Subscription'}
    </button>
  );
}
```

---

## Subscription Upgrades/Downgrades

### Update Subscription (Cloud Function)

```typescript
// functions/src/stripe/updateSubscription.ts
import * as functions from 'firebase-functions';
import { stripe, STRIPE_PRICES } from '../lib/stripe';
import * as admin from 'firebase-admin';

interface UpdateSubscriptionData {
  newPriceId: string;
  prorationBehavior?: 'create_prorations' | 'none' | 'always_invoice';
}

export const updateSubscription = functions.https.onCall(
  async (data: UpdateSubscriptionData, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
    }

    const { newPriceId, prorationBehavior = 'create_prorations' } = data;
    const userId = context.auth.uid;

    try {
      // Get subscription ID from Firestore
      const userDoc = await admin.firestore().collection('users').doc(userId).get();
      const subscriptionId = userDoc.data()?.stripeSubscriptionId;

      if (!subscriptionId) {
        throw new functions.https.HttpsError('not-found', 'No active subscription found');
      }

      // Retrieve subscription
      const subscription = await stripe.subscriptions.retrieve(subscriptionId);

      // Update subscription with new price
      const updatedSubscription = await stripe.subscriptions.update(subscriptionId, {
        items: [
          {
            id: subscription.items.data[0].id,
            price: newPriceId,
          },
        ],
        proration_behavior: prorationBehavior,
      });

      // Update Firestore
      await admin.firestore().collection('users').doc(userId).update({
        plan: getPlanFromPriceId(newPriceId),
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      });

      return { 
        success: true, 
        subscription: updatedSubscription,
        message: 'Subscription updated successfully',
      };
    } catch (error: any) {
      console.error('Error updating subscription:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);

function getPlanFromPriceId(priceId: string): string {
  if (priceId === STRIPE_PRICES.PRO) return 'pro';
  if (priceId === STRIPE_PRICES.ENTERPRISE) return 'enterprise';
  return 'free';
}
```

---

## Cancel Subscription

### Cancel at Period End (Cloud Function)

```typescript
// functions/src/stripe/cancelSubscription.ts
import * as functions from 'firebase-functions';
import { stripe } from '../lib/stripe';
import * as admin from 'firebase-admin';

interface CancelSubscriptionData {
  immediately?: boolean;
}

export const cancelSubscription = functions.https.onCall(
  async (data: CancelSubscriptionData, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
    }

    const { immediately = false } = data;
    const userId = context.auth.uid;

    try {
      // Get subscription ID
      const userDoc = await admin.firestore().collection('users').doc(userId).get();
      const subscriptionId = userDoc.data()?.stripeSubscriptionId;

      if (!subscriptionId) {
        throw new functions.https.HttpsError('not-found', 'No active subscription');
      }

      if (immediately) {
        // Cancel immediately
        await stripe.subscriptions.cancel(subscriptionId);

        await admin.firestore().collection('users').doc(userId).update({
          subscriptionStatus: 'canceled',
          plan: 'free',
          canceledAt: admin.firestore.FieldValue.serverTimestamp(),
        });

        return { success: true, message: 'Subscription canceled immediately' };
      } else {
        // Cancel at period end
        await stripe.subscriptions.update(subscriptionId, {
          cancel_at_period_end: true,
        });

        await admin.firestore().collection('users').doc(userId).update({
          cancelAtPeriodEnd: true,
          updatedAt: admin.firestore.FieldValue.serverTimestamp(),
        });

        return { success: true, message: 'Subscription will cancel at period end' };
      }
    } catch (error: any) {
      console.error('Error canceling subscription:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

---

## One-time Payments

### Create Payment Intent (for one-time purchases)

```typescript
// functions/src/stripe/createPaymentIntent.ts
import * as functions from 'firebase-functions';
import { stripe } from '../lib/stripe';

interface PaymentIntentData {
  amount: number; // in cents
  currency: string;
  description?: string;
}

export const createPaymentIntent = functions.https.onCall(
  async (data: PaymentIntentData, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
    }

    const { amount, currency, description } = data;

    // Validate amount
    if (amount < 50) { // Stripe minimum is 50 cents
      throw new functions.https.HttpsError('invalid-argument', 'Amount too small');
    }

    try {
      const paymentIntent = await stripe.paymentIntents.create({
        amount,
        currency,
        description,
        metadata: {
          firebaseUID: context.auth.uid,
        },
      });

      return { clientSecret: paymentIntent.client_secret };
    } catch (error: any) {
      console.error('Error creating payment intent:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

### Client-side Payment Form (React)

```tsx
// components/PaymentForm.tsx
import { useState } from 'react';
import { Elements, PaymentElement, useStripe, useElements } from '@stripe/react-stripe-js';
import { getStripe } from '@/lib/stripe-client';
import { httpsCallable } from 'firebase/functions';
import { functions } from '@/lib/firebase';

function CheckoutForm({ amount }: { amount: number }) {
  const stripe = useStripe();
  const elements = useElements();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) return;

    setLoading(true);
    setError(null);

    try {
      const { error: submitError } = await elements.submit();
      if (submitError) {
        setError(submitError.message || 'Payment failed');
        setLoading(false);
        return;
      }

      // Create payment intent
      const createPaymentIntent = httpsCallable(functions, 'createPaymentIntent');
      const { data } = await createPaymentIntent({
        amount: amount * 100, // Convert to cents
        currency: 'usd',
        description: 'One-time payment',
      }) as { data: { clientSecret: string } };

      // Confirm payment
      const { error: confirmError } = await stripe.confirmPayment({
        elements,
        clientSecret: data.clientSecret,
        confirmParams: {
          return_url: `${window.location.origin}/payment-success`,
        },
      });

      if (confirmError) {
        setError(confirmError.message || 'Payment failed');
      }
    } catch (err: any) {
      setError(err.message || 'Something went wrong');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      {error && <div className="error">{error}</div>}
      <button type="submit" disabled={!stripe || loading}>
        {loading ? 'Processing...' : `Pay $${amount}`}
      </button>
    </form>
  );
}

export default function PaymentForm({ amount }: { amount: number }) {
  const stripePromise = getStripe();

  return (
    <Elements stripe={stripePromise}>
      <CheckoutForm amount={amount} />
    </Elements>
  );
}
```

---

## Security Best Practices

### 1. Verify Webhook Signatures (CRITICAL)

```typescript
// ✅ ALWAYS verify webhook signatures
const event = stripe.webhooks.constructEvent(req.rawBody, sig, webhookSecret);

// ❌ NEVER trust webhook data without verification
```

### 2. Use Idempotency Keys

```typescript
// For retry-safe operations
const paymentIntent = await stripe.paymentIntents.create(
  {
    amount: 2000,
    currency: 'usd',
  },
  {
    idempotencyKey: `payment-${userId}-${Date.now()}`,
  }
);
```

### 3. Validate Amounts Server-side

```typescript
// ❌ NEVER trust amount from client
// const amount = req.body.amount; // DANGEROUS

// ✅ Calculate amount server-side
const plan = PLANS[req.body.planName];
const amount = plan.price * 100; // Convert to cents
```

### 4. Store Minimal Payment Data

```typescript
// ✅ Store only necessary info in Firestore
{
  stripeCustomerId: 'cus_xxx',
  stripeSubscriptionId: 'sub_xxx',
  plan: 'pro',
  subscriptionStatus: 'active'
}

// ❌ NEVER store card details or payment methods
```

---

## Testing

### Test Cards

```typescript
// Stripe test cards
const TEST_CARDS = {
  SUCCESS: '4242424242424242',
  DECLINED: '4000000000000002',
  INSUFFICIENT_FUNDS: '4000000000009995',
  REQUIRES_3D_SECURE: '4000002500003155',
};
```

### Test Mode vs Production

```typescript
// Always check environment
const isProduction = process.env.NODE_ENV === 'production';

if (!isProduction) {
  console.warn('Running in TEST mode. Use test cards.');
}

// Use different keys
const STRIPE_KEY = isProduction 
  ? process.env.STRIPE_LIVE_SECRET_KEY 
  : process.env.STRIPE_TEST_SECRET_KEY;
```

---

## Troubleshooting

### Common Issues

#### 1. Webhook Not Receiving Events

```bash
# Verify webhook is configured in Stripe Dashboard
# Check endpoint URL is correct and accessible
# Test locally with Stripe CLI
stripe listen --forward-to localhost:5001/your-project/us-central1/stripeWebhook
```

#### 2. Payment Failed

```typescript
// Check error codes
if (error.code === 'card_declined') {
  // Handle declined card
} else if (error.code === 'insufficient_funds') {
  // Handle insufficient funds
}
```

#### 3. Subscription Not Updating

```typescript
// Check webhook is processing events
// Verify Firestore rules allow updates
// Check Cloud Function logs for errors
```

---

## Best Practices Summary

### Security
- ✅ Always verify webhook signatures
- ✅ Validate all amounts server-side
- ✅ Use HTTPS only
- ✅ Never store sensitive payment data
- ✅ Use idempotency keys for critical operations

### User Experience
- ✅ Show clear pricing and terms
- ✅ Offer trial periods
- ✅ Enable Customer Portal for self-service
- ✅ Send email confirmations for all transactions
- ✅ Handle failed payments gracefully

### Development
- ✅ Use test mode during development
- ✅ Test webhooks with Stripe CLI
- ✅ Log all payment events
- ✅ Handle errors with user-friendly messages
- ✅ Monitor failed payments and cancellations

### Cost Optimization
- ✅ Use proration for upgrades/downgrades
- ✅ Set up automatic retries for failed payments
- ✅ Archive old subscription data
- ✅ Monitor Stripe fees and optimize

---

## Resources

- **Stripe Documentation**: https://stripe.com/docs
- **Stripe Testing**: https://stripe.com/docs/testing
- **Webhook Events**: https://stripe.com/docs/api/events/types
- **Customer Portal**: https://stripe.com/docs/billing/subscriptions/integrating-customer-portal
- **Security Best Practices**: https://stripe.com/docs/security/guide

---

## Quick Reference

### Essential Stripe Endpoints

```typescript
// Create subscription
stripe.checkout.sessions.create()

// Cancel subscription
stripe.subscriptions.update(id, { cancel_at_period_end: true })

// Immediate cancel
stripe.subscriptions.cancel(id)

// Update subscription
stripe.subscriptions.update(id, { items: [...] })

// Create portal session
stripe.billingPortal.sessions.create()

// Retrieve customer
stripe.customers.retrieve(id)

// List invoices
stripe.invoices.list({ customer: customerId })
```

### Webhook Events to Handle

- `checkout.session.completed` - New subscription
- `customer.subscription.created` - Subscription created
- `customer.subscription.updated` - Plan changed
- `customer.subscription.deleted` - Subscription canceled
- `invoice.payment_succeeded` - Payment successful
- `invoice.payment_failed` - Payment failed

**Always test your payment flows with test cards before going live!**
