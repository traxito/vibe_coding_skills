---
name: SaaS Stack & Architecture
description: Tech stack, project structure, and coding standards for building SaaS applications. Based on proven architecture from entiendetunomina and valuebetanalytics. Use this skill for all new projects and when generating code to ensure consistency with established patterns and technologies.
applyTo: "**/*.{ts,tsx,js,jsx,json,css}"
---

# SaaS Stack & Architecture Standards

Complete tech stack and architectural patterns for building production-ready SaaS applications.

## When to Use This Skill

- Starting a new SaaS project
- Generating any frontend or backend code
- Setting up project structure
- Making technology decisions
- Writing components, hooks, or utilities
- Configuring build tools or deployments

---

## Technology Stack

### Frontend

```json
{
  "framework": "React 18",
  "language": "TypeScript (strict mode)",
  "buildTool": "Vite",
  "styling": "Tailwind CSS",
  "uiComponents": "Shadcn UI",
  "icons": "Lucide React",
  "routing": "React Router v6",
  "forms": "React Hook Form + Zod",
  "stateManagement": "Context API + Custom Hooks",
  "notifications": "Sonner (toast)",
  "dateHandling": "date-fns",
  "http": "Fetch API with retry logic"
}
```

### Backend

```json
{
  "platform": "Firebase",
  "authentication": "Firebase Auth (Email, Google OAuth)",
  "database": "Cloud Firestore",
  "storage": "Firebase Storage",
  "functions": "Firebase Cloud Functions (Node.js/TypeScript)",
  "hosting": "Firebase Hosting",
  "payments": "Stripe",
  "email": "Gmail SMTP + Nodemailer",
  "monitoring": "Firebase Analytics + Cloud Logging"
}
```

### External Services

```json
{
  "dns": "Cloudflare",
  "domain": "Cloudflare Registrar",
  "cdn": "Cloudflare (optional, Firebase has built-in CDN)",
  "email": "Gmail SMTP (transactional)",
  "payments": "Stripe"
}
```

---

## Project Structure

### Standard Directory Layout

```
project-root/
├── src/
│   ├── components/
│   │   ├── common/              # Reusable components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Toast.tsx
│   │   │   └── LoadingSpinner.tsx
│   │   ├── layout/              # Layout components
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── PageLayout.tsx
│   │   └── features/            # Feature-specific components
│   │       ├── auth/
│   │       ├── dashboard/
│   │       └── pricing/
│   ├── contexts/                # React contexts
│   │   ├── AuthContext.tsx
│   │   └── ThemeContext.tsx
│   ├── hooks/                   # Custom hooks
│   │   ├── useAuth.ts
│   │   ├── useToast.ts
│   │   ├── useFirestore.ts
│   │   └── useLocalStorage.ts
│   ├── lib/                     # External service clients
│   │   ├── firebase.ts
│   │   ├── stripe-client.ts
│   │   └── analytics.ts
│   ├── pages/                   # Page components
│   │   ├── public/              # Public pages
│   │   │   ├── Home.tsx
│   │   │   ├── Pricing.tsx
│   │   │   ├── Blog.tsx
│   │   │   └── Contact.tsx
│   │   ├── auth/                # Auth pages
│   │   │   ├── Login.tsx
│   │   │   ├── Signup.tsx
│   │   │   └── ResetPassword.tsx
│   │   └── dashboard/           # Protected pages
│   │       ├── Dashboard.tsx
│   │       ├── Settings.tsx
│   │       └── Profile.tsx
│   ├── services/                # API clients & business logic
│   │   ├── authService.ts
│   │   ├── firestoreService.ts
│   │   ├── stripeService.ts
│   │   └── emailService.ts
│   ├── types/                   # TypeScript types
│   │   ├── user.ts
│   │   ├── subscription.ts
│   │   └── api.ts
│   ├── utils/                   # Utility functions
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   └── constants.ts
│   ├── App.tsx                  # Main app component
│   ├── main.tsx                 # Entry point
│   └── index.css                # Global styles
├── functions/                   # Firebase Cloud Functions
│   ├── src/
│   │   ├── index.ts             # Functions entry
│   │   ├── auth/                # Auth triggers
│   │   ├── stripe/              # Stripe webhooks
│   │   ├── emails/              # Email functions
│   │   ├── api/                 # HTTP callable functions
│   │   └── scheduled/           # Scheduled functions
│   ├── package.json
│   └── tsconfig.json
├── public/                      # Static assets
│   ├── favicon.ico
│   ├── logo.png
│   └── robots.txt
├── .env.local                   # Local environment variables
├── .gitignore
├── firebase.json                # Firebase config
├── firestore.rules              # Firestore security rules
├── firestore.indexes.json       # Composite indexes
├── storage.rules                # Storage security rules
├── package.json
├── tsconfig.json                # TypeScript config
├── tailwind.config.js           # Tailwind config
├── vite.config.ts               # Vite config
└── README.md
```

