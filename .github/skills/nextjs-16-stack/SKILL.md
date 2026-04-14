---
name: nextjs-16-stack
description: Professional Next.js 16 development with TypeScript, TailwindCSS, shadcn/ui, MongoDB, and Nodemailer for building scalable, production-grade applications. Use this skill for Next.js 16 projects, full-stack applications, API design, database architecture, scalability patterns, algorithm implementations, performance optimization, caching strategies, microservices, queue systems, rate limiting, search optimization, or when the user mentions building scalable products, production architecture, Next.js, React Server Components, App Router, shadcn, MongoDB, system design, algorithms, data structures, performance, or deployment strategies. Also trigger for questions about scalable architecture, distributed systems, real-world algorithms, production patterns, or building products that can handle growth.
version: 1.0.0
author: claude-skills-collective
license: MIT
tags: [nextjs-16, typescript, production-architecture, full-stack, mongodb, nodejs, scalability, system-design]
compatible_tools: [claude-code, codex-cli, cursor, gemini-cli, opencode, antigravity]
---

# Next.js 16 Professional Stack

A comprehensive guide for building production-ready Next.js 16 applications with TypeScript, TailwindCSS, shadcn/ui, MongoDB, and Nodemailer.

## Core Principles

1. **Type Safety First**: Leverage TypeScript across the entire stack
2. **Server Components by Default**: Use Client Components only when needed
3. **Colocation**: Keep related files together (components, types, utils)
4. **Convention over Configuration**: Follow Next.js 16 conventions
5. **Progressive Enhancement**: Build apps that work without JavaScript when possible

## Project Structure

```
project-root/
├── src/
│   ├── app/                      # App Router (Next.js 16)
│   │   ├── (auth)/              # Route groups for layout organization
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx       # Shared dashboard layout
│   │   │   └── dashboard/
│   │   │       └── page.tsx
│   │   ├── api/                 # API routes
│   │   │   ├── auth/
│   │   │   │   └── route.ts
│   │   │   └── users/
│   │   │       └── route.ts
│   │   ├── layout.tsx           # Root layout
│   │   ├── page.tsx             # Home page
│   │   ├── globals.css          # Global styles + Tailwind directives
│   │   └── error.tsx            # Error boundary
│   ├── components/              # Reusable components
│   │   ├── ui/                  # shadcn/ui components
│   │   ├── forms/               # Form components
│   │   ├── layouts/             # Layout components
│   │   └── shared/              # Shared components
│   ├── lib/                     # Utilities and configurations
│   │   ├── db/                  # Database utilities
│   │   │   ├── mongodb.ts       # MongoDB connection
│   │   │   └── models/          # Mongoose models
│   │   ├── email/               # Email utilities
│   │   │   ├── nodemailer.ts    # Email client
│   │   │   └── templates/       # Email templates
│   │   ├── auth/                # Authentication utilities
│   │   ├── validations/         # Zod schemas
│   │   └── utils.ts             # General utilities
│   ├── types/                   # TypeScript type definitions
│   │   ├── index.ts
│   │   ├── api.ts
│   │   └── database.ts
│   ├── hooks/                   # Custom React hooks
│   ├── actions/                 # Server Actions
│   └── middleware.ts            # Next.js middleware
├── public/                      # Static assets
├── .env.local                   # Environment variables
├── .env.example                 # Environment template
├── next.config.ts               # Next.js configuration
├── tailwind.config.ts           # Tailwind configuration
├── tsconfig.json                # TypeScript configuration
├── components.json              # shadcn/ui configuration
└── package.json
```

## Initial Setup

### 1. Create Next.js 16 Project

```bash
npx create-next-app@latest project-name
cd project-name
```

### 2. Install Core Dependencies

```bash
# shadcn/ui
pnpm dlx shadcn@latest init

# Database
pnpm add mongoose
pnpm add -D @types/mongoose

# Email
pnpm add nodemailer
pnpm add -D @types/nodemailer

# Validation
pnpm add zod

# Additional utilities
pnpm add clsx tailwind-merge
pnpm add class-variance-authority
pnpm add lucide-react
```

### 3. Environment Variables

Create `.env.local`:

```env
# Database
MONGODB_URI=mongodb://localhost:27017/your-database
# or for MongoDB Atlas:
# MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/database

# Email (Nodemailer)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASSWORD=your-app-password
EMAIL_FROM="Your App <noreply@yourapp.com>"

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Authentication (if using NextAuth)
NEXTAUTH_SECRET=your-secret-key-here
NEXTAUTH_URL=http://localhost:3000
```

## MongoDB Integration

### Connection Singleton

**`src/lib/db/mongodb.ts`**:

```typescript
import mongoose from 'mongoose';

const MONGODB_URI = process.env.MONGODB_URI!;

if (!MONGODB_URI) {
  throw new Error('Please define the MONGODB_URI environment variable');
}

interface MongooseCache {
  conn: typeof mongoose | null;
  promise: Promise<typeof mongoose> | null;
}

// Use global to persist connection across hot reloads in development
declare global {
  var mongoose: MongooseCache | undefined;
}

let cached: MongooseCache = global.mongoose || { conn: null, promise: null };

if (!global.mongoose) {
  global.mongoose = cached;
}

export async function connectDB(): Promise<typeof mongoose> {
  if (cached.conn) {
    return cached.conn;
  }

  if (!cached.promise) {
    const opts = {
      bufferCommands: false,
    };

    cached.promise = mongoose.connect(MONGODB_URI, opts);
  }

  try {
    cached.conn = await cached.promise;
  } catch (e) {
    cached.promise = null;
    throw e;
  }

  return cached.conn;
}
```

### Mongoose Models

**`src/lib/db/models/User.ts`**:

```typescript
import mongoose, { Document, Model, Schema } from 'mongoose';

export interface IUser extends Document {
  email: string;
  name: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

const UserSchema = new Schema<IUser>(
  {
    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,
      lowercase: true,
      trim: true,
    },
    name: {
      type: String,
      required: [true, 'Name is required'],
      trim: true,
    },
    password: {
      type: String,
      required: [true, 'Password is required'],
      minlength: 6,
    },
  },
  {
    timestamps: true,
  }
);

// Prevent model overwrite during hot reload
const User: Model<IUser> = 
  mongoose.models.User || mongoose.model<IUser>('User', UserSchema);

export default User;
```

### Using Models in API Routes

**`src/app/api/users/route.ts`**:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { connectDB } from '@/lib/db/mongodb';
import User from '@/lib/db/models/User';
import { z } from 'zod';

// Validation schema
const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
  password: z.string().min(6),
});

