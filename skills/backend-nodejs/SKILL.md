---
name: backend-nodejs
description: Node.js backend development with Express, NestJS, middleware patterns, authentication, and API design best practices
tags: [nodejs, express, nestjs, api, backend, typescript]
author: Antigravity Team
version: 1.0.0
---

# Node.js Backend Development Skill

Comprehensive guide for building scalable Node.js backends.

## Express.js Patterns

### Project Structure
```
src/
├── controllers/     # Route handlers
├── middleware/      # Custom middleware
├── models/          # Data models
├── routes/          # Route definitions
├── services/        # Business logic
├── utils/           # Helpers
├── config/          # Configuration
└── app.js           # Express app
```

### Middleware Chain
```javascript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';

const app = express();

// Security middleware
app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') }));
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Custom request logging
app.use((req, res, next) => {
  req.startTime = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.path} ${res.statusCode} ${Date.now() - req.startTime}ms`);
  });
  next();
});
```

### Error Handling
```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Async handler wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Global error handler
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    success: false,
    error: err.message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});
```

## Authentication

### JWT Authentication
```javascript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';

// Generate tokens
function generateTokens(userId) {
  const accessToken = jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  const refreshToken = jwt.sign(
    { userId },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );
  return { accessToken, refreshToken };
}

// Auth middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) throw new AppError('Unauthorized', 401);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
  } catch {
    throw new AppError('Invalid token', 401);
  }
}
```

## NestJS Patterns

### Module Structure
```typescript
// users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService]
})
export class UsersModule {}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepo: Repository<User>
  ) {}

  async findAll(): Promise<User[]> {
    return this.usersRepo.find();
  }
}

// users.controller.ts
@Controller('users')
@UseGuards(JwtAuthGuard)
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

## Database

### Prisma ORM
```javascript
// prisma/schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
}

// Usage
const user = await prisma.user.create({
  data: { email, name },
  include: { posts: true }
});
```

## Best Practices

1. **Environment Variables**: Use `dotenv` and validate with `zod`
2. **Logging**: Use structured logging with `pino` or `winston`
3. **Validation**: Input validation with `zod` or `class-validator`
4. **Testing**: Unit tests with Jest, integration tests with Supertest
5. **Documentation**: Auto-generate with Swagger/OpenAPI