---

## Configuration Files

### package.json (Frontend)

```json
{
  "name": "your-saas-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "deploy": "npm run build && npx firebase deploy --only hosting"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "react-router-dom": "^6.22.0",
    "firebase": "^10.8.0",
    "@stripe/stripe-js": "^3.0.0",
    "@stripe/react-stripe-js": "^2.5.0",
    "react-hook-form": "^7.50.0",
    "zod": "^3.22.0",
    "@hookform/resolvers": "^3.3.0",
    "lucide-react": "^0.344.0",
    "sonner": "^1.4.0",
    "date-fns": "^3.3.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "typescript": "^5.3.0",
    "vite": "^5.1.0",
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### vite.config.ts

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'firebase-vendor': ['firebase/app', 'firebase/auth', 'firebase/firestore', 'firebase/storage'],
          'stripe-vendor': ['@stripe/stripe-js', '@stripe/react-stripe-js'],
        },
      },
    },
  },
});
```

### tailwind.config.js

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

### firebase.json

```json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(jpg|jpeg|gif|png|svg|webp|js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ]
  },
  "functions": {
    "source": "functions",
    "runtime": "nodejs18"
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "storage": {
    "rules": "storage.rules"
  }
}
```

---

## Code Patterns & Standards

### Component Structure

```tsx
// ✅ CORRECT: Consistent component pattern
import { useState, useEffect } from 'react';
import { useAuth } from '@/hooks/useAuth';
import type { User } from '@/types/user';

interface ComponentProps {
  title: string;
  onAction?: () => void;
}

export default function Component({ title, onAction }: ComponentProps) {
  // 1. Hooks first
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);

  // 2. Effects
  useEffect(() => {
    // Side effects
  }, []);

  // 3. Event handlers
  const handleClick = () => {
    setLoading(true);
    // Logic
    onAction?.();
    setLoading(false);
  };

  // 4. Early returns
  if (!user) {
    return <div>Please login</div>;
  }

  // 5. Main render
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold">{title}</h1>
      <button 
        onClick={handleClick}
        disabled={loading}
        className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        {loading ? 'Loading...' : 'Click Me'}
      </button>
    </div>
  );
}
```

### Custom Hook Pattern

```typescript
// ✅ CORRECT: Custom hook pattern
import { useState, useEffect } from 'react';
import { collection, query, where, onSnapshot } from 'firebase/firestore';
import { db } from '@/lib/firebase';

interface UseFirestoreOptions<T> {
  collectionName: string;
  queryConstraints?: any[];
}

export function useFirestore<T>(options: UseFirestoreOptions<T>) {
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const q = options.queryConstraints
      ? query(collection(db, options.collectionName), ...options.queryConstraints)
      : collection(db, options.collectionName);

    const unsubscribe = onSnapshot(
      q,
      (snapshot) => {
        const items = snapshot.docs.map(doc => ({
          id: doc.id,
          ...doc.data(),
        })) as T[];

        setData(items);
        setLoading(false);
      },
      (err) => {
        setError(err as Error);
        setLoading(false);
      }
    );

    return () => unsubscribe();
  }, [options.collectionName, options.queryConstraints]);

  return { data, loading, error };
}
```

### Service Pattern

```typescript
// ✅ CORRECT: Service pattern for business logic
import { doc, getDoc, setDoc, updateDoc, deleteDoc } from 'firebase/firestore';
import { db } from '@/lib/firebase';
import type { User } from '@/types/user';

export class UserService {
  private static COLLECTION = 'users';

  static async getUser(userId: string): Promise<User | null> {
    try {
      const docRef = doc(db, this.COLLECTION, userId);
      const docSnap = await getDoc(docRef);

      if (docSnap.exists()) {
        return { id: docSnap.id, ...docSnap.data() } as User;
      }
      return null;
    } catch (error) {
      console.error('Error getting user:', error);
      throw error;
    }
  }

  static async createUser(userId: string, data: Partial<User>): Promise<void> {
    try {
      await setDoc(doc(db, this.COLLECTION, userId), {
        ...data,
        createdAt: new Date(),
        updatedAt: new Date(),
      });
    } catch (error) {
      console.error('Error creating user:', error);
      throw error;
    }
  }

  static async updateUser(userId: string, data: Partial<User>): Promise<void> {
    try {
      await updateDoc(doc(db, this.COLLECTION, userId), {
        ...data,
        updatedAt: new Date(),
      });
    } catch (error) {
      console.error('Error updating user:', error);
      throw error;
    }
  }

  static async deleteUser(userId: string): Promise<void> {
    try {
      await deleteDoc(doc(db, this.COLLECTION, userId));
    } catch (error) {
      console.error('Error deleting user:', error);
      throw error;
    }
  }
}
```

### Protected Route Pattern

```tsx
// ✅ CORRECT: Protected route wrapper
import { Navigate } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import LoadingSpinner from '@/components/common/LoadingSpinner';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requireEmailVerification?: boolean;
}

export default function ProtectedRoute({ 
  children, 
  requireEmailVerification = false 
}: ProtectedRouteProps) {
  const { user, loading } = useAuth();

  if (loading) {
    return <LoadingSpinner />;
  }

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  if (requireEmailVerification && !user.emailVerified) {
    return <Navigate to="/verify-email" replace />;
  }

  return <>{children}</>;
}
```

### Error Handling Pattern

```typescript
// ✅ CORRECT: Consistent error handling
export async function fetchData<T>(url: string): Promise<T> {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch error:', error);

    // Re-throw with context
    if (error instanceof Error) {
      throw new Error(`Failed to fetch data: ${error.message}`);
    }

    throw new Error('Failed to fetch data: Unknown error');
  }
}
```

---

## Naming Conventions

### Files and Folders

```
✅ CORRECT:
- Components: PascalCase (Button.tsx, UserProfile.tsx)
- Hooks: camelCase with 'use' prefix (useAuth.ts, useFirestore.ts)
- Utilities: camelCase (formatDate.ts, validateEmail.ts)
- Types: camelCase (user.ts, subscription.ts)
- Services: camelCase with 'Service' suffix (authService.ts, emailService.ts)
- Pages: PascalCase (Dashboard.tsx, Pricing.tsx)

❌ WRONG:
- button.tsx (should be Button.tsx)
- UseAuth.ts (should be useAuth.ts)
- format-date.ts (should be formatDate.ts)
```

### Variables and Functions

```typescript
// ✅ CORRECT:
const userName = 'John';
const isLoading = false;
const userCount = 10;
const API_KEY = 'abc123';

function fetchUserData() {}
function calculateTotal(items: Item[]) {}
async function sendEmail(to: string) {}

// ❌ WRONG:
const user_name = 'John';  // Use camelCase
const IsLoading = false;   // Variables start lowercase
const USERCOUNT = 10;      // Not a constant

function FetchUserData() {} // Functions start lowercase
```

### Components and Classes

```typescript
// ✅ CORRECT:
class UserService {}
class EmailValidator {}
function UserProfile() {}
function PricingCard() {}

// ❌ WRONG:
class userService {}      // Classes use PascalCase
function userProfile() {} // Components use PascalCase
```

---

## TypeScript Best Practices

### Type Definitions

```typescript
// ✅ CORRECT: Explicit types
import type { Timestamp } from 'firebase/firestore';

export interface User {
  id: string;
  email: string;
  displayName: string | null;
  photoURL: string | null;
  plan: 'free' | 'pro' | 'enterprise';
  subscriptionStatus: 'active' | 'canceled' | 'past_due';
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

export interface Subscription {
  id: string;
  userId: string;
  priceId: string;
  status: 'active' | 'canceled' | 'incomplete' | 'past_due';
  currentPeriodStart: Date;
  currentPeriodEnd: Date;
  cancelAtPeriodEnd: boolean;
}

// ❌ WRONG: Using 'any'
export interface User {
  id: string;
  data: any;  // Avoid 'any', define explicit types
}
```

