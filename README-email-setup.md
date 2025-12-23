# Email Setup Guide for Bugsink

This guide walks you through setting up email functionality securely using Resend and Coolify environment variables.

## Overview

Email is configured using:

- **Service**: Resend SMTP
- **SMTP Host**: smtp.resend.com
- **Port**: 465 (SSL)
- **From Address**: Bugsink <onboarding@resend.dev>

## Security Note

The Resend API key is stored as an environment variable in Coolify, not in the git repository. This prevents sensitive credentials from being exposed in version control.

## Step 1: Create/Rotate Resend API Key

### If you need a NEW key (rotating the old exposed one):

1. Go to [resend.com/api-keys](https://resend.com/api-keys)
2. Log in to your account
3. Click **"Create API Key"**
4. Name it: `Bugsink Production` (or similar)
5. Select permission: **Sending access**
6. Click **Create**
7. **Copy the API key** (starts with `re_`)
   - ⚠️ Save it immediately - you can only see it once!
8. **Delete the old key** `re_bWLQUypM_ExTEaUbRWUdc5qNzFaR3xvDF` from Resend dashboard (Security → API Keys → Delete)

### If you're keeping the existing key:

Skip to Step 2 and use: `re_bWLQUypM_ExTEaUbRWUdc5qNzFaR3xvDF`

## Step 2: Set Environment Variables in Coolify

1. **Log in to Coolify Dashboard**

   - URL: http://91.99.12.107/

2. **Navigate to your Bugsink service**

   - Projects → Your Project → Bugsink service

3. **Go to Environment Variables**

   - Click on the **Environment Variables** tab

4. **Add the API key variable**

   - Click **+ Add**
   - Variable Name: `RESEND_API_KEY`
   - Value: `re_your_new_api_key_here` (paste your actual key)
   - Click **Save**

5. **Add the from email variable (Optional but Recommended)**

   - Click **+ Add**
   - Variable Name: `DEFAULT_FROM_EMAIL`
   - Value: `Bugsink <no-reply@tubedatabase.co>` (or your custom domain)
   - Click **Save**
   - See "Custom Domain Setup" section below for how to set up your own domain

6. **Verify the variables**
   - Ensure both `RESEND_API_KEY` and `DEFAULT_FROM_EMAIL` appear in the list
   - Values should be masked/hidden for security

## Step 3: Deploy the Changes

### Option A: Auto-Deploy (if enabled)

If you have auto-deployment enabled, Coolify will automatically deploy when you push to git:

```bash
git push origin main
```

### Option B: Manual Deploy

In Coolify Dashboard:

1. Go to your Bugsink service
2. Click **"Redeploy"** button
3. Wait for deployment to complete (watch the logs)

## Step 4: Test Email Functionality

After deployment completes:

### Test 1: User Registration

1. Go to your Bugsink URL:
   - http://bugsink-8000-xss8occg0k4kogog0kkkw8g8.91.99.12.107.sslip.io
2. Click **"Sign Up"** or create a new user
3. Check for verification email

### Test 2: Password Reset

1. Go to login page
2. Click **"Forgot Password"**
3. Enter an email address
4. Check for password reset email

### Test 3: Monitor in Resend

1. Go to [resend.com/emails](https://resend.com/emails)
2. View delivery status
3. Check email previews
4. Verify no errors

## Troubleshooting

### No emails being sent?

1. **Check Coolify logs**:

   - In Coolify → Your service → Logs
   - Look for email-related errors

2. **Verify environment variable**:

   - Coolify → Environment Variables
   - Ensure `RESEND_API_KEY` is set

3. **Check Resend dashboard**:

   - [resend.com/emails](https://resend.com/emails)
   - See if requests are reaching Resend

4. **Verify API key permissions**:
   - Resend → API Keys
   - Ensure the key has "Sending access"

### Emails going to spam?

This is expected when using `onboarding@resend.dev`. To fix:

1. Verify your own domain in Resend
2. Update `DEFAULT_FROM_EMAIL` in docker-compose.yml:
   ```yaml
   DEFAULT_FROM_EMAIL: "Bugsink <noreply@yourdomain.com>"
   ```
3. Commit and redeploy

## Configuration Reference

### Environment Variables (Set in Coolify)

| Variable         | Value    | Description                            |
| ---------------- | -------- | -------------------------------------- |
| `RESEND_API_KEY` | `re_...` | Resend API key for SMTP authentication |

### Email Settings (in docker-compose.yml)

```yaml
EMAIL_HOST: "smtp.resend.com"
EMAIL_PORT: "465"
EMAIL_USE_SSL: "True"
EMAIL_HOST_USER: "resend"
EMAIL_HOST_PASSWORD: ${RESEND_API_KEY} # Set in Coolify UI
DEFAULT_FROM_EMAIL: "Bugsink <onboarding@resend.dev>"
```

## Security Best Practices

✅ **Do:**

- Store API keys in Coolify environment variables
- Rotate API keys if exposed
- Use descriptive names for API keys
- Monitor Resend dashboard for unusual activity

❌ **Don't:**

- Commit API keys to git
- Share API keys in chat/email
- Use production keys for testing
- Leave old keys active after rotation

## Custom Domain Setup

Using a custom domain (e.g., `no-reply@tubedatabase.co`) significantly improves email deliverability and builds trust with recipients.

### Why Use a Custom Domain?

- ✅ Better deliverability (emails less likely to go to spam)
- ✅ Professional appearance
- ✅ Brand recognition
- ✅ Control over sender reputation
- ⚠️ Default `onboarding@resend.dev` is only for testing

### Step 1: Add Domain to Resend

1. **Log into Resend**

   - Go to https://resend.com
   - Navigate to **Domains** in the left menu

2. **Add Your Domain**

   - Click **Add Domain**
   - Enter your domain: `tubedatabase.co`
   - Click **Add Domain**

3. **Copy DNS Records**
   - Resend will generate 3 DNS records:
     - 1 MX record (for bounce handling)
     - 2 TXT records (SPF and DKIM for authentication)

### Step 2: Configure DNS Records

Add these records to your DNS provider (e.g., Cloudflare, Namecheap, GoDaddy):

**1. MX Record**

```
Type: MX
Name: @ (or tubedatabase.co)
Value: feedback-smtp.us-east-1.amazonses.com
Priority: 10
```

**2. SPF Record (TXT)**

```
Type: TXT
Name: @ (or tubedatabase.co)
Value: v=spf1 include:amazonses.com ~all
```

**3. DKIM Record (TXT)**

```
Type: TXT
Name: [Resend provides - usually: resend._domainkey]
Value: [Long string provided by Resend]
```

#### If Using Cloudflare:

1. Go to Cloudflare dashboard
2. Select your domain: **tubedatabase.co**
3. Click **DNS** → **Records**
4. For each record:
   - Click **Add Record**
   - Select the **Type** (MX or TXT)
   - Enter the **Name** exactly as shown
   - Enter the **Content/Value**
   - For MX: Set **Priority** to 10
   - Click **Save**

### Step 3: Verify Domain in Resend

1. **Wait for DNS Propagation**

   - Usually takes 15-30 minutes
   - Can take up to 72 hours in rare cases

2. **Check DNS Propagation** (Optional)

   - Use online tools like: https://dnschecker.org
   - Search for your domain with record type

3. **Verify in Resend**

   - Go back to Resend → Domains
   - Click **Verify DNS Records** button
   - Each record should show **Verified** with a green checkmark

4. **Troubleshooting Verification**
   - If not verified after 30 minutes, double-check DNS records
   - Ensure no typos in record values
   - Some DNS providers require `@` for root domain
   - Others require the full domain name `tubedatabase.co`

### Step 4: Add DMARC Record (Recommended)

DMARC builds additional trust with email providers:

**Option 1: Simple DMARC**

```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@tubedatabase.co
```

**Option 2: Use DMARC Generator (Recommended)**

1. Go to https://dmarcdkim.com/dmarc-check
2. Enter `tubedatabase.co`
3. Copy the suggested DMARC value
4. Add it to your DNS provider
5. Sign up at DmarcDkim.com for report monitoring

### Step 5: Update Environment Variable

1. **In Coolify Dashboard**

   - Go to your Bugsink service
   - Navigate to **Environment Variables**
   - Find or add: `DEFAULT_FROM_EMAIL`
   - Update value to: `Bugsink <no-reply@tubedatabase.co>`
   - Click **Save**

2. **Redeploy Service**
   - Click **Redeploy** button
   - Wait for deployment to complete

### Step 6: Test Custom Domain

1. **Send Test Email from Bugsink**

   - Trigger a password reset or user registration
   - Check that email comes from `no-reply@tubedatabase.co`

2. **Check Resend Dashboard**

   - Go to https://resend.com/emails
   - Verify emails show your custom domain
   - Check delivery status

3. **Check Email Headers**
   - Open received email
   - View email source/headers
   - Verify SPF, DKIM pass

### Using Subdomains (Alternative)

For better reputation isolation, consider using a subdomain:

**Example:** `send.tubedatabase.co` or `mail.tubedatabase.co`

**Benefits:**

- Isolates sending reputation from main domain
- Allows different policies for different types of emails
- Industry best practice

**Setup:**

1. In Resend, add: `send.tubedatabase.co`
2. Follow same DNS configuration steps
3. Use: `Bugsink <no-reply@send.tubedatabase.co>`

### Coolify Notifications with Custom Domain

To use your custom domain for Coolify notifications:

1. **Configure Coolify Email**

   - Go to Coolify → Settings → Notifications → Email
   - Choose **SMTP Server**
   - Enter:
     ```
     From Name: Coolify
     From Address: coolify@tubedatabase.co
     Host: smtp.resend.com
     Port: 587
     Username: resend
     Password: [Your Resend API Key]
     Encryption: SSL (not TLS - see known issues)
     ```

2. **Enable and Test**
   - Enable via the **Enabled** checkbox
   - Click **Send Test Email**
   - Check delivery

### Important Notes

- **DNS Propagation**: Be patient - usually 15-30 minutes
- **Copy-Paste Carefully**: DKIM records are long and must be exact
- **Test Thoroughly**: Send test emails before production use
- **Monitor Deliverability**: Check Resend dashboard regularly
- **Known Issue**: Coolify SMTP with port 587 + TLS doesn't work in some versions; use 587 + SSL instead

## Future Improvements

### 1. Email Templates

Bugsink supports customizing email templates. See Bugsink documentation for details.

### 2. Rate Limiting

Resend free tier: 3,000 emails/month

- Monitor usage in Resend dashboard
- Upgrade plan if needed

### 3. Multiple Domains

You can configure multiple domains in Resend:

- Different domains for different applications
- Separate marketing vs transactional emails
- Improved reputation management

## Support

- **Bugsink Docs**: https://www.bugsink.com/docs/settings/#email
- **Resend Docs**: https://resend.com/docs
- **Coolify Docs**: https://coolify.io/docs
