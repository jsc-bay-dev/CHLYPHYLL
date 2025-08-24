# Frontend Setup Guide - E-Commerce Platform
## Step-by-Step Instructions

### Prerequisites
- Node.js (v18 or higher) installed on your system
- Git for version control
- A code editor (VS Code recommended)
- Basic knowledge of React, TypeScript, and modern web development

---

## Phase 1: Project Initialization

### Step 1: Create Next.js Project
```bash
# Navigate to your project directory
cd CHLYPHYLL

# Create a new Next.js project with TypeScript
npx create-next-app@latest client --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

# Navigate into the client directory
cd client
```

### Step 2: Install Core Dependencies
```bash
# Install essential packages for e-commerce functionality
npm install @reduxjs/toolkit react-redux
npm install @tanstack/react-query
npm install axios
npm install react-hook-form @hookform/resolvers zod
npm install framer-motion
npm install lucide-react
npm install clsx tailwind-merge
npm install @headlessui/react
npm install react-hot-toast
```

### Step 3: Install Development Dependencies
```bash
# Install development tools
npm install -D @types/node
npm install -D prettier prettier-plugin-tailwindcss
npm install -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

---

## Phase 2: Project Structure Setup

### Step 4: Create Directory Structure
```bash
# Create the following directory structure in your client/src directory
mkdir -p src/components/{ui,layout,product,cart,checkout,user,auth}
mkdir -p src/hooks
mkdir -p src/lib
mkdir -p src/store
mkdir -p src/types
mkdir -p src/utils
mkdir -p src/services
mkdir -p src/styles
mkdir -p src/app/{products,cart,checkout,account,admin}
```

### Step 5: Configure TypeScript
Update `tsconfig.json` to include strict type checking:
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

---

## Phase 3: Core Configuration

### Step 6: Setup Tailwind CSS Configuration
Create `tailwind.config.ts`:
```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
        secondary: {
          50: '#f8fafc',
          500: '#64748b',
          600: '#475569',
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [],
}

export default config
```

### Step 7: Setup Environment Variables
Create `.env.local` file:
```env
NEXT_PUBLIC_API_URL=http://localhost:3001/api
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your_stripe_key_here
NEXT_PUBLIC_APP_NAME=CHLYPHYLL
```

---

## Phase 4: State Management Setup

### Step 8: Configure Redux Store
Create `src/store/index.ts`:
```typescript
import { configureStore } from '@reduxjs/toolkit'
import cartReducer from './cartSlice'
import userReducer from './userSlice'
import productReducer from './productSlice'

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    user: userReducer,
    product: productReducer,
  },
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

### Step 9: Create Cart Slice
Create `src/store/cartSlice.ts`:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

interface CartItem {
  id: string
  name: string
  price: number
  quantity: number
  image: string
}

interface CartState {
  items: CartItem[]
  total: number
}

const initialState: CartState = {
  items: [],
  total: 0,
}

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem: (state, action: PayloadAction<CartItem>) => {
      const existingItem = state.items.find(item => item.id === action.payload.id)
      if (existingItem) {
        existingItem.quantity += action.payload.quantity
      } else {
        state.items.push(action.payload)
      }
      state.total = state.items.reduce((sum, item) => sum + (item.price * item.quantity), 0)
    },
    removeItem: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter(item => item.id !== action.payload)
      state.total = state.items.reduce((sum, item) => sum + (item.price * item.quantity), 0)
    },
    updateQuantity: (state, action: PayloadAction<{ id: string; quantity: number }>) => {
      const item = state.items.find(item => item.id === action.payload.id)
      if (item) {
        item.quantity = action.payload.quantity
        state.total = state.items.reduce((sum, item) => sum + (item.price * item.quantity), 0)
      }
    },
    clearCart: (state) => {
      state.items = []
      state.total = 0
    },
  },
})

export const { addItem, removeItem, updateQuantity, clearCart } = cartSlice.actions
export default cartSlice.reducer
```

---

## Phase 5: Component Development

### Step 10: Create Layout Components
Create `src/components/layout/Header.tsx`:
```typescript
'use client'

import Link from 'next/link'
import { ShoppingCart, User, Search } from 'lucide-react'
import { useSelector } from 'react-redux'
import { RootState } from '@/store'