### Type Imports

```typescript
// ✅ CORRECT: Use 'type' for type-only imports
import type { User } from '@/types/user';
import type { FC, ReactNode } from 'react';

// Regular imports for values
import { useState } from 'react';
import { db } from '@/lib/firebase';
```

### Generic Types

```typescript
// ✅ CORRECT: Generic utility functions
export async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json();
}

// Usage
const user = await fetchData<User>('/api/user');
```

---

## Styling Standards

### Tailwind CSS Patterns

```tsx
// ✅ CORRECT: Consistent Tailwind usage

// 1. Utility classes directly in JSX
<button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 disabled:opacity-50">
  Click Me
</button>

// 2. Conditional classes with clsx
import { clsx } from 'clsx';

<div className={clsx(
  'p-4 rounded',
  isActive && 'bg-blue-500 text-white',
  isDisabled && 'opacity-50 cursor-not-allowed'
)}>
  Content
</div>

// 3. Reusable class combinations
const buttonClasses = 'px-4 py-2 rounded font-medium transition-colors';

<button className={`${buttonClasses} bg-blue-500 hover:bg-blue-600`}>
  Primary
</button>
<button className={`${buttonClasses} bg-gray-500 hover:bg-gray-600`}>
  Secondary
</button>

// ❌ WRONG: Inline styles (avoid when possible)
<div style={{ padding: '1rem', color: 'blue' }}>
  Use Tailwind classes instead
</div>
```

### Responsive Design

```tsx
// ✅ CORRECT: Mobile-first responsive design
<div className="
  p-4 
  md:p-6 
  lg:p-8

  grid 
  grid-cols-1 
  md:grid-cols-2 
  lg:grid-cols-3 
  gap-4
">
  {/* Content */}
</div>
```

---

## Authentication Pattern

### Firebase Auth Integration

```typescript
// lib/firebase.ts - Auth configuration
import { initializeApp } from 'firebase/app';
import { getAuth, connectAuthEmulator } from 'firebase/auth';
import { getFirestore, connectFirestoreEmulator } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);

// Emulators in development
if (import.meta.env.DEV) {
  connectAuthEmulator(auth, 'http://localhost:9099', { disableWarnings: true });
  connectFirestoreEmulator(db, 'localhost', 8080);
}
```

### AuthContext Implementation

```typescript
// contexts/AuthContext.tsx
import { createContext, useContext, useEffect, useState } from 'react';
import type { User } from 'firebase/auth';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from '@/lib/firebase';

interface AuthContextType {
  user: User | null;
  loading: boolean;
}

const AuthContext = createContext<AuthContextType>({
  user: null,
  loading: true,
});

export function useAuth() {
  return useContext(AuthContext);
}

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setUser(user);
      setLoading(false);
    });

    return () => unsubscribe();
  }, []);

  return (
    <AuthContext.Provider value={{ user, loading }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

## Routing Pattern

### React Router Setup

```tsx
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { lazy, Suspense } from 'react';
import ProtectedRoute from '@/components/auth/ProtectedRoute';
import LoadingSpinner from '@/components/common/LoadingSpinner';

// Lazy load pages
const Home = lazy(() => import('@/pages/public/Home'));
const Pricing = lazy(() => import('@/pages/public/Pricing'));
const Login = lazy(() => import('@/pages/auth/Login'));
const Dashboard = lazy(() => import('@/pages/dashboard/Dashboard'));

export default function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          {/* Public routes */}
          <Route path="/" element={<Home />} />
          <Route path="/pricing" element={<Pricing />} />
          <Route path="/login" element={<Login />} />

          {/* Protected routes */}
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }
          />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

---

## Deployment Workflow

### Build and Deploy

```bash
# 1. Install dependencies
npm install

# 2. Build for production
npm run build

# 3. Deploy to Firebase (PowerShell: use npx)
npx firebase deploy

# Deploy only hosting
npx firebase deploy --only hosting

# Deploy only functions
npx firebase deploy --only functions

# Deploy specific function
npx firebase deploy --only functions:functionName
```

### Environment Variables

```bash
# .env.local (frontend)
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123456789:web:abcdef
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_xxxxx

# functions/.env (backend)
STRIPE_SECRET_KEY=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
GMAIL_USER=your-email@gmail.com
GMAIL_APP_PASSWORD=your_app_password
```

