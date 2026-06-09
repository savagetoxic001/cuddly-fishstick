# LITTLE ANGELS SCHOOL MANAGEMENT SYSTEM - DEPLOYMENT GUIDE

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Backend Deployment](#backend-deployment)
3. [Frontend Deployment](#frontend-deployment)
4. [Database Setup](#database-setup)
5. [Environment Configuration](#environment-configuration)
6. [Domain & DNS Setup](#domain--dns-setup)
7. [Launch Checklist](#launch-checklist)
8. [Post-Launch Monitoring](#post-launch-monitoring)

---

## Prerequisites

Before starting deployment, ensure you have:
- ✅ Node.js v16+ installed locally
- ✅ MongoDB Atlas account (free tier available)
- ✅ Cloudinary account (for file uploads)
- ✅ GitHub account with repository access
- ✅ A domain name (e.g., `littleangelsschool.com`)
- ✅ Credit card for hosting services (most offer free tiers)

**Accounts to create:**
- [Render](https://render.com) - Backend hosting
- [Netlify](https://netlify.com) or [Vercel](https://vercel.com) - Frontend hosting
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) - Database
- [Cloudinary](https://cloudinary.com) - Image/file storage

---

## Backend Deployment

### Option 1: Deploy to Render (Recommended - Free Tier Available)

#### Step 1: Prepare Backend for Deployment

```bash
cd backend
```

Ensure your `package.json` has a start script:
```json
{
  "scripts": {
    "dev": "nodemon index.js",
    "start": "node index.js"
  }
}
```

Create a `.gitignore` file (if not exists):
```
node_modules/
.env
.env.local
```

#### Step 2: Create `.env` File (Backend)

Copy `.env.example` to `.env` and fill in:
```env
# Server
PORT=5000
NODE_ENV=production

# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/little-angels

# JWT
JWT_SECRET=your-super-secret-jwt-key-min-32-characters-long

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Frontend URL (for CORS)
FRONTEND_URL=https://littleangelsschool.com
```

#### Step 3: Deploy on Render

1. **Push code to GitHub**:
   ```bash
   git add .
   git commit -m "Prepare for production deployment"
   git push origin main
   ```

2. **Create Render Service**:
   - Go to [Render Dashboard](https://dashboard.render.com)
   - Click **New +** → **Web Service**
   - Connect your GitHub repository
   - Select the repository and branch

3. **Configure Render Service**:
   - **Name**: `little-angels-backend`
   - **Runtime**: `Node`
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Instance Type**: Free (or Starter for production)

4. **Add Environment Variables**:
   - In Render, go to **Environment** section
   - Add all variables from your `.env` file:
     - `MONGODB_URI`
     - `JWT_SECRET`
     - `CLOUDINARY_CLOUD_NAME`
     - `CLOUDINARY_API_KEY`
     - `CLOUDINARY_API_SECRET`
     - `FRONTEND_URL`
     - `NODE_ENV=production`

5. **Deploy**:
   - Click **Deploy**
   - Wait for deployment to complete (2-5 minutes)
   - Note your backend URL: `https://little-angels-backend.onrender.com`

---

### Option 2: Deploy to Railway.app

1. Go to [Railway.app](https://railway.app)
2. Click **New Project** → **Deploy from GitHub**
3. Select your repository
4. Add environment variables in project settings
5. Railway will automatically deploy on push to main

**Backend URL**: `https://little-angels-backend-production.up.railway.app`

---

### Option 3: Deploy to Fly.io

1. Install Fly CLI: `curl -L https://fly.io/install.sh | sh`
2. Create `Dockerfile` in backend root:
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 5000
   CMD ["npm", "start"]
   ```

3. Deploy:
   ```bash
   fly launch
   fly deploy
   ```

---

## Frontend Deployment

### Option 1: Deploy to Netlify (Recommended)

#### Step 1: Prepare Frontend

Ensure `netlify.toml` exists in frontend root:
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

Ensure `vite.config.js` is configured:
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
  }
})
```

#### Step 2: Create `.env` File (Frontend)

```env
VITE_API_BASE_URL=https://little-angels-backend.onrender.com
```

#### Step 3: Deploy to Netlify

1. **Push code to GitHub**:
   ```bash
   git add .
   git commit -m "Prepare frontend for deployment"
   git push origin main
   ```

2. **Connect to Netlify**:
   - Go to [Netlify Dashboard](https://app.netlify.com)
   - Click **Add new site** → **Import an existing project**
   - Connect GitHub and select your repository

3. **Configure Build Settings**:
   - **Base directory**: `frontend`
   - **Build command**: `npm run build`
   - **Publish directory**: `frontend/dist`

4. **Add Environment Variables**:
   - In Netlify, go to **Site settings** → **Build & deploy** → **Environment**
   - Add: `VITE_API_BASE_URL=https://little-angels-backend.onrender.com`

5. **Deploy**:
   - Click **Deploy**
   - Your frontend URL: `https://little-angels-school.netlify.app`

---

### Option 2: Deploy to Vercel

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. Click **Add New** → **Project**
3. Import your GitHub repository
4. Set **Root Directory**: `frontend`
5. Add environment variable: `VITE_API_BASE_URL`
6. Click **Deploy**

---

## Database Setup

### MongoDB Atlas (Free Tier)

1. **Create Account**: Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)

2. **Create Cluster**:
   - Click **Create** → **Shared Cluster**
   - Select **Free** tier
   - Choose region close to your users
   - Click **Create Cluster**

3. **Create Database User**:
   - Go to **Database Access**
   - Click **Add New Database User**
   - Username: `school_admin`
   - Auto-generate password or set custom
   - Click **Add User**

4. **Get Connection String**:
   - Go to **Clusters** → **Connect**
   - Select **Connect your application**
   - Copy connection string
   - Replace `<username>` and `<password>` with credentials
   - Add database name: `/little-angels`

5. **Whitelist IP** (Allow all for development):
   - Go to **Network Access**
   - Click **Add IP Address**
   - Enter `0.0.0.0/0` (allow all - use IP restriction in production)
   - Click **Confirm**

Example MongoDB URI:
```
mongodb+srv://school_admin:your_password@cluster.mongodb.net/little-angels?retryWrites=true&w=majority
```

---

## Environment Configuration

### Backend Environment Variables Checklist

```env
# Core
PORT=5000
NODE_ENV=production

# Database
MONGODB_URI=mongodb+srv://school_admin:password@cluster.mongodb.net/little-angels

# Security
JWT_SECRET=generate-a-random-string-at-least-32-characters-long
JWT_EXPIRE=7d

# File Storage
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# CORS
FRONTEND_URL=https://littleangelsschool.com
ALLOWED_ORIGINS=https://littleangelsschool.com,https://www.littleangelsschool.com

# Email (Optional - for notifications)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# App Settings
SCHOOL_NAME=Little Angels School
SCHOOL_TIMEZONE=UTC
```

### Frontend Environment Variables Checklist

```env
VITE_API_BASE_URL=https://little-angels-backend.onrender.com
VITE_APP_NAME=Little Angels School Management
```

---

## Domain & DNS Setup

### Register Domain

1. Go to domain registrar (GoDaddy, Namecheap, Google Domains, etc.)
2. Register: `littleangelsschool.com`
3. Note your domain registrar's DNS management interface

### Connect Frontend to Custom Domain (Netlify)

1. In Netlify, go to **Domain settings**
2. Click **Add custom domain**
3. Enter: `littleangelsschool.com`
4. Follow Netlify's instructions to update DNS records:
   - Add `CNAME` record pointing to Netlify
   - Add `A` record for apex domain

Example DNS Records (Netlify):
```
Type    Name                    Value
CNAME   www                     little-angels-school.netlify.app
A       @                       75.2.60.5 (Netlify IP)
```

### Connect Backend to Custom Domain (Render)

1. In Render, go to **Custom Domains**
2. Add: `api.littleangelsschool.com`
3. Render will provide DNS records
4. Update your domain registrar with the provided DNS values

Example DNS Records (Render):
```
Type    Name                    Value
CNAME   api                     your-render-service.onrender.com
```

### SSL Certificate (Free via Let's Encrypt)

- Netlify: ✅ Automatic HTTPS
- Render: ✅ Automatic HTTPS
- Railway: ✅ Automatic HTTPS

---

## Launch Checklist

### Before Going Live

- [ ] **Database**
  - [ ] MongoDB cluster created and secured
  - [ ] Database user created with strong password
  - [ ] Connection string tested
  - [ ] Backup enabled

- [ ] **Backend**
  - [ ] All environment variables set correctly
  - [ ] API endpoints tested (use Postman or similar)
  - [ ] CORS configured for frontend domain
  - [ ] Error logging enabled
  - [ ] Rate limiting enabled (if needed)

- [ ] **Frontend**
  - [ ] All API endpoints connected to production backend
  - [ ] Tested on multiple browsers (Chrome, Firefox, Safari, Edge)
  - [ ] Tested on mobile devices
  - [ ] Images and assets loading correctly
  - [ ] Form submissions working end-to-end

- [ ] **Security**
  - [ ] JWT secret is strong (32+ characters)
  - [ ] No sensitive data in code (API keys, passwords)
  - [ ] HTTPS enabled on all domains
  - [ ] Input validation on all forms
  - [ ] SQL injection/NoSQL injection protection

- [ ] **DNS & Domain**
  - [ ] Domain DNS records propagated (can take 24 hours)
  - [ ] Both `littleangelsschool.com` and `www.littleangelsschool.com` work
  - [ ] API domain (`api.littleangelsschool.com`) resolves correctly

- [ ] **Testing**
  - [ ] User registration works
  - [ ] Login with different roles (Admin, Teacher, Parent) works
  - [ ] File uploads (profile pictures, documents) work
  - [ ] Report PDF generation works
  - [ ] Attendance marking and submission works
  - [ ] Fee management displays correctly

### Launch Steps

1. **Update Frontend API URL**:
   ```bash
   # In frontend .env
   VITE_API_BASE_URL=https://api.littleangelsschool.com
   ```

2. **Deploy Frontend**:
   ```bash
   cd frontend
   npm run build
   # Push to GitHub - Netlify auto-deploys
   git push origin main
   ```

3. **Verify Deployment**:
   - Visit `https://littleangelsschool.com`
   - Check browser console for errors (F12 → Console)
   - Test login functionality

4. **Monitor Initial Traffic**:
   - Check Render/Railway dashboard for errors
   - Monitor MongoDB performance
   - Check Cloudinary bandwidth usage

---

## Post-Launch Monitoring

### Daily Checks (First Week)

- ✅ Check dashboard for errors
- ✅ Monitor API response times
- ✅ Check database storage usage
- ✅ Review user sign-ups and logins
- ✅ Monitor file upload success rate

### Weekly Checks

- ✅ Review server logs for errors
- ✅ Check database backups
- ✅ Monitor costs on all services
- ✅ Update dependencies for security patches
- ✅ Test backup restore process

### Regular Maintenance

- **Updates**: Run `npm update` monthly for dependencies
- **Backups**: Enable automatic MongoDB backups
- **Monitoring**: Set up alerts in Render/Railway for downtime
- **Analytics**: Add Google Analytics to track user behavior
- **Security**: Regular penetration testing and code reviews

### Useful Monitoring Tools

- **Uptime Monitoring**: [Uptime Robot](https://uptimerobot.com) (Free)
- **Error Tracking**: [Sentry](https://sentry.io) (Free tier)
- **Database Monitoring**: MongoDB Atlas built-in monitoring
- **Performance**: [New Relic](https://newrelic.com) (Free tier)

---

## Troubleshooting

### Backend Won't Deploy

**Problem**: Deployment fails with build error
**Solution**:
```bash
# Check for missing dependencies
npm install

# Ensure package.json has start script
npm start  # Test locally first

# Check logs on Render/Railway dashboard
```

### Frontend Shows "Cannot connect to API"

**Problem**: API requests fail with CORS error
**Solution**:
1. Check `VITE_API_BASE_URL` environment variable
2. Verify backend CORS configuration
3. Ensure API domain is correct in backend

### MongoDB Connection Error

**Problem**: "MongoNetworkError" in logs
**Solution**:
1. Check IP whitelist in MongoDB Atlas
2. Verify connection string is correct
3. Ensure database user has correct password
4. Test connection with MongoDB Compass

### SSL Certificate Issues

**Problem**: HTTPS showing as insecure
**Solution**:
1. Netlify/Render handle SSL automatically
2. Wait 24 hours for DNS propagation
3. Clear browser cache and try again

---

## Cost Estimation (Monthly)

| Service | Free Tier | Starter | Notes |
|---------|-----------|---------|-------|
| Render Backend | ✅ $0 | $7+ | Free sleeps after 15 min inactivity |
| Netlify Frontend | ✅ $0 | $19+ | 300 build minutes/month free |
| MongoDB Atlas | ✅ $0 | $57+ | 512 MB free, scales with data |
| Cloudinary | ✅ $0 | $99+ | 10 GB storage free |
| Domain | N/A | $10-15 | Annual cost |
| **Total** | **$0-15** | **$100+** | **Estimated annual**: $120-180 |

---

## Additional Resources

- [Render Deployment Docs](https://render.com/docs)
- [Netlify Docs](https://docs.netlify.com)
- [MongoDB Atlas Guide](https://docs.atlas.mongodb.com)
- [Cloudinary Upload Guide](https://cloudinary.com/documentation)
- [Node.js Best Practices](https://nodejs.org/en/docs/guides/nodejs-docker-webapp)

---

## Support

For deployment issues:
1. Check service status pages (Render, Netlify, MongoDB)
2. Review logs in respective dashboards
3. Community forums: Reddit r/webdev, Stack Overflow
4. Contact hosting provider support

**Good luck with your launch! 🚀**
