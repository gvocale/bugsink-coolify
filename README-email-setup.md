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

## Step 2: Set Environment Variable in Coolify

1. **Log in to Coolify Dashboard**

   - URL: http://91.99.12.107/

2. **Navigate to your Bugsink service**

   - Projects → Your Project → Bugsink service

3. **Go to Environment Variables**

   - Click on the **Environment Variables** tab

4. **Add the new variable**

   - Click **+ Add**
   - Variable Name: `RESEND_API_KEY`
   - Value: `re_your_new_api_key_here` (paste your actual key)
   - Click **Save**

5. **Verify the variable**
   - Ensure `RESEND_API_KEY` appears in the list
   - The value should be masked/hidden for security

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

## Future Improvements

### 1. Custom Domain

For better deliverability:

1. Add your domain in Resend
2. Configure DNS records (SPF, DKIM, DMARC)
3. Update `DEFAULT_FROM_EMAIL` to use your domain
4. Redeploy

### 2. Email Templates

Bugsink supports customizing email templates. See Bugsink documentation for details.

### 3. Rate Limiting

Resend free tier: 3,000 emails/month

- Monitor usage in Resend dashboard
- Upgrade plan if needed

## Support

- **Bugsink Docs**: https://www.bugsink.com/docs/settings/#email
- **Resend Docs**: https://resend.com/docs
- **Coolify Docs**: https://coolify.io/docs
