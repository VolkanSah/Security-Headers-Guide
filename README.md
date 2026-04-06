# Security Headers — Complete Implementation Guide

**Production-ready HTTP security headers for Apache, Nginx, Node.js, and more.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Security Rating: A+](https://img.shields.io/badge/Security-A+-brightgreen.svg)](https://securityheaders.com)

---

## Why Security Headers Matter

Security headers are your first line of defense against:

- **XSS (Cross-Site Scripting)** — malicious scripts injected into your site
- **Clickjacking** — invisible frames tricking users into clicking malicious content
- **Data leaks** — unauthorized access to cross-origin resources
- **MIME-type attacks** — browsers executing malicious content
- **Referrer leaks** — sensitive URL data exposed to third parties
- **Base tag injection** — attackers manipulating relative URLs
- **FLoC/Topics tracking** — privacy-invasive browser features

**Result:** Setting proper headers can boost your security rating from F to A+ in minutes, with zero code changes to your application.

---

## Quick Start

### Test your current security
```bash
curl -I https://yoursite.com | grep -i "x-frame\|content-security\|strict-transport"

# Or use online tools:
# https://securityheaders.com
# https://observatory.mozilla.org
```

### Choose your platform
- [Apache (.htaccess)](#apache-configuration)
- [Nginx](#nginx-configuration)
- [Node.js / Express](#nodejs--express)
- [Docker / Containers](#docker-containers)
- [Cloudflare Workers](#cloudflare-workers)
- [WordPress](#wordpress-specific)

---

## Apache Configuration

### Complete .htaccess example

Add this to your `.htaccess` or Apache virtual host config:

```apache
# ============================================
# Security Headers — Production Config (2025)
# ============================================

# Hide PHP version (if using PHP)
php_flag expose_php off

<IfModule mod_headers.c>
    # Isolation headers
    Header always set Cross-Origin-Opener-Policy "same-origin"
    Header always set Cross-Origin-Resource-Policy "same-origin"
    Header always set Cross-Origin-Embedder-Policy "require-corp"
    
    # Content Security Policy
    # OPTION 1: Strict (recommended for new sites)
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'; object-src 'none'; upgrade-insecure-requests"
    
    # OPTION 2: Relaxed (for sites with inline scripts/styles)
    # Header always set Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline'; connect-src 'self'; base-uri 'self'; object-src 'none'"
    
    # Frame protection
    Header always set X-Frame-Options "SAMEORIGIN"
    
    # MIME-type protection
    Header always set X-Content-Type-Options "nosniff"
    
    # Download handling (Legacy IE8 - optional in 2025)
    # Header always set X-Download-Options "noopen"
    
    # Feature policies (2025 updated)
    # Note: browsing-topics blocks Google's Topics API (FLoC successor)
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), browsing-topics=(), interest-cohort=()"
    
    # Referrer policy
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    
    # HTTPS enforcement (only if you have SSL!)
    # WARNING: Only enable if your ENTIRE site uses HTTPS permanently
    # Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    
    # Adobe policy restrictions
    Header always set X-Permitted-Cross-Domain-Policies "none"
</IfModule>

# ============================================
# Additional Security Measures
# ============================================

# Disable directory browsing
Options -Indexes

# Disable server signature
ServerSignature Off

# Block suspicious request methods
<LimitExcept GET POST HEAD>
    deny from all
</LimitExcept>
```

### Important Apache Notes

**`Header always` vs `Header set`:**
- `Header set` → Only applied to successful responses (2xx)
- `Header always` → Applied to ALL responses including errors (3xx/4xx/5xx)
- **Always use `always`** for security headers to ensure protection even on error pages

### Enable required Apache modules
```bash
# Enable mod_headers
sudo a2enmod headers
sudo systemctl restart apache2
```

---

## Nginx Configuration

Add to your `nginx.conf` or site-specific config in `/etc/nginx/sites-available/`:

```nginx
server {
    listen 443 ssl http2;
    server_name yoursite.com;
    
    # Hide Nginx version
    server_tokens off;
    
    # Security headers
    add_header Cross-Origin-Opener-Policy "same-origin" always;
    add_header Cross-Origin-Resource-Policy "same-origin" always;
    add_header Cross-Origin-Embedder-Policy "require-corp" always;
    
    # Content Security Policy (2025)
    # Note: 'https:' allows loading from ANY https source - restrict to specific domains in production
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'; object-src 'none'; upgrade-insecure-requests" always;
    
    # Frame protection
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # MIME-type protection
    add_header X-Content-Type-Options "nosniff" always;
    
    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # HTTPS enforcement
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    
    # Permissions policy (2025)
    # browsing-topics blocks Google's Topics API (FLoC successor for ad targeting)
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=(), browsing-topics=(), interest-cohort=()" always;
    
    # Adobe policies
    add_header X-Permitted-Cross-Domain-Policies "none" always;
    
    # Your site config continues here...
    root /var/www/html;
    index index.html;
}
```

### Test and reload
```bash
# Test configuration
sudo nginx -t

# Reload if successful
sudo systemctl reload nginx
```

---

## Node.js / Express

### Using Helmet.js (recommended)

```javascript
const express = require('express');
const helmet = require('helmet');
const crypto = require('crypto');

const app = express();

// Apply all security headers with 2025 best practices
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'"],
            styleSrc: ["'self'"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'"],
            fontSrc: ["'self'"],
            objectSrc: ["'none'"],
            mediaSrc: ["'self'"],
            frameSrc: ["'none'"],
            baseUri: ["'self'"],
            formAction: ["'self'"],
            upgradeInsecureRequests: [],
        },
    },
    crossOriginEmbedderPolicy: true,
    crossOriginOpenerPolicy: { policy: "same-origin" },
    crossOriginResourcePolicy: { policy: "same-origin" },
    hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true
    },
    referrerPolicy: { policy: "strict-origin-when-cross-origin" }
}));

// Additional Permissions-Policy header (Helmet doesn't cover all 2025 features yet)
app.use((req, res, next) => {
    res.setHeader('Permissions-Policy', 
        'geolocation=(), microphone=(), camera=(), payment=(), usb=(), browsing-topics=(), interest-cohort=()');
    next();
});

app.get('/', (req, res) => {
    res.send('Secured with Helmet!');
});

app.listen(3000);
```

### Advanced: CSP with Nonces (recommended for inline scripts)

```javascript
const crypto = require('crypto');

// Middleware to generate nonce per request
app.use((req, res, next) => {
    res.locals.nonce = crypto.randomBytes(16).toString('base64');
    next();
});

app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.nonce}'`],
            styleSrc: ["'self'", (req, res) => `'nonce-${res.locals.nonce}'`],
            objectSrc: ["'none'"],
            baseUri: ["'self'"],
        },
    },
}));

// Use nonce in your templates
app.get('/', (req, res) => {
    res.send(`
        <!DOCTYPE html>
        <html>
        <head>
            <script nonce="${res.locals.nonce}">
                console.log('Inline script allowed with nonce!');
            </script>
        </head>
        <body>Nonce-based CSP</body>
        </html>
    `);
});
```

### Manual implementation (without Helmet)

```javascript
app.use((req, res, next) => {
    res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
    res.setHeader('Cross-Origin-Resource-Policy', 'same-origin');
    res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
    res.setHeader('Content-Security-Policy', "default-src 'self'; object-src 'none'; base-uri 'self'");
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
    res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=(), browsing-topics=(), interest-cohort=()');
    next();
});
```

---

## Docker Containers

### Nginx in Docker

Create `nginx-security.conf`:

```nginx
# Include this in your Nginx Docker image
add_header Cross-Origin-Opener-Policy "same-origin" always;
add_header Cross-Origin-Resource-Policy "same-origin" always;
add_header Content-Security-Policy "default-src 'self'; object-src 'none'; base-uri 'self'" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "interest-cohort=(), browsing-topics=()" always;
```

**Dockerfile:**
```dockerfile
FROM nginx:alpine
COPY nginx-security.conf /etc/nginx/conf.d/security.conf
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80 443
```

---

## Cloudflare Workers

Add headers via Workers or Transform Rules:

```javascript
addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
    const response = await fetch(request)
    
    // Clone response to modify headers
    const newResponse = new Response(response.body, response)
    
    // Add security headers (2025)
    newResponse.headers.set('Cross-Origin-Opener-Policy', 'same-origin')
    newResponse.headers.set('Cross-Origin-Resource-Policy', 'same-origin')
    newResponse.headers.set('Content-Security-Policy', "default-src 'self'; object-src 'none'; base-uri 'self'")
    newResponse.headers.set('X-Frame-Options', 'SAMEORIGIN')
    newResponse.headers.set('X-Content-Type-Options', 'nosniff')
    newResponse.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin')
    newResponse.headers.set('Permissions-Policy', 'browsing-topics=(), interest-cohort=()')
    
    return newResponse
}
```

---

## WordPress Specific

### Common issues with WordPress

WordPress often requires relaxed CSP due to:
- Inline scripts in themes/plugins
- Third-party assets (fonts, analytics, CDNs)
- Admin panel dynamic content

### Recommended WordPress .htaccess

```apache
<IfModule mod_headers.c>
    # Isolation headers (safe for WordPress)
    Header always set Cross-Origin-Opener-Policy "same-origin-allow-popups"
    Header always set Cross-Origin-Resource-Policy "cross-origin"
    
    # Relaxed CSP for WordPress (2025)
    Header always set Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval'; img-src 'self' https: data:; font-src 'self' https: data:; connect-src 'self' https:; base-uri 'self'; object-src 'none'"
    
    # Frame protection (allows same-origin for admin)
    Header always set X-Frame-Options "SAMEORIGIN"
    
    # MIME protection
    Header always set X-Content-Type-Options "nosniff"
    
    # Referrer policy
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    
    # Permissions policy
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=(), browsing-topics=(), interest-cohort=()"
    
    # HSTS (only if SSL is configured)
    # Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>
