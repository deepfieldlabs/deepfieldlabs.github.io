# Company Website -- Complete Guide

This document covers the domain choice, company naming, website structure, hosting, and everything needed to go live.

---

## 1. Domain & Company Name

### Company Name: **DeepField Labs**

**Why this name works:**
- "Deep" references both deep learning and deep-sky astronomy (Hubble Deep Field, James Webb Deep Field)
- "Field" connects to field of view (astronomical surveys) and deep-field imaging
- "Labs" signals innovation, R&D, and open-source culture without overpromising scale
- Short, memorable, professional, not tied to a single product

**Conflict check (completed):**
- No exact match for "DeepField Labs" as a registered company
- "DeepField" (deepfield-ai.com) exists -- AI consumer research company. Different space entirely.
- "Deep Labs" (deep-labs.com) -- cyber threat intelligence. Unrelated.
- "Dark Field Labs" (darkfieldlabs.com) -- R&D consulting. Different name.
- "Dawnfield Labs" (dawnfieldlabs.com) -- AI infrastructure. Different name.
- Conclusion: **"DeepField Labs" is clear of conflicts in the astronomy/AI space.** The name is safe to use.

**Note:** This is a brand/domain name, not necessarily the legal company name. The GitHub Organization can be named `deepfieldlabs` independently of any business registration.

### Domain: `deepfieldlabs.ai`

**Recommendation: Purchase ONE domain -- `deepfieldlabs.ai`**

- `.ai` signals AI expertise immediately
- Most attractive and credible TLD for an AI-focused brand
- Estimated cost: ~$80-100/year
- No need to purchase multiple TLDs at this stage

**Where to purchase:**
Use Route 53 directly on AWS (see DNS section below).

---

## 2. DNS & Hosting: Route 53 vs Cloudflare

### Recommendation: Use Route 53 + CloudFront (all-AWS stack)

Since you're already an AWS specialist and will host on S3 + CloudFront, **Route 53 is the right choice**. Here's why:

**Why Route 53 (not Cloudflare):**

| Factor | Route 53 | Cloudflare |
|--------|----------|------------|
| AWS integration | Native -- Alias records to CloudFront, S3, ELB | Requires CNAME flattening workarounds |
| Domain purchase | Buy directly in Route 53 console | Buy on Namecheap, then configure |
| SSL certificates | ACM integrates seamlessly (must be us-east-1) | Cloudflare issues its own certs |
| Billing | Single AWS bill | Separate billing |
| Hosted zones | $0.50/month per hosted zone | Free tier available |
| DDoS protection | AWS Shield Standard (free) | Cloudflare has stronger free DDoS |
| CDN | CloudFront (same AWS ecosystem) | Cloudflare CDN (would bypass CloudFront) |

**The original suggestion to use Cloudflare was for its free DDoS protection and CDN.** However, for a static company site with minimal traffic, AWS Shield Standard + CloudFront gives you everything you need, and you avoid managing DNS across two providers.

**If Cloudflare makes sense later:** If the site grows to significant traffic or faces attacks, Cloudflare's free WAF and DDoS protection can be added in front of CloudFront. But this isn't necessary at launch.

### Hosting Architecture

```
Route 53 (DNS)
    |
    v
CloudFront (CDN + SSL)
    |
    v
S3 Bucket (Static Files)
    ├── index.html
    ├── profile.jpeg
    ├── detect.gif
    ├── viewer.gif
    └── visualise.gif
```

### Step-by-Step Deployment

```bash
# 1. Purchase domain in Route 53
#    AWS Console → Route 53 → Registered Domains → Register Domain
#    Domain: deepfieldlabs.ai
#    Route 53 auto-creates a hosted zone

# 2. Request SSL certificate in ACM (MUST be us-east-1 for CloudFront)
aws acm request-certificate \
  --domain-name deepfieldlabs.ai \
  --subject-alternative-names "*.deepfieldlabs.ai" \
  --validation-method DNS \
  --region us-east-1

# 3. Create S3 bucket
aws s3 mb s3://deepfieldlabs.ai --region ap-southeast-2

# 4. Upload website files
aws s3 sync /path/to/website/ s3://deepfieldlabs.ai \
  --exclude "howto.md" --exclude ".DS_Store" --exclude "*.md"

# 5. Create CloudFront distribution
#    - Origin: S3 bucket (use OAC, not OAI)
#    - Default root object: index.html
#    - Alternate domains: deepfieldlabs.ai
#    - SSL: Select ACM certificate from step 2
#    - Custom error responses: 403 → /index.html (200), 404 → /index.html (200)
#    - Cache policy: CachingOptimized for static assets

# 6. Add DNS records in Route 53
#    A record (Alias): deepfieldlabs.ai → CloudFront distribution
#    AAAA record (Alias): deepfieldlabs.ai → CloudFront distribution
```

