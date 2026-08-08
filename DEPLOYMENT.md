# Deploying to transcriber.Healthaid.ai

## Overview

This guide will help you deploy the Healthaid Medical AI transcription app to `transcriber.Healthaid.ai`.

## What's Included

- **Landing Page**: `landing.html` - Marketing page with hero section, features, and CTAs
- **App**: `src/index.html` - The actual transcription application
- **Server**: `server.js` - Node.js server that handles routing and OpenMRS proxy

## URL Structure

- `https://transcriber.Healthaid.ai/` - Landing page
- `https://transcriber.Healthaid.ai/src/index.html` - Direct link to app (used in CTA buttons)
- `https://transcriber.Healthaid.ai/app` - Alternative app URL

## Deployment Options

### Option 1: Deploy to Vercel (Recommended)

1. **Install Vercel CLI** (if not already installed):
   ```bash
   npm install -g vercel
   ```

2. **Create `vercel.json` configuration**:
   ```json
   {
     "version": 2,
     "builds": [
       {
         "src": "server.js",
         "use": "@vercel/node"
       }
     ],
     "routes": [
       {
         "src": "/",
         "dest": "/landing.html"
       },
       {
         "src": "/app",
         "dest": "/src/index.html"
       },
       {
         "src": "/src/(.*)",
         "dest": "/src/$1"
       },
       {
         "src": "/connectors/(.*)",
         "dest": "/connectors/$1"
       },
       {
         "src": "/config/(.*)",
         "dest": "/config/$1"
       },
       {
         "src": "/openmrs-proxy/(.*)",
         "dest": "/server.js"
       }
     ]
   }
   ```

3. **Deploy**:
   ```bash
   cd /path/to/ai_medical_transcriber
   vercel --prod
   ```

4. **Set custom domain**:
   - Go to Vercel dashboard â†’ Your project â†’ Settings â†’ Domains
   - Add `transcriber.Healthaid.ai`
   - Update your DNS records as instructed by Vercel

5. **Add environment variable** (if using .env for API keys):
   - Go to Settings â†’ Environment Variables
   - Add `OPENAI_API_KEY` (if you plan to move it server-side)

### Option 2: Deploy to DigitalOcean App Platform

1. **Push to GitHub** (if not already):
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/yourusername/Healthaid-transcriber.git
   git push -u origin main
   ```

2. **Create App on DigitalOcean**:
   - Go to Apps â†’ Create App
   - Connect your GitHub repository
   - Select the repository
   - Configure:
     - **Type**: Web Service
     - **Run Command**: `node server.js`
     - **HTTP Port**: 3000

3. **Set custom domain**:
   - Go to Settings â†’ Domains
   - Add `transcriber.Healthaid.ai`
   - Update your DNS records

### Option 3: Traditional VPS (Ubuntu)

1. **SSH into your server**:
   ```bash
   ssh user@your-server-ip
   ```

2. **Install Node.js**:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

3. **Clone or upload the project**:
   ```bash
   cd /var/www
   git clone https://github.com/yourusername/Healthaid-transcriber.git
   cd Healthaid-transcriber
   ```

4. **Install PM2** (process manager):
   ```bash
   sudo npm install -g pm2
   ```

5. **Start the app**:
   ```bash
   pm2 start server.js --name Healthaid-transcriber
   pm2 save
   pm2 startup
   ```

6. **Set up Nginx reverse proxy**:
   ```nginx
   server {
       listen 80;
       server_name transcriber.Healthaid.ai;

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

7. **Enable SSL with Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d transcriber.Healthaid.ai
   ```

## DNS Configuration

Add these records to your DNS provider (Cloudflare, Route53, etc.):

```
Type: A
Name: transcriber
Value: [Your server IP or Vercel IP]
TTL: Auto

OR

Type: CNAME
Name: transcriber
Value: [Your Vercel/DigitalOcean domain]
TTL: Auto
```

## Post-Deployment

1. **Test the landing page**: Visit `https://transcriber.Healthaid.ai`
2. **Test the app**: Click "Try Demo" button or visit `https://transcriber.Healthaid.ai/src/index.html`
3. **Verify OpenMRS proxy**: Try connecting to OpenMRS in the app
4. **Check SSL certificate**: Ensure HTTPS is working

## Environment Variables

If you're moving the OpenAI API key to the server (recommended for production):

```bash
# .env file
OPENAI_API_KEY=sk-proj-your-key-here
```

Then update `src/index.html` to fetch the key from the server instead of the meta tag.

## Monitoring

- **Vercel**: Built-in analytics and logs
- **DigitalOcean**: App Platform includes monitoring
- **VPS**: Use PM2 logs:
  ```bash
  pm2 logs Healthaid-transcriber
  pm2 monit
  ```

## Updating the Site

### Vercel
```bash
vercel --prod
```

### DigitalOcean
Push to GitHub, auto-deploys

### VPS
```bash
cd /var/www/Healthaid-transcriber
git pull
pm2 restart Healthaid-transcriber
```

## Security Checklist

- [ ] HTTPS enabled (SSL certificate)
- [ ] API keys moved to environment variables
- [ ] CORS configured properly
- [ ] Rate limiting on API endpoints
- [ ] OpenMRS proxy secured (if needed)
- [ ] HIPAA compliance review completed

## Troubleshooting

**Landing page not loading:**
- Check server.js is serving landing.html for `/`
- Verify landing.html exists in project root

**App not loading:**
- Check `/src/index.html` path
- Verify all connectors are accessible

**OpenMRS proxy failing:**
- Check server logs for proxy errors
- Verify dev3.openmrs.org is accessible from server
- Check CORS headers

**CTA buttons not working:**
- Update all `/src/index.html` links in landing.html
- Test navigation after deployment

## Support

For issues or questions:
- Email: hello@Healthaid.ai
- GitHub: [Your repo URL]