```

### WordPress plugin alternative

If `.htaccess` doesn't work, use this in `functions.php`:

```php
// Add security headers via PHP
function add_security_headers() {
    header('Cross-Origin-Opener-Policy: same-origin-allow-popups');
    header('Cross-Origin-Resource-Policy: cross-origin');
    header('X-Frame-Options: SAMEORIGIN');
    header('X-Content-Type-Options: nosniff');
    header('Referrer-Policy: strict-origin-when-cross-origin');
    header('Permissions-Policy: geolocation=(), microphone=(), camera=(), browsing-topics=(), interest-cohort=()');
    header("Content-Security-Policy: default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval'; base-uri 'self'; object-src 'none'");
}
add_action('send_headers', 'add_security_headers');
```

---

## Header Breakdown

### Critical Headers (Must Have)

#### Content-Security-Policy (CSP)
**Purpose:** Controls which resources can be loaded  

**Strict (2025 recommended):**
```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'; base-uri 'self'; upgrade-insecure-requests
```

**Key 2025 additions:**
- `object-src 'none'` — Blocks Flash and legacy plugins
- `base-uri 'self'` — Prevents base tag injection attacks
- `upgrade-insecure-requests` — Auto-upgrades HTTP to HTTPS

**Relaxed (for legacy sites):**
```
Content-Security-Policy: default-src 'self' https: data: 'unsafe-inline'; base-uri 'self'; object-src 'none'
```

**Common CSP issues:**
- Inline scripts blocked → Use nonces or move to external `.js` files
- Google Analytics blocked → Add `script-src 'self' https://www.google-analytics.com`
- Fonts not loading → Add `font-src 'self' https://fonts.gstatic.com`

