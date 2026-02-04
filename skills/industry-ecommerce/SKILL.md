---
name: industry-ecommerce
description: E-commerce development patterns including product catalogs, shopping carts, checkout flows, and inventory management
tags: [ecommerce, cart, checkout, inventory, catalog]
author: Antigravity Team
version: 1.0.0
---

# E-commerce Development Skill

Build robust e-commerce applications.

## Product Catalog

```javascript
// Product schema
const ProductSchema = {
  id: 'string',
  name: 'string',
  slug: 'string',
  description: 'string',
  price: 'number',
  compareAtPrice: 'number?',
  images: 'string[]',
  variants: [{
    id: 'string',
    name: 'string',
    sku: 'string',
    price: 'number',
    inventory: 'number',
    attributes: { size: 'string', color: 'string' }
  }],
  categories: 'string[]',
  tags: 'string[]',
  metadata: 'object'
};

// Product service
class ProductService {
  async search({ query, category, minPrice, maxPrice, page = 1, limit = 20 }) {
    const filters = [];
    
    if (query) filters.push({ $text: { $search: query } });
    if (category) filters.push({ categories: category });
    if (minPrice) filters.push({ price: { $gte: minPrice } });
    if (maxPrice) filters.push({ price: { $lte: maxPrice } });
    
    const products = await Product.find(
      filters.length ? { $and: filters } : {}
    )
    .skip((page - 1) * limit)
    .limit(limit)
    .sort({ score: { $meta: 'textScore' } });
    
    return { products, page, totalPages };
  }
}
```

## Shopping Cart

```javascript
class CartService {
  constructor(redis) {
    this.redis = redis;
    this.TTL = 7 * 24 * 60 * 60; // 7 days
  }

  getCartKey(userId) {
    return `cart:${userId}`;
  }

  async getCart(userId) {
    const cart = await this.redis.get(this.getCartKey(userId));
    return cart ? JSON.parse(cart) : { items: [], updatedAt: null };
  }

  async addItem(userId, productId, variantId, quantity = 1) {
    const cart = await this.getCart(userId);
    const existingItem = cart.items.find(
      i => i.productId === productId && i.variantId === variantId
    );

    if (existingItem) {
      existingItem.quantity += quantity;
    } else {
      const product = await ProductService.getProduct(productId);
      const variant = product.variants.find(v => v.id === variantId);
      
      cart.items.push({
        productId,
        variantId,
        name: product.name,
        variantName: variant.name,
        price: variant.price,
        image: product.images[0],
        quantity
      });
    }

    cart.updatedAt = new Date().toISOString();
    await this.redis.setex(
      this.getCartKey(userId),
      this.TTL,
      JSON.stringify(cart)
    );
    
    return cart;
  }

  async updateQuantity(userId, productId, variantId, quantity) {
    const cart = await this.getCart(userId);
    const item = cart.items.find(
      i => i.productId === productId && i.variantId === variantId
    );
    
    if (quantity <= 0) {
      cart.items = cart.items.filter(i => i !== item);
    } else {
      item.quantity = quantity;
    }
    
    await this.redis.setex(this.getCartKey(userId), this.TTL, JSON.stringify(cart));
    return cart;
  }

  calculateTotals(cart) {
    const subtotal = cart.items.reduce(
      (sum, item) => sum + item.price * item.quantity, 0
    );
    const tax = subtotal * 0.1;
    const shipping = subtotal > 50 ? 0 : 5.99;
    const total = subtotal + tax + shipping;
    
    return { subtotal, tax, shipping, total };
  }
}
```

## Checkout Flow

```javascript
class CheckoutService {
  async createOrder(userId, cart, shippingAddress, paymentMethod) {
    // Validate inventory
    for (const item of cart.items) {
      const available = await InventoryService.check(item.variantId);
      if (available < item.quantity) {
        throw new Error(`${item.name} is out of stock`);
      }
    }

    // Reserve inventory
    await InventoryService.reserve(cart.items);

    try {
      // Process payment
      const payment = await PaymentService.charge({
        amount: cart.total,
        paymentMethod,
        metadata: { userId }
      });

      // Create order
      const order = await Order.create({
        userId,
        items: cart.items,
        totals: CartService.calculateTotals(cart),
        shippingAddress,
        paymentId: payment.id,
        status: 'confirmed'
      });

      // Commit inventory
      await InventoryService.commit(cart.items);
      
      // Clear cart
      await CartService.clear(userId);

      // Send confirmation email
      await EmailService.sendOrderConfirmation(order);

      return order;
    } catch (error) {
      // Release inventory on failure
      await InventoryService.release(cart.items);
      throw error;
    }
  }
}
```

## Inventory Management

```javascript
class InventoryService {
  async check(variantId) {
    const variant = await Variant.findById(variantId);
    return variant.inventory - variant.reserved;
  }

  async reserve(items) {
    for (const item of items) {
      await Variant.updateOne(
        { _id: item.variantId },
        { $inc: { reserved: item.quantity } }
      );
    }
  }

  async commit(items) {
    for (const item of items) {
      await Variant.updateOne(
        { _id: item.variantId },
        { 
          $inc: { 
            inventory: -item.quantity,
            reserved: -item.quantity 
          }
        }
      );
    }
  }

  async release(items) {
    for (const item of items) {
      await Variant.updateOne(
        { _id: item.variantId },
        { $inc: { reserved: -item.quantity } }
      );
    }
  }
}
```