// POST /api/users
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    // Validate input
    const validatedData = createUserSchema.parse(body);
    
    // Connect to database
    await connectDB();
    
    // Check if user exists
    const existingUser = await User.findOne({ email: validatedData.email });
    if (existingUser) {
      return NextResponse.json(
        { error: 'User already exists' },
        { status: 400 }
      );
    }
    
    // Create user (in production, hash password first!)
    const user = await User.create(validatedData);
    
    // Remove password from response
    const { password, ...userWithoutPassword } = user.toObject();
    
    return NextResponse.json(
      { user: userWithoutPassword },
      { status: 201 }
    );
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    
    console.error('Error creating user:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// GET /api/users
export async function GET() {
  try {
    await connectDB();
    
    const users = await User.find({})
      .select('-password') // Exclude password field
      .sort({ createdAt: -1 });
    
    return NextResponse.json({ users });
  } catch (error) {
    console.error('Error fetching users:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

## Nodemailer Integration

### Email Client Setup

**`src/lib/email/nodemailer.ts`**:

```typescript
import nodemailer from 'nodemailer';
import type { Transporter } from 'nodemailer';

let transporter: Transporter | null = null;

export function getEmailTransporter(): Transporter {
  if (transporter) {
    return transporter;
  }

  transporter = nodemailer.createTransport({
    host: process.env.EMAIL_HOST,
    port: parseInt(process.env.EMAIL_PORT || '587'),
    secure: process.env.EMAIL_PORT === '465', // true for 465, false for other ports
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASSWORD,
    },
  });

  return transporter;
}

export interface SendEmailOptions {
  to: string | string[];
  subject: string;
  text?: string;
  html?: string;
  from?: string;
}

export async function sendEmail({
  to,
  subject,
  text,
  html,
  from = process.env.EMAIL_FROM,
}: SendEmailOptions) {
  const transporter = getEmailTransporter();

  try {
    const info = await transporter.sendMail({
      from,
      to: Array.isArray(to) ? to.join(', ') : to,
      subject,
      text,
      html,
    });

    console.log('Email sent:', info.messageId);
    return { success: true, messageId: info.messageId };
  } catch (error) {
    console.error('Error sending email:', error);
    throw error;
  }
}
```

### Email Templates

**`src/lib/email/templates/welcome.ts`**:

```typescript
export interface WelcomeEmailData {
  name: string;
  appUrl: string;
}

export function getWelcomeEmailHtml({ name, appUrl }: WelcomeEmailData): string {
  return `
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <style>
          body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
          }
          .button {
            display: inline-block;
            padding: 12px 24px;
            background-color: #0070f3;
            color: white;
            text-decoration: none;
            border-radius: 6px;
            margin: 20px 0;
          }
        </style>
      </head>
      <body>
        <h1>Welcome, ${name}!</h1>
        <p>Thank you for joining our platform. We're excited to have you on board.</p>
        <a href="${appUrl}/dashboard" class="button">Get Started</a>
        <p>If you have any questions, feel free to reply to this email.</p>
        <p>Best regards,<br>The Team</p>
      </body>
    </html>
  `;
}

export function getWelcomeEmailText({ name, appUrl }: WelcomeEmailData): string {
  return `
Welcome, ${name}!

Thank you for joining our platform. We're excited to have you on board.

Get started: ${appUrl}/dashboard

If you have any questions, feel free to reply to this email.

Best regards,
The Team
  `.trim();
}
```

### Using Email in API Routes

**`src/app/api/auth/register/route.ts`**:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { sendEmail } from '@/lib/email/nodemailer';
import { getWelcomeEmailHtml, getWelcomeEmailText } from '@/lib/email/templates/welcome';

export async function POST(request: NextRequest) {
  try {
    // ... user registration logic ...
    
    // Send welcome email
    await sendEmail({
      to: user.email,
      subject: 'Welcome to Our Platform',
      text: getWelcomeEmailText({
        name: user.name,
        appUrl: process.env.NEXT_PUBLIC_APP_URL!,
      }),
      html: getWelcomeEmailHtml({
        name: user.name,
        appUrl: process.env.NEXT_PUBLIC_APP_URL!,
      }),
    });
    
    return NextResponse.json({ success: true });
  } catch (error) {
    // Error handling...
  }
}
```

## shadcn/ui Integration

### Adding Components

```bash
# Add commonly used components
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add form
npx shadcn@latest add input
npx shadcn@latest add label
npx shadcn@latest add toast
npx shadcn@latest add dialog
npx shadcn@latest add dropdown-menu
npx shadcn@latest add table
```

### Custom Component Example

**`src/components/forms/LoginForm.tsx`**:

```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { toast } from '@/components/ui/use-toast';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
});

export function LoginForm() {
  const router = useRouter();
  const [isLoading, setIsLoading] = useState(false);
  const [formData, setFormData] = useState({
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setErrors({});
    
    // Validate
    const result = loginSchema.safeParse(formData);
    if (!result.success) {
      const fieldErrors: Record<string, string> = {};
      result.error.errors.forEach((err) => {
        if (err.path[0]) {
          fieldErrors[err.path[0].toString()] = err.message;
        }
      });
      setErrors(fieldErrors);
      return;
    }

    setIsLoading(true);
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Login failed');
      }

      toast({
        title: 'Success',
        description: 'You have been logged in successfully',
      });
      
      router.push('/dashboard');
    } catch (error) {
      toast({
        title: 'Error',
        description: error instanceof Error ? error.message : 'An error occurred',
        variant: 'destructive',
      });
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Login</CardTitle>
        <CardDescription>Enter your credentials to access your account</CardDescription>
      </CardHeader>
      <CardContent>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>
            <Input
              id="email"
              type="email"
              placeholder="you@example.com"
              value={formData.email}
              onChange={(e) => setFormData({ ...formData, email: e.target.value })}
              disabled={isLoading}
            />
            {errors.email && (
              <p className="text-sm text-destructive">{errors.email}</p>
            )}
          </div>
          
          <div className="space-y-2">
            <Label htmlFor="password">Password</Label>
            <Input
              id="password"
              type="password"
              value={formData.password}
              onChange={(e) => setFormData({ ...formData, password: e.target.value })}
              disabled={isLoading}
            />
            {errors.password && (
              <p className="text-sm text-destructive">{errors.password}</p>
            )}
          </div>

          <Button type="submit" className="w-full" disabled={isLoading}>
            {isLoading ? 'Logging in...' : 'Login'}
          </Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

## Server Actions (Next.js 16)

Server Actions provide a powerful way to handle form submissions and mutations.

**`src/actions/users.ts`**:

```typescript
'use server';

import { revalidatePath } from 'next/cache';
import { connectDB } from '@/lib/db/mongodb';
import User from '@/lib/db/models/User';
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
  password: z.string().min(6),
});

export async function createUser(formData: FormData) {
  try {
    // Extract and validate data
    const rawData = {
      email: formData.get('email'),
      name: formData.get('name'),
      password: formData.get('password'),
    };
    
    const validatedData = createUserSchema.parse(rawData);
    
    // Connect to database
    await connectDB();
    
    // Check if user exists
    const existingUser = await User.findOne({ email: validatedData.email });
    if (existingUser) {
      return { success: false, error: 'User already exists' };
    }
    
    // Create user
    const user = await User.create(validatedData);
    
    // Revalidate relevant paths
    revalidatePath('/admin/users');
    
    return { success: true, user: { id: user._id.toString(), email: user.email } };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: 'Validation failed', details: error.errors };
    }
    
    console.error('Error creating user:', error);
    return { success: false, error: 'Failed to create user' };
  }
}
```

**Using Server Action in Component**:

```typescript
'use client';

import { useTransition } from 'react';
import { createUser } from '@/actions/users';
import { toast } from '@/components/ui/use-toast';

export function UserForm() {
  const [isPending, startTransition] = useTransition();

  const handleSubmit = async (formData: FormData) => {
    startTransition(async () => {
      const result = await createUser(formData);
      
      if (result.success) {
        toast({ title: 'Success', description: 'User created successfully' });
      } else {
        toast({ title: 'Error', description: result.error, variant: 'destructive' });
      }
    });
  };

  return (
    <form action={handleSubmit}>
      {/* Form fields */}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

## TypeScript Best Practices

### Type Definitions

**`src/types/api.ts`**:

```typescript
// API Response types
export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// API Error types
export class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
    this.name = 'ApiError';
  }
}
```

**`src/types/database.ts`**:

```typescript
import { Document } from 'mongoose';

export interface BaseDocument extends Document {
  createdAt: Date;
  updatedAt: Date;
}

export interface UserDocument extends BaseDocument {
  email: string;
  name: string;
  password: string;
}

// Frontend-safe types (without sensitive data)
export type SafeUser = Omit<UserDocument, 'password'>;
```

## Middleware

**`src/middleware.ts`**:

```typescript
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');
  
  // Protect routes
  const isAuthPage = request.nextUrl.pathname.startsWith('/auth');
  const isDashboard = request.nextUrl.pathname.startsWith('/dashboard');
  
  // Get token from cookies (example)
  const token = request.cookies.get('token')?.value;
  
  if (isDashboard && !token) {
    return NextResponse.redirect(new URL('/auth/login', request.url));
  }
  
  if (isAuthPage && token) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }
  
  return response;
}