---

## Best Practices Summary

### Code Quality

- ✅ Use TypeScript strict mode
- ✅ No 'any' types (use 'unknown' if needed)
- ✅ Explicit return types for functions
- ✅ Handle errors consistently
- ✅ Use async/await over promises
- ✅ Comment complex logic only
- ✅ Keep functions small and focused

### Performance

- ✅ Lazy load routes
- ✅ Code splitting (manual chunks in Vite)
- ✅ Optimize images (WebP, lazy loading)
- ✅ Use production builds
- ✅ Enable Firebase persistence
- ✅ Cache static assets
- ✅ Minimize bundle size

### Security

- ✅ Use Firebase Security Rules
- ✅ Validate all inputs
- ✅ Never trust client data
- ✅ Use environment variables for secrets
- ✅ Implement proper authentication
- ✅ Sanitize user content
- ✅ Use HTTPS only

### UX

- ✅ Show loading states
- ✅ Handle errors gracefully
- ✅ Provide feedback (toasts, notifications)
- ✅ Mobile-first design
- ✅ Accessible components
- ✅ Fast page loads
- ✅ Consistent UI patterns

---

## Don'ts (Anti-Patterns)

### ❌ NEVER Do This:

```typescript
// Don't use 'any'
function process(data: any) {}  // ❌

// Don't ignore errors
try {
  await somethingRisky();
} catch (e) {} // ❌

// Don't use inline styles
<div style={{ color: 'red' }}>Text</div> // ❌

// Don't mutate state directly
user.name = 'New Name'; // ❌
setUser(user); // ❌

// Don't use var
var count = 0; // ❌

// Don't use require
const React = require('react'); // ❌

// Don't skip key prop in lists
items.map(item => <div>{item}</div>) // ❌

// Don't use string refs
<input ref="myInput" /> // ❌
```

### ✅ DO This Instead:

```typescript
// Use explicit types
function process(data: User) {}  // ✅

// Handle errors
try {
  await somethingRisky();
} catch (error) {
  console.error('Error:', error);
  throw error;
} // ✅

// Use Tailwind classes
<div className="text-red-500">Text</div> // ✅

// Use immutable updates
setUser({ ...user, name: 'New Name' }); // ✅

// Use const/let
const count = 0; // ✅

// Use ES6 imports
import React from 'react'; // ✅

// Always use key prop
items.map(item => <div key={item.id}>{item}</div>) // ✅

// Use useRef hook
const inputRef = useRef<HTMLInputElement>(null); // ✅
```

---

## Quick Start Checklist

When starting a new project, follow this checklist:

### Initial Setup
- [ ] Create Firebase project
- [ ] Set up Vite + React + TypeScript
- [ ] Install core dependencies (see package.json above)
- [ ] Configure Tailwind CSS
- [ ] Set up path aliases (@/*)
- [ ] Create directory structure
- [ ] Initialize Firebase (firebase init)
- [ ] Set up environment variables

### Configuration
- [ ] Configure tsconfig.json
- [ ] Configure vite.config.ts
- [ ] Configure tailwind.config.js
- [ ] Configure firebase.json
- [ ] Set up Firestore rules
- [ ] Set up Storage rules

### Core Features
- [ ] Implement AuthContext
- [ ] Create ProtectedRoute component
- [ ] Set up routing (React Router)
- [ ] Create layout components (Header, Footer)
- [ ] Set up toast notifications (Sonner)
- [ ] Implement error boundaries

### Deployment
- [ ] Test build locally
- [ ] Deploy to Firebase Hosting
- [ ] Configure custom domain (Cloudflare)
- [ ] Set up SSL certificate
- [ ] Test production deployment

---

## Resources

- **React Documentation**: https://react.dev/
- **TypeScript Handbook**: https://www.typescriptlang.org/docs/
- **Vite Guide**: https://vitejs.dev/guide/
- **Tailwind CSS Docs**: https://tailwindcss.com/docs
- **Firebase Docs**: https://firebase.google.com/docs
- **Shadcn UI**: https://ui.shadcn.com/
- **React Router**: https://reactrouter.com/

---

**This is the proven stack from entiendetunomina and valuebetanalytics. Always follow these standards for consistency and maintainability.**