**CSP with Nonces (best practice):**
```html
<!-- Server generates nonce per request -->
<script nonce="2726c7f26c">
  console.log('Allowed!');
</script>
```

**Subresource Integrity (SRI) for CDNs:**
```html
<script src="https://cdn.example.com/lib.js" 
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
        crossorigin="anonymous"></script>
```

#### Strict-Transport-Security (HSTS)
**Purpose:** Forces HTTPS for all connections  
**Configuration:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

**⚠️ CRITICAL WARNINGS:**
1. Only enable if your **entire site** uses SSL permanently
2. Once set, browsers will refuse HTTP for the specified duration
3. `preload` directive submits your domain to browser HSTS lists
4. **Preload is IRREVERSIBLE** — Removal takes months and requires support tickets

**Preload Submission Process:**
1. Ensure HTTPS works on all subdomains
2. Set `max-age=31536000` minimum
3. Submit at https://hstspreload.org
4. Wait for inclusion in Chromium/Firefox/Safari lists
5. **Cannot easily undo** — Only submit if 100% certain

**Safe rollout strategy:**
```apache
# Week 1: Short max-age for testing
Header always set Strict-Transport-Security "max-age=300"

# Week 2-4: Increase gradually
Header always set Strict-Transport-Security "max-age=86400"

# Month 2+: Full deployment
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

# Only after 6+ months of stability:
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

#### X-Frame-Options
**Purpose:** Prevents clickjacking attacks  
**Options:**
- `DENY` — Never allow framing
- `SAMEORIGIN` — Allow framing from same domain only
- ~~`ALLOW-FROM`~~ — (deprecated, use CSP `frame-ancestors` instead)

**Modern alternative:** Use CSP `frame-ancestors` directive:
```
Content-Security-Policy: frame-ancestors 'self'
```

### Important Headers

#### X-Content-Type-Options
```
X-Content-Type-Options: nosniff
```
Prevents browsers from MIME-sniffing responses away from declared content-type.

**Why it matters:** Without this, browsers might execute `image.jpg` as JavaScript if it contains code.

#### Referrer-Policy
**Options:**
- `no-referrer` — Never send referrer (breaks some analytics)
- `strict-origin-when-cross-origin` — **(recommended)** Full URL for same-origin, origin only for cross-origin
- `same-origin` — Only send for same-origin requests

**Privacy consideration:** Balance between analytics needs and user privacy.

#### Permissions-Policy (formerly Feature-Policy)

**2025/2026 updated syntax:**
```
Permissions-Policy: geolocation=(), microphone=(), camera=(), payment=(), usb=(), browsing-topics=(), interest-cohort=()
```

**New in 2025+:**
- `browsing-topics=()` — Blocks Google Topics API (FLoC successor for ad targeting)
- `interest-cohort=()` — Blocks legacy FLoC (kept for older browser compatibility)

**Common permissions:**
- `geolocation` — GPS location
- `microphone` — Audio input
- `camera` — Video input
- `payment` — Payment Request API
- `usb` — USB device access
- `magnetometer`, `gyroscope` — Motion sensors

### Advanced Headers

#### Cross-Origin-Opener-Policy (COOP)
**Purpose:** Isolates browsing context from cross-origin documents  
**Options:**
- `same-origin` — Strictest, breaks pop-ups
- `same-origin-allow-popups` — Allows pop-ups (better for WordPress)

#### Cross-Origin-Resource-Policy (CORP)
**Purpose:** Prevents resources from being loaded by other origins  
**Options:**
- `same-origin` — Only same domain
- `same-site` — Same site (includes subdomains)
- `cross-origin` — Allow all (least secure)

#### Cross-Origin-Embedder-Policy (COEP)
**Purpose:** Required for advanced features like SharedArrayBuffer  
**Configuration:**
```
Cross-Origin-Embedder-Policy: require-corp
```
**⚠️ Warning:** Can break third-party integrations if not configured carefully.

### Deprecated Headers (2025)

#### ❌ X-XSS-Protection (DO NOT USE)
```
# WRONG — This header is deprecated and harmful
X-XSS-Protection: 1; mode=block
```

**Why deprecated:**
- Chrome removed support in 2019
- Safari implementation has security bugs
- Can introduce vulnerabilities instead of preventing them
- Superseded by Content-Security-Policy

**Correct approach:** Rely on CSP instead. If you must set it, disable it:
```
X-XSS-Protection: 0
```

---

## Testing & Validation

### Online Tools
1. **Security Headers** — https://securityheaders.com  
   Quick grade (A+ to F) with actionable recommendations

2. **Mozilla Observatory** — https://observatory.mozilla.org  
   Comprehensive scan with detailed explanations

3. **CSP Evaluator** — https://csp-evaluator.withgoogle.com  
   Validates Content-Security-Policy syntax

4. **HSTS Preload** — https://hstspreload.org  
   Check preload eligibility and status

### Command Line Testing
```bash
# Check all security headers
curl -I https://yoursite.com | grep -iE "content-security|x-frame|strict-transport|x-content|referrer"

