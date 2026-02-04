---
name: testing-e2e
description: End-to-end testing with Playwright and Cypress including page objects, fixtures, and CI integration
tags: [testing, e2e, playwright, cypress, automation]
author: Antigravity Team
version: 1.0.0
---

# E2E Testing Skill

End-to-end testing with modern frameworks.

## Playwright

### Setup
```javascript
// playwright.config.js
export default {
  testDir: './tests',
  timeout: 30000,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure'
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'mobile', use: { ...devices['iPhone 13'] } }
  ]
};
```

### Basic Tests
```javascript
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('should login successfully', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toContainText('Welcome');
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[data-testid="email"]', 'wrong@email.com');
    await page.fill('[data-testid="password"]', 'wrongpass');
    await page.click('[data-testid="login-button"]');
    
    await expect(page.locator('.error-message')).toBeVisible();
    await expect(page.locator('.error-message')).toContainText('Invalid');
  });
});
```

### Page Object Model
```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email"]');
    this.passwordInput = page.locator('[data-testid="password"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.errorMessage = page.locator('.error-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}

// Usage in test
test('login flow', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@test.com', 'password');
  await expect(page).toHaveURL('/dashboard');
});
```

### Fixtures
```javascript
// fixtures.js
import { test as base } from '@playwright/test';

export const test = base.extend({
  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login');
    await page.fill('#email', 'test@test.com');
    await page.fill('#password', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
    await use(page);
  }
});

// Usage
test('authenticated test', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('/settings');
  // Already logged in!
});
```

## Cypress

```javascript
describe('Shopping Cart', () => {
  beforeEach(() => {
    cy.login('user@test.com', 'password');
    cy.visit('/products');
  });

  it('should add item to cart', () => {
    cy.get('[data-cy="product-card"]').first().within(() => {
      cy.get('[data-cy="add-to-cart"]').click();
    });
    
    cy.get('[data-cy="cart-count"]').should('contain', '1');
  });

  it('should complete checkout', () => {
    cy.addToCart('product-1');
    cy.visit('/checkout');
    
    cy.get('[data-cy="payment-form"]').within(() => {
      cy.get('#card-number').type('4242424242424242');
      cy.get('#expiry').type('12/25');
      cy.get('#cvc').type('123');
    });
    
    cy.get('[data-cy="pay-button"]').click();
    cy.url().should('include', '/order-confirmation');
  });
});

// Custom commands (support/commands.js)
Cypress.Commands.add('login', (email, password) => {
  cy.session([email], () => {
    cy.request('POST', '/api/auth/login', { email, password })
      .then(({ body }) => {
        window.localStorage.setItem('token', body.token);
      });
  });
});
```

## CI Integration

```yaml
# GitHub Actions
- name: Run E2E Tests
  run: npx playwright test
  env:
    BASE_URL: ${{ secrets.TEST_URL }}

- uses: actions/upload-artifact@v3
  if: failure()
  with:
    name: playwright-report
    path: playwright-report/
```

## Best Practices

1. **Use data-testid** for stable selectors
2. **Isolate tests** - each test should be independent
3. **Mock external services** in CI
4. **Visual testing** for UI regression
5. **Parallel execution** for speed