export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * - public folder
     */
    '/((?!_next/static|_next/image|favicon.ico|public).*)',
  ],
};
```

## Configuration Files

### next.config.ts

```typescript
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  images: {
    domains: ['your-image-domain.com'],
    formats: ['image/avif', 'image/webp'],
  },
  env: {
    CUSTOM_ENV_VAR: process.env.CUSTOM_ENV_VAR,
  },
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          { key: 'Access-Control-Allow-Origin', value: '*' },
          { key: 'Access-Control-Allow-Methods', value: 'GET,POST,PUT,DELETE,OPTIONS' },
          { key: 'Access-Control-Allow-Headers', value: 'Content-Type, Authorization' },
        ],
      },
    ];
  },
};

export default nextConfig;
```

### tailwind.config.ts

```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class'],
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        // ... other shadcn/ui colors
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};

export default config;
```

## Common Patterns

### API Route Error Handling

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { ZodError } from 'zod';
import { ApiError } from '@/types/api';

export async function handleApiError(error: unknown) {
  console.error('API Error:', error);

  if (error instanceof ZodError) {
    return NextResponse.json(
      {
        success: false,
        error: 'Validation failed',
        details: error.errors,
      },
      { status: 400 }
    );
  }

  if (error instanceof ApiError) {
    return NextResponse.json(
      {
        success: false,
        error: error.message,
        code: error.code,
      },
      { status: error.statusCode }
    );
  }

  return NextResponse.json(
    {
      success: false,
      error: 'Internal server error',
    },
    { status: 500 }
  );
}
```

### Data Fetching in Server Components

```typescript
// src/app/users/page.tsx
import { connectDB } from '@/lib/db/mongodb';
import User from '@/lib/db/models/User';

// This is a Server Component
export default async function UsersPage() {
  await connectDB();
  
  const users = await User.find({})
    .select('-password')
    .lean() // Convert to plain objects
    .exec();

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user._id.toString()}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Scalable Product Architecture

### System Design Principles

Building a scalable product requires thinking beyond just code - you need to design systems that can handle growth.

#### 1. **Horizontal Scalability**

Design your application to scale out (add more servers) rather than up (bigger servers).

```typescript
// Good: Stateless API design
// Each request is independent - can be handled by any server

export async function GET(request: NextRequest) {
  // Get user from token, not from server memory
  const token = request.cookies.get('token')?.value;
  const user = await validateToken(token);
  
  // Fetch data from database, not from server memory
  const data = await fetchFromDatabase(user.id);
  
  return NextResponse.json({ data });
}

// Bad: Stateful design
let userSessions = {}; // This won't work across multiple servers!
```

#### 2. **Database Architecture Patterns**

**Indexing Strategy**:

```typescript
// User model with optimized indexes
const UserSchema = new Schema({
  email: { 
    type: String, 
    unique: true, 
    index: true  // Single field index
  },
  username: { 
    type: String, 
    index: true 
  },
  createdAt: { 
    type: Date, 
    index: true 
  },
  status: String,
  lastLoginAt: Date,
});

// Compound indexes for common queries
UserSchema.index({ status: 1, createdAt: -1 }); // For filtering active users by date
UserSchema.index({ email: 1, status: 1 }); // For login queries

// Text search index
UserSchema.index({ 
  name: 'text', 
  bio: 'text' 
}); // Full-text search
```

**Read Replicas Pattern**:

```typescript
// src/lib/db/mongodb-read.ts
import mongoose from 'mongoose';

// Separate connection for read-heavy operations
let readConnection: typeof mongoose | null = null;

export async function getReadConnection() {
  if (readConnection) return readConnection;
  
  readConnection = await mongoose.createConnection(
    process.env.MONGODB_READ_REPLICA_URI!,
    { readPreference: 'secondaryPreferred' }
  ).asPromise();
  
  return readConnection;
}

// Usage: Route read-heavy queries to replicas
export async function GET() {
  const readDb = await getReadConnection();
  const User = readDb.model('User');
  
  const users = await User.find({})
    .select('name email')
    .lean(); // Use lean() for read-only operations
    
  return NextResponse.json({ users });
}
```

#### 3. **Caching Strategy**

**Multi-Layer Caching**:

```typescript
// src/lib/cache/redis.ts
import { Redis } from '@upstash/redis'; // Or ioredis

const redis = new Redis({
  url: process.env.REDIS_URL!,
  token: process.env.REDIS_TOKEN!,
});

export async function getCached<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 3600
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return cached as T;
  }
  
  // Cache miss - fetch and store
  const data = await fetcher();
  await redis.setex(key, ttl, JSON.stringify(data));
  
  return data;
}