# Test specific header
curl -I https://yoursite.com | grep -i "x-frame-options"

# Check from different location (for CDN testing)
curl -H "Host: yoursite.com" -I https://cdn-ip-address/

# Verify CSP is applied
curl -I https://yoursite.com | grep -i "content-security-policy"
```

### Browser DevTools
1. Open DevTools (F12)
2. Go to **Network** tab
3. Refresh page
4. Click on main document
5. Check **Response Headers** section
6. Look for CSP violations in **Console** tab

### Monitor CSP Violations

**Setup CSP reporting:**
```apache
# Apache
Header always set Content-Security-Policy "default-src 'self'; report-uri https://yoursite.com/csp-report"

# Or use report-only mode during testing
Header always set Content-Security-Policy-Report-Only "default-src 'self'; report-uri https://yoursite.com/csp-report"
```

**Node.js CSP report endpoint:**
```javascript
app.post('/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
    console.log('CSP Violation:', req.body);
    res.status(204).end();
});
```

---

## Common Issues & Solutions

### Issue: CSP blocks everything
**Symptom:** Site appears broken, console shows CSP violations  
**Solution:** Start with a relaxed policy, then tighten:
```apache
# Phase 1: Report-only mode (doesn't block, only logs)
Header set Content-Security-Policy-Report-Only "default-src 'self'; report-uri /csp-report"

