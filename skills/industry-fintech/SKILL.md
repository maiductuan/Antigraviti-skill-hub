---
name: industry-fintech
description: FinTech development patterns including payment processing, PCI compliance, banking APIs, and financial calculations
tags: [fintech, payments, stripe, banking, compliance]
author: Antigravity Team
version: 1.0.0
---

# FinTech Development Skill

Build secure financial applications.

## Payment Processing (Stripe)

```javascript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

// Create payment intent
async function createPayment(amount, currency, customerId) {
  return stripe.paymentIntents.create({
    amount: Math.round(amount * 100), // Convert to cents
    currency,
    customer: customerId,
    automatic_payment_methods: { enabled: true },
    metadata: { orderId: generateOrderId() }
  });
}

// Webhook handler
app.post('/webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];
  
  try {
    const event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET
    );
    
    switch (event.type) {
      case 'payment_intent.succeeded':
        await handlePaymentSuccess(event.data.object);
        break;
      case 'payment_intent.payment_failed':
        await handlePaymentFailure(event.data.object);
        break;
    }
    
    res.json({ received: true });
  } catch (err) {
    res.status(400).send(`Webhook Error: ${err.message}`);
  }
});
```

## Money Handling

```javascript
// NEVER use floating point for money!
import Decimal from 'decimal.js';

class Money {
  constructor(amount, currency = 'USD') {
    this.amount = new Decimal(amount);
    this.currency = currency;
  }

  add(other) {
    this.checkCurrency(other);
    return new Money(this.amount.plus(other.amount), this.currency);
  }

  multiply(factor) {
    return new Money(
      this.amount.times(factor).toDecimalPlaces(2, Decimal.ROUND_HALF_UP),
      this.currency
    );
  }

  toCents() {
    return this.amount.times(100).toNumber();
  }

  format() {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: this.currency
    }).format(this.amount.toNumber());
  }
}

// Usage
const price = new Money('19.99');
const tax = price.multiply(0.1);
const total = price.add(tax); // $21.99
```

## PCI Compliance

```javascript
// Never store raw card data!
// Use tokenization

// Client-side (Stripe Elements)
const elements = stripe.elements();
const card = elements.create('card');
card.mount('#card-element');

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  const { token } = await stripe.createToken(card);
  
  // Send only token to your server
  await fetch('/api/pay', {
    method: 'POST',
    body: JSON.stringify({ token: token.id, amount: 1999 })
  });
});

// Server-side: only handle tokens, not card numbers
```

## Banking API Integration

```javascript
// Open Banking API example (Plaid)
import { PlaidApi, Configuration, PlaidEnvironments } from 'plaid';

const plaid = new PlaidApi(new Configuration({
  basePath: PlaidEnvironments.sandbox,
  baseOptions: {
    headers: {
      'PLAID-CLIENT-ID': process.env.PLAID_CLIENT_ID,
      'PLAID-SECRET': process.env.PLAID_SECRET
    }
  }
}));

// Get account balances
async function getBalances(accessToken) {
  const response = await plaid.accountsBalanceGet({ access_token: accessToken });
  return response.data.accounts.map(acc => ({
    name: acc.name,
    balance: acc.balances.current,
    currency: acc.balances.iso_currency_code
  }));
}

// Get transactions
async function getTransactions(accessToken, startDate, endDate) {
  const response = await plaid.transactionsGet({
    access_token: accessToken,
    start_date: startDate,
    end_date: endDate
  });
  return response.data.transactions;
}
```

## Security Checklist

- [ ] Use HTTPS everywhere
- [ ] Implement rate limiting
- [ ] Log all financial transactions
- [ ] Two-factor authentication for sensitive operations
- [ ] Encrypt sensitive data at rest
- [ ] Regular security audits
- [ ] PCI DSS compliance validation