// Usage in API route
export async function GET(request: NextRequest) {
  const userId = request.nextUrl.searchParams.get('userId');
  
  const userData = await getCached(
    `user:${userId}`,
    async () => {
      await connectDB();
      return await User.findById(userId).lean();
    },
    1800 // 30 minutes
  );
  
  return NextResponse.json({ user: userData });
}
```

**Next.js Built-in Caching**:

```typescript
// Revalidate cache every hour
export const revalidate = 3600;

// Or use fetch with cache options
const data = await fetch('https://api.example.com/data', {
  next: { 
    revalidate: 3600,
    tags: ['products'] 
  }
});

// Revalidate on-demand
import { revalidateTag } from 'next/cache';
revalidateTag('products');
```

#### 4. **Rate Limiting**

```typescript
// src/lib/rate-limit.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL!,
  token: process.env.REDIS_TOKEN!,
});

export async function rateLimit(
  identifier: string,
  limit: number = 10,
  window: number = 60
): Promise<{ success: boolean; remaining: number }> {
  const key = `ratelimit:${identifier}`;
  
  const count = await redis.incr(key);
  
  if (count === 1) {
    await redis.expire(key, window);
  }
  
  const remaining = Math.max(0, limit - count);
  
  return {
    success: count <= limit,
    remaining,
  };
}

// Usage in API route
export async function POST(request: NextRequest) {
  const ip = request.headers.get('x-forwarded-for') || 'unknown';
  
  const { success, remaining } = await rateLimit(ip, 10, 60);
  
  if (!success) {
    return NextResponse.json(
      { error: 'Too many requests' },
      { 
        status: 429,
        headers: {
          'X-RateLimit-Remaining': remaining.toString(),
          'Retry-After': '60',
        }
      }
    );
  }
  
  // Process request...
}
```

#### 5. **Background Jobs & Queue System**

```typescript
// src/lib/queue/bullmq.ts
import { Queue, Worker } from 'bullmq';
import IORedis from 'ioredis';

const connection = new IORedis(process.env.REDIS_URL!);

// Create queue
export const emailQueue = new Queue('emails', { connection });

// Add job
export async function sendWelcomeEmail(userId: string, email: string) {
  await emailQueue.add('welcome', {
    userId,
    email,
    timestamp: Date.now(),
  }, {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000,
    },
  });
}

// Worker process (separate file or process)
const worker = new Worker('emails', async (job) => {
  const { email, userId } = job.data;
  
  // Send email
  await sendEmail({
    to: email,
    subject: 'Welcome!',
    html: getWelcomeEmailHtml({ userId }),
  });
  
  return { sent: true, timestamp: Date.now() };
}, { connection });

worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err);
});
```

**Simpler Alternative with Vercel (no queue needed)**:

```typescript
// Use Edge Functions for background tasks
export async function POST(request: NextRequest) {
  const data = await request.json();
  
  // Return response immediately
  const response = NextResponse.json({ success: true });
  
  // Execute background task after response
  request.waitUntil(
    (async () => {
      await sendEmail({ /* ... */ });
      await updateDatabase({ /* ... */ });
    })()
  );
  
  return response;
}
```

## Algorithm Implementations

### 1. **Search & Filtering Algorithms**

**Fuzzy Search Implementation**:

```typescript
// src/lib/algorithms/fuzzy-search.ts
export function fuzzyMatch(pattern: string, str: string): number {
  pattern = pattern.toLowerCase();
  str = str.toLowerCase();
  
  let patternIdx = 0;
  let score = 0;
  let consecutiveBonus = 0;
  
  for (let i = 0; i < str.length; i++) {
    if (pattern[patternIdx] === str[i]) {
      score += 1 + consecutiveBonus;
      consecutiveBonus += 5;
      patternIdx++;
      
      if (patternIdx === pattern.length) {
        return score;
      }
    } else {
      consecutiveBonus = 0;
    }
  }
  
  return patternIdx === pattern.length ? score : 0;
}

export function fuzzySearch<T>(
  items: T[],
  query: string,
  getSearchable: (item: T) => string[]
): T[] {
  const results = items
    .map(item => {
      const searchableFields = getSearchable(item);
      const maxScore = Math.max(
        ...searchableFields.map(field => fuzzyMatch(query, field))
      );
      return { item, score: maxScore };
    })
    .filter(({ score }) => score > 0)
    .sort((a, b) => b.score - a.score)
    .map(({ item }) => item);
    
  return results;
}

// Usage in API
export async function GET(request: NextRequest) {
  const query = request.nextUrl.searchParams.get('q') || '';
  
  await connectDB();
  const allUsers = await User.find({}).lean();
  
  const results = fuzzySearch(allUsers, query, (user) => [
    user.name,
    user.email,
    user.username,
  ]);
  
  return NextResponse.json({ results: results.slice(0, 20) });
}
```

**Elasticsearch Integration for Advanced Search**:

```typescript
// src/lib/search/elasticsearch.ts
import { Client } from '@elastic/elasticsearch';

const client = new Client({
  node: process.env.ELASTICSEARCH_URL!,
  auth: {
    apiKey: process.env.ELASTICSEARCH_API_KEY!,
  },
});

export async function searchProducts(query: string, filters: any = {}) {
  const { body } = await client.search({
    index: 'products',
    body: {
      query: {
        bool: {
          must: [
            {
              multi_match: {
                query,
                fields: ['name^3', 'description', 'tags'],
                fuzziness: 'AUTO',
              },
            },
          ],
          filter: Object.entries(filters).map(([key, value]) => ({
            term: { [key]: value },
          })),
        },
      },
      highlight: {
        fields: {
          name: {},
          description: {},
        },
      },
      size: 20,
    },
  });
  
  return body.hits.hits.map((hit: any) => ({
    ...hit._source,
    highlights: hit.highlight,
    score: hit._score,
  }));
}
```

### 2. **Pagination Algorithms**

**Cursor-Based Pagination (for infinite scroll)**:

```typescript
// More efficient than offset-based for large datasets
export async function GET(request: NextRequest) {
  const cursor = request.nextUrl.searchParams.get('cursor');
  const limit = 20;
  
  await connectDB();
  
  const query: any = {};
  if (cursor) {
    query._id = { $lt: cursor }; // Get items before cursor
  }
  
  const items = await Item.find(query)
    .sort({ _id: -1 }) // Descending order
    .limit(limit + 1) // Fetch one extra to check if there's more
    .lean();
  
  const hasMore = items.length > limit;
  const results = hasMore ? items.slice(0, -1) : items;
  const nextCursor = hasMore ? results[results.length - 1]._id : null;
  
  return NextResponse.json({
    data: results,
    nextCursor,
    hasMore,
  });
}
```

### 3. **Sorting & Ranking Algorithms**

**Multi-Criteria Sorting**:

```typescript
// src/lib/algorithms/sorting.ts
export interface SortCriteria<T> {
  key: keyof T;
  direction: 'asc' | 'desc';
  weight?: number;
}

