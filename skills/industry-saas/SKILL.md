---
name: industry-saas
description: SaaS development patterns including multi-tenancy, subscription billing, feature flags, and tenant isolation
tags: [saas, multi-tenant, billing, feature-flags]
author: Antigravity Team
version: 1.0.0
---

# SaaS Development Skill

Build scalable SaaS applications.

## Multi-Tenancy

### Database Per Tenant
```javascript
// Connection manager
class TenantConnectionManager {
  private connections = new Map();

  async getConnection(tenantId) {
    if (!this.connections.has(tenantId)) {
      const config = await this.getTenantConfig(tenantId);
      const connection = await createConnection(config.databaseUrl);
      this.connections.set(tenantId, connection);
    }
    return this.connections.get(tenantId);
  }
}

// Middleware
async function tenantMiddleware(req, res, next) {
  const tenantId = req.headers['x-tenant-id'] || extractFromSubdomain(req);
  req.db = await connectionManager.getConnection(tenantId);
  req.tenantId = tenantId;
  next();
}
```

### Row-Level Isolation
```javascript
// All queries filtered by tenant
class Repository {
  constructor(private db, private tenantId) {}

  async findAll() {
    return this.db.query(
      'SELECT * FROM items WHERE tenant_id = $1',
      [this.tenantId]
    );
  }

  async create(data) {
    return this.db.query(
      'INSERT INTO items (tenant_id, name) VALUES ($1, $2)',
      [this.tenantId, data.name]
    );
  }
}
```

## Subscription Billing (Stripe)

```javascript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

// Create subscription
async function createSubscription(customerId, priceId) {
  return stripe.subscriptions.create({
    customer: customerId,
    items: [{ price: priceId }],
    payment_behavior: 'default_incomplete',
    expand: ['latest_invoice.payment_intent']
  });
}

// Handle webhooks
app.post('/webhook', async (req, res) => {
  const event = stripe.webhooks.constructEvent(
    req.body, req.headers['stripe-signature'], WEBHOOK_SECRET
  );

  switch (event.type) {
    case 'customer.subscription.created':
      await activateTenant(event.data.object.metadata.tenantId);
      break;
    case 'customer.subscription.deleted':
      await deactivateTenant(event.data.object.metadata.tenantId);
      break;
    case 'invoice.payment_failed':
      await notifyPaymentFailed(event.data.object);
      break;
  }
  res.json({ received: true });
});

// Usage metering
async function reportUsage(subscriptionItemId, quantity) {
  return stripe.subscriptionItems.createUsageRecord(
    subscriptionItemId,
    { quantity, timestamp: Math.floor(Date.now() / 1000) }
  );
}
```

## Feature Flags

```javascript
class FeatureFlags {
  constructor(private flagService) {}

  async isEnabled(flag, tenantId, userId) {
    const config = await this.flagService.getFlag(flag);
    
    if (!config.enabled) return false;
    
    // Check tenant tier
    const tenant = await getTenant(tenantId);
    if (config.tiers && !config.tiers.includes(tenant.tier)) {
      return false;
    }
    
    // Percentage rollout
    if (config.percentage < 100) {
      const hash = this.hashUser(userId);
      return hash < config.percentage;
    }
    
    return true;
  }

  hashUser(userId) {
    const hash = crypto.createHash('md5').update(userId).digest('hex');
    return parseInt(hash.slice(0, 8), 16) % 100;
  }
}

// Usage
if (await features.isEnabled('new-dashboard', tenantId, userId)) {
  return <NewDashboard />;
}
```

## Plan Limits

```javascript
const PLAN_LIMITS = {
  free: { users: 3, storage: 100, apiCalls: 1000 },
  pro: { users: 20, storage: 1000, apiCalls: 50000 },
  enterprise: { users: Infinity, storage: Infinity, apiCalls: Infinity }
};

async function checkLimit(tenantId, resource) {
  const tenant = await getTenant(tenantId);
  const limits = PLAN_LIMITS[tenant.plan];
  const usage = await getUsage(tenantId, resource);
  
  if (usage >= limits[resource]) {
    throw new LimitExceededError(resource, limits[resource]);
  }
}
```

## Best Practices

1. **Isolate tenant data** completely
2. **Audit all actions** with tenant context
3. **Graceful degradation** when limits reached
4. **Plan upgrade prompts** at key moments
