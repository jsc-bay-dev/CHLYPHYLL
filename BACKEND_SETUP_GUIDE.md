# CHLYPHYLL Backend Setup Guide

## Prerequisites
- Node.js (v18 or higher) installed
- PostgreSQL installed and running
- Git installed
- Code editor (VS Code recommended)

## Step 1: Initialize the Backend Project

```bash
# Navigate to your project root
cd /path/to/CHLYPHYLL

# Create server directory
mkdir server
cd server

# Initialize Node.js project
npm init -y

# Install core dependencies
npm install express cors helmet morgan dotenv

# Install development dependencies
npm install -D nodemon @types/node typescript ts-node @types/express @types/cors
```

## Step 2: Set Up TypeScript Configuration

```bash
# Create TypeScript config
npx tsc --init
```

Edit `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Step 3: Set Up Prisma and Database

```bash
# Install Prisma
npm install prisma @prisma/client

# Initialize Prisma
npx prisma init
```

Edit `prisma/schema.prisma`:
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  auth0Id   String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  orders Order[]
}

model Product {
  id          String   @id @default(cuid())
  name        String
  description String?
  price       Decimal
  category    String
  imageUrl    String?
  stock       Int      @default(0)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  orderItems OrderItem[]
}

model Order {
  id        String   @id @default(cuid())
  userId    String
  status    String   @default("pending")
  total     Decimal
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  user      User        @relation(fields: [userId], references: [id])
  items     OrderItem[]
}

model OrderItem {
  id        String   @id @default(cuid())
  orderId   String
  productId String
  quantity  Int
  price     Decimal
  
  order   Order   @relation(fields: [orderId], references: [id])
  product Product @relation(fields: [productId], references: [id])
}
```

## Step 4: Configure Environment Variables

Create `.env` file:
```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/chlyphyll_db"

# Server
PORT=5000
NODE_ENV=development

# Auth0 (add later)
AUTH0_AUDIENCE=
AUTH0_ISSUER_BASE_URL=
```

## Step 5: Create Project Structure

```bash
# Create directory structure
mkdir -p src/{controllers,models,routes,middleware,services,utils}
mkdir -p src/types
```

## Step 6: Set Up Express Server

Create `src/app.ts`:
```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import dotenv from 'dotenv';

// Load environment variables
dotenv.config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(helmet());
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Basic route
app.get('/', (req, res) => {
  res.json({ message: 'Welcome to CHLYPHYLL API' });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

export default app;
```

## Step 7: Create Database Connection

Create `src/lib/prisma.ts`:
```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export default prisma;
```

## Step 8: Set Up Basic Routes

Create `src/routes/index.ts`:
```typescript
import { Router } from 'express';
import productRoutes from './products';
import userRoutes from './users';
import orderRoutes from './orders';

const router = Router();

router.use('/products', productRoutes);
router.use('/users', userRoutes);
router.use('/orders', orderRoutes);

export default router;
```

## Step 9: Create Product Routes

Create `src/routes/products.ts`:
```typescript
import { Router } from 'express';
import prisma from '../lib/prisma';

const router = Router();

// Get all products
router.get('/', async (req, res) => {
  try {
    const products = await prisma.product.findMany();
    res.json(products);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch products' });
  }
});

// Get product by ID
router.get('/:id', async (req, res) => {
  try {
    const product = await prisma.product.findUnique({
      where: { id: req.params.id }
    });
    
    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }
    
    res.json(product);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch product' });
  }
});

export default router;
```

## Step 10: Update Package.json Scripts

Edit `package.json`:
```json
{
  "scripts": {
    "dev": "nodemon src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio"
  }
}
```

## Step 11: Set Up Database

```bash
# Create database (in PostgreSQL)
createdb chlyphyll_db

# Generate Prisma client
npm run db:generate

# Push schema to database
npm run db:push
```

## Step 12: Test Your Setup

```bash
# Start development server
npm run dev
```

Visit `http://localhost:5000` to see your API response.

## Step 13: Add More Features

### Authentication Middleware
Create `src/middleware/auth.ts`:
```typescript
import { Request, Response, NextFunction } from 'express';

export const authenticateToken = (req: Request, res: Response, next: NextFunction) => {
  // TODO: Implement Auth0 JWT verification
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  // TODO: Verify JWT with Auth0
  next();
};
```

### Error Handling Middleware
Create `src/middleware/errorHandler.ts`:
```typescript
import { Request, Response, NextFunction } from 'express';

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
};
```

## Step 14: Environment-Specific Configuration

Create `src/config/index.ts`:
```typescript
export const config = {
  port: process.env.PORT || 5000,
  nodeEnv: process.env.NODE_ENV || 'development',
  database: {
    url: process.env.DATABASE_URL!,
  },
  auth0: {
    audience: process.env.AUTH0_AUDIENCE,
    issuerBaseUrl: process.env.AUTH0_ISSUER_BASE_URL,
  },
};
```

## Next Steps

1. **Add more routes** for users, orders, and other features
2. **Implement Auth0 integration** using their SDK
3. **Add input validation** using libraries like Joi or Zod
4. **Set up testing** with Jest
5. **Add logging** with Winston or Pino
6. **Implement caching** with Redis
7. **Add rate limiting** for API protection
8. **Set up CI/CD** pipeline

## Useful Commands

```bash
# Generate Prisma client after schema changes
npm run db:generate

# Create and apply database migrations
npm run db:migrate

# Open Prisma Studio (database GUI)
npm run db:studio

# Reset database (development only)
npx prisma db push --force-reset
```

## Troubleshooting

- **Port already in use**: Change PORT in .env file
- **Database connection issues**: Check DATABASE_URL and PostgreSQL status
- **Prisma errors**: Run `npm run db:generate` after schema changes
- **TypeScript errors**: Check tsconfig.json and run `npm run build`

This setup gives you a solid foundation for your ecommerce backend that you can build upon!
