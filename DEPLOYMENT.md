# Deployment Guide for Internet Blueprint

This guide covers deploying the Internet Blueprint Next.js application across various platforms, with specific focus on Nixpacks-based deployment platforms like Easypanel, Railway, and others.

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ 
- npm 9+
- Git repository access

### Environment Variables
Create a `.env.local` file with the following variables:
```bash
# Optional - if using external APIs
NEXT_PUBLIC_API_URL=your_api_url
# Add any other environment variables your app needs
```

## 📦 Platform-Specific Deployment

### Nixpacks Platforms (Easypanel, Railway, Render)

The application is configured with `nixpacks.toml` for automatic deployment:

```toml
[variables]
NODE_ENV = "production"
NEXT_TELEMETRY_DISABLED = "1"

[phases.setup]
nixPkgs = ["nodejs_18", "npm-9_x"]

[phases.install]
cmd = "npm ci --only=production"

[phases.build]
cmd = "npm run build"

[start]
cmd = "npm start"
```

#### Deployment Steps:
1. Push your code to your Git repository
2. Connect repository to your deployment platform
3. The platform will automatically detect the `nixpacks.toml` configuration
4. Build and deployment will proceed automatically

### Vercel Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Production deployment
vercel --prod
```

### Netlify Deployment

Build settings:
- **Build command**: `npm run build`
- **Publish directory**: `.next`
- **Node version**: 18

## 🔧 Build Optimization

### Docker Deployment

Create a `Dockerfile`:
```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm ci --only=production && npm cache clean --force

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

## 🐛 Troubleshooting Common Issues

### Issue 1: "UndefinedVar: Usage of undefined variable '$NIXPACKS_PATH'"

**Solution**: This is a Nixpacks warning that can be safely ignored. The deployment will still work correctly.

**Alternative Fix**: Add to `nixpacks.toml`:
```toml
[variables]
NIXPACKS_PATH = "/app"
```

### Issue 2: "npm run build failed with exit code 1"

**Causes & Solutions**:

1. **TypeScript Errors**
   ```bash
   # Check for type errors
   npm run build
   # Fix type errors in the reported files
   ```

2. **Framer Motion Animation Variants**
   ```typescript
   // ❌ Wrong
   const variants = {
     animate: {
       transition: { ease: "easeInOut" }
     }
   }

   // ✅ Correct
   import { Variants } from "framer-motion";
   const variants: Variants = {
     animate: {
       transition: { ease: [0.4, 0, 0.2, 1] }
     }
   }
   ```

3. **Missing Dependencies**
   ```bash
   npm install
   # or
   npm ci
   ```

### Issue 3: "Module not found" errors

**Solution**: Ensure all imports use correct paths:
```typescript
// Use absolute imports with @ prefix
import { Component } from "@/components/Component"
// Instead of relative imports
import { Component } from "../../components/Component"
```

### Issue 4: Build succeeds but app doesn't start

**Check**:
1. Port configuration in deployment platform
2. Environment variables are set correctly
3. Next.js output is set to standalone if needed:

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone', // Add this for Docker deployments
  // ... rest of config
}
```

### Issue 5: Images not loading in production

**Solution**: Update `next.config.js`:
```javascript
const nextConfig = {
  images: {
    unoptimized: true, // For static exports
    domains: ["your-domain.com"], // Add your domains
  },
}
```

## 🔍 Performance Monitoring

### Build Analysis
```bash
# Analyze bundle size
npm run build

# For detailed analysis, add to package.json:
{
  "scripts": {
    "analyze": "ANALYZE=true npm run build"
  }
}
```

### Production Checks
- [ ] All environment variables are set
- [ ] Build completes without errors
- [ ] TypeScript compilation succeeds
- [ ] No console errors in browser
- [ ] All routes are accessible
- [ ] Images and assets load correctly
- [ ] API routes respond correctly

## 📊 Monitoring and Logs

### Accessing Logs

**Easypanel/Railway/Render**:
- Check the deployment platform's dashboard for build and runtime logs

**Vercel**:
- Visit the Functions tab in your project dashboard
- Real-time logs available during deployment

**Docker/Self-hosted**:
```bash
# View container logs
docker logs container_name

# Follow logs in real-time
docker logs -f container_name
```

## 🔒 Security Considerations

### Environment Variables
- Never commit `.env` files to version control
- Use platform-specific environment variable settings
- Prefix client-side variables with `NEXT_PUBLIC_`

### Headers and CORS
The app includes security headers in `next.config.js`:
```javascript
async headers() {
  return [
    {
      source: "/(.*)",
      headers: [
        {
          key: "Cross-Origin-Embedder-Policy",
          value: "unsafe-none",
        },
        {
          key: "Cross-Origin-Opener-Policy", 
          value: "same-origin-allow-popups",
        },
      ],
    },
  ];
}
```

## 📞 Support

If you encounter issues not covered in this guide:

1. Check the platform-specific documentation
2. Review build logs for specific error messages  
3. Ensure all dependencies are up to date
4. Verify Node.js and npm versions match requirements

For platform-specific issues:
- **Easypanel**: Check Docker container logs
- **Vercel**: Review function logs and deployment details
- **Netlify**: Check build logs and form handling settings
- **Railway**: Monitor service logs and metrics

## 🔄 Deployment Checklist

Before deploying:
- [ ] All TypeScript errors resolved
- [ ] Build completes locally (`npm run build`)
- [ ] Environment variables configured
- [ ] Domain/subdomain configured (if applicable)
- [ ] SSL certificate configured (automatic on most platforms)
- [ ] Performance optimizations applied

After deployment:
- [ ] Test all major user flows
- [ ] Verify API endpoints work
- [ ] Check mobile responsiveness
- [ ] Test form submissions
- [ ] Monitor initial performance metrics