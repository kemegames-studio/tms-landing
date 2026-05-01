# Landing Page & Backend Integration Guide

## Overview

The CargoFlow landing page is now fully integrated with the Node.js Express backend. This guide explains how the integration works and how to deploy it.

## Architecture

```
Landing Page (Static HTML)
    ↓
    ├─→ /public/plans (GET) - Load pricing plans
    ├─→ /public/features (GET) - Load features list
    ├─→ /public/signup (POST) - Register new company
    ├─→ /public/contact (POST) - Submit contact form
    └─→ /public/stats (GET) - Load company statistics
    ↓
Express Backend (Node.js)
    ↓
    ├─→ PostgreSQL Database
    ├─→ Stripe Integration
    └─→ JWT Authentication
```

## API Endpoints

### Public Endpoints (No Authentication Required)

#### 1. Get Subscription Plans
```
GET /public/plans
```

**Response:**
```json
{
  "success": true,
  "plans": [
    {
      "id": "starter",
      "name": "Starter",
      "description": "Perfect for small operations",
      "priceMonthly": 2999,
      "maxVehicles": 5,
      "priceFormatted": "$29.99"
    },
    {
      "id": "professional",
      "name": "Professional",
      "description": "For growing businesses",
      "priceMonthly": 7999,
      "maxVehicles": 20,
      "priceFormatted": "$79.99"
    },
    {
      "id": "enterprise",
      "name": "Enterprise",
      "description": "For large enterprises",
      "priceMonthly": 19999,
      "maxVehicles": 999,
      "priceFormatted": "$199.99"
    }
  ]
}
```

#### 2. Get Features List
```
GET /public/features
```

**Response:**
```json
{
  "success": true,
  "features": [
    {
      "id": "tracking",
      "title": "Real-Time Tracking",
      "description": "Track your vehicles and drivers in real-time with GPS integration and live updates.",
      "icon": "tracking"
    },
    ...
  ]
}
```

#### 3. Register New Company (Sign Up)
```
POST /public/signup
Content-Type: application/json

{
  "name": "My Logistics Company",
  "email": "admin@mycompany.com",
  "password": "SecurePassword123!"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Company registered successfully with 7-day free trial",
  "company": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "My Logistics Company",
    "email": "admin@mycompany.com"
  },
  "subscription": {
    "plan": "starter",
    "status": "active",
    "trialEndDate": "2026-05-08T14:30:00Z"
  }
}
```

#### 4. Submit Contact Form
```
POST /public/contact
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@company.com",
  "company": "My Company",
  "message": "I'd like to schedule a demo"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Thank you for your message. We'll get back to you soon!"
}
```

#### 5. Get Company Statistics
```
GET /public/stats
```

**Response:**
```json
{
  "success": true,
  "stats": {
    "activeCompanies": 234,
    "vehiclesManaged": 5678,
    "tripsCompleted": 45230,
    "countriesCovered": 12
  }
}
```

## Setup Instructions

### 1. Configure Backend

Update `.env` file in `/home/ubuntu/tms-backend/`:

```env
# Server Configuration
PORT=5000
NODE_ENV=development

# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=tms_db
DB_USER=postgres
DB_PASSWORD=postgres

# JWT Configuration
JWT_SECRET=your_super_secret_jwt_key_change_this_in_production
JWT_EXPIRY=7d

# CORS Configuration (includes landing page URLs)
CORS_ORIGIN=http://localhost:3000,http://localhost:5173,http://localhost:8888,http://localhost:8080

# Stripe Configuration
STRIPE_SECRET_KEY=sk_test_your_secret_key_here
STRIPE_PUBLISHABLE_KEY=pk_test_your_publishable_key_here
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret_here
```

### 2. Start Backend Server

```bash
cd /home/ubuntu/tms-backend
npm install
npm run migrate
npm start
```

Server will run on `http://localhost:5000`

### 3. Configure Landing Page

Update the `API_URL` in `/home/ubuntu/tms-landing/index.html`:

```javascript
const API_URL = 'http://localhost:5000'; // Change to your backend URL
```

For production:
```javascript
const API_URL = 'https://api.cargoflow.com'; // Your production API URL
```

### 4. Start Landing Page Server

```bash
cd /home/ubuntu/tms-landing
npx http-server -p 8888
```

Landing page will be available at `http://localhost:8888`

## User Flow

### Sign Up Flow

1. **User clicks "Start Free Trial"** on landing page
2. **Sign-up modal opens** with form fields
3. **User enters company name, email, password**
4. **Frontend sends POST request** to `/public/signup`
5. **Backend creates company** and generates JWT token
6. **Backend creates free trial subscription** (7 days, Starter plan)
7. **Backend sends confirmation email** (optional)
8. **Frontend shows success message** and redirects to dashboard
9. **User logs in** with JWT token to access dashboard

### Contact Form Flow

1. **User clicks "Schedule Demo"** or "Contact"
2. **Contact modal opens** with form fields
3. **User enters name, email, company, message**
4. **Frontend sends POST request** to `/public/contact`
5. **Backend logs contact request** (or sends email)
6. **Frontend shows success message**
7. **Sales team receives notification** and follows up

## Error Handling

### Common Errors

#### Email Already Exists
```json
{
  "success": false,
  "error": "Email already registered"
}
```

#### Invalid Password
```json
{
  "success": false,
  "error": "Password must be at least 8 characters"
}
```

#### CORS Error
```
Access to XMLHttpRequest at 'http://localhost:5000/public/plans' from origin 'http://localhost:8888' 
has been blocked by CORS policy
```

**Solution:** Update `CORS_ORIGIN` in backend `.env` to include landing page URL

#### Backend Not Running
```
Failed to fetch: Cannot connect to http://localhost:5000
```

**Solution:** Start backend server with `npm start`

## Testing Integration

### Using cURL

```bash
# Test backend health
curl http://localhost:5000/health

# Get pricing plans
curl http://localhost:5000/public/plans

# Get features
curl http://localhost:5000/public/features

# Test sign up
curl -X POST http://localhost:5000/public/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Company",
    "email": "test@example.com",
    "password": "TestPassword123!"
  }'

# Test contact form
curl -X POST http://localhost:5000/public/contact \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "company": "Test Co",
    "message": "Hello"
  }'
```

### Using Browser DevTools

1. Open landing page in browser
2. Open DevTools (F12)
3. Go to Network tab
4. Click "Start Free Trial"
5. Observe network requests to backend
6. Check response in Network tab

## Deployment

### Deploy Backend

**Option 1: Heroku**
```bash
cd /home/ubuntu/tms-backend
heroku create cargoflow-api
git push heroku main
```

**Option 2: Railway**
```bash
cd /home/ubuntu/tms-backend
railway init
railway up
```

**Option 3: AWS EC2**
```bash
# SSH into EC2 instance
ssh -i key.pem ubuntu@your-instance.com

# Clone repo and deploy
git clone your-repo
cd tms-backend
npm install
npm run migrate
npm start
```

### Deploy Landing Page

**Option 1: Netlify**
```bash
cd /home/ubuntu/tms-landing
npm install -g netlify-cli
netlify deploy --prod
```

**Option 2: Vercel**
```bash
cd /home/ubuntu/tms-landing
npm install -g vercel
vercel --prod
```

**Option 3: AWS S3 + CloudFront**
```bash
cd /home/ubuntu/tms-landing
aws s3 sync . s3://your-bucket-name
```

## Environment Variables

### Backend (.env)

| Variable | Description | Example |
|----------|-------------|---------|
| PORT | Server port | 5000 |
| NODE_ENV | Environment | development/production |
| DB_HOST | Database host | localhost |
| DB_PORT | Database port | 5432 |
| DB_NAME | Database name | tms_db |
| DB_USER | Database user | postgres |
| DB_PASSWORD | Database password | postgres |
| JWT_SECRET | JWT signing key | your_secret_key |
| JWT_EXPIRY | JWT expiry time | 7d |
| CORS_ORIGIN | Allowed origins | http://localhost:8888 |
| STRIPE_SECRET_KEY | Stripe secret key | sk_test_... |
| STRIPE_PUBLISHABLE_KEY | Stripe public key | pk_test_... |
| STRIPE_WEBHOOK_SECRET | Stripe webhook secret | whsec_... |

### Landing Page (index.html)

| Variable | Description | Example |
|----------|-------------|---------|
| API_URL | Backend API URL | http://localhost:5000 |

## Troubleshooting

### Issue: CORS Error

**Error:** `Access to XMLHttpRequest has been blocked by CORS policy`

**Solution:**
1. Check backend is running on correct port
2. Verify `CORS_ORIGIN` includes landing page URL
3. Restart backend server

### Issue: 404 Not Found

**Error:** `POST /public/signup 404 Not Found`

**Solution:**
1. Verify backend is running
2. Check API_URL in landing page is correct
3. Verify publicRoutes.js is imported in index.js

### Issue: Sign Up Fails

**Error:** `Email already registered` or `Invalid password`

**Solution:**
1. Use unique email address
2. Password must be at least 8 characters
3. Check backend logs for detailed error

### Issue: Database Connection Error

**Error:** `ECONNREFUSED 127.0.0.1:5432`

**Solution:**
1. Start PostgreSQL: `sudo service postgresql start`
2. Verify DB credentials in .env
3. Run migrations: `npm run migrate`

## Next Steps

1. **Add authentication to dashboard** - Implement JWT token verification
2. **Create user dashboard** - Show trips, drivers, vehicles
3. **Add payment processing** - Integrate Stripe checkout
4. **Setup email notifications** - Send confirmation emails
5. **Add analytics** - Track sign-ups, conversions
6. **Setup monitoring** - Monitor API performance and errors

## Support

For issues or questions:
- Check API_DOCUMENTATION.md for detailed API reference
- Check STRIPE_BILLING.md for payment integration
- Review backend logs: `tail -f /home/ubuntu/tms-backend/logs.txt`
