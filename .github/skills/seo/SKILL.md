---
name: SEO Best Practices
description: Implement comprehensive SEO following best practices for SaaS apps. Includes meta tags, Schema.org markup, sitemap generation, Open Graph, Twitter Cards, and performance optimization. Use when creating public pages, landing pages, blog posts, or improving organic search rankings.
applyTo: "**/*.{tsx,ts,jsx,js,html,astro,vue}"
---

# SEO Best Practices Skill

Complete SEO implementation for SaaS applications and modern web apps.

## When to Use This Skill

- Creating any public page (landing, pricing, blog, about)
- Improving Google search rankings
- Implementing rich snippets and social cards
- Optimizing Core Web Vitals
- Setting up structured data markup
- Configuring sitemaps and robots.txt

---

## Required Meta Tags

Every public page MUST include these meta tags for optimal SEO:

### SEO Head Component (React/TypeScript)

```tsx
// src/components/common/SEOHead.tsx
import { Helmet } from 'react-helmet-async';

interface SEOProps {
  title: string;
  description: string;
  canonical?: string;
  image?: string;
  type?: 'website' | 'article' | 'product';
  publishedTime?: string;
  modifiedTime?: string;
  author?: string;
  keywords?: string[];
  noindex?: boolean;
}

export default function SEOHead({
  title,
  description,
  canonical,
  image = '/og-image.jpg',
  type = 'website',
  publishedTime,
  modifiedTime,
  author,
  keywords = [],
  noindex = false
}: SEOProps) {
  const siteName = 'YourApp';
  const fullTitle = `${title} | ${siteName}`;
  const url = canonical || (typeof window !== 'undefined' ? window.location.href : '');
  const fullImageUrl = image.startsWith('http') 
    ? image 
    : `${typeof window !== 'undefined' ? window.location.origin : ''}${image}`;

  return (
    <Helmet>
      {/* Basic Meta Tags */}
      <title>{fullTitle}</title>
      <meta name="description" content={description} />
      {keywords.length > 0 && (
        <meta name="keywords" content={keywords.join(', ')} />
      )}
      <link rel="canonical" href={url} />

      {/* Robots */}
      {noindex && <meta name="robots" content="noindex, nofollow" />}

      {/* Open Graph (Facebook, LinkedIn) */}
      <meta property="og:type" content={type} />
      <meta property="og:title" content={title} />
      <meta property="og:description" content={description} />
      <meta property="og:url" content={url} />
      <meta property="og:image" content={fullImageUrl} />
      <meta property="og:image:width" content="1200" />
      <meta property="og:image:height" content="630" />
      <meta property="og:locale" content="en_US" />
      <meta property="og:site_name" content={siteName} />

      {/* Twitter Cards */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={title} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={fullImageUrl} />
      <meta name="twitter:site" content="@yourapp" />
      <meta name="twitter:creator" content="@yourapp" />

      {/* Article Meta Tags (for blog posts) */}
      {type === 'article' && publishedTime && (
        <>
          <meta property="article:published_time" content={publishedTime} />
          {modifiedTime && (
            <meta property="article:modified_time" content={modifiedTime} />
          )}
          {author && <meta property="article:author" content={author} />}
        </>
      )}

      {/* Mobile Optimization */}
      <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=5" />
      <meta name="theme-color" content="#000000" />
    </Helmet>
  );
}
```

### Usage Example:

```tsx
// In any public page component
<SEOHead
  title="Pricing Plans"
  description="Choose the perfect plan for your needs. Starting at $19/month."
  canonical="https://yourapp.com/pricing"
  keywords={['pricing', 'plans', 'subscription', 'saas pricing']}
/>
```

---

## Schema.org Structured Data

Structured data helps search engines understand your content and display rich results.

### 1. Landing Page (Organization + Website Schema)

