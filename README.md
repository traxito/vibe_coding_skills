This repository contains a reusable SaaS template designed to work hand-in-hand with GitHub Copilot (and similar AI agents) using project-wide instructions and modular skills. It standardizes the tech stack, architecture, and best practices so every new app (like entiendetunomina or valuebetanalytics) starts from a solid, AI-friendly base.

Overview
Frontend: React 18, TypeScript, Vite, Tailwind CSS, Shadcn UI, React Router, React Hook Form + Zod.

Backend: Firebase (Auth, Firestore, Functions, Hosting, Storage).

Integrations: Stripe (subscriptions), Gmail SMTP (emails).

AI-First UX: Modern UI, visible AI usage (“AI is analyzing…”, badges, confidence scores).

Developer Experience: Central instructions + per-domain skills to guide AI code generation.

The goal is that any AI assistant working in this repo automatically:

Uses the same stack and patterns.

Follows the same architecture and naming conventions.

Implements Stripe, Firebase, Email, and AI UX in a consistent way.

Repository Structure
text
.github/
  copilot-instructions.md    # Global project & design instructions for AI assistants
  skills/
    saas-stack/
      SKILL.md               # Tech stack, architecture, project structure, patterns
    firebase/
      SKILL.md               # Firebase setup, Auth, Firestore, Functions, Hosting, Storage
    stripe-payments/
      SKILL.md               # Stripe subscriptions, webhooks, portal, one-time payments
    email-notifications/
      SKILL.md               # Gmail SMTP, templates, notifications, verification, deliverability

src/
  ...                        # Frontend app (React + TS + Vite + Tailwind)

functions/
  ...                        # Firebase Cloud Functions (Node + TypeScript)
The global instructions file sets the high-level rules (tone, design, AI visibility, languages, GDPR, etc.).

Each skill file focuses on one technical area and is scoped by file types (e.g. ts, tsx, etc.).

How the AI Guidance Works
1. Global Instructions
.github/copilot-instructions.md explains to AI assistants:

Business context (B2C SaaS, Spanish users, freemium model).

Design principles (modern, clean, AI-visible UI).

Tone:

Code & comments: English.

UI text: Spanish, “cercano pero riguroso” (friendly but rigorous).

AI visibility:

Badges like “Powered by AI”.

“La IA está analizando tu documento…” loading states.

Confidence indicators and explanations of AI decisions.

Non-functional requirements:

GDPR, security, SEO, performance targets.

App-specific notes for projects like payroll analysis or value-betting analytics.

When you write or edit code, the assistant should read this file as the “source of truth” for style, tone, and UX expectations.

2. Skills (Technical Domains)
Under .github/skills/, each directory contains one SKILL.md describing how to implement a specific area:

saas-stack/SKILL.md

Standard project structure (components, hooks, services, pages, etc.).

Recommended package.json, tsconfig, vite.config, tailwind.config, firebase.json.

Patterns for components, hooks, services, routing, error handling.

Naming conventions and TypeScript best practices.

firebase/SKILL.md

Firebase initialization, emulators, Auth flows, Firestore patterns.

Security rules, indexes, Cloud Functions structure.

Deployment commands and troubleshooting.

stripe-payments/SKILL.md

Subscription creation (Checkout), trial periods, upgrades/downgrades, cancellations.

Webhooks for all key events and Firestore sync.

Customer Portal, one-time payments, security checks.

email-notifications/SKILL.md

Gmail SMTP with app passwords, Nodemailer service.

HTML templates (welcome, reset, payment, alerts).

Email verification flow, unsubscribe preferences, SPF/DKIM/DMARC notes.

In-app notifications (toasts, notification bell).

When you edit files that match the applyTo patterns in these skills (e.g. **/*.ts,tsx,js,jsx), the assistant should use the corresponding skill as the detailed “how-to”.

How to Use This Template
1. Clone and Install
bash
git clone <your-repo-url> my-saas-app
cd my-saas-app

# Install frontend dependencies
npm install

# Install Cloud Functions dependencies
cd functions
npm install
cd ..
2. Configure Environment Variables
Create .env.local for the frontend:

bash
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...

VITE_STRIPE_PUBLISHABLE_KEY=pk_test_...
Create functions/.env (or your preferred env loading strategy) for the backend:

bash
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

GMAIL_USER=your-email@gmail.com
GMAIL_APP_PASSWORD=your-app-password
GMAIL_FROM_NAME=YourApp
GMAIL_FROM_EMAIL=noreply@yourapp.com
APP_NAME=YourApp
APP_URL=https://yourapp.com
SUPPORT_EMAIL=support@yourapp.com
3. Initialize Firebase
bash
# Login and initialize (if not already)
npx firebase login
npx firebase init
Ensure firebase.json, firestore.rules, firestore.indexes.json, storage.rules follow the patterns defined in the stack and Firebase skills.

4. Run Locally
bash
# Frontend (Vite)
npm run dev

# In another terminal: Firebase emulators
npx firebase emulators:start
Use the emulator suite for Firestore, Auth, and Functions during development to avoid unnecessary costs.

5. Deploy
bash
# Build frontend
npm run build

# Deploy hosting + functions (PowerShell: always use npx)
npx firebase deploy

# Deploy only hosting
npx firebase deploy --only hosting

# Deploy only functions
npx firebase deploy --only functions
How to Work with AI Assistants
1. For Global Behavior
When prompting your assistant, you can say things like:

“Follow the project instructions in .github/copilot-instructions.md.”

“Use the standard SaaS stack defined in the stack skill.”

“Keep the UI text in Spanish with a friendly but rigorous tone and make AI usage visible.”

This helps align the assistant with the overall design and product strategy.

2. For Specific Features
Reference the relevant skills:

Firebase-related code:
“Implement this using the Firebase skill (Auth + Firestore patterns).”

Payments/Stripe:
“Use the Stripe & Payments skill for subscriptions and webhooks.”

Email & notifications:
“Follow the Email & Notifications skill with Gmail SMTP.”

Because the skills define concrete patterns, the assistant should produce code that plugs into the existing architecture without you repeating instructions.

Conventions and Expectations
Code: English (variables, functions, comments, commit messages).

UI copy: Spanish, “cercano pero riguroso”.

Design: Modern, clean, AI-visible, mobile-first, accessible.

Security & Privacy: GDPR-aware, minimal data collection, secure defaults.

Architecture: Follow the structure, naming conventions, and patterns defined in the stack skill.

If you extend the template (new integrations, domains, or patterns), add or update a skill under .github/skills/ and, if it affects global behavior, update .github/copilot-instructions.md accordingly.
