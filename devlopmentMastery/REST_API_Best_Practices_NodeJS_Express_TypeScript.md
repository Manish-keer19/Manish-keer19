# REST API Industry Standard — Node.js + Express + TypeScript

> Production-ready best practices for scalable backend systems.

---

## Table of Contents

1. [Project Architecture](#1-project-architecture)
2. [REST API Naming Conventions](#2-rest-api-naming-conventions)
3. [HTTP Methods Best Practice](#3-http-methods-best-practice)
4. [Standard Response Format](#4-standard-response-format)
5. [HTTP Status Code Standards](#5-http-status-code-standards)
6. [Controller Best Practices](#6-controller-best-practices)
7. [Service Layer Best Practice](#7-service-layer-best-practice)
8. [Validation Layer](#8-validation-layer)
9. [Error Handling Middleware](#9-error-handling-middleware)
10. [Authentication Best Practice](#10-authentication-best-practice)
11. [Pagination Best Practice](#11-pagination-best-practice)
12. [Filtering & Sorting](#12-filtering--sorting)
13. [API Versioning](#13-api-versioning)
14. [Middleware Best Practices](#14-middleware-best-practices)
15. [Production Security Best Practices](#15-production-security-best-practices)
16. [Environment Variables](#16-environment-variables)
17. [Logging Best Practice](#17-logging-best-practice)
18. [Example Production User Module](#18-example-production-user-module)
19. [Golden Rules](#19-golden-rules)

---

## 1. Project Architecture

A well-structured project separates concerns clearly, making the codebase maintainable, testable, and scalable. Follow **Clean Architecture** principles by organizing your `src/` directory as follows:

```
src/
 ├── controllers/      # Handle HTTP requests and responses
 ├── services/         # Business logic and application rules
 ├── repositories/     # Data access layer (DB queries)
 ├── routes/           # Express route definitions
 ├── middleware/       # Request pipeline interceptors
 ├── validators/       # Input validation schemas
 ├── models/           # ORM/ODM models and schema definitions
 ├── utils/            # Reusable utility/helper functions
 ├── config/           # App configuration (DB, env, etc.)
 ├── types/            # TypeScript interfaces, types, enums
 ├── constants/        # App-wide constant values
 └── app.ts            # Express app initialization entry point
```

### Folder Responsibilities

| Folder | Responsibility |
|---|---|
| `controllers/` | Receive requests, call services, send responses. No business logic. |
| `services/` | All business logic, calculations, and orchestration. |
| `repositories/` | All direct database interactions (CRUD). Services call repositories. |
| `routes/` | Define API endpoints and map them to controllers. |
| `middleware/` | Cross-cutting concerns: auth, logging, error handling, rate limiting. |
| `validators/` | Zod/Joi schemas to validate incoming request data. |
| `models/` | Mongoose/Prisma/TypeORM model definitions. |
| `utils/` | Shared helpers — date formatters, token generators, etc. |
| `config/` | Load and export environment config, DB connections, third-party clients. |
| `types/` | Shared TypeScript types and interfaces used across the app. |
| `constants/` | Enums and string constants (roles, statuses, messages). |
| `app.ts` | Bootstrap Express app, register middleware, and mount routes. |

> **Rule of thumb:** Each layer only communicates with the layer directly below it. Controllers → Services → Repositories → Models.

---

## 2. REST API Naming Conventions

Well-named endpoints are self-documenting and predictable. Follow these conventions consistently.

### Use Plural Nouns

Resources are collections. Use plural nouns to represent them.

**✅ Good**

```
GET    /users
GET    /users/:id
POST   /users
PATCH  /users/:id
DELETE /users/:id
```

**❌ Bad**

```
/getUsers
/createUser
/deleteUser
/fetchUserById
```

### Avoid Verbs in Endpoints

HTTP methods already describe the action. Embedding verbs in URLs is redundant.

```
❌ POST /createUser
✅ POST /users

❌ GET /getAllProducts
✅ GET /products

❌ DELETE /removeOrder/:id
✅ DELETE /orders/:id
```

### Use Nested Routes for Relationships

When a resource belongs to another resource, express that in the URL.

```
GET  /users/:userId/orders           → Get all orders for a user
GET  /users/:userId/orders/:orderId  → Get a specific order for a user
POST /posts/:postId/comments         → Create a comment on a post
```

### Use Kebab-case for Multi-word Resources

```
✅ /product-categories
✅ /order-items
❌ /productCategories
❌ /ProductCategories
```

---

## 3. HTTP Methods Best Practice

Each HTTP method has a defined semantic meaning. Use them correctly.

| Method | Purpose | Idempotent | Body |
|---|---|---|---|
| `GET` | Retrieve resource(s) | ✅ Yes | ❌ No |
| `POST` | Create a new resource | ❌ No | ✅ Yes |
| `PUT` | Replace a resource entirely | ✅ Yes | ✅ Yes |
| `PATCH` | Partially update a resource | ✅ Yes | ✅ Yes |
| `DELETE` | Remove a resource | ✅ Yes | Optional |

### Usage Examples

```
GET    /products              → Return list of all products
GET    /products/:id          → Return a single product

POST   /products              → Create a new product (body required)

PUT    /products/:id          → Replace full product document

PATCH  /products/:id          → Update only changed fields (e.g., price only)

DELETE /products/:id          → Delete a product
```

> **PATCH vs PUT:** Use `PATCH` when updating partial fields. Use `PUT` only when replacing the entire resource.

---

## 4. Standard Response Format

All API responses must follow a **consistent envelope structure**. This makes client-side handling predictable and reduces integration friction.

### Success Response

```ts
{
  success: true,
  message: string,
  data?: any,
  meta?: {
    page?: number,
    limit?: number,
    total?: number
  }
}
```

### Error Response

```ts
{
  success: false,
  message: string,
  error?: string   // Optional: stack trace or detailed error (dev only)
}
```

### TypeScript Response Helper

```ts
// src/utils/response.ts

import { Response } from 'express';

export const sendSuccess = (
  res: Response,
  data: any,
  message = 'Success',
  statusCode = 200,
  meta?: object
) => {
  return res.status(statusCode).json({
    success: true,
    message,
    data,
    ...(meta && { meta })
  });
};

export const sendError = (
  res: Response,
  message = 'Something went wrong',
  statusCode = 500,
  error?: string
) => {
  return res.status(statusCode).json({
    success: false,
    message,
    ...(error && { error })
  });
};
```

> **Why consistency matters:** Every consumer (web, mobile, third-party) can rely on the same shape. Reduces bugs, speeds up onboarding, and simplifies error handling across clients.

---

## 5. HTTP Status Code Standards

Always return the most semantically accurate status code.

### 2xx — Success

| Code | Name | When to Use |
|---|---|---|
| `200` | OK | Successful GET, PATCH, PUT, DELETE |
| `201` | Created | Resource successfully created via POST |
| `204` | No Content | Successful DELETE with no response body |

### 4xx — Client Errors

| Code | Name | When to Use |
|---|---|---|
| `400` | Bad Request | Malformed request, missing required fields |
| `401` | Unauthorized | Missing or invalid authentication token |
| `403` | Forbidden | Authenticated but lacks permission |
| `404` | Not Found | Resource does not exist |
| `409` | Conflict | Duplicate resource (e.g., email already exists) |
| `422` | Unprocessable Entity | Validation failed on valid syntax |
| `429` | Too Many Requests | Rate limit exceeded |

### 5xx — Server Errors

| Code | Name | When to Use |
|---|---|---|
| `500` | Internal Server Error | Unexpected server-side failure |
| `503` | Service Unavailable | Server overloaded or down for maintenance |

### Examples

```ts
res.status(200).json({ success: true, data: users });        // List users
res.status(201).json({ success: true, data: newUser });      // Created user
res.status(204).send();                                       // Deleted user
res.status(400).json({ success: false, message: 'Invalid request' });
res.status(401).json({ success: false, message: 'Unauthorized' });
res.status(404).json({ success: false, message: 'User not found' });
res.status(500).json({ success: false, message: 'Internal server error' });
```

---

## 6. Controller Best Practices

Controllers are the **entry point for HTTP requests**. They are thin by design — their only job is to receive input, delegate to the service layer, and return a response.

### Controllers Should

- Extract request data (params, query, body)
- Call the appropriate service method
- Return a formatted response
- Handle async errors with try/catch or a wrapper

### Controllers Should NOT

- Contain business logic
- Query the database directly
- Perform complex data transformations

### Example: `user.controller.ts`

```ts
import { Request, Response } from 'express';
import * as userService from '../services/user.service';
import { sendSuccess, sendError } from '../utils/response';

export const getUsers = async (req: Request, res: Response) => {
  try {
    const users = await userService.getUsers();
    return sendSuccess(res, users, 'Users fetched successfully');
  } catch (error: any) {
    return sendError(res, error.message, 500);
  }
};

export const getUserById = async (req: Request, res: Response) => {
  try {
    const user = await userService.getUserById(req.params.id);
    if (!user) return sendError(res, 'User not found', 404);
    return sendSuccess(res, user);
  } catch (error: any) {
    return sendError(res, error.message, 500);
  }
};

export const createUser = async (req: Request, res: Response) => {
  try {
    const user = await userService.createUser(req.body);
    return sendSuccess(res, user, 'User created successfully', 201);
  } catch (error: any) {
    return sendError(res, error.message, 500);
  }
};
```

> **Tip:** Use an `asyncHandler` wrapper to avoid repeating try/catch in every controller.

```ts
// src/utils/asyncHandler.ts
import { Request, Response, NextFunction } from 'express';

export const asyncHandler =
  (fn: Function) => (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
```

---

## 7. Service Layer Best Practice

The **service layer owns all business logic**. It sits between controllers and repositories, orchestrating data access and applying rules.

### Services Should

- Implement all business rules and logic
- Call repository methods for data access
- Throw descriptive errors for edge cases
- Be fully testable in isolation

### Services Should NOT

- Handle HTTP request/response objects
- Return raw HTTP status codes

### Example: `user.service.ts`

```ts
import * as userRepository from '../repositories/user.repository';
import { CreateUserDTO } from '../types/user.types';
import { hashPassword } from '../utils/hash';

export const getUsers = async () => {
  return await userRepository.findAll();
};

export const getUserById = async (id: string) => {
  const user = await userRepository.findById(id);
  if (!user) throw new Error('User not found');
  return user;
};

export const createUser = async (dto: CreateUserDTO) => {
  const existing = await userRepository.findByEmail(dto.email);
  if (existing) throw new Error('Email already in use');

  const hashedPassword = await hashPassword(dto.password);
  return await userRepository.create({ ...dto, password: hashedPassword });
};
```

### Repository Pattern: `user.repository.ts`

```ts
import User from '../models/user.model';
import { CreateUserDTO } from '../types/user.types';

export const findAll = () => User.find().select('-password');
export const findById = (id: string) => User.findById(id).select('-password');
export const findByEmail = (email: string) => User.findOne({ email });
export const create = (data: CreateUserDTO) => User.create(data);
export const updateById = (id: string, data: Partial<CreateUserDTO>) =>
  User.findByIdAndUpdate(id, data, { new: true });
export const deleteById = (id: string) => User.findByIdAndDelete(id);
```

---

## 8. Validation Layer

**Never trust incoming request data.** Validate all inputs at the boundary before they reach your service layer. Use a schema-based validation library.

### Recommended Libraries

| Library | Best For |
|---|---|
| `zod` | TypeScript-first, type inference, modern |
| `joi` | Feature-rich, battle-tested |
| `class-validator` | Decorator-based, great with NestJS |

### Zod Example: `user.validator.ts`

```ts
import { z } from 'zod';

export const createUserSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  role: z.enum(['admin', 'user']).optional().default('user')
});

export const updateUserSchema = createUserSchema.partial();

export type CreateUserDTO = z.infer<typeof createUserSchema>;
export type UpdateUserDTO = z.infer<typeof updateUserSchema>;
```

### Validation Middleware

```ts
// src/middleware/validate.ts
import { Request, Response, NextFunction } from 'express';
import { ZodSchema } from 'zod';

export const validate =
  (schema: ZodSchema) => (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({
        success: false,
        message: 'Validation failed',
        errors: result.error.flatten().fieldErrors
      });
    }
    req.body = result.data;
    next();
  };
```

### Usage in Routes

```ts
router.post('/users', validate(createUserSchema), createUser);
```

---

## 9. Error Handling Middleware

Centralize error handling in a single global middleware to avoid scattered try/catch logic and ensure consistent error responses.

### Custom Error Class

```ts
// src/utils/AppError.ts
export class AppError extends Error {
  statusCode: number;
  isOperational: boolean;

  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

### Global Error Handler

```ts
// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '../utils/AppError';

export const errorHandler = (
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const statusCode = (err as AppError).statusCode || 500;
  const isOperational = (err as AppError).isOperational || false;

  console.error(`[ERROR] ${err.message}`, {
    stack: err.stack,
    path: req.path,
    method: req.method
  });

  return res.status(statusCode).json({
    success: false,
    message: isOperational ? err.message : 'Internal server error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};
```

### Register in `app.ts`

```ts
import { errorHandler } from './middleware/errorHandler';

// Must be registered AFTER all routes
app.use(errorHandler);
```

### Throw Errors from Services

```ts
import { AppError } from '../utils/AppError';

export const getUserById = async (id: string) => {
  const user = await userRepository.findById(id);
  if (!user) throw new AppError('User not found', 404);
  return user;
};
```

---

## 10. Authentication Best Practice

### JWT Authentication Flow

1. User logs in with credentials
2. Server validates and returns a signed JWT
3. Client stores token and sends it in the `Authorization` header
4. Server verifies the token on every protected route

### Authorization Header Format

```
Authorization: Bearer <token>
```

### JWT Auth Middleware

```ts
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { AppError } from '../utils/AppError';

export interface AuthRequest extends Request {
  user?: { id: string; role: string };
}

export const authenticate = (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return next(new AppError('No token provided', 401));
  }

  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
      id: string;
      role: string;
    };
    req.user = decoded;
    next();
  } catch {
    return next(new AppError('Invalid or expired token', 401));
  }
};
```

### Role-Based Authorization

```ts
export const authorize =
  (...roles: string[]) =>
  (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return next(new AppError('Access denied', 403));
    }
    next();
  };
```

### Usage

```ts
router.get('/admin/users', authenticate, authorize('admin'), getUsers);
```

### Best Practices

- Set short expiry on access tokens (15m – 1h)
- Use refresh tokens for extended sessions
- Store `JWT_SECRET` in environment variables — never hardcode
- Use HTTPS to prevent token interception

---

## 11. Pagination Best Practice

Never return unbounded lists of data. Always paginate large collections.

### Query Parameters

```
GET /users?page=1&limit=10
```

### Paginated Response Structure

```ts
{
  success: true,
  message: "Users fetched successfully",
  data: [...],
  meta: {
    page: 1,
    limit: 10,
    total: 100,
    totalPages: 10,
    hasNextPage: true,
    hasPrevPage: false
  }
}
```

### Pagination Utility

```ts
// src/utils/pagination.ts
export const paginate = (query: any) => {
  const page = Math.max(parseInt(query.page) || 1, 1);
  const limit = Math.min(parseInt(query.limit) || 10, 100);
  const skip = (page - 1) * limit;
  return { page, limit, skip };
};
```

### Service with Pagination

```ts
export const getUsers = async (query: any) => {
  const { page, limit, skip } = paginate(query);
  const [users, total] = await Promise.all([
    User.find().skip(skip).limit(limit).select('-password'),
    User.countDocuments()
  ]);

  return {
    data: users,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
      hasNextPage: page * limit < total,
      hasPrevPage: page > 1
    }
  };
};
```

---

## 12. Filtering & Sorting

Expose flexible query parameters for filtering and sorting without creating custom endpoints.

### Filtering

```
GET /users?role=admin
GET /products?category=electronics&inStock=true
GET /orders?status=pending&userId=abc123
```

### Sorting

```
GET /users?sort=createdAt          → Ascending by createdAt
GET /users?sort=-createdAt         → Descending by createdAt (prefix -)
GET /products?sort=-price,name     → Sort by price desc, then name asc
```

### Filter & Sort Helper

```ts
// src/utils/queryBuilder.ts
export const buildFilter = (query: Record<string, any>, allowedFields: string[]) => {
  const filter: Record<string, any> = {};
  allowedFields.forEach((field) => {
    if (query[field] !== undefined) filter[field] = query[field];
  });
  return filter;
};

export const buildSort = (sortQuery: string = 'createdAt') => {
  const sort: Record<string, 1 | -1> = {};
  sortQuery.split(',').forEach((field) => {
    if (field.startsWith('-')) sort[field.slice(1)] = -1;
    else sort[field] = 1;
  });
  return sort;
};
```

### Usage

```ts
const filter = buildFilter(req.query, ['role', 'status']);
const sort = buildSort(req.query.sort as string);
const users = await User.find(filter).sort(sort);
```

---

## 13. API Versioning

Versioning prevents breaking changes from affecting existing clients. Always version your API from day one.

### URL Path Versioning (Recommended)

```
/api/v1/users
/api/v2/users
```

### Route Setup

```ts
// src/app.ts
import v1Routes from './routes/v1';
import v2Routes from './routes/v2';

app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);
```

### When to Version

- Breaking changes to response structure
- Removing or renaming fields
- Changing authentication mechanism
- Major behavior changes to existing endpoints

### Alternatives

| Strategy | Example | Notes |
|---|---|---|
| URL Path | `/api/v1/users` | Most visible, easiest to test |
| Header | `API-Version: 2` | Cleaner URLs, harder to test in browser |
| Query Param | `?version=2` | Easy but pollutes query string |

> URL path versioning is the most widely adopted in production systems.

---

## 14. Middleware Best Practices

Middleware handles cross-cutting concerns applied across multiple routes.

### Auth Middleware

```ts
// Apply to protected routes only
router.use(authenticate);
```

### Request Logger Middleware

```ts
// src/middleware/logger.ts
import { Request, Response, NextFunction } from 'express';
import logger from '../config/logger';

export const requestLogger = (req: Request, res: Response, next: NextFunction) => {
  const start = Date.now();
  res.on('finish', () => {
    logger.info({
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration: `${Date.now() - start}ms`,
      ip: req.ip
    });
  });
  next();
};
```

### Not Found Handler

```ts
// src/middleware/notFound.ts
import { Request, Response } from 'express';

export const notFound = (req: Request, res: Response) => {
  res.status(404).json({
    success: false,
    message: `Route ${req.method} ${req.originalUrl} not found`
  });
};
```

### Middleware Registration Order in `app.ts`

```ts
app.use(helmet());
app.use(cors(corsOptions));
app.use(express.json({ limit: '10kb' }));
app.use(requestLogger);
app.use(rateLimiter);

// Routes
app.use('/api/v1', v1Routes);

// Catch-all
app.use(notFound);

// Global error handler (must be last)
app.use(errorHandler);
```

---

## 15. Production Security Best Practices

### Install Security Dependencies

```bash
npm install helmet cors express-rate-limit express-mongo-sanitize
```

### Helmet — HTTP Security Headers

```ts
import helmet from 'helmet';
app.use(helmet());
```

Helmet sets headers like `X-Content-Type-Options`, `Strict-Transport-Security`, `X-Frame-Options`, and more.

### CORS

```ts
import cors from 'cors';

const corsOptions = {
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
};

app.use(cors(corsOptions));
```

### Rate Limiting

```ts
import rateLimit from 'express-rate-limit';

export const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    success: false,
    message: 'Too many requests. Please try again later.'
  }
});

app.use('/api', rateLimiter);
```

### NoSQL Injection Sanitization

```ts
import mongoSanitize from 'express-mongo-sanitize';
app.use(mongoSanitize());
```

### Body Size Limiting

```ts
app.use(express.json({ limit: '10kb' }));
```

### Security Checklist

- [ ] Enable HTTPS in production
- [ ] Store secrets in environment variables
- [ ] Validate and sanitize all inputs
- [ ] Implement rate limiting on auth routes
- [ ] Use parameterized DB queries
- [ ] Disable `X-Powered-By` header (`app.disable('x-powered-by')`)
- [ ] Set secure, HttpOnly cookies for session tokens

---

## 16. Environment Variables

Never hardcode configuration values. All environment-specific values belong in `.env`.

### `.env` File

```env
# Server
NODE_ENV=development
PORT=5000

# Database
DB_URL=mongodb://localhost:27017/myapp

# Auth
JWT_SECRET=your_super_secret_key_here
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your_refresh_secret
JWT_REFRESH_EXPIRES_IN=7d

# CORS
ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com

# Logging
LOG_LEVEL=info
```

### `.env.example` — Always Commit This

```env
NODE_ENV=
PORT=
DB_URL=
JWT_SECRET=
JWT_EXPIRES_IN=
ALLOWED_ORIGINS=
LOG_LEVEL=
```

### Typed Config Module

```ts
// src/config/env.ts
import dotenv from 'dotenv';
dotenv.config();

export const config = {
  nodeEnv: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT || '5000', 10),
  dbUrl: process.env.DB_URL!,
  jwt: {
    secret: process.env.JWT_SECRET!,
    expiresIn: process.env.JWT_EXPIRES_IN || '15m'
  },
  allowedOrigins: process.env.ALLOWED_ORIGINS?.split(',') || []
};
```

> **Always add `.env` to `.gitignore`.** Never commit secrets to version control.

---

## 17. Logging Best Practice

Use a structured logging library instead of `console.log` in production.

### Recommended Libraries

| Library | Best For |
|---|---|
| `winston` | Flexible, feature-rich, widely used |
| `pino` | Extremely fast, JSON-first, low overhead |

### Winston Logger Setup

```ts
// src/config/logger.ts
import winston from 'winston';
import { config } from './env';

const logger = winston.createLogger({
  level: config.nodeEnv === 'production' ? 'warn' : 'debug',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    config.nodeEnv === 'production'
      ? winston.format.json()
      : winston.format.colorize(),
    config.nodeEnv !== 'production'
      ? winston.format.simple()
      : winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

export default logger;
```

### Structured Log Output (Production)

```json
{
  "level": "info",
  "message": "Request processed",
  "method": "GET",
  "url": "/api/v1/users",
  "status": 200,
  "duration": "23ms",
  "timestamp": "2025-01-15T10:30:00.000Z"
}
```

### Structured Logging Principles

- **Log structured data (JSON)** — not string concatenation
- **Never log sensitive data** — passwords, tokens, PII
- **Use log levels correctly** — `debug`, `info`, `warn`, `error`
- **Persist error logs** to files or external services (Datadog, Logtail, Sentry)
- **Correlate logs** with a request ID for tracing

---

## 18. Example Production User Module

A complete, production-ready user module showing the full request lifecycle.

### `src/routes/v1/user.routes.ts`

```ts
import { Router } from 'express';
import {
  getUsers,
  getUserById,
  createUser,
  updateUser,
  deleteUser
} from '../../controllers/user.controller';
import { validate } from '../../middleware/validate';
import { authenticate, authorize } from '../../middleware/auth';
import {
  createUserSchema,
  updateUserSchema
} from '../../validators/user.validator';

const router = Router();

router.get('/', authenticate, authorize('admin'), getUsers);
router.get('/:id', authenticate, getUserById);
router.post('/', validate(createUserSchema), createUser);
router.patch('/:id', authenticate, validate(updateUserSchema), updateUser);
router.delete('/:id', authenticate, authorize('admin'), deleteUser);

export default router;
```

### `src/controllers/user.controller.ts`

```ts
import { Request, Response, NextFunction } from 'express';
import * as userService from '../services/user.service';
import { sendSuccess } from '../utils/response';
import { asyncHandler } from '../utils/asyncHandler';

export const getUsers = asyncHandler(async (req: Request, res: Response) => {
  const result = await userService.getUsers(req.query);
  return sendSuccess(res, result.data, 'Users fetched', 200, result.meta);
});

export const getUserById = asyncHandler(async (req: Request, res: Response) => {
  const user = await userService.getUserById(req.params.id);
  return sendSuccess(res, user);
});

export const createUser = asyncHandler(async (req: Request, res: Response) => {
  const user = await userService.createUser(req.body);
  return sendSuccess(res, user, 'User created successfully', 201);
});

export const updateUser = asyncHandler(async (req: Request, res: Response) => {
  const user = await userService.updateUser(req.params.id, req.body);
  return sendSuccess(res, user, 'User updated successfully');
});

export const deleteUser = asyncHandler(async (req: Request, res: Response) => {
  await userService.deleteUser(req.params.id);
  return res.status(204).send();
});
```

### `src/services/user.service.ts`

```ts
import * as userRepo from '../repositories/user.repository';
import { AppError } from '../utils/AppError';
import { paginate } from '../utils/pagination';
import { buildFilter, buildSort } from '../utils/queryBuilder';
import { hashPassword } from '../utils/hash';
import { CreateUserDTO, UpdateUserDTO } from '../types/user.types';

export const getUsers = async (query: any) => {
  const { page, limit, skip } = paginate(query);
  const filter = buildFilter(query, ['role', 'status']);
  const sort = buildSort(query.sort);

  const [users, total] = await Promise.all([
    userRepo.findAll(filter, sort, skip, limit),
    userRepo.count(filter)
  ]);

  return {
    data: users,
    meta: { page, limit, total, totalPages: Math.ceil(total / limit) }
  };
};

export const getUserById = async (id: string) => {
  const user = await userRepo.findById(id);
  if (!user) throw new AppError('User not found', 404);
  return user;
};

export const createUser = async (dto: CreateUserDTO) => {
  const existing = await userRepo.findByEmail(dto.email);
  if (existing) throw new AppError('Email already registered', 409);
  dto.password = await hashPassword(dto.password);
  return await userRepo.create(dto);
};

export const updateUser = async (id: string, dto: UpdateUserDTO) => {
  const user = await userRepo.updateById(id, dto);
  if (!user) throw new AppError('User not found', 404);
  return user;
};

export const deleteUser = async (id: string) => {
  const user = await userRepo.deleteById(id);
  if (!user) throw new AppError('User not found', 404);
};
```

---

## 19. Golden Rules

The non-negotiables for production-quality REST APIs.

### Architecture

- **Thin controllers** — No business logic. Delegate everything to services.
- **Fat services** — All business rules, validation logic, and orchestration live here.
- **Isolated repositories** — Database access is centralized and easily swappable.

### API Design

- **Consistent responses** — Every response follows the same envelope structure.
- **Proper status codes** — Use semantically correct HTTP codes every time.
- **Plural nouns, no verbs** — Let HTTP methods do the talking.
- **Version from day one** — `/api/v1/` before you ever have v2.

### Quality & Safety

- **Validation everywhere** — Validate all inputs at the boundary. Never trust the client.
- **Centralized error handling** — One global error handler, custom error class.
- **Structured logging** — JSON logs with request IDs, no `console.log` in production.
- **Security by default** — Helmet, CORS, rate limiting, input sanitization on every app.

### Team Practices

- **Never commit `.env`** — Use `.env.example` as a template.
- **Type everything** — Use TypeScript interfaces and DTOs for all data shapes.
- **Document as you build** — Keep this guide updated with your team's decisions.
- **Write tests** — Unit test services, integration test routes.

---

> *This document reflects industry-standard practices for Node.js + Express + TypeScript APIs at scale. Review and adapt to your team's specific needs.*

---

*Last updated: 2025 · Stack: Node.js · Express.js · TypeScript · Zod · Winston · JWT*