```tsx
// src/pages/Home.tsx
export default function Home() {
  const organizationSchema = {
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "YourApp",
    "url": "https://www.yourapp.com",
    "logo": "https://www.yourapp.com/logo.png",
    "description": "Brief description of your SaaS product",
    "foundingDate": "2024",
    "founders": [
      {
        "@type": "Person",
        "name": "Founder Name"
      }
    ],
    "contactPoint": {
      "@type": "ContactPoint",
      "email": "contact@yourapp.com",
      "contactType": "Customer Service",
      "availableLanguage": ["English", "Spanish"]
    },
    "sameAs": [
      "https://twitter.com/yourapp",
      "https://www.linkedin.com/company/yourapp",
      "https://github.com/yourapp"
    ]
  };

  const websiteSchema = {
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": "YourApp",
    "url": "https://www.yourapp.com",
    "potentialAction": {
      "@type": "SearchAction",
      "target": {
        "@type": "EntryPoint",
        "urlTemplate": "https://www.yourapp.com/search?q={search_term_string}"
      },
      "query-input": "required name=search_term_string"
    }
  };

  return (
    <>
      <SEOHead
        title="Home"
        description="Your SEO-optimized description here. Keep it under 160 characters."
        keywords={['saas', 'productivity', 'automation']}
      />

      {/* JSON-LD Structured Data */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(organizationSchema) }}
      />
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(websiteSchema) }}
      />

      {/* Page content */}
    </>
  );
}
```

### 2. Blog Post (Article Schema)

```tsx
// src/pages/BlogPost.tsx
interface BlogPostProps {
  post: {
    title: string;
    excerpt: string;
    slug: string;
    featuredImage: string;
    publishedAt: string;
    updatedAt: string;
    author: {
      name: string;
      slug: string;
      image: string;
    };
    content: string;
    readingTime: number;
  };
}

export default function BlogPost({ post }: BlogPostProps) {
  const articleSchema = {
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    "headline": post.title,
    "description": post.excerpt,
    "image": post.featuredImage,
    "datePublished": post.publishedAt,
    "dateModified": post.updatedAt,
    "author": {
      "@type": "Person",
      "name": post.author.name,
      "url": `https://www.yourapp.com/author/${post.author.slug}`,
      "image": post.author.image
    },
    "publisher": {
      "@type": "Organization",
      "name": "YourApp",
      "logo": {
        "@type": "ImageObject",
        "url": "https://www.yourapp.com/logo.png"
      }
    },
    "mainEntityOfPage": {
      "@type": "WebPage",
      "@id": `https://www.yourapp.com/blog/${post.slug}`
    },
    "wordCount": post.content.split(' ').length,
    "timeRequired": `PT${post.readingTime}M`
  };

  const breadcrumbSchema = {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
      {
        "@type": "ListItem",
        "position": 1,
        "name": "Home",
        "item": "https://www.yourapp.com"
      },
      {
        "@type": "ListItem",
        "position": 2,
        "name": "Blog",
        "item": "https://www.yourapp.com/blog"
      },
      {
        "@type": "ListItem",
        "position": 3,
        "name": post.title,
        "item": `https://www.yourapp.com/blog/${post.slug}`
      }
    ]
  };

  return (
    <>
      <SEOHead
        title={post.title}
        description={post.excerpt}
        canonical={`https://www.yourapp.com/blog/${post.slug}`}
        image={post.featuredImage}
        type="article"
        publishedTime={post.publishedAt}
        modifiedTime={post.updatedAt}
        author={post.author.name}
      />

      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(articleSchema) }}
      />
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(breadcrumbSchema) }}
      />

      {/* Article content */}
      <article>
        <h1>{post.title}</h1>
        <div dangerouslySetInnerHTML={{ __html: post.content }} />
      </article>
    </>
  );
}
```

### 3. Pricing Page (Product/Offer Schema)

```tsx
// src/pages/Pricing.tsx
export default function Pricing() {
  const plans = [
    {
      name: 'Starter',
      price: 9.99,
      currency: 'USD',
      description: 'Perfect for individuals getting started'
    },
    {
      name: 'Pro',
      price: 29.99,
      currency: 'USD',
      description: 'For professionals and small teams'
    },
    {
      name: 'Enterprise',
      price: 99.99,
      currency: 'USD',
      description: 'Advanced features for large organizations'
    }
  ];

  const productSchema = plans.map(plan => ({
    "@context": "https://schema.org",
    "@type": "Product",
    "name": `YourApp ${plan.name}`,
    "description": plan.description,
    "brand": {
      "@type": "Brand",
      "name": "YourApp"
    },
    "offers": {
      "@type": "Offer",
      "url": "https://www.yourapp.com/pricing",
      "priceCurrency": plan.currency,
      "price": plan.price.toString(),
      "priceValidUntil": "2026-12-31",
      "itemCondition": "https://schema.org/NewCondition",
      "availability": "https://schema.org/InStock",
      "seller": {
        "@type": "Organization",
        "name": "YourApp"
      }
    }
  }));

  const faqSchema = {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
      {
        "@type": "Question",
        "name": "Can I cancel my subscription anytime?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Yes, you can cancel your subscription at any time with no penalties."
        }
      },
      {
        "@type": "Question",
        "name": "Do you offer a free trial?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Yes, we offer a 14-day free trial for all plans. No credit card required."
        }
      }
    ]
  };

  return (
    <>
      <SEOHead
        title="Pricing Plans"
        description="Choose the perfect plan for your needs. Starting at $9.99/month. 14-day free trial available."
        canonical="https://www.yourapp.com/pricing"
        keywords={['pricing', 'plans', 'subscription', 'cost', 'free trial']}
      />

      {productSchema.map((schema, index) => (
        <script
          key={index}
          type="application/ld+json"
          dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
        />
      ))}

      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(faqSchema) }}
      />

      {/* Pricing content */}
    </>
  );
}
```

### 4. SaaS Software Application Schema

```tsx
// For your main landing page
const softwareAppSchema = {
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "YourApp",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web, iOS, Android",
  "offers": {
    "@type": "Offer",
    "price": "9.99",
    "priceCurrency": "USD"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "ratingCount": "256"
  },
  "screenshot": "https://www.yourapp.com/screenshots/dashboard.png"
};
```

---

## Dynamic Sitemap Generation

### Sitemap Utility (TypeScript)

```typescript
// src/utils/generateSitemap.ts
import { collection, getDocs, query, where } from 'firebase/firestore';
import { db } from '@/lib/firebase';