export function multiSort<T>(
  items: T[],
  criteria: SortCriteria<T>[]
): T[] {
  return items.sort((a, b) => {
    for (const criterion of criteria) {
      const aVal = a[criterion.key];
      const bVal = b[criterion.key];
      
      const multiplier = criterion.direction === 'asc' ? 1 : -1;
      const weight = criterion.weight || 1;
      
      if (aVal < bVal) return -1 * multiplier * weight;
      if (aVal > bVal) return 1 * multiplier * weight;
    }
    return 0;
  });
}

// Usage: Sort products by relevance, then price, then rating
const sortedProducts = multiSort(products, [
  { key: 'relevanceScore', direction: 'desc', weight: 2 },
  { key: 'price', direction: 'asc', weight: 1 },
  { key: 'rating', direction: 'desc', weight: 1.5 },
]);
```

### 4. **Recommendation Algorithm**

**Collaborative Filtering (Simple)**:

```typescript
// src/lib/algorithms/recommendations.ts
export interface UserItem {
  userId: string;
  itemId: string;
  rating: number;
}

export function calculateSimilarity(
  user1Ratings: Map<string, number>,
  user2Ratings: Map<string, number>
): number {
  const commonItems = Array.from(user1Ratings.keys()).filter(item =>
    user2Ratings.has(item)
  );
  
  if (commonItems.length === 0) return 0;
  
  // Cosine similarity
  let dotProduct = 0;
  let magnitude1 = 0;
  let magnitude2 = 0;
  
  for (const item of commonItems) {
    const rating1 = user1Ratings.get(item)!;
    const rating2 = user2Ratings.get(item)!;
    
    dotProduct += rating1 * rating2;
    magnitude1 += rating1 * rating1;
    magnitude2 += rating2 * rating2;
  }
  
  return dotProduct / (Math.sqrt(magnitude1) * Math.sqrt(magnitude2));
}

export async function getRecommendations(
  userId: string,
  limit: number = 10
): Promise<string[]> {
  // Get user's ratings
  const userRatings = await Rating.find({ userId }).lean();
  const userRatingsMap = new Map(
    userRatings.map(r => [r.itemId, r.rating])
  );
  
  // Get all other users' ratings
  const allRatings = await Rating.find({
    userId: { $ne: userId },
  }).lean();
  
  // Group by user
  const otherUsers = new Map<string, Map<string, number>>();
  for (const rating of allRatings) {
    if (!otherUsers.has(rating.userId)) {
      otherUsers.set(rating.userId, new Map());
    }
    otherUsers.get(rating.userId)!.set(rating.itemId, rating.rating);
  }
  
  // Find similar users
  const similarities = Array.from(otherUsers.entries())
    .map(([otherUserId, ratings]) => ({
      userId: otherUserId,
      similarity: calculateSimilarity(userRatingsMap, ratings),
      ratings,
    }))
    .filter(({ similarity }) => similarity > 0)
    .sort((a, b) => b.similarity - a.similarity)
    .slice(0, 10); // Top 10 similar users
  
  // Get items they liked that current user hasn't rated
  const recommendations = new Map<string, number>();
  
  for (const { similarity, ratings } of similarities) {
    for (const [itemId, rating] of ratings.entries()) {
      if (!userRatingsMap.has(itemId)) {
        const currentScore = recommendations.get(itemId) || 0;
        recommendations.set(itemId, currentScore + similarity * rating);
      }
    }
  }
  
  return Array.from(recommendations.entries())
    .sort((a, b) => b[1] - a[1])
    .slice(0, limit)
    .map(([itemId]) => itemId);
}
```

### 5. **Data Structure Implementations**

**LRU Cache**:

```typescript
// src/lib/algorithms/lru-cache.ts
export class LRUCache<K, V> {
  private capacity: number;
  private cache: Map<K, V>;
  
  constructor(capacity: number) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key: K): V | undefined {
    if (!this.cache.has(key)) return undefined;
    
    // Move to end (most recently used)
    const value = this.cache.get(key)!;
    this.cache.delete(key);
    this.cache.set(key, value);
    
    return value;
  }
  
  set(key: K, value: V): void {
    // Remove if exists
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    
    // Add to end
    this.cache.set(key, value);
    
    // Evict least recently used if over capacity
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
  
  has(key: K): boolean {
    return this.cache.has(key);
  }
}

// Usage: API response caching
const responseCache = new LRUCache<string, any>(100);

export async function GET(request: NextRequest) {
  const cacheKey = request.url;
  
  if (responseCache.has(cacheKey)) {
    return NextResponse.json(responseCache.get(cacheKey));
  }
  
  const data = await fetchExpensiveData();
  responseCache.set(cacheKey, data);
  
  return NextResponse.json(data);
}
```

### 6. **Debounce & Throttle**

```typescript
// src/lib/algorithms/debounce.ts
export function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout | null = null;
  
  return function executedFunction(...args: Parameters<T>) {
    const later = () => {
      timeout = null;
      func(...args);
    };
    
    if (timeout) clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

export function throttle<T extends (...args: any[]) => any>(
  func: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;
  
  return function executedFunction(...args: Parameters<T>) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage in component
'use client';

import { debounce } from '@/lib/algorithms/debounce';

const handleSearch = debounce((query: string) => {
  fetch(`/api/search?q=${query}`);
}, 300);
```

## Performance Optimization

### 1. **Database Query Optimization**

```typescript
// ❌ Bad: N+1 Query Problem
const users = await User.find({});
for (const user of users) {
  const posts = await Post.find({ userId: user._id }); // N queries!
  user.posts = posts;
}

// ✅ Good: Use aggregation
const users = await User.aggregate([
  {
    $lookup: {
      from: 'posts',
      localField: '_id',
      foreignField: 'userId',
      as: 'posts',
    },
  },
  {
    $project: {
      name: 1,
      email: 1,
      posts: { $slice: ['$posts', 5] }, // Limit posts
    },
  },
]);

// ✅ Even better: Use population with select
const users = await User.find({})
  .populate({
    path: 'posts',
    select: 'title createdAt',
    options: { limit: 5, sort: { createdAt: -1 } },
  })
  .lean();
```

### 2. **Image Optimization**

```typescript
// Use Next.js Image component
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // Load immediately for above-fold images
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>

// For dynamic images
<Image
  src={user.avatar}
  alt={user.name}
  width={100}
  height={100}
  loading="lazy" // Lazy load for below-fold images
/>
```

### 3. **Code Splitting**

```typescript
// Dynamic imports for heavy components
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false, // Disable SSR for this component
});