# Phase 2: After reviewing violations, enable blocking with relaxed policy
Header set Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline'; base-uri 'self'"

# Phase 3: Tighten after confirming no issues
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; base-uri 'self'; object-src 'none'"
```

### Issue: HSTS locks users out after SSL expires
**Symptom:** Users can't access site even after fixing SSL  
**Solution:** 
1. Renew SSL immediately
2. Use short `max-age` initially: `max-age=300` (5 minutes)
3. Gradually increase after confirming stability
4. **Never use `preload` until 100% certain**

### Issue: WordPress admin breaks with strict headers
**Symptom:** Can't save posts, plugins fail to update  
**Solution:** Use WordPress-specific config (see [WordPress section](#wordpress-specific))

### Issue: Third-party embeds don't load
**Symptom:** YouTube videos, Google Maps, etc. blocked  
**Solution:** Adjust CSP frame-src and connect-src:
```apache
Header set Content-Security-Policy "default-src 'self'; frame-src 'self' https://www.youtube.com https://www.google.com; connect-src 'self' https://www.google-analytics.com"
```

### Issue: Google Analytics / Tag Manager blocked
**Solution:** Add to CSP:
```apache
Header set Content-Security-Policy "default-src 'self'; script-src 'self' https://www.googletagmanager.com https://www.google-analytics.com; connect-src 'self' https://www.google-analytics.com"
```

### Issue: Fonts from Google Fonts blocked
**Solution:** Add font-src and style-src:
```apache
Header set Content-Security-Policy "default-src 'self'; font-src 'self' https://fonts.gstatic.com; style-src 'self' https://fonts.googleapis.com"
```

---

## Best Practices

#### 1. Start conservatively
Begin with relaxed policies, monitor for issues, then tighten gradually.

#### 2. Test in staging first
Never deploy security headers directly to production without testing.

#### 3. Monitor CSP violations
Implement CSP reporting to catch issues before users do.

#### 4. Version control your configs
Keep `.htaccess` / `nginx.conf` in Git to track changes.

#### 5. Document exceptions
If you must use `'unsafe-inline'`, document WHY in comments:
```apache
# 'unsafe-inline' required for:
# - WordPress admin dashboard (uses inline scripts)
# - Theme customizer (inline styles)
# TODO: Migrate to nonce-based CSP when possible
Header set Content-Security-Policy "default-src 'self' 'unsafe-inline'"
```

#### 6. Regular audits
Re-scan with securityheaders.com monthly to catch regressions.

#### 7. Use nonces instead of 'unsafe-inline'
Nonces allow specific inline scripts without blanket permission:
```javascript
// Generate per request
const nonce = crypto.randomBytes(16).toString('base64');
res.setHeader('Content-Security-Policy', `script-src 'nonce-${nonce}'`);
```

#### 8. Implement Subresource Integrity (SRI)
For all third-party scripts:
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.0"
        integrity="sha384-..."
        crossorigin="anonymous"></script>
```

---

## Platform-Specific Guides

#### Apache + WordPress + SSL
```apache
<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval'; base-uri 'self'; object-src 'none'"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "interest-cohort=(), browsing-topics=()"
</IfModule>
```