interface SitemapEntry {
  loc: string;
  lastmod?: string;
  changefreq?: 'always' | 'hourly' | 'daily' | 'weekly' | 'monthly' | 'yearly' | 'never';
  priority?: number;
}

export async function generateSitemap(): Promise<string> {
  const baseUrl = 'https://www.yourapp.com';

  // Static pages
  const staticPages: SitemapEntry[] = [
    { loc: '/', changefreq: 'weekly', priority: 1.0 },
    { loc: '/pricing', changefreq: 'monthly', priority: 0.9 },
    { loc: '/features', changefreq: 'monthly', priority: 0.8 },
    { loc: '/blog', changefreq: 'daily', priority: 0.9 },
    { loc: '/about', changefreq: 'monthly', priority: 0.6 },
    { loc: '/contact', changefreq: 'monthly', priority: 0.6 },
    { loc: '/privacy-policy', changefreq: 'yearly', priority: 0.3 },
    { loc: '/terms-of-service', changefreq: 'yearly', priority: 0.3 }
  ];

  // Dynamic blog posts
  const postsQuery = query(
    collection(db, 'blog-posts'),
    where('published', '==', true)
  );
  const postsSnapshot = await getDocs(postsQuery);

  const blogPosts: SitemapEntry[] = postsSnapshot.docs.map(doc => {
    const data = doc.data();
    return {
      loc: `/blog/${data.slug}`,
      lastmod: data.updatedAt?.toDate().toISOString() || new Date().toISOString(),
      changefreq: 'monthly',
      priority: 0.7
    };
  });

  // Dynamic category pages (if applicable)
  const categoriesSnapshot = await getDocs(collection(db, 'categories'));
  const categoryPages: SitemapEntry[] = categoriesSnapshot.docs.map(doc => ({
    loc: `/category/${doc.data().slug}`,
    changefreq: 'weekly',
    priority: 0.6
  }));

  const allPages = [...staticPages, ...blogPosts, ...categoryPages];

  // Generate XML
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:news="http://www.google.com/schemas/sitemap-news/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml"
        xmlns:mobile="http://www.google.com/schemas/sitemap-mobile/1.0"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
${allPages.map(page => `  <url>
    <loc>${baseUrl}${page.loc}</loc>
    ${page.lastmod ? `<lastmod>${page.lastmod}</lastmod>` : ''}
    ${page.changefreq ? `<changefreq>${page.changefreq}</changefreq>` : ''}
    ${page.priority !== undefined ? `<priority>${page.priority.toFixed(1)}</priority>` : ''}
  </url>`).join('\n')}
</urlset>`;

  return sitemap;
}
```

### Server-side Sitemap Endpoint (Express/Node.js)

```typescript
// api/sitemap.ts (Vercel/Netlify Functions)
import { generateSitemap } from '../utils/generateSitemap';

export default async function handler(req: any, res: any) {
  try {
    const xml = await generateSitemap();

    res.setHeader('Content-Type', 'application/xml');
    res.setHeader('Cache-Control', 'public, max-age=3600, s-maxage=3600');
    res.status(200).send(xml);
  } catch (error) {
    console.error('Error generating sitemap:', error);
    res.status(500).json({ error: 'Failed to generate sitemap' });
  }
}
```

### Firebase Cloud Function Sitemap

```typescript
// functions/src/sitemap.ts
import * as functions from 'firebase-functions';
import { generateSitemap } from './utils/generateSitemap';

export const sitemap = functions.https.onRequest(async (req, res) => {
  try {
    const xml = await generateSitemap();

    res.set('Content-Type', 'application/xml');
    res.set('Cache-Control', 'public, max-age=3600');
    res.status(200).send(xml);
  } catch (error) {
    console.error('Error generating sitemap:', error);
    res.status(500).send('Error generating sitemap');
  }
});
```

---

## robots.txt Configuration

```txt
# public/robots.txt

# Allow all crawlers
User-agent: *
Allow: /

# Block private pages
Disallow: /dashboard
Disallow: /dashboard/*
Disallow: /settings
Disallow: /settings/*
Disallow: /admin
Disallow: /admin/*
Disallow: /api/*

# Block sensitive paths
Disallow: /auth/
Disallow: /checkout/
Disallow: /*.json$

# Sitemap location
Sitemap: https://www.yourapp.com/sitemap.xml

# Crawl delay (optional, use if server load is an issue)
# Crawl-delay: 10
```

---

## Core Web Vitals Optimization

### 1. Image Optimization

```tsx
// Always use optimized images with proper attributes

// ✅ CORRECT: Modern image with lazy loading
<img 
  src="/images/hero.webp" 
  alt="Descriptive alt text for SEO"
  loading="lazy"
  width={1200}
  height={630}
  decoding="async"
/>

// ✅ BETTER: Responsive images with srcset
<img 
  srcSet="/images/hero-320w.webp 320w,
          /images/hero-640w.webp 640w,
          /images/hero-1280w.webp 1280w"
  sizes="(max-width: 640px) 320px,
         (max-width: 1280px) 640px,
         1280px"
  src="/images/hero-1280w.webp"
  alt="Descriptive alt text"
  loading="lazy"
  width={1280}
  height={720}
/>

// ✅ BEST: Next.js Image component (automatic optimization)
import Image from 'next/image';

<Image
  src="/images/hero.jpg"
  alt="Descriptive alt text"
  width={1200}
  height={630}
  priority={false} // true for above-the-fold images
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

### 2. Code Splitting by Routes

```tsx
// src/router.tsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import LoadingSpinner from '@/components/common/LoadingSpinner';

// ✅ Lazy load routes for better initial load performance
const Home = lazy(() => import('@/pages/public/Home'));
const Pricing = lazy(() => import('@/pages/public/Pricing'));
const Blog = lazy(() => import('@/pages/public/Blog'));
const BlogPost = lazy(() => import('@/pages/public/BlogPost'));
const Dashboard = lazy(() => import('@/pages/dashboard/Dashboard'));
const Features = lazy(() => import('@/pages/public/Features'));

export default function Router() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/pricing" element={<Pricing />} />
          <Route path="/features" element={<Features />} />
          <Route path="/blog" element={<Blog />} />
          <Route path="/blog/:slug" element={<BlogPost />} />
          <Route path="/dashboard/*" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### 3. Preload Critical Resources

```tsx
// src/main.tsx or index.html
import { createRoot } from 'react-dom/client';

// Preload critical fonts
const preloadFont = (href: string) => {
  const link = document.createElement('link');
  link.rel = 'preload';
  link.as = 'font';
  link.type = 'font/woff2';
  link.href = href;
  link.crossOrigin = 'anonymous';
  document.head.appendChild(link);
};

preloadFont('/fonts/Inter-var.woff2');
preloadFont('/fonts/Inter-Bold.woff2');

// Preconnect to external domains
const preconnect = (href: string) => {
  const link = document.createElement('link');
  link.rel = 'preconnect';
  link.href = href;
  document.head.appendChild(link);
};

preconnect('https://fonts.googleapis.com');
preconnect('https://cdn.yourcdn.com');

createRoot(document.getElementById('root')!).render(<App />);
```

### 4. Vite Build Optimization

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true }) // Analyze bundle size
  ],
  build: {
    rollupOptions: {
      output: {
        // Manual chunk splitting for optimal caching
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'firebase-vendor': ['firebase/app', 'firebase/auth', 'firebase/firestore', 'firebase/storage'],
          'ui-vendor': ['lucide-react', 'sonner', '@radix-ui/react-dialog'],
          'form-vendor': ['react-hook-form', 'zod', '@hookform/resolvers']
        }
      }
    },
    chunkSizeWarningLimit: 1000,
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true, // Remove console.log in production
        drop_debugger: true
      }
    }
  },
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom']
  }
});
```

### 5. Next.js Configuration for Performance

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,

  // Image optimization
  images: {
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },

  // Compression
  compress: true,

  // Headers for caching
  async headers() {
    return [
      {
        source: '/:all*(svg|jpg|png|webp|avif)',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

### 6. Performance Monitoring Component

```tsx
// src/components/PerformanceMonitor.tsx
import { useEffect } from 'react';