// Load only when needed
const AdminPanel = dynamic(() => import('@/components/AdminPanel'));
```

## Microservices Architecture Pattern

For truly scalable products, consider splitting into microservices:

```typescript
// API Gateway pattern (main Next.js app)
export async function GET(request: NextRequest) {
  const service = request.nextUrl.searchParams.get('service');
  
  const serviceUrls = {
    auth: process.env.AUTH_SERVICE_URL,
    payment: process.env.PAYMENT_SERVICE_URL,
    notification: process.env.NOTIFICATION_SERVICE_URL,
  };
  
  const targetUrl = serviceUrls[service as keyof typeof serviceUrls];
  
  if (!targetUrl) {
    return NextResponse.json({ error: 'Invalid service' }, { status: 400 });
  }
  
  // Forward request to microservice
  const response = await fetch(targetUrl, {
    method: request.method,
    headers: request.headers,
    body: request.body,
  });
  
  return response;
}
```

## Monitoring & Observability

```typescript
// src/lib/monitoring/logger.ts
export function logError(error: Error, context: Record<string, any> = {}) {
  console.error({
    message: error.message,
    stack: error.stack,
    timestamp: new Date().toISOString(),
    ...context,
  });
  
  // Send to monitoring service (e.g., Sentry, DataDog)
  if (process.env.SENTRY_DSN) {
    // Sentry.captureException(error, { extra: context });
  }
}

// Performance monitoring
export function measurePerformance<T>(
  name: string,
  fn: () => Promise<T>
): Promise<T> {
  const start = performance.now();
  
  return fn().then(result => {
    const duration = performance.now() - start;
    console.log(`${name} took ${duration.toFixed(2)}ms`);
    
    // Send to monitoring
    // track(name, duration);
    
    return result;
  });
}
```

## Deployment Checklist

### Pre-Deployment

1. **Environment Variables**: Set all required env vars in your hosting platform
2. **Database**: 
   - Ensure MongoDB connection string is correct and database is accessible
   - Set up indexes for frequently queried fields
   - Configure read replicas if needed
   - Set up database backups
3. **Email**: Verify SMTP credentials and test email sending
4. **Build**: Run `npm run build` to check for errors
5. **Type Checking**: Run `tsc --noEmit` to verify no type errors
6. **Linting**: Run `npm run lint` to catch any issues

### Security

7. **Secrets Management**: 
   - Never commit `.env.local`
   - Use environment variables for all secrets
   - Rotate API keys regularly
8. **Rate Limiting**: 
   - Implement rate limiting for API routes
   - Add CAPTCHA for public forms
9. **Input Validation**: 
   - Validate all user input with Zod
   - Sanitize HTML inputs
10. **Authentication**:
    - Hash passwords with bcrypt (10+ rounds)
    - Implement secure session management
    - Add CSRF protection
    - Use secure, httpOnly cookies
11. **CORS**: Configure proper CORS headers
12. **SQL Injection Protection**: Use parameterized queries (Mongoose handles this)

### Performance & Scalability

13. **Caching**:
    - Set up Redis/Upstash for caching
    - Configure Next.js caching strategies
    - Add CDN for static assets
14. **Database Optimization**:
    - Review and add necessary indexes
    - Implement connection pooling
    - Consider read replicas for read-heavy apps
15. **Image Optimization**:
    - Use next/image for all images
    - Configure image domains in next.config.ts
    - Set up image CDN
16. **Code Splitting**:
    - Use dynamic imports for heavy components
    - Lazy load below-fold content
17. **API Optimization**:
    - Implement pagination for list endpoints
    - Add response compression
    - Use cursor-based pagination for infinite scroll

### Monitoring & Observability

18. **Error Tracking**: Set up Sentry or similar
19. **Performance Monitoring**: Configure APM (Application Performance Monitoring)
20. **Logging**: 
    - Set up structured logging
    - Configure log aggregation (e.g., DataDog, LogRocket)
21. **Uptime Monitoring**: Set up health checks and alerts
22. **Analytics**: Configure Google Analytics or Posthog

### Infrastructure

23. **Hosting**: 
    - Vercel (recommended for Next.js)
    - AWS / GCP / Azure
    - Configure auto-scaling if not using Vercel
24. **Database Hosting**:
    - MongoDB Atlas (recommended)
    - Set up connection limits
    - Configure automatic backups
25. **CDN**: CloudFront / Cloudflare for static assets
26. **Load Balancer**: If self-hosting, configure load balancing
27. **DNS**: Set up proper DNS records
28. **SSL/TLS**: Ensure HTTPS is enforced

### Testing

29. **Load Testing**: Test with expected traffic (use k6, Artillery)
30. **Security Scanning**: Run security audit (npm audit, Snyk)
31. **Accessibility**: Test with screen readers and lighthouse
32. **Browser Testing**: Test on major browsers
33. **Mobile Testing**: Test responsive design on real devices

### Documentation

34. **API Documentation**: Document all API endpoints
35. **Runbook**: Create incident response procedures
36. **Architecture Diagram**: Document system architecture
37. **Deployment Playbook**: Document deployment steps

## Real-World Production Patterns

### 1. **Feature Flags**

```typescript
// src/lib/feature-flags.ts
export interface FeatureFlags {
  newDashboard: boolean;
  aiAssistant: boolean;
  betaFeatures: boolean;
}

export async function getFeatureFlags(userId: string): Promise<FeatureFlags> {
  // Fetch from database or feature flag service (LaunchDarkly, Flagsmith)
  const userFlags = await FeatureFlagModel.findOne({ userId });
  
  return {
    newDashboard: userFlags?.newDashboard ?? false,
    aiAssistant: userFlags?.aiAssistant ?? false,
    betaFeatures: userFlags?.betaFeatures ?? false,
  };
}

// Usage in component
'use client';

export function Dashboard({ flags }: { flags: FeatureFlags }) {
  return (
    <div>
      {flags.newDashboard ? <NewDashboard /> : <OldDashboard />}
      {flags.aiAssistant && <AIAssistant />}
    </div>
  );
}
```

### 2. **A/B Testing**

```typescript
// src/lib/ab-testing.ts
export function assignVariant(userId: string, experimentId: string): 'A' | 'B' {
  // Consistent hashing - same user always gets same variant
  const hash = userId.split('').reduce((acc, char) => {
    return acc + char.charCodeAt(0);
  }, 0);
  
  return hash % 2 === 0 ? 'A' : 'B';
}

export async function trackEvent(
  userId: string,
  experimentId: string,
  event: string,
  value?: number
) {
  // Track to analytics service
  await AnalyticsEvent.create({
    userId,
    experimentId,
    event,
    value,
    variant: assignVariant(userId, experimentId),
    timestamp: new Date(),
  });
}

// Usage
const variant = assignVariant(user.id, 'checkout-button-color');

<Button 
  variant={variant === 'A' ? 'default' : 'outline'}
  onClick={() => {
    trackEvent(user.id, 'checkout-button-color', 'clicked');
    handleCheckout();
  }}
>
  Checkout