export default function Header() {
  const cartItems = useSelector((state: RootState) => state.cart.items)
  const cartItemCount = cartItems.reduce((sum, item) => sum + item.quantity, 0)

  return (
    <header className="bg-white shadow-sm border-b">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between items-center h-16">
          <Link href="/" className="text-2xl font-bold text-primary-600">
            CHLYPHYLL
          </Link>
          
          <div className="flex items-center space-x-4">
            <div className="relative">
              <input
                type="text"
                placeholder="Search products..."
                className="w-64 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary-500 focus:border-transparent"
              />
              <Search className="absolute right-3 top-2.5 h-5 w-5 text-gray-400" />
            </div>
            
            <Link href="/cart" className="relative">
              <ShoppingCart className="h-6 w-6 text-gray-600" />
              {cartItemCount > 0 && (
                <span className="absolute -top-2 -right-2 bg-red-500 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center">
                  {cartItemCount}
                </span>
              )}
            </Link>
            
            <Link href="/account">
              <User className="h-6 w-6 text-gray-600" />
            </Link>
          </div>
        </div>
      </div>
    </header>
  )
}
```

### Step 11: Create Product Card Component
Create `src/components/product/ProductCard.tsx`:
```typescript
'use client'

import Image from 'next/image'
import { ShoppingCart } from 'lucide-react'
import { useDispatch } from 'react-redux'
import { addItem } from '@/store/cartSlice'
import { Product } from '@/types/product'

interface ProductCardProps {
  product: Product
}

export default function ProductCard({ product }: ProductCardProps) {
  const dispatch = useDispatch()

  const handleAddToCart = () => {
    dispatch(addItem({
      id: product.id,
      name: product.name,
      price: product.price,
      quantity: 1,
      image: product.images[0],
    }))
  }

  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
      <div className="relative h-48 w-full">
        <Image
          src={product.images[0]}
          alt={product.name}
          fill
          className="object-cover"
        />
      </div>
      
      <div className="p-4">
        <h3 className="text-lg font-semibold text-gray-900 mb-2">
          {product.name}
        </h3>
        <p className="text-gray-600 text-sm mb-3 line-clamp-2">
          {product.description}
        </p>
        
        <div className="flex items-center justify-between">
          <span className="text-xl font-bold text-primary-600">
            ${product.price.toFixed(2)}
          </span>
          
          <button
            onClick={handleAddToCart}
            className="bg-primary-600 text-white px-4 py-2 rounded-lg hover:bg-primary-700 transition-colors flex items-center space-x-2"
          >
            <ShoppingCart className="h-4 w-4" />
            <span>Add to Cart</span>
          </button>
        </div>
      </div>
    </div>
  )
}
```

---

## Phase 6: Page Development

### Step 12: Create Home Page
Update `src/app/page.tsx`:
```typescript
import { Suspense } from 'react'
import ProductGrid from '@/components/product/ProductGrid'
import Hero from '@/components/layout/Hero'

export default function HomePage() {
  return (
    <main className="min-h-screen">
      <Hero />
      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <h2 className="text-3xl font-bold text-gray-900 mb-8">Featured Products</h2>
        <Suspense fallback={<div>Loading products...</div>}>
          <ProductGrid />
        </Suspense>
      </section>
    </main>
  )
}
```

### Step 13: Create Products Page
Create `src/app/products/page.tsx`:
```typescript
import ProductGrid from '@/components/product/ProductGrid'
import ProductFilters from '@/components/product/ProductFilters'

export default function ProductsPage() {
  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      <h1 className="text-3xl font-bold text-gray-900 mb-8">All Products</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-4 gap-8">
        <aside className="lg:col-span-1">
          <ProductFilters />
        </aside>
        
        <main className="lg:col-span-3">
          <ProductGrid />
        </main>
      </div>
    </div>
  )
}
```

---

## Phase 7: API Integration

### Step 14: Create API Service
Create `src/services/api.ts`:
```typescript
import axios from 'axios'

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
})

// Request interceptor for adding auth token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('authToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Response interceptor for error handling
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('authToken')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
```

### Step 15: Create Product Service
Create `src/services/productService.ts`:
```typescript
import api from './api'
import { Product } from '@/types/product'

export const productService = {
  async getProducts(params?: {
    category?: string
    search?: string
    sort?: string
    page?: number
    limit?: number
  }): Promise<{ products: Product[]; total: number }> {
    const response = await api.get('/products', { params })
    return response.data
  },

  async getProduct(id: string): Promise<Product> {
    const response = await api.get(`/products/${id}`)
    return response.data
  },

  async getCategories(): Promise<string[]> {
    const response = await api.get('/products/categories')
    return response.data
  },
}
```

---

## Phase 8: Authentication Setup

### Step 16: Create Auth Context
Create `src/contexts/AuthContext.tsx`:
```typescript
'use client'

import { createContext, useContext, useState, useEffect, ReactNode } from 'react'
import { User } from '@/types/user'

interface AuthContextType {
  user: User | null
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  isLoading: boolean
}