export default function PerformanceMonitor() {
  useEffect(() => {
    // Measure Core Web Vitals
    if (typeof window !== 'undefined' && 'web-vitals' in window) {
      import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
        getCLS(console.log);
        getFID(console.log);
        getFCP(console.log);
        getLCP(console.log);
        getTTFB(console.log);
      });
    }

    // Log performance metrics
    if (window.performance) {
      const perfData = window.performance.timing;
      const pageLoadTime = perfData.loadEventEnd - perfData.navigationStart;
      console.log(`Page load time: ${pageLoadTime}ms`);
    }
  }, []);

  return null;
}

// Add to your App component
function App() {
  return (
    <>
      <PerformanceMonitor />
      {/* Rest of your app */}
    </>
  );
}
```

---

## SEO Pre-Deploy Checklist

### Meta Tags & Structured Data
- [ ] All pages have unique `<title>` tags (50-60 characters)
- [ ] All pages have unique meta descriptions (150-160 characters)
- [ ] Canonical URLs set on all pages
- [ ] Open Graph tags implemented (title, description, image, url)
- [ ] Twitter Card tags implemented
- [ ] Schema.org markup on homepage (Organization/WebSite)
- [ ] Schema.org markup on blog posts (Article)
- [ ] Schema.org markup on pricing (Product/Offer)
- [ ] Breadcrumb schema on inner pages
- [ ] FAQ schema where applicable

### Technical SEO
- [ ] sitemap.xml generated and accessible
- [ ] robots.txt configured correctly
- [ ] All images have descriptive alt text
- [ ] Images optimized (WebP/AVIF format)
- [ ] Lazy loading implemented for below-fold images
- [ ] HTTPS enabled site-wide
- [ ] 404 page exists and is helpful
- [ ] Redirects (301) set up for changed URLs
- [ ] No broken internal links
- [ ] Mobile-responsive design verified

### Performance
- [ ] Lighthouse score > 90 on mobile
- [ ] Lighthouse score > 95 on desktop
- [ ] LCP (Largest Contentful Paint) < 2.5s
- [ ] FID (First Input Delay) < 100ms
- [ ] CLS (Cumulative Layout Shift) < 0.1
- [ ] Code splitting implemented
- [ ] Critical CSS inlined
- [ ] Fonts preloaded
- [ ] External resources preconnected

### Analytics & Monitoring
- [ ] Google Search Console configured
- [ ] Google Analytics 4 installed
- [ ] Core Web Vitals tracking enabled
- [ ] Conversion tracking set up
- [ ] Error tracking configured (Sentry, etc.)

### Content
- [ ] H1 tag on every page (only one per page)
- [ ] Proper heading hierarchy (H1 → H2 → H3)
- [ ] Internal linking strategy implemented
- [ ] Blog content plan created
- [ ] Target keywords researched
- [ ] Content length adequate (1000+ words for pillar content)

---

## Tools & Testing

### Testing Your SEO Implementation

```bash
# 1. Test with Lighthouse (Chrome DevTools)
# Open DevTools → Lighthouse → Run audit