#### Nginx + Static Site + Cloudflare
```nginx
# Cloudflare already provides some headers, avoid duplication
add_header Content-Security-Policy "default-src 'self'; object-src 'none'; base-uri 'self'" always;
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Permissions-Policy "interest-cohort=()" always;
```

#### Node.js API (No Frontend)
```javascript
app.use(helmet({
    contentSecurityPolicy: false, // Not needed for APIs
    frameguard: { action: 'deny' },
    hsts: { maxAge: 31536000 },
    noSniff: true
}));
```

---

## Advanced Topics

### Network Error Logging (NEL)

Monitor network-level errors:

```apache
# Apache
Header always set NEL '{"report_to":"default","max_age":31536000,"include_subdomains":true}'
Header always set Report-To '{"group":"default","max_age":31536000,"endpoints":[{"url":"https://yoursite.com/nel-report"}],"include_subdomains":true}'
```

```javascript
// Node.js endpoint
app.post('/nel-report', express.json(), (req, res) => {
    console.log('Network Error:', req.body);
    res.status(204).end();
});
```

### Certificate Transparency (Expect-CT)

**Note:** Deprecated as of June 2021 (CT now mandatory), but still useful for older browsers:

```apache
Header always set Expect-CT "max-age=86400, enforce, report-uri='https://yoursite.com/ct-report'"
```

---

## Resources

#### Official Documentation
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [Content Security Policy Reference](https://content-security-policy.com/)

#### Tools
- [CSP Generator](https://report-uri.com/home/generate)
- [HSTS Preload List](https://hstspreload.org/)
- [Security Headers Scanner](https://securityheaders.com/)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [SRI Hash Generator](https://www.srihash.org/)

#### Further Reading
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Google Web Fundamentals — Security](https://web.dev/secure/)
- [Scott Helme's Security Headers Blog](https://scotthelme.co.uk/)

---

## Contributing

Improvements welcome!

1. Fork repository
2. Add your platform-specific config
3. Test thoroughly
4. Submit PR with documentation

---

## License

Public Domain. (Publick nowlege)

---

---

## 💬 Support This Project

The AI ecosystem has made security more critical than ever — spam, scams, and automated attacks are everywhere and getting smarter.

If you find these tips useful:

- ⭐ **Star this repository** to show your support
- 💬 **Say hello in the [Discussions](../../discussions)** — feedback and war stories welcome
- 🐛 **[Report issues](../../issues)** if something's broken or outdated
- 💡 **Suggest improvements** or submit a PR
- 💖 **[Sponsor my free time and tests](https://github.com/sponsors/volkansah)** — keeps this project alive

> Simple tips. Hard results. Stay secure. Stay paranoid. 🔒

---

### 🔗 Related Projects

#### Security Guides

- [ModSecurity Webserver Protection Guide](https://github.com/VolkanSah/ModSecurity-Webserver-Protection-Guide)
- [Securing FastAPI Applications](https://github.com/VolkanSah/Securing-FastAPI-Applications)
- [AI API (Wrapper) — Security Best Practices](https://github.com/VolkanSah/AI-API-Security-Best-Practices)
- [WPScan — WordPress Security Scanner Guide](https://github.com/VolkanSah/WordPress-Security-Scanner-advanced-use)
- [Detection Labs for Palantir-Style Activity](https://github.com/VolkanSah/Detection-Labs-for-Palantir-Style-Activity)
- [Block SQL Injection Attacks in PHP](https://github.com/VolkanSah/ModSecurity-rule-to-block-SQL-injection-attacks-in-PHP)

#### Other

- 🤖 **[Multi-LLM API Gateway](https://github.com/VolkanSah/Multi-LLM-API-Gateway)** — A self-hosted AI hub boilerplate, works locally and on Hugging Face free tiers.
- 🚨 **[GitHub & Social Media Scam Exposure](https://github.com/Wall-of-Shames/scammer-analysis-guide)** — Identifying and dismantling phishing campaigns and social engineering attacks on GitHub.
- 🔐 **[How to Secure Your Git Ass](https://github.com/VolkanSah/How-to-Secure-Your-Git-Ass)** — A practical guide to protecting your GitHub presence.

---

> © [S. Volkan Kücükbudak](https://github.com/volkansah)