</Button>
```

### 3. **Circuit Breaker Pattern**

```typescript
// src/lib/patterns/circuit-breaker.ts
export class CircuitBreaker {
  private failures: number = 0;
  private lastFailureTime: number = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  
  constructor(
    private threshold: number = 5,
    private timeout: number = 60000
  ) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }
  
  private onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();
    
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// Usage: Protect external API calls
const paymentCircuit = new CircuitBreaker(5, 60000);

export async function processPayment(data: PaymentData) {
  return await paymentCircuit.execute(async () => {
    const response = await fetch('https://payment-api.com/charge', {
      method: 'POST',
      body: JSON.stringify(data),
    });
    
    if (!response.ok) throw new Error('Payment failed');
    return response.json();
  });
}
```

### 4. **Retry Logic with Exponential Backoff**

```typescript
// src/lib/patterns/retry.ts
export async function retry<T>(
  fn: () => Promise<T>,
  maxAttempts: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt < maxAttempts) {
        const waitTime = delay * Math.pow(2, attempt - 1); // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
  }
  
  throw lastError!;
}

// Usage
const data = await retry(
  () => fetch('https://api.example.com/data').then(r => r.json()),
  3,
  1000
);
```

### 5. **Idempotency Keys**

```typescript
// Prevent duplicate operations (e.g., double payments)
export async function POST(request: NextRequest) {
  const idempotencyKey = request.headers.get('Idempotency-Key');
  
  if (!idempotencyKey) {
    return NextResponse.json(
      { error: 'Idempotency-Key header required' },
      { status: 400 }
    );
  }
  
  // Check if operation already completed
  const existing = await Operation.findOne({ idempotencyKey });
  if (existing) {
    return NextResponse.json(existing.result, { status: 200 });
  }
  
  // Process operation
  const result = await processPayment(await request.json());
  
  // Store result with idempotency key
  await Operation.create({
    idempotencyKey,
    result,
    createdAt: new Date(),
  });
  
  return NextResponse.json(result);
}
```

### 6. **Webhook Handling**

```typescript
// src/app/api/webhooks/stripe/route.ts
import { headers } from 'next/headers';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = headers().get('stripe-signature')!;
  
  let event: Stripe.Event;
  
  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (error) {
    console.error('Webhook signature verification failed', error);
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }
  
  // Handle event
  switch (event.type) {
    case 'payment_intent.succeeded':
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      await handlePaymentSuccess(paymentIntent);
      break;
      
    case 'customer.subscription.deleted':
      const subscription = event.data.object as Stripe.Subscription;
      await handleSubscriptionCanceled(subscription);
      break;
      
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
  
  return NextResponse.json({ received: true });
}

async function handlePaymentSuccess(paymentIntent: Stripe.PaymentIntent) {
  await Order.updateOne(
    { paymentIntentId: paymentIntent.id },
    { status: 'paid', paidAt: new Date() }
  );
  
  // Send confirmation email
  await sendEmail({
    to: paymentIntent.receipt_email!,
    subject: 'Payment Confirmed',
    html: getPaymentConfirmationEmail(paymentIntent),
  });
}
```

### 7. **Multi-Tenancy Pattern**

```typescript
// Middleware to identify tenant
export function middleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';
  
  // Extract subdomain
  const subdomain = hostname.split('.')[0];
  
  // Add tenant to request
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-tenant-id', subdomain);
  
  return NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });
}

// Use in API routes
export async function GET(request: NextRequest) {
  const tenantId = request.headers.get('x-tenant-id');
  
  await connectDB();
  
  const data = await Data.find({ tenantId });
  
  return NextResponse.json({ data });
}
```

### 8. **Graceful Degradation**

```typescript
// Fallback when services are unavailable
export async function GET(request: NextRequest) {
  try {
    // Try primary service
    const data = await fetchFromPrimaryAPI();
    return NextResponse.json({ data, source: 'primary' });
  } catch (error) {
    console.error('Primary API failed, trying cache', error);
    
    try {
      // Fallback to cache
      const cachedData = await getCachedData();
      if (cachedData) {
        return NextResponse.json({ 
          data: cachedData, 
          source: 'cache',
          warning: 'Using cached data',
        });
      }
    } catch (cacheError) {
      console.error('Cache failed, trying backup API', cacheError);
    }
    
    try {
      // Fallback to backup API
      const backupData = await fetchFromBackupAPI();
      return NextResponse.json({ data: backupData, source: 'backup' });
    } catch (backupError) {
      // All failed - return error with helpful message
      return NextResponse.json(
        { error: 'All data sources unavailable', details: 'Try again later' },
        { status: 503 }
      );
    }
  }
}
```

### 9. **Request Deduplication**

```typescript
// Prevent duplicate in-flight requests
const pendingRequests = new Map<string, Promise<any>>();

export async function deduplicatedFetch(key: string, fetcher: () => Promise<any>) {
  if (pendingRequests.has(key)) {
    return pendingRequests.get(key);
  }
  
  const promise = fetcher().finally(() => {
    pendingRequests.delete(key);
  });
  
  pendingRequests.set(key, promise);
  
  return promise;
}

// Usage in API route
export async function GET(request: NextRequest) {
  const userId = request.nextUrl.searchParams.get('userId')!;
  
  const data = await deduplicatedFetch(`user:${userId}`, async () => {
    await connectDB();
    return await User.findById(userId).lean();
  });
  
  return NextResponse.json({ data });
}
```

### 10. **Health Check Endpoint**

```typescript
// src/app/api/health/route.ts
export async function GET() {
  const checks = {
    database: false,
    redis: false,
    email: false,
  };
  
  // Check database
  try {
    await connectDB();
    await mongoose.connection.db.admin().ping();
    checks.database = true;
  } catch (error) {
    console.error('Database health check failed', error);
  }
  
  // Check Redis
  try {
    await redis.ping();
    checks.redis = true;
  } catch (error) {
    console.error('Redis health check failed', error);
  }
  
  // Check email
  try {
    await getEmailTransporter().verify();
    checks.email = true;
  } catch (error) {
    console.error('Email health check failed', error);
  }
  
  const allHealthy = Object.values(checks).every(Boolean);
  
  return NextResponse.json(
    {
      status: allHealthy ? 'healthy' : 'degraded',
      checks,
      timestamp: new Date().toISOString(),
    },
    { status: allHealthy ? 200 : 503 }
  );
}
```

## Anti-Patterns to Avoid

### Component & Architecture Anti-Patterns

❌ **Don't**: Use Client Components unnecessarily
✅ **Do**: Default to Server Components, add 'use client' only when needed (hooks, event handlers, browser APIs)

❌ **Don't**: Create new MongoDB connections on every request
✅ **Do**: Use the connection singleton pattern shown above

❌ **Don't**: Store passwords in plain text
✅ **Do**: Hash passwords with bcrypt (10+ rounds) before storing

❌ **Don't**: Expose sensitive data in API responses
✅ **Do**: Filter out sensitive fields like passwords, use `.select('-password')` or create safe types

❌ **Don't**: Skip input validation
✅ **Do**: Validate all inputs with Zod schemas before processing

❌ **Don't**: Use inline styles when Tailwind can handle it
✅ **Do**: Leverage Tailwind's utility classes for consistency and performance

❌ **Don't**: Create custom UI components when shadcn/ui has them
✅ **Do**: Use shadcn/ui components and customize as needed

### Performance Anti-Patterns

❌ **Don't**: Fetch all records without pagination
```typescript
// Bad
const users = await User.find({});
```
✅ **Do**: Always implement pagination
```typescript
// Good
const users = await User.find({})
  .skip((page - 1) * limit)
  .limit(limit);