### Cost Estimate (Monthly)
| Service | Cost |
|---------|------|
| Route 53 hosted zone | $0.50 |
| Route 53 queries | ~$0.01 (minimal traffic) |
| S3 storage | ~$0.01 (< 100MB) |
| CloudFront | Free tier (1TB/month) |
| ACM certificate | Free |
| **Total** | **~$0.52/month** + domain registration |

---

## 3. GitHub Organization

### Setup Steps

1. Go to https://github.com/organizations/plan
2. Create organization: `deepfieldlabs` (free plan)
3. Transfer repositories:
   - `samantaba/astroLens` → `deepfieldlabs/astroLens`
   - `samantaba/astroSETI` → `deepfieldlabs/astroSETI`
4. Old URLs auto-redirect. Stars, watchers, issues, forks all preserved.
5. Set org profile: avatar, description, pinned repos

---

## 4. Website Content Summary

### Sections in index.html

| Section | Content |
|---------|---------|
| **Hero** | Headline, tagline, starfield animation, CTAs |
| **Products** | AstroLens (released) + AstroSETI (in dev) with demo GIF |
| **Results** | Key stats grid (20,997 images, 99.5% accuracy, etc.) with gallery GIF |
| **Technology** | Pipeline diagram, tech stack, dashboard GIF |
| **From Research to Production** | Three subsections: Local ML/AI, AWS Cloud Architecture (9 services with diagram), DevOps & Platform Engineering |
| **About** | Profile photo, title, bio, competency tags, LinkedIn/email |
| **Open Source** | Philosophy cards |
| **Contact** | Three engagement paths + links |
| **Footer** | Brand, links, copyright |

### About Section Content

**Title:** Saman Tabatabaeian
**Role:** Technical Director -- Cloud, DevOps & Platform Engineering
**Bio:** Designing universal, production-grade AWS solutions and delivering end-to-end technical packages across software architecture, cloud infrastructure, DevOps, and platform engineering. Focused on building scalable, reusable solution patterns that take ML and AI workloads from research environments into production-ready cloud deployments. Member of the AWS Cloud Warrior ANZ community.

---

## 5. Profile Photo

Two options discussed:
- Formal photo (suit and tie) -- too formal for a tech/AI site
- **Relaxed professional photo (jacket + T-shirt, red background) -- SELECTED**

The selected photo is at `profile.jpeg` (800x800, 150KB). This conveys approachability while maintaining professionalism. Good choice for a tech company site.

---

## 6. Company Logo

The website currently uses text branding ("DeepField Labs" in the header). When ready to add a logo:

**Recommended approach:**
- Simple geometric mark that works at 32px (favicon) and 200px (header)
- Astronomy motif: telescope aperture, constellation pattern, or spectral line
- Color palette: match the site (#4da6ff blue, #7c3aed purple on dark)
- Formats needed: SVG (web), PNG (social media), ICO (favicon)

**Where to get one:**
- Fiverr: $20-100 for professional logo
- Looka.com: AI-generated, $20 basic package
- Figma: Free, DIY if you have design skills

---

## 7. Files in This Folder

| File | Purpose | Size |
|------|---------|------|
| `index.html` | Complete website | ~50KB |
| `howto.md` | This guide | -- |
| `profile.jpeg` | Professional profile photo | 150KB |
| `detect.gif` | Anomaly detection demo | 13MB |
| `viewer.gif` | Gallery and viewer demo | 8MB |
| `visualise.gif` | Web dashboard demo | 10MB |

**To preview:** `open index.html`

**To deploy:** Upload all files except `howto.md` to S3 bucket.

---

## 8. Post-Launch Checklist

- [ ] Purchase `deepfieldlabs.ai` via Route 53
- [ ] Create GitHub Organization `deepfieldlabs`
- [ ] Transfer repos to org
- [ ] Request ACM certificate (us-east-1)
- [ ] Create S3 bucket + CloudFront distribution
- [ ] Deploy website files
- [ ] Configure DNS in Route 53
- [ ] Verify HTTPS works
- [ ] Update og:url and og:image meta tags in index.html
- [ ] Add favicon (once logo is designed)
- [ ] Add Google Analytics or Plausible for traffic tracking
- [ ] Consider adding a blog later (Hugo or similar static site generator)
