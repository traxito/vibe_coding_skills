# SaaS Template – AI‑First + Copilot Ready

Modern SaaS template designed to work **with AI assistants** (GitHub Copilot, Claude, etc.), using a consistent stack, architecture and a system of project instructions + skills.

## Features

- **AI‑first UX**: visible AI usage (“La IA está analizando…”, badges, confidence, explanations).
- **Unified stack**: React 18, TypeScript, Vite, Tailwind, Firebase, Stripe, Gmail SMTP.
- **AI guidance**:
  - Global project instructions (`.github/copilot-instructions.md`).
  - Modular skills (`.github/skills/*/SKILL.md`) for Firebase, Stripe, Email, Stack.
- **Production‑ready** patterns: auth, subscriptions, emails, dashboards, protected routes.
- **Reusable** for new apps (e.g. payroll, sports analytics, other SaaS ideas).

---

## Tech Stack

| Layer        | Technologies                                   |
|--------------|-----------------------------------------------|
| Frontend     | React 18, TypeScript, Vite, React Router      |
| UI           | Tailwind CSS, Shadcn UI, Lucide Icons, Sonner |
| Backend      | Firebase (Auth, Firestore, Functions, Hosting, Storage) |
| Payments     | Stripe (subscriptions + webhooks)             |
| Email        | Gmail SMTP + Nodemailer                       |
| DNS / Domain | Cloudflare                                    |

---

## AI & Instructions System
# SaaS Template — AI‑First & Copilot Ready

![CI](https://img.shields.io/badge/build-passing-brightgreen) ![License: MIT](https://img.shields.io/badge/license-MIT-blue) ![Stack](https://img.shields.io/badge/stack-React%20%7C%20TS%20%7C%20Tailwind-blueviolet)

Modern, production-ready SaaS starter focused on AI-first UX and clear AI assistant instructions. Includes a set of reusable "skills" and project guidelines so assistants (like GitHub Copilot) can act consistently across the codebase.

—

## Table of Contents

- [What is this](#what-is-this)
- [Key features](#key-features)
- [Tech stack](#tech-stack)
- [Quick start](#quick-start)
- [AI & instructions system](#ai--instructions-system)
- [Contributing](#contributing)
- [License](#license)

## What is this

A starter template for modern B2C SaaS products with an emphasis on:

- AI-visible UX (processing states, confidence, explanations)
- Clear project-level instructions for assistants (`.github/copilot-instructions.md`)
- Reusable skills to keep implementation consistent across features

Ideal for Spanish-speaking end users (UI text) while keeping code and comments in English.

## Key features

- AI-first UX patterns and badges
- Auth, subscriptions, dashboards, protected routes
- Firebase (Auth, Firestore, Functions, Hosting) integration
- Stripe subscriptions + webhooks support
- Email via Gmail SMTP + Nodemailer
- Mobile-first, accessible UI built with Tailwind and Shadcn UI

## Tech stack

- Frontend: React 18, TypeScript, Vite
- UI: Tailwind CSS, Shadcn UI, Lucide Icons
- Backend: Firebase (Auth, Firestore, Functions, Hosting)
- Payments: Stripe (subscriptions + webhooks)
- Email: Gmail SMTP + Nodemailer

## Quick start

1. Clone the repo

```bash
git clone <your-repo-url> my-saas-app
cd my-saas-app
```

2. Install dependencies

```bash
npm install
cd functions && npm install && cd ..
```

3. Add environment variables

- Frontend: create `.env.local` with Vite-prefixed vars (VITE_FIREBASE_*, VITE_STRIPE_PUBLISHABLE_KEY)
- Functions / backend: set `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `GMAIL_*` creds

4. Run locally

```bash
npm run dev            # frontend (Vite)
npx firebase emulators:start   # local emulators for Auth/Firestore/Functions
```

5. Deploy

```bash
npm run build
npx firebase deploy
```

## AI & instructions system

This repo contains project-level instructions and modular skills so assistants know project constraints, tone, and architecture. Key files:

- `.github/copilot-instructions.md` — business context, tone, design rules, language rules
- `.github/skills/*/SKILL.md` — domain skills (firebase, stripe, email, saas-stack)

Guidelines (short):

- Code & comments: English
- UI text: Spanish (tone: "cercano pero riguroso")
- Always show AI processing states and confidence where appropriate

When prompting an assistant, include: "Follow .github/copilot-instructions.md and the relevant skill for this feature."

## Contributing

- Follow TypeScript strict mode and Tailwind for styles
- Write UI text in Spanish; keep code/comments in English
- Open PRs against `main`; add small, atomic commits and a clear description

If you want help bootstrapping a new feature, mention the skill(s) the assistant should follow (e.g., `firebase`, `stripe-payments`).

## License

This project is available under the MIT License.

---

Updated to improve clarity, structure and quick onboarding for contributors and AI assistants.
