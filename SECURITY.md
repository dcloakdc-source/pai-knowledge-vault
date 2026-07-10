# PAI Knowledge Vault - Security Configuration

## Protection Status

✅ **Protected by Cloudflare Access**

The vault at https://quartz.cloudenz.org is now protected with Cloudflare Access authentication.

## Access Control

**Method:** Email-based authentication  
**Allowed:** All @cloudenz.org email addresses  
**Session Duration:** 24 hours  
**Identity Provider:** Cloudflare One-Time PIN (OTP)

## How to Access

1. Visit https://quartz.cloudenz.org
2. You'll be redirected to Cloudflare Access login
3. Enter your @cloudenz.org email address
4. Check your email for one-time PIN code
5. Enter the PIN to access the vault
6. Session valid for 24 hours

## Allowed Users

Anyone with an @cloudenz.org email address can access after email verification:
- duane@cloudenz.org
- melissa@cloudenz.org
- (any other @cloudenz.org addresses)

## Cloudflare Access Configuration

**Application Details:**
- **ID:** c5e12722-ecd6-4f28-84e9-089be61b39e3
- **Name:** PAI Knowledge Vault
- **Domain:** quartz.cloudenz.org
- **Type:** Self-hosted

**Policy Details:**
- **ID:** 129193c6-7eef-4a1f-81eb-42f24c4fb794
- **Name:** Allow Family Access
- **Decision:** Allow
- **Include:** email_domain = cloudenz.org
- **Precedence:** 1

## Managing Access

### Add Individual Email

```bash
curl -X POST "https://api.cloudflare.com/client/v4/accounts/3a7266f44a39cd3458824597a7837830/access/apps/c5e12722-ecd6-4f28-84e9-089be61b39e3/policies" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Allow Specific User",
    "decision": "allow",
    "precedence": 2,
    "include": [{"email": {"email": "user@example.com"}}]
  }'
```

### Update Existing Policy

```bash
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/3a7266f44a39cd3458824597a7837830/access/apps/c5e12722-ecd6-4f28-84e9-089be61b39e3/policies/129193c6-7eef-4a1f-81eb-42f24c4fb794" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Allow Family Access",
    "decision": "allow",
    "precedence": 1,
    "include": [{"email_domain": {"domain": "cloudenz.org"}}]
  }'
```

### Remove Protection (Emergency)

```bash
curl -X DELETE "https://api.cloudflare.com/client/v4/accounts/3a7266f44a39cd3458824597a7837830/access/apps/c5e12722-ecd6-4f28-84e9-089be61b39e3" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

## Session Management

**Duration:** 24 hours  
**Cookie:** CF_AppSession (HttpOnly, Secure)  
**Binding Cookie:** Disabled  
**Auto-redirect:** Disabled (shows login page first)

## Additional Security Features Available

### IP Allowlist (Not Currently Active)

Add IP restrictions:
```json
{
  "include": [
    {"ip": {"ip": "YOUR_HOME_IP"}},
    {"ip_list": {"id": "tailscale_ips"}}
  ]
}
```

### Require Multiple Factors (Not Currently Active)

Require email + another factor:
```json
{
  "require": [
    {"email_domain": {"domain": "cloudenz.org"}},
    {"geo": {"country_code": "US"}}
  ]
}
```

### Service Tokens (Not Currently Active)

For API/automated access:
```bash
curl -X POST ".../access/apps/.../service_tokens" \
  --data '{"name": "API Access", "duration": "8760h"}'
```

## Monitoring

**View Access Logs:**
```bash
curl -X GET "https://api.cloudflare.com/client/v4/accounts/3a7266f44a39cd3458824597a7837830/access/logs/access_requests" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

## Cloudflare Zero Trust Dashboard

**Direct Link:** https://one.dash.cloudflare.com/3a7266f44a39cd3458824597a7837830/access/apps

View and manage:
- Application settings
- Access policies
- Authentication logs
- Identity providers
- Service tokens

## Testing

**Test Protection:**
```bash
# Should return 302 redirect to login
curl -I https://quartz.cloudenz.org

# Should show Cloudflare Access login page
curl https://quartz.cloudenz.org
```

**Test After Login:**
After authenticating, you'll have a CF_AppSession cookie valid for 24 hours.

## Troubleshooting

**"Access Denied"**
- Check email domain is @cloudenz.org
- Verify email in spam folder
- Try requesting new PIN

**"Cookie Error"**
- Clear browser cookies for quartz.cloudenz.org
- Try incognito/private window

**"Redirect Loop"**
- Check policy configuration
- Verify no conflicting WAF rules
- Check Cloudflare Access app status

## Costs

**Cloudflare Access Free Tier:**
- Up to 50 users
- Unlimited applications
- Email OTP authentication included
- Zero cost for family use

---

**Configured:** 2026-07-10  
**Last Updated:** 2026-07-10  
**Maintained by:** PAI Nova
