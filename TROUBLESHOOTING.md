# Troubleshooting Guide - Internet Blueprint

This guide helps you resolve common deployment and development issues with the Internet Blueprint application.

## 🚨 Common Deployment Issues

### Issue 1: "Couldn't find any `pages` or `app` directory"

**Error Message:**
```
Error: > Couldn't find any `pages` or `app` directory. Please create one under the project root
```

**Solution:**
1. Ensure your project structure includes the `app` directory
2. Check that `.dockerignore` isn't excluding the `app` directory
3. Verify the build context is correct in your deployment platform

**Quick Fix:**
```bash
# Verify app directory exists
ls -la app/

# If missing, your project structure should look like:
project-root/
├── app/
├── package.json
├── next.config.js
└── ...other files
```

### Issue 2: Nixpacks "$NIXPACKS_PATH" Undefined Variable Warning

**Error Message:**
```
UndefinedVar: Usage of undefined variable '$NIXPACKS_PATH'
```

**Solution:**
Update your `nixpacks.toml` file with proper configuration:

```toml
[variables]
NODE_VERSION = "18"
NPM_VERSION = "9"
NIXPACKS_BUILD_TIME_ENV = "true"

[phases.setup]
nixPkgs = ["nodejs_18", "npm"]

[phases.install]
cmds = [
  "npm ci --prefer-offline --no-audit"
]

[phases.build]
cmds = [
  "npm run build"
]

[start]
cmd = "npm start"
```

### Issue 3: Framer Motion Type Errors

**Error Message:**
```
Type 'string' is not assignable to type 'Transition | undefined'
```

**Solution:**
Update animation configurations with proper TypeScript types:

```typescript
import { Variants } from 'framer-motion';

const containerVariants: Variants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      duration: 0.6,
      ease: [0.4, 0, 0.2, 1], // Use array instead of string
      staggerChildren: 0.1
    }
  }
};
```

### Issue 4: npm Security Vulnerabilities

**Error Message:**
```
6 vulnerabilities (1 low, 3 moderate, 1 high, 1 critical)
```

**Solution:**
```bash
# Run audit and fix
npm audit
npm audit fix

# For force fix (use with caution)
npm audit fix --force

# Update dependencies
npm update
```

## 🔧 Development Issues

### Issue 5: TypeScript Compilation Errors

**Common Error:**
```
Type 'undefined' is not assignable to type 'string'
```

**Solution:**
Add proper type checking and null guards:

```typescript
// Before (problematic)
const userMessage = searchParams.message;

// After (fixed)
const userMessage = searchParams?.message || '';

// Or with proper typing
const userMessage: string = searchParams?.message ?? 'Default message';
```

### Issue 6: API Route Type Errors

**Error Message:**
```
Property 'createdAt' is missing in type
```

**Solution:**
Ensure all required properties are included in type definitions:

```typescript
const session: ChatSession = {
  id: 'fallback-session',
  userId: 'anonymous',
  createdAt: new Date(), // Add missing property
  messages: []
};
```

### Issue 7: Environment Variables Not Loading

**Solution:**
1. Create `.env.local` file in project root
2. Add variables with `NEXT_PUBLIC_` prefix for client-side access:

```bash
# Server-side only
DATABASE_URL=your_database_url

# Client-side accessible
NEXT_PUBLIC_API_URL=your_api_url
```

## 🚀 Platform-Specific Solutions

### Easypanel Deployment

1. Ensure `nixpacks.toml` is properly configured
2. Use Node.js 18 or higher
3. Set build environment variables:
   ```
   NODE_ENV=production
   NEXT_TELEMETRY_DISABLED=1
   ```

### Railway Deployment

1. Add `railway.toml`:
   ```toml
   [build]
   builder = "nixpacks"
   
   [deploy]
   startCommand = "npm start"
   ```

### Vercel Deployment

1. Add `vercel.json`:
   ```json
   {
     "builds": [
       {
         "src": "package.json",
         "use": "@vercel/next"
       }
     ]
   }
   ```

### Docker Deployment

1. Use the provided `Dockerfile` with standalone output
2. Ensure `next.config.js` has `output: "standalone"`
3. Build with proper context:
   ```bash
   docker build -t internet-blueprint .
   docker run -p 3000:3000 internet-blueprint
   ```

## 🛠️ Debugging Commands

### Quick Diagnostics

```bash
# Check project structure
ls -la app/

# Verify dependencies
npm list --depth=0

# Check for TypeScript errors
npx tsc --noEmit

# Test local build
npm run build

# Start development server
npm run dev
```

### Build Analysis

```bash
# Analyze bundle size
npm run build -- --analyze

# Check for duplicate dependencies
npm ls --depth=0 | grep -E "WARN|ERROR"

# Clean build cache
rm -rf .next node_modules/.cache
```

## 📋 Pre-Deployment Checklist

- [ ] All TypeScript errors resolved
- [ ] Local build completes successfully
- [ ] Environment variables configured
- [ ] `.env.example` file updated
- [ ] Security vulnerabilities addressed
- [ ] Tests passing (if applicable)
- [ ] Git repository up to date

## 🆘 Emergency Fixes

### Quick Reset
If everything is broken, try this sequence:

```bash
# Clean everything
rm -rf node_modules .next package-lock.json

# Fresh install
npm install

# Test build
npm run build
```

### Rollback Strategy
```bash
# If deployment fails, rollback to working commit
git log --oneline -5
git reset --hard <working-commit-hash>
git push --force-with-lease
```

## 🔗 Useful Resources

- [Next.js Deployment Documentation](https://nextjs.org/docs/deployment)
- [Nixpacks Documentation](https://nixpacks.com/docs)
- [Easypanel Deployment Guide](https://easypanel.io/docs)
- [Railway Deployment Guide](https://railway.app/docs)

## 💡 Getting Help

1. **Check the logs** - Most deployment platforms provide detailed logs
2. **Search the error** - Copy the exact error message and search online
3. **Community support** - Join the Next.js Discord or Stack Overflow
4. **Platform support** - Contact your deployment platform's support team

---

**Last Updated:** January 2025  
**Next.js Version:** 14.x  
**Node.js Version:** 18.x  

For more specific issues not covered here, please create an issue in the project repository with:
- Full error message
- Steps to reproduce
- Environment details (OS, Node version, deployment platform)