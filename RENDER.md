# Render Deployment Guide

## 🚀 Quick Fix for 404 Error

The 404 error occurs because the application is now deployed at the **root path**. After the recent changes:

### ✅ Correct URLs:
- **Homepage (Email List):** `https://sqlgatewayapp.onrender.com/`
- **SQL Gateway:** `https://sqlgatewayapp.onrender.com/sqlGateway`

## 📋 Deployment Steps for Render

### 1. Prerequisites
- GitHub repository with your code
- Render account (free tier available)

### 2. Database Setup on Render

1. **Create PostgreSQL Database:**
   - Go to Render Dashboard → New → PostgreSQL
   - Name: `sqlgateway-db`
   - Database: `murach`
   - User: `postgres` (or custom)
   - Region: Choose closest to you
   - Plan: Free or Starter
   - Click "Create Database"

2. **Initialize Database:**
   - Wait for database to be ready
   - Go to database → "Connect" tab
   - Copy "PSQL Command"
   - Run locally or use Render Shell:
   ```bash
   # Paste the PSQL command, then run:
   CREATE TABLE "user" (
       userid SERIAL PRIMARY KEY,
       email VARCHAR(50) NOT NULL UNIQUE,
       firstname VARCHAR(50) NOT NULL,
       lastname VARCHAR(50) NOT NULL
   );
   
   INSERT INTO "user" (email, firstname, lastname) VALUES 
       ('jsmith@gmail.com', 'John', 'Smith'),
       ('andi@murach.com', 'Andrea', 'Steelman'),
       ('joelmurach@yahoo.com', 'Joel', 'Murach'),
       ('mike@murach.com', 'Mike', 'Murach');
   ```

### 3. Application Setup on Render

1. **Create Web Service:**
   - Go to Render Dashboard → New → Web Service
   - Connect your GitHub repository
   - Configure:
     - **Name:** `sqlgateway-app`
     - **Environment:** Docker
     - **Region:** Same as database
     - **Branch:** main (or your branch)
     - **Plan:** Free or Starter

2. **Environment Variables:**
   Add these in the "Environment" section:
   - `DB_HOST`: [Your Render PostgreSQL hostname]
   - `DB_PORT`: `5432`
   - `DB_NAME`: `murach`
   - `DB_USER`: `postgres`
   - `DB_PASSWORD`: [Your database password]

3. **Update context-docker.xml for Render:**
   You need to update the database URL to use environment variables or Render's internal connection string.

### 4. Create Render-specific Context File

Create `context-render.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context path="">
    <Resource name="jdbc/murach" 
              auth="Container"
              driverClassName="org.postgresql.Driver"
              url="jdbc:postgresql://[RENDER_DB_HOST]:5432/murach"
              username="postgres" 
              password="[RENDER_DB_PASSWORD]"
              maxTotal="100" 
              maxIdle="30" 
              maxWaitMillis="10000"
              removeAbandonedOnBorrow="true"
              removeAbandonedTimeout="60" 
              type="javax.sql.DataSource" />
</Context>
```

Replace:
- `[RENDER_DB_HOST]` with your Render PostgreSQL hostname
- `[RENDER_DB_PASSWORD]` with your database password

### 5. Update Dockerfile for Render

Modify Dockerfile to use `context-render.xml`:
```dockerfile
# Copy Render-specific context.xml for database configuration
COPY context-render.xml /usr/local/tomcat/conf/Catalina/localhost/ROOT.xml
```

### 6. Deploy

1. Push changes to GitHub
2. Render will automatically detect and deploy
3. Wait for build to complete (5-10 minutes)
4. Access your app at: `https://sqlgatewayapp.onrender.com/`

## 🔧 Troubleshooting

### Application won't start
- Check Render logs: Dashboard → Your Service → Logs
- Verify database connection string
- Ensure database is running

### Database connection errors
- Verify database credentials in context-render.xml
- Check if database and app are in same region
- Test database connection from Render Shell

### 404 Errors
After the recent changes, the app deploys at root:
- ✅ `https://sqlgatewayapp.onrender.com/`
- ✅ `https://sqlgatewayapp.onrender.com/sqlGateway`
- ❌ ~~`https://sqlgatewayapp.onrender.com/SQLGatewayApp/`~~ (old path)

## 📝 Important Notes

### Free Tier Limitations
- Database: 90 days free, then $7/month
- Web Service: Spins down after 15 min inactivity
- First request after spin-down takes 30-60 seconds

### Production Recommendations
- Use paid tier for always-on service
- Enable automatic backups for database
- Set up custom domain
- Configure SSL (automatic on Render)

## 🔄 Local vs Render Configuration

| Environment | Context File | Database Host | URL Path |
|-------------|--------------|---------------|----------|
| Local Docker | `context-docker.xml` | `postgres` | `/` |
| Render | `context-render.xml` | Render hostname | `/` |
| NetBeans Local | `context.xml` | `localhost` | `/SQLGatewayApp` |
