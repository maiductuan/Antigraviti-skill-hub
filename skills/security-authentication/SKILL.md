---
name: security-authentication
description: Authentication and authorization patterns including OAuth2, JWT, SSO, RBAC, and secure session management
tags: [security, oauth, jwt, sso, authentication]
author: Antigravity Team
version: 1.0.0
---

# Authentication & Authorization Skill

Secure authentication implementation patterns.

## JWT Authentication

```javascript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';

const JWT_SECRET = process.env.JWT_SECRET;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

// Password hashing
async function hashPassword(password) {
  return bcrypt.hash(password, 12);
}

async function verifyPassword(password, hash) {
  return bcrypt.compare(password, hash);
}

// Token generation
function generateTokens(userId, roles = []) {
  const accessToken = jwt.sign(
    { sub: userId, roles },
    JWT_SECRET,
    { expiresIn: '15m', algorithm: 'HS256' }
  );
  
  const refreshToken = jwt.sign(
    { sub: userId, type: 'refresh' },
    JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
}

// Token refresh
async function refreshAccessToken(refreshToken) {
  const payload = jwt.verify(refreshToken, JWT_REFRESH_SECRET);
  
  // Check if token is blacklisted
  const isBlacklisted = await redis.get(`blacklist:${refreshToken}`);
  if (isBlacklisted) throw new Error('Token revoked');
  
  return generateTokens(payload.sub);
}
```

## OAuth2 Integration

```javascript
import { OAuth2Client } from 'google-auth-library';

// Google OAuth
const googleClient = new OAuth2Client(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  'http://localhost:3000/auth/google/callback'
);

app.get('/auth/google', (req, res) => {
  const url = googleClient.generateAuthUrl({
    scope: ['profile', 'email'],
    access_type: 'offline'
  });
  res.redirect(url);
});

app.get('/auth/google/callback', async (req, res) => {
  const { code } = req.query;
  const { tokens } = await googleClient.getToken(code);
  
  const ticket = await googleClient.verifyIdToken({
    idToken: tokens.id_token,
    audience: process.env.GOOGLE_CLIENT_ID
  });
  
  const { email, name, picture } = ticket.getPayload();
  
  // Create or update user
  let user = await db.users.findByEmail(email);
  if (!user) {
    user = await db.users.create({ email, name, avatar: picture });
  }
  
  const appTokens = generateTokens(user.id);
  res.redirect(`/dashboard?token=${appTokens.accessToken}`);
});
```

## RBAC (Role-Based Access Control)

```javascript
// Permission definitions
const PERMISSIONS = {
  'users:read': ['admin', 'manager', 'user'],
  'users:write': ['admin', 'manager'],
  'users:delete': ['admin'],
  'reports:read': ['admin', 'manager', 'analyst'],
  'reports:write': ['admin', 'analyst']
};

// Middleware
function requirePermission(permission) {
  return (req, res, next) => {
    const userRoles = req.user.roles;
    const allowedRoles = PERMISSIONS[permission] || [];
    
    const hasPermission = userRoles.some(role => 
      allowedRoles.includes(role)
    );
    
    if (!hasPermission) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Usage
app.delete('/users/:id', 
  authenticate, 
  requirePermission('users:delete'),
  deleteUser
);
```

## Session Management

```javascript
import session from 'express-session';
import RedisStore from 'connect-redis';

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  name: 'sid',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));
```

## Security Headers

```javascript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:']
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true }
}));

// CSRF protection
import csrf from 'csurf';
app.use(csrf({ cookie: true }));
```

## Best Practices

1. **Never store plain passwords** - Always hash with bcrypt
2. **Short-lived access tokens** - 15-30 minutes max
3. **Secure cookies** - httpOnly, secure, sameSite
4. **Token rotation** - Rotate refresh tokens on use
5. **Audit logging** - Log all auth events