const AuthContext = createContext<AuthContextType | undefined>(undefined)

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    // Check for existing auth token on app load
    const token = localStorage.getItem('authToken')
    if (token) {
      // Validate token and set user
      validateToken(token)
    } else {
      setIsLoading(false)
    }
  }, [])

  const validateToken = async (token: string) => {
    try {
      // Call API to validate token and get user info
      // Set user if valid
      setIsLoading(false)
    } catch (error) {
      localStorage.removeItem('authToken')
      setIsLoading(false)
    }
  }

  const login = async (email: string, password: string) => {
    // Implement login logic
  }

  const logout = () => {
    localStorage.removeItem('authToken')
    setUser(null)
  }

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  )
}

export function useAuth() {
  const context = useContext(AuthContext)
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider')
  }
  return context
}
```

---

## Phase 9: Testing Setup

### Step 17: Configure Testing Environment
Update `jest.config.js`:
```javascript
const nextJest = require('next/jest')

const createJestConfig = nextJest({
  dir: './',
})

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jsdom',
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
}

module.exports = createJestConfig(customJestConfig)
```

### Step 18: Create Test Files
Create `src/components/__tests__/ProductCard.test.tsx`:
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Provider } from 'react-redux'
import { configureStore } from '@reduxjs/toolkit'
import ProductCard from '../product/ProductCard'
import cartReducer from '@/store/cartSlice'

const mockProduct = {
  id: '1',
  name: 'Test Product',
  price: 99.99,
  description: 'A test product',
  images: ['/test-image.jpg'],
}

const store = configureStore({
  reducer: {
    cart: cartReducer,
  },
})

describe('ProductCard', () => {
  it('renders product information correctly', () => {
    render(
      <Provider store={store}>
        <ProductCard product={mockProduct} />
      </Provider>
    )

    expect(screen.getByText('Test Product')).toBeInTheDocument()
    expect(screen.getByText('$99.99')).toBeInTheDocument()
    expect(screen.getByText('A test product')).toBeInTheDocument()
  })

  it('adds product to cart when add to cart button is clicked', () => {
    render(
      <Provider store={store}>
        <ProductCard product={mockProduct} />
      </Provider>
    )

    const addToCartButton = screen.getByText('Add to Cart')
    fireEvent.click(addToCartButton)

    // Verify cart state was updated
    const state = store.getState()
    expect(state.cart.items).toHaveLength(1)
    expect(state.cart.items[0].id).toBe('1')
  })
})
```

---

## Phase 10: Build and Deployment

### Step 19: Build Configuration
Update `next.config.ts`:
```typescript
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    appDir: true,
  },
  images: {
    domains: ['your-image-domain.com'],
  },
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
}

export default nextConfig
```

### Step 20: Build and Test
```bash
# Build the application
npm run build

# Test the build locally
npm start

# Run tests
npm test

# Run linting
npm run lint
```

---

## Phase 11: Performance Optimization

### Step 21: Implement Code Splitting
Update your components to use dynamic imports:
```typescript
import dynamic from 'next/dynamic'

const ProductGrid = dynamic(() => import('@/components/product/ProductGrid'), {
  loading: () => <div>Loading...</div>,
  ssr: false,
})
```

### Step 22: Add Loading States
Create loading components for better UX:
```typescript
// src/components/ui/LoadingSpinner.tsx
export default function LoadingSpinner() {
  return (
    <div className="flex justify-center items-center">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary-600"></div>
    </div>
  )
}
```

---

## Final Steps

### Step 23: Environment Setup
1. Copy `.env.local.example` to `.env.local`
2. Fill in your actual API endpoints and keys
3. Update image domains in `next.config.ts`

### Step 24: Run Development Server
```bash
npm run dev
```

Your frontend should now be running at `http://localhost:3000`

---

## Troubleshooting Common Issues

### Issue: TypeScript Errors
- Ensure all dependencies are properly installed
- Check `tsconfig.json` configuration
- Verify import paths are correct

### Issue: Styling Not Working
- Verify Tailwind CSS is properly configured
- Check that CSS files are imported in `layout.tsx`
- Ensure PostCSS configuration is correct

### Issue: Redux Store Not Working
- Verify Provider is wrapping your app
- Check that slices are properly exported
- Ensure actions are dispatched correctly

---

## Next Steps

After completing this setup:
1. Implement remaining components (checkout, user management, etc.)
2. Add error boundaries and error handling
3. Implement SEO optimization
4. Add analytics and monitoring
5. Set up CI/CD pipeline
6. Configure production deployment

This guide provides a solid foundation for your e-commerce frontend. Each step builds upon the previous one, ensuring a robust and scalable application structure.