# 2. Check structured data
# Visit: https://search.google.com/test/rich-results
# Enter your URL

# 3. Validate sitemap
# Visit: https://www.xml-sitemaps.com/validate-xml-sitemap.html

# 4. Check mobile-friendliness
# Visit: https://search.google.com/test/mobile-friendly

# 5. Analyze Core Web Vitals
# Visit: https://pagespeed.web.dev/
```

### Recommended Tools

**SEO Analysis:**
- Ahrefs - Comprehensive SEO toolset
- SEMrush - Keyword research & competitor analysis
- Moz - Domain authority tracking
- Screaming Frog - Technical SEO crawler

**Performance:**
- Lighthouse - Built into Chrome DevTools
- WebPageTest - Detailed performance analysis
- GTmetrix - Performance & optimization recommendations
- Google PageSpeed Insights - Core Web Vitals

**Schema Testing:**
- Google Rich Results Test
- Schema Markup Validator
- JSON-LD Playground

---

## Common SEO Mistakes to Avoid

### ❌ Don't Do This:

1. **Duplicate title tags** across multiple pages
2. **Missing alt text** on images
3. **Blocking CSS/JS** in robots.txt
4. **Slow page load times** (> 3 seconds)
5. **Non-responsive design** on mobile
6. **Broken links** (404 errors)
7. **Thin content** (< 300 words on important pages)
8. **Keyword stuffing** in meta tags or content
9. **Using only h1 tags** or skipping heading levels
10. **Not setting canonical URLs** (causes duplicate content issues)

### ✅ Best Practices:

1. **Write unique, descriptive titles** for each page
2. **Create compelling meta descriptions** that encourage clicks
3. **Optimize images** (compress, use modern formats, add alt text)
4. **Implement proper URL structure** (readable, hierarchical)
5. **Create high-quality content** that answers user questions
6. **Build internal links** to important pages
7. **Update content regularly** to keep it fresh
8. **Monitor and fix errors** in Google Search Console
9. **Use HTTPS** everywhere
10. **Focus on user experience** first, SEO second

---

## Resources

- **Google Search Central**: https://developers.google.com/search
- **Schema.org Documentation**: https://schema.org/
- **Web.dev SEO Guide**: https://web.dev/learn/seo/
- **Moz Beginner's Guide to SEO**: https://moz.com/beginners-guide-to-seo
- **Core Web Vitals**: https://web.dev/vitals/

---

## Summary

This SEO skill provides everything needed to implement comprehensive SEO for modern SaaS applications:

- **Meta tags** for social sharing and search engines
- **Structured data** (Schema.org) for rich results
- **Dynamic sitemap** generation
- **Performance optimization** for Core Web Vitals
- **Complete checklist** for pre-deploy verification

Apply these practices consistently across all public pages to maximize organic search visibility.
