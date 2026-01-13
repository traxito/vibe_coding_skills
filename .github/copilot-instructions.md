# GitHub Copilot Instructions

## Project Overview

This is a modern B2C SaaS application built with AI-first principles. The project targets Spanish-speaking users (primarily in Spain) who need [specific solution]. We emphasize transparency about AI usage throughout the user experience.

## Business Context

### Target Audience
- **Primary Market**: Spain (ES), Spanish language
- **User Profile**: Non-technical users seeking automated solutions
- **User Needs**: Simple, fast, and trustworthy tools that solve real problems
- **Age Range**: 25-55 years old, comfortable with technology but not developers

### Value Proposition
- Solve complex problems with simple, AI-powered interfaces
- Transparent AI usage (users see and understand AI is helping them)
- Freemium model with clear value progression
- Fast, reliable, and privacy-conscious

### Pricing Strategy
- **Free Tier**: Basic features, limited usage (hook users, demonstrate value)
- **Pro Tier**: €19.99/month, full features, unlimited usage
- **Enterprise Tier**: Custom pricing, dedicated support, SLA

## Development Principles

### 1. Modern & AI-Visible Design
- **Modern UI**: Clean, minimalist, with subtle animations and micro-interactions
- **AI Transparency**: Show AI working (loading states, "AI is analyzing...", confidence scores)
- **Visual Feedback**: Progress indicators, real-time updates, skeleton loaders
- **Professional Yet Approachable**: Not corporate-stiff, not startup-playful, balanced

### 2. User-First Experience
- **Simplicity**: One clear action per screen, minimize cognitive load
- **Speed**: Fast page loads (< 2s), instant feedback, optimistic UI updates
- **Mobile-First**: Design for mobile, enhance for desktop
- **Accessibility**: WCAG AA minimum, keyboard navigation, screen reader friendly
- **Trust**: Clear pricing, no hidden fees, transparent data usage

### 3. Performance & Cost-Consciousness
- **Optimize Bundle Size**: Code splitting, lazy loading, tree shaking
- **Minimize API Calls**: Cache aggressively, batch requests where possible
- **Firebase Optimization**: Limit reads/writes, use composite indexes
- **Stripe Cost Efficiency**: Minimize API calls, use webhooks correctly

### 4. Security & Privacy
- **GDPR Compliance**: Cookie consent, privacy policy, data export/deletion
- **Data Minimization**: Only collect what's necessary
- **Secure by Default**: Firebase rules, input validation, XSS prevention
- **Transparent Communication**: Explain what data we collect and why

### 5. Maintainability
- **Clear Over Clever**: Readable code beats clever optimizations
- **TypeScript Strict**: Catch errors at compile time
- **Consistent Patterns**: Follow established conventions (see skills/)
- **Self-Documenting**: Good naming > excessive comments

## Tech Stack (High-Level)

```
Frontend:  React 18 + TypeScript + Next.js + Tailwind CSS
UI:        Shadcn UI + Lucide Icons + Sonner (toasts)
Backend:   Firebase (Auth, Firestore, Functions, Hosting)
Payments:  Stripe (subscriptions)
Email:     Gmail SMTP + Nodemailer
DNS:       Cloudflare
```

**For detailed implementation, refer to `.github/skills/saas-stack/SKILL.md`**

## Design & UX Guidelines

### Visual Style

**Modern & Clean**
- Minimalist design with plenty of white space
- Subtle gradients and shadows (avoid flat design extremes)
- Rounded corners (4px-8px), smooth transitions
- Contemporary color palette (not too bright, not too muted)

**Typography**
- **Primary Font**: Inter, SF Pro, or similar modern sans-serif
- **Headings**: Bold (600-700), clear hierarchy (H1 > H2 > H3)
- **Body Text**: Regular (400), 16px minimum for readability
- **Code/Data**: Monospace font (Firacode, JetBrains Mono)

**Color Palette**
- **Primary**: Professional blue (#0ea5e9 or similar)
- **Success**: Green (#22c55e)
- **Warning**: Amber (#f59e0b)
- **Error**: Red (#ef4444)
- **Neutral**: Gray scale (#f8fafc to #0f172a)

**AI Visual Indicators**
- Sparkle icons (✨) for AI-powered features
- Gradient accents on AI sections
- Loading states: "AI is analyzing..." with progress
- Confidence indicators: "95% confidence" or visual bars

### Tone & Language

**Spanish UI Text**
- **Tone**: Cercano pero riguroso (friendly but rigorous)
- **Voice**: Professional yet approachable, use "tú" (informal you)
- **Style**: Clear, direct, avoid jargon unless necessary
- **Examples**:
  - ✅ "Analiza tu nómina con IA en segundos"
  - ✅ "Nuestra IA ha detectado un error en tu nómina"
  - ❌ "Utilice nuestro sistema de inteligencia artificial" (too formal)
  - ❌ "¡Flipante! La IA hace magia 🎉" (too casual)

**English Code & Comments**
- All code, variable names, comments in English
- Type definitions, function names, documentation in English
- Commit messages in English

**AI Transparency Examples**
- "AI-powered analysis" badges on features
- "Analyzed by AI" labels on results
- "AI confidence: 98%" on predictions
- "Our AI scanned 1,234 data points" on insights
- Progress indicators: "AI is processing your document..."

### UX Patterns

**Landing Page**
- Hero section: Clear value prop + AI mention in first 3 seconds
- Social proof: "Trusted by X users" + testimonials
- Feature highlights: Each feature mentions AI usage
- Clear CTA: "Try Free (No Credit Card)" above the fold

**Dashboard**
- Overview cards with AI-generated insights
- Quick actions: Most common tasks front and center
- AI suggestions: "Based on your usage, we recommend..."
- Activity feed: "AI detected X new items"

**Forms**
- Real-time validation with friendly errors
- Auto-save drafts (show "Saved" indicator)
- Smart defaults: "AI suggests: [option]"
- Progress indicators for multi-step forms

**Loading States**
- Skeleton loaders (not spinners alone)
- "AI is analyzing..." with animated icon
- Progress percentage when possible
- Estimated time remaining for long operations

**Empty States**
- Helpful illustrations (not just text)
- Clear next action: "Upload your first document"
- AI onboarding tips: "Try asking AI to..."

## Code Guidelines

### Language & Conventions

**Code Language**: English only
```typescript
// ✅ CORRECT
const userName = 'John';
function calculateTotal() {}

// ❌ WRONG
const nombreUsuario = 'John';
function calcularTotal() {}
```

**UI Text Language**: Spanish
```tsx
// ✅ CORRECT
<button>Analizar con IA</button>
<p>Tu nómina ha sido procesada correctamente</p>

// ❌ WRONG
<button>Analyze with AI</button>
<p>Your payroll has been processed successfully</p>
```

### TypeScript Standards

**Always Use Strict Mode**
```typescript
// ✅ CORRECT: Explicit types
interface User {
  id: string;
  email: string;
  plan: 'free' | 'pro' | 'enterprise';
}

function getUser(id: string): Promise<User> {
  // Implementation
}

// ❌ WRONG: Using 'any'
function getUser(id: any): any {
  // Implementation
}
```

**Type Imports**
```typescript
// ✅ CORRECT
import type { User } from '@/types/user';
import { useState } from 'react';

// ❌ WRONG
import { User } from '@/types/user'; // Not type-only
```

### Component Structure

**Consistent Pattern**
```tsx
import { useState, useEffect } from 'react';
import type { Props } from './types';

export default function Component({ prop1, prop2 }: Props) {
  // 1. Hooks
  const [state, setState] = useState();

  // 2. Effects
  useEffect(() => {
    // Side effects
  }, []);

  // 3. Handlers
  const handleClick = () => {
    // Logic
  };

  // 4. Early returns
  if (!data) return <Loading />;

  // 5. Main render
  return (
    <div className="container">
      {/* JSX */}
    </div>
  );
}
```

### Styling Standards

**Tailwind CSS Only**
```tsx
// ✅ CORRECT: Tailwind classes
<button className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors">
  Analizar con IA
</button>

// ❌ WRONG: Inline styles
<button style={{ padding: '8px 16px', backgroundColor: 'blue' }}>
  Analizar con IA
</button>
```

**Mobile-First Responsive**
```tsx
// ✅ CORRECT: Mobile first, then tablet/desktop
<div className="p-4 md:p-6 lg:p-8 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
  {/* Content */}
</div>
```

**AI Visual Indicators**
```tsx
// ✅ Add AI visual cues
<div className="flex items-center gap-2 px-3 py-1 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full text-white text-sm">
  <Sparkles className="w-4 h-4" />
  <span>Powered by AI</span>
</div>
```

## Specific Requirements

### GDPR Compliance
- Cookie consent banner on first visit
- Privacy policy page (link in footer)
- Terms of service page
- Data export functionality (user settings)
- Data deletion request handling
- Clear explanation of data collection

### Email Communications
- **Transactional**: Always sent (confirmations, receipts, password reset)
- **Marketing**: Must include unsubscribe link
- **User Preferences**: Allow users to opt-out of non-essential emails
- **Tone**: Professional but friendly (Spanish)
- **AI Mentions**: Highlight AI features in onboarding emails

### Payment & Pricing
- Always validate amounts server-side (never trust client)
- Show clear pricing before subscription
- No hidden fees or surprise charges
- Easy cancellation (no dark patterns)
- Trial periods clearly communicated
- Invoices sent automatically

### SEO & Performance
- Meta tags on all public pages (title, description, OG tags)
- Semantic HTML (proper heading hierarchy)
- Alt text on all images
- Structured data (JSON-LD) for rich snippets
- Page load < 2 seconds
- Core Web Vitals: LCP < 2.5s, FID < 100ms, CLS < 0.1

### Security
- Firebase Security Rules (strict, deny by default)
- Input validation on all forms (Zod schemas)
- XSS prevention (sanitize user input)
- HTTPS only (no HTTP)
- Environment variables for secrets
- Rate limiting on API endpoints

## AI Integration Best Practices

### Show AI Working
```tsx
// ✅ CORRECT: Show AI processing
const [analyzing, setAnalyzing] = useState(false);

{analyzing && (
  <div className="flex items-center gap-2 text-blue-600">
    <Loader2 className="w-4 h-4 animate-spin" />
    <span>La IA está analizando tu documento...</span>
  </div>
)}
```

### Display Confidence Scores
```tsx
// ✅ Show AI confidence when relevant
<div className="flex items-center gap-2">
  <span>Resultado:</span>
  <span className="font-semibold">{result}</span>
  <span className="text-xs text-gray-500">(Confianza: {confidence}%)</span>
</div>
```

### Highlight AI Features
```tsx
// ✅ Badge AI-powered features
<div className="relative">
  <div className="absolute -top-2 -right-2 px-2 py-1 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full text-white text-xs flex items-center gap-1">
    <Sparkles className="w-3 h-3" />
    <span>IA</span>
  </div>
  <Card>
    <h3>Análisis Automático</h3>
    <p>Nuestra IA revisa tu documento en segundos</p>
  </Card>
</div>
```

### Explain AI Decisions
```tsx
// ✅ Make AI transparent
<div className="bg-blue-50 border border-blue-200 rounded-lg p-4">
  <div className="flex items-start gap-3">
    <Brain className="w-5 h-5 text-blue-600 mt-0.5" />
    <div>
      <h4 className="font-semibold text-blue-900">Por qué la IA recomienda esto</h4>
      <p className="text-sm text-blue-700">
        Analizamos {dataPoints} puntos de datos y detectamos que...
      </p>
    </div>
  </div>
</div>
```

## What to Avoid

### ❌ Don't Do This

**Design**
- Don't use outdated UI patterns (tables without styling, plain buttons)
- Don't hide AI usage (users want to know AI is helping)
- Don't use complex layouts on mobile
- Don't use small fonts (< 14px)
- Don't forget loading states

**Code**
- Don't use 'any' type in TypeScript
- Don't inline styles (use Tailwind)
- Don't mutate state directly
- Don't ignore errors (always handle catch blocks)
- Don't use 'var' (use const/let)

**UX**
- Don't use dark patterns (trick users into subscriptions)
- Don't hide pricing or fees
- Don't make cancellation difficult
- Don't use confusing language or jargon
- Don't skip error messages

**Business**
- Don't collect unnecessary data
- Don't sell user data (we never do this)
- Don't send marketing emails without consent
- Don't ignore GDPR requirements

## Testing & Quality

### Before Committing
- [ ] Code compiles (no TypeScript errors)
- [ ] All imports resolve
- [ ] No console errors in browser
- [ ] Mobile responsive (test on small screen)
- [ ] Loading states work
- [ ] Error handling implemented
- [ ] Spanish text for UI, English for code

### Before Deploying
- [ ] Build succeeds (`npm run build`)
- [ ] Environment variables set
- [ ] Firebase rules updated
- [ ] No hardcoded secrets
- [ ] Meta tags correct
- [ ] Analytics tracking works
- [ ] Payment flow tested (Stripe test mode)

## Resources

### Documentation
- Tech Stack Details: `.github/skills/saas-stack/SKILL.md`
- Firebase Patterns: `.github/skills/firebase/SKILL.md`
- Stripe Integration: `.github/skills/stripe-payments/SKILL.md`
- Email Setup: `.github/skills/email-notifications/SKILL.md`

### Design Inspiration
- Modern SaaS: Linear, Vercel, Stripe, Notion
- AI Transparency: Perplexity AI, ChatGPT, Grammarly
- Spanish SaaS: Factorial, Holded

### Libraries & Tools
- React: https://react.dev/
- TypeScript: https://www.typescriptlang.org/
- Tailwind: https://tailwindcss.com/
- Shadcn UI: https://ui.shadcn.com/
- Firebase: https://firebase.google.com/docs
- Stripe: https://stripe.com/docs

## Project-Specific Adaptations

**For entiendetunomina (Payroll Analysis)**
- Focus: Spanish payroll complexity, tax calculations
- AI Usage: PDF analysis, error detection, tax optimization suggestions
- Target: Employees reviewing their payslips, HR managers
- Tone: Professional, trustworthy, educational

**For valuebetanalytics (Sports Betting Analysis)**
- Focus: Value bet detection, odds comparison, ROI tracking
- AI Usage: Probability calculations, pattern detection, recommendations
- Target: Sports bettors, analytics enthusiasts
- Tone: Data-driven, analytical, confident

---

**Remember**: We build modern, AI-transparent SaaS products that users trust. Show the AI working, use contemporary design, and always prioritize user experience over technical complexity.
