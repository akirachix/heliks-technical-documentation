# RICA Informational Website — Technical Documentation

## 1. Overview

The RICA informational website is the public-facing site for Heliks, presenting RICA — a mineral value-chain intelligence platform that helps mining companies track mineral batches, uncover value losses, and capture more value from Zambia's copper. 

It is separate from any authenticated administrative dashboard or backend console.

## 2. Purpose

- Introduce the Heliks/RICA platform and its purpose
- Explain core capabilities: mineral tracking & traceability, value-chain positioning, scenario modelling & decision support
- Describe impact: transparency, higher value capture, stronger decisions
- Provide a public entry point and link to relevant services
- No direct database access

## 3. Website Structure

```
Informational Website
├── Home (Hero)
├── About Us
├── Our Product (RICA)
│   ├── Features
│   ├── How it Works (3 steps)
│   └── Impact
├── Team
├── Contact Us
└── Navigation (hash anchors)
```

## 4. Navigation

Navbar (`components/layout/Navbar.tsx`) is a client component with links:

| Link | Destination |
|---|---|
| HOME | / |
| ABOUT US | /#about-section |
| PRODUCT | /#our-product |
| TEAM | /#team-section |
| CONTACT US | /#contact-section |

Sticky header; scroll-aware divider; responsive mobile menu.

## 5. Project Placement

```
heliks-informational-website/
├── app/
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── layout/Navbar.tsx
│   ├── home/Hero.tsx, Features.tsx
│   ├── About-us/about-us.tsx
│   ├── our_product/our_product.tsx
│   ├── team/team.tsx
│   └── contact/contact.tsx
└── public/   # images, icons, team photos, logo
```

## 6. User Access

Publicly accessible; no authentication required. Administrative functionality (if any) remains separately protected.

## 7. Backend Relationship

Static content is rendered by the frontend. Dynamic data (if needed) flows via API to the backend; the site never connects directly to PostgreSQL.

## 8. Styling

- Tailwind CSS v4 with custom theme tokens (brand-maroon, brand-orange, ink, divider)
- Container max-width 1440px, horizontal padding 37px
- Our Product section uses CSS Modules for the timeline layout
- Instrument Sans font

## 9. Responsive Design

Supports desktop, tablet, and mobile via Tailwind responsive utilities and a mobile nav menu.

## 10. Platform Relationship

```
Heliks Platform
├── Informational Website (this repo)
├── (future) Mobile App
└── (future) Admin Dashboard
        └── Backend → PostgreSQL
```

## 11. Deployment

- **Platform:** Vercel (Next.js build)
- **Flow:** source → Git → hosting → deployed site
- **Production URL:** [https://heliks-informational-website.vercel.app/](https://heliks-informational-website.vercel.app/)

## 12. Summary

The RICA informational website is a public, responsive Next.js site presenting Heliks/RICA's value-chain intelligence capabilities, with no direct database access and clear separation from any admin functionality.