```

❌ **Don't**: Use offset-based pagination for large datasets
```typescript
// Bad - slow for large offsets
.skip(10000).limit(20)
```
✅ **Do**: Use cursor-based pagination
```typescript
// Good - consistent performance
.find({ _id: { $gt: cursor } }).limit(20)
```

❌ **Don't**: Load all relations by default
```typescript
// Bad
const user = await User.findById(id).populate('friends posts comments');
```
✅ **Do**: Only populate what you need
```typescript
// Good
const user = await User.findById(id)
  .populate({ path: 'friends', select: 'name avatar' });
```

❌ **Don't**: Run heavy computations in API routes
```typescript
// Bad - blocks the request
export async function POST() {
  const result = await heavyComputation(); // Takes 30 seconds
  return NextResponse.json({ result });
}
```
✅ **Do**: Use background jobs
```typescript
// Good
export async function POST() {
  await queue.add('heavy-task', data);
  return NextResponse.json({ queued: true });
}
```

### Database Anti-Patterns

❌ **Don't**: Query in loops (N+1 problem)
```typescript
// Bad
for (const post of posts) {
  post.author = await User.findById(post.authorId);
}
```
✅ **Do**: Use aggregation or populate
```typescript
// Good
const posts = await Post.find({}).populate('author');
```

❌ **Don't**: Forget indexes on frequently queried fields
```typescript
// Bad - full collection scan
await User.find({ email: 'user@example.com' });
```
✅ **Do**: Add indexes
```typescript
// Good
UserSchema.index({ email: 1 });
```

❌ **Don't**: Store large arrays in documents
```typescript
// Bad - document can grow indefinitely
const user = {
  name: 'John',
  posts: [/* thousands of posts */]
};
```
✅ **Do**: Use references or separate collections
```typescript
// Good
const Post = { userId: ObjectId, title: String };
```

### Caching Anti-Patterns

❌ **Don't**: Cache everything forever
```typescript
// Bad
cache.set(key, data); // No expiration
```
✅ **Do**: Set appropriate TTLs
```typescript
// Good
cache.setex(key, 3600, data); // 1 hour
```

❌ **Don't**: Cache user-specific data in shared cache
```typescript
// Bad - cache pollution
cache.set('current-user', userData);
```
✅ **Do**: Include user ID in cache key
```typescript
// Good
cache.set(`user:${userId}:data`, userData);
```

### Security Anti-Patterns

❌ **Don't**: Trust client-side data
```typescript
// Bad
const isAdmin = request.body.isAdmin;
```
✅ **Do**: Verify on server
```typescript
// Good
const user = await getUserFromToken(token);
const isAdmin = user.role === 'admin';
```

❌ **Don't**: Expose internal error details
```typescript
// Bad
catch (error) {
  return NextResponse.json({ error: error.message });
}
```
✅ **Do**: Return generic messages
```typescript
// Good
catch (error) {
  logError(error);
  return NextResponse.json({ error: 'Something went wrong' });
}
```

❌ **Don't**: Skip rate limiting on expensive operations
✅ **Do**: Implement rate limiting for all public endpoints

### Scalability Anti-Patterns

❌ **Don't**: Store state in server memory
```typescript
// Bad - won't work across multiple servers
let activeUsers = new Set();
```
✅ **Do**: Use Redis or database
```typescript
// Good
await redis.sadd('active-users', userId);
```

❌ **Don't**: Use long-running requests for async work
✅ **Do**: Use webhooks or polling for long operations

❌ **Don't**: Hardcode limits that can't scale
```typescript
// Bad
const MAX_USERS = 1000;
```
✅ **Do**: Use configuration and design for growth
```typescript
// Good
const MAX_USERS = parseInt(process.env.MAX_USERS || '1000000');
```

## Quick Reference

### Add a new API route
1. Create `src/app/api/[route]/route.ts`
2. Export async functions: GET, POST, PUT, DELETE, PATCH
3. Use `NextRequest` and `NextResponse`
4. Connect to DB if needed
5. Validate inputs with Zod
6. Handle errors properly
7. Add rate limiting for public endpoints
8. Implement caching where appropriate

### Add a new page
1. Create `src/app/[route]/page.tsx`
2. Default to Server Component
3. Add 'use client' only if needed (interactivity, hooks, browser APIs)
4. Use async/await for data fetching in Server Components
5. Implement proper SEO metadata

### Add a new Mongoose model
1. Create `src/lib/db/models/[Model].ts`
2. Define interface extending Document
3. Create schema with validation
4. Add indexes for frequently queried fields
5. Export with hot-reload prevention pattern
6. Add corresponding TypeScript types

### Send an email
1. Create template in `src/lib/email/templates/`
2. Use `sendEmail()` function
3. Provide both HTML and text versions
4. Handle errors appropriately
5. Consider using a queue for bulk emails

### Implement caching
1. Identify cacheable data (infrequent changes, expensive queries)
2. Choose cache layer (Next.js cache, Redis, or both)
3. Set appropriate TTL
4. Implement cache invalidation strategy
5. Add cache key namespacing

### Add pagination
1. Use cursor-based for infinite scroll
2. Use offset-based for page numbers (small datasets only)
3. Always include `limit` parameter
4. Return `hasMore` or `nextCursor` in response
5. Add indexes on sort fields

### Optimize database queries
1. Add indexes for WHERE clauses
2. Use `.lean()` for read-only operations
3. Use `.select()` to limit fields
4. Use aggregation for complex queries
5. Avoid N+1 queries with populate or aggregation

### Implement search
1. For simple: Use MongoDB text indexes
2. For fuzzy: Implement fuzzy matching algorithm
3. For advanced: Use Elasticsearch
4. Add debouncing on frontend
5. Implement result ranking

### Add background jobs
1. Set up Redis connection
2. Create queue (BullMQ or similar)
3. Define job processor
4. Add job to queue from API
5. Implement retry logic
6. Monitor job failures

### Scale the application
1. Ensure stateless API design
2. Implement horizontal scaling
3. Add load balancer
4. Set up database read replicas
5. Use CDN for static assets
6. Implement caching at all layers
7. Monitor performance metrics
8. Set up auto-scaling

This skill provides the foundation for building production-ready, scalable Next.js 16 applications with enterprise-grade patterns and algorithms.
