---
name: Email & Notifications
description: Implement email sending, notifications, and user communication following best practices. Includes Gmail SMTP integration with Firebase, transactional email templates, in-app notifications, email verification flows, and deliverability configuration. Use when implementing user communication, alerts, or notification systems.
applyTo: "**/*.{ts,tsx,js,jsx,html}"
---

# Email & Notifications Best Practices Skill

Complete guide for implementing email and notification systems with Firebase and Gmail SMTP.

## When to Use This Skill

- Setting up Gmail SMTP with Firebase Cloud Functions
- Sending transactional emails (welcome, password reset, invoices)
- Creating email templates
- Implementing in-app notifications (toasts, alerts)
- Setting up email verification flows
- Managing unsubscribe functionality
- Configuring email deliverability (SPF, DKIM, DMARC)
- Sending alerts and system notifications

---

## Gmail SMTP Setup with Firebase

### 1. Create Gmail App Password

**Important: Use Gmail App Password, NOT your regular password**

1. Go to Google Account settings: https://myaccount.google.com/
2. Security → 2-Step Verification (enable if not already)
3. Security → App passwords
4. Generate app password for "Mail"
5. Save the 16-character password

### 2. Environment Variables

```bash
# .env (Backend/Cloud Functions)
GMAIL_USER=your-email@gmail.com
GMAIL_APP_PASSWORD=abcd efgh ijkl mnop  # 16-character app password
GMAIL_FROM_NAME=YourApp
GMAIL_FROM_EMAIL=noreply@yourapp.com  # Display name (actual sender is GMAIL_USER)

# App configuration
APP_NAME=YourApp
APP_URL=https://yourapp.com
SUPPORT_EMAIL=support@yourapp.com
```

### 3. Install Nodemailer

```bash
# In functions directory
cd functions
npm install nodemailer
npm install --save-dev @types/nodemailer
```

---

## Email Service Configuration

### Email Service Utility

```typescript
// functions/src/services/emailService.ts
import * as nodemailer from 'nodemailer';
import * as admin from 'firebase-admin';

// Email configuration
const EMAIL_CONFIG = {
  user: process.env.GMAIL_USER!,
  password: process.env.GMAIL_APP_PASSWORD!,
  fromName: process.env.GMAIL_FROM_NAME || 'YourApp',
  fromEmail: process.env.GMAIL_FROM_EMAIL || process.env.GMAIL_USER!,
};

// Create transporter (singleton)
let transporter: nodemailer.Transporter;

function getTransporter(): nodemailer.Transporter {
  if (!transporter) {
    transporter = nodemailer.createTransport({
      service: 'gmail',
      auth: {
        user: EMAIL_CONFIG.user,
        pass: EMAIL_CONFIG.password,
      },
    });
  }
  return transporter;
}

// Email interface
export interface EmailOptions {
  to: string | string[];
  subject: string;
  html: string;
  text?: string;
  attachments?: Array<{
    filename: string;
    content?: string | Buffer;
    path?: string;
  }>;
}

// Send email function
export async function sendEmail(options: EmailOptions): Promise<boolean> {
  try {
    const transporter = getTransporter();

    const mailOptions: nodemailer.SendMailOptions = {
      from: `"${EMAIL_CONFIG.fromName}" <${EMAIL_CONFIG.fromEmail}>`,
      to: Array.isArray(options.to) ? options.to.join(', ') : options.to,
      subject: options.subject,
      html: options.html,
      text: options.text || stripHtml(options.html),
      attachments: options.attachments,
    };

    const info = await transporter.sendMail(mailOptions);

    console.log('Email sent successfully:', {
      messageId: info.messageId,
      to: options.to,
      subject: options.subject,
    });

    // Log email in Firestore (optional but recommended)
    await logEmail({
      to: options.to,
      subject: options.subject,
      status: 'sent',
      messageId: info.messageId,
      sentAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return true;
  } catch (error) {
    console.error('Error sending email:', error);

    // Log failed email
    await logEmail({
      to: options.to,
      subject: options.subject,
      status: 'failed',
      error: (error as Error).message,
      sentAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    throw error;
  }
}

// Log email to Firestore
async function logEmail(data: any): Promise<void> {
  try {
    await admin.firestore().collection('email_logs').add(data);
  } catch (error) {
    console.error('Error logging email:', error);
  }
}

// Strip HTML for plain text fallback
function stripHtml(html: string): string {
  return html.replace(/<[^>]*>/g, '');
}

// Verify transporter configuration
export async function verifyEmailConfig(): Promise<boolean> {
  try {
    const transporter = getTransporter();
    await transporter.verify();
    console.log('Email configuration verified successfully');
    return true;
  } catch (error) {
    console.error('Email configuration verification failed:', error);
    return false;
  }
}
```

---

## Email Templates

### Base Email Template (HTML)

```typescript
// functions/src/templates/baseTemplate.ts

interface BaseTemplateParams {
  title: string;
  preheader?: string;
  content: string;
  footerText?: string;
}

export function baseEmailTemplate(params: BaseTemplateParams): string {
  const appName = process.env.APP_NAME || 'YourApp';
  const appUrl = process.env.APP_URL || 'https://yourapp.com';
  const currentYear = new Date().getFullYear();

  return `
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>${params.title}</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
      background-color: #f4f4f4;
      color: #333333;
    }
    .email-container {
      max-width: 600px;
      margin: 0 auto;
      background-color: #ffffff;
    }
    .header {
      background-color: #000000;
      padding: 20px;
      text-align: center;
    }
    .header h1 {
      color: #ffffff;
      margin: 0;
      font-size: 24px;
    }
    .content {
      padding: 40px 20px;
      line-height: 1.6;
    }
    .button {
      display: inline-block;
      padding: 12px 30px;
      background-color: #000000;
      color: #ffffff !important;
      text-decoration: none;
      border-radius: 5px;
      margin: 20px 0;
      font-weight: 600;
    }
    .footer {
      background-color: #f8f8f8;
      padding: 20px;
      text-align: center;
      font-size: 12px;
      color: #666666;
    }
    .footer a {
      color: #666666;
      text-decoration: underline;
    }
    @media only screen and (max-width: 600px) {
      .content {
        padding: 20px 15px;
      }
    }
  </style>
</head>
<body>
  ${params.preheader ? `<div style="display:none;font-size:1px;color:#ffffff;line-height:1px;max-height:0px;max-width:0px;opacity:0;overflow:hidden;">${params.preheader}</div>` : ''}

  <div class="email-container">
    <div class="header">
      <h1>${appName}</h1>
    </div>

    <div class="content">
      ${params.content}
    </div>

    <div class="footer">
      <p>${params.footerText || `© ${currentYear} ${appName}. All rights reserved.`}</p>
      <p>
        <a href="${appUrl}/unsubscribe">Unsubscribe</a> | 
        <a href="${appUrl}/privacy">Privacy Policy</a> | 
        <a href="${appUrl}/contact">Contact Us</a>
      </p>
    </div>
  </div>
</body>
</html>
  `.trim();
}
```

### Welcome Email Template

```typescript
// functions/src/templates/welcomeEmail.ts
import { baseEmailTemplate } from './baseTemplate';

export function welcomeEmailTemplate(userName: string, verificationUrl?: string): string {
  const content = `
    <h2>Welcome to ${process.env.APP_NAME || 'YourApp'}! 🎉</h2>
    <p>Hi ${userName},</p>
    <p>Thank you for signing up! We're excited to have you on board.</p>

    ${verificationUrl ? `
      <p>To get started, please verify your email address:</p>
      <p style="text-align: center;">
        <a href="${verificationUrl}" class="button">Verify Email Address</a>
      </p>
      <p style="font-size: 12px; color: #666;">
        If the button doesn't work, copy and paste this link into your browser:<br>
        ${verificationUrl}
      </p>
    ` : `
      <p>You're all set! You can now access all features of your account.</p>
      <p style="text-align: center;">
        <a href="${process.env.APP_URL}/dashboard" class="button">Go to Dashboard</a>
      </p>
    `}

    <p>If you have any questions, feel free to reply to this email or contact our support team.</p>
    <p>Best regards,<br>The ${process.env.APP_NAME} Team</p>
  `;

  return baseEmailTemplate({
    title: 'Welcome to YourApp',
    preheader: 'Get started with your new account',
    content,
  });
}
```

### Password Reset Email Template

```typescript
// functions/src/templates/passwordResetEmail.ts
import { baseEmailTemplate } from './baseTemplate';

export function passwordResetEmailTemplate(userName: string, resetUrl: string): string {
  const content = `
    <h2>Password Reset Request</h2>
    <p>Hi ${userName},</p>
    <p>We received a request to reset your password. Click the button below to create a new password:</p>

    <p style="text-align: center;">
      <a href="${resetUrl}" class="button">Reset Password</a>
    </p>

    <p style="font-size: 12px; color: #666;">
      If the button doesn't work, copy and paste this link into your browser:<br>
      ${resetUrl}
    </p>

    <p><strong>This link will expire in 1 hour.</strong></p>

    <p>If you didn't request a password reset, you can safely ignore this email. Your password will not be changed.</p>

    <p>Best regards,<br>The ${process.env.APP_NAME} Team</p>
  `;

  return baseEmailTemplate({
    title: 'Reset Your Password',
    preheader: 'Click to reset your password',
    content,
  });
}
```

### Payment Confirmation Email Template

```typescript
// functions/src/templates/paymentConfirmationEmail.ts
import { baseEmailTemplate } from './baseTemplate';

interface PaymentDetails {
  userName: string;
  planName: string;
  amount: number;
  currency: string;
  invoiceUrl: string;
  nextBillingDate: string;
}

export function paymentConfirmationEmailTemplate(details: PaymentDetails): string {
  const content = `
    <h2>Payment Confirmation</h2>
    <p>Hi ${details.userName},</p>
    <p>Thank you for your payment! Your subscription has been confirmed.</p>

    <div style="background-color: #f8f8f8; padding: 20px; border-radius: 5px; margin: 20px 0;">
      <h3 style="margin-top: 0;">Payment Details</h3>
      <p style="margin: 5px 0;"><strong>Plan:</strong> ${details.planName}</p>
      <p style="margin: 5px 0;"><strong>Amount:</strong> ${details.currency.toUpperCase()} ${details.amount.toFixed(2)}</p>
      <p style="margin: 5px 0;"><strong>Next Billing Date:</strong> ${details.nextBillingDate}</p>
    </div>

    <p style="text-align: center;">
      <a href="${details.invoiceUrl}" class="button">View Invoice</a>
    </p>

    <p>If you have any questions about your subscription, please don't hesitate to contact us.</p>

    <p>Best regards,<br>The ${process.env.APP_NAME} Team</p>
  `;

  return baseEmailTemplate({
    title: 'Payment Confirmation',
    preheader: 'Your payment was successful',
    content,
  });
}
```

### Subscription Cancelled Email Template

```typescript
// functions/src/templates/subscriptionCancelledEmail.ts
import { baseEmailTemplate } from './baseTemplate';

export function subscriptionCancelledEmailTemplate(
  userName: string,
  planName: string,
  endDate: string
): string {
  const content = `
    <h2>Subscription Cancelled</h2>
    <p>Hi ${userName},</p>
    <p>We're sorry to see you go. Your ${planName} subscription has been cancelled.</p>

    <div style="background-color: #fff3cd; padding: 20px; border-left: 4px solid #ffc107; margin: 20px 0;">
      <p style="margin: 0;"><strong>Important:</strong> You'll continue to have access to your ${planName} features until <strong>${endDate}</strong>.</p>
    </div>

    <p>After this date, your account will be downgraded to the Free plan.</p>

    <p>We'd love to hear your feedback. What could we have done better?</p>

    <p style="text-align: center;">
      <a href="${process.env.APP_URL}/feedback" class="button">Share Feedback</a>
    </p>

    <p>If you change your mind, you can reactivate your subscription anytime from your dashboard.</p>

    <p>Best regards,<br>The ${process.env.APP_NAME} Team</p>
  `;

  return baseEmailTemplate({
    title: 'Subscription Cancelled',
    preheader: 'Your subscription has been cancelled',
    content,
  });
}
```

### Custom Alert Email Template

```typescript
// functions/src/templates/alertEmail.ts
import { baseEmailTemplate } from './baseTemplate';

interface AlertDetails {
  userName: string;
  alertTitle: string;
  alertMessage: string;
  actionUrl?: string;
  actionText?: string;
}

export function alertEmailTemplate(details: AlertDetails): string {
  const content = `
    <h2>${details.alertTitle}</h2>
    <p>Hi ${details.userName},</p>
    <p>${details.alertMessage}</p>

    ${details.actionUrl && details.actionText ? `
      <p style="text-align: center;">
        <a href="${details.actionUrl}" class="button">${details.actionText}</a>
      </p>
    ` : ''}

    <p>This is an automated notification. If you have questions, please contact our support team.</p>

    <p>Best regards,<br>The ${process.env.APP_NAME} Team</p>
  `;

  return baseEmailTemplate({
    title: details.alertTitle,
    preheader: details.alertMessage.substring(0, 100),
    content,
  });
}
```

---

## Cloud Functions for Email Sending

### Welcome Email Function

```typescript
// functions/src/emails/sendWelcomeEmail.ts
import * as functions from 'firebase-functions';
import { sendEmail } from '../services/emailService';
import { welcomeEmailTemplate } from '../templates/welcomeEmail';

export const sendWelcomeEmail = functions.auth.user().onCreate(async (user) => {
  try {
    const displayName = user.displayName || 'there';
    const email = user.email;

    if (!email) {
      console.log('User has no email, skipping welcome email');
      return;
    }

    // Generate email verification link (if needed)
    // const verificationUrl = await admin.auth().generateEmailVerificationLink(email);

    const html = welcomeEmailTemplate(displayName);

    await sendEmail({
      to: email,
      subject: `Welcome to ${process.env.APP_NAME || 'YourApp'}!`,
      html,
    });

    console.log(`Welcome email sent to ${email}`);
  } catch (error) {
    console.error('Error sending welcome email:', error);
  }
});
```

### Password Reset Email Function

```typescript
// functions/src/emails/sendPasswordResetEmail.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import { sendEmail } from '../services/emailService';
import { passwordResetEmailTemplate } from '../templates/passwordResetEmail';

interface PasswordResetData {
  email: string;
}

export const sendPasswordResetEmail = functions.https.onCall(
  async (data: PasswordResetData, context) => {
    const { email } = data;

    try {
      // Verify email exists
      const userRecord = await admin.auth().getUserByEmail(email);

      // Generate password reset link
      const resetUrl = await admin.auth().generatePasswordResetLink(email);

      const html = passwordResetEmailTemplate(
        userRecord.displayName || 'User',
        resetUrl
      );

      await sendEmail({
        to: email,
        subject: 'Reset Your Password',
        html,
      });

      return { success: true, message: 'Password reset email sent' };
    } catch (error: any) {
      console.error('Error sending password reset email:', error);

      // Don't reveal if email exists or not (security best practice)
      return { success: true, message: 'If the email exists, a reset link has been sent' };
    }
  }
);
```

### Payment Confirmation Email (Triggered by Stripe Webhook)

```typescript
// functions/src/emails/sendPaymentConfirmation.ts
import { sendEmail } from '../services/emailService';
import { paymentConfirmationEmailTemplate } from '../templates/paymentConfirmationEmail';
import * as admin from 'firebase-admin';

export async function sendPaymentConfirmationEmail(
  userId: string,
  invoiceData: {
    planName: string;
    amount: number;
    currency: string;
    invoiceUrl: string;
    nextBillingDate: string;
  }
): Promise<void> {
  try {
    const userDoc = await admin.firestore().collection('users').doc(userId).get();
    const userData = userDoc.data();

    if (!userData?.email) {
      throw new Error('User email not found');
    }

    const html = paymentConfirmationEmailTemplate({
      userName: userData.displayName || 'User',
      ...invoiceData,
    });

    await sendEmail({
      to: userData.email,
      subject: 'Payment Confirmation - Invoice Received',
      html,
    });

    console.log(`Payment confirmation sent to ${userData.email}`);
  } catch (error) {
    console.error('Error sending payment confirmation:', error);
    throw error;
  }
}

// Call this from your Stripe webhook handler
// In functions/src/stripe/webhooks.ts:
// import { sendPaymentConfirmationEmail } from '../emails/sendPaymentConfirmation';
//
// case 'invoice.payment_succeeded':
//   await sendPaymentConfirmationEmail(userId, {
//     planName: 'Pro',
//     amount: invoice.amount_paid / 100,
//     currency: invoice.currency,
//     invoiceUrl: invoice.hosted_invoice_url,
//     nextBillingDate: new Date(subscription.current_period_end * 1000).toLocaleDateString(),
//   });
```

### Custom Alert Email Function

```typescript
// functions/src/emails/sendAlertEmail.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import { sendEmail } from '../services/emailService';
import { alertEmailTemplate } from '../templates/alertEmail';

interface AlertData {
  userId: string;
  alertTitle: string;
  alertMessage: string;
  actionUrl?: string;
  actionText?: string;
}

export const sendAlertEmail = functions.https.onCall(
  async (data: AlertData, context) => {
    // Verify authentication
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
    }

    const { userId, alertTitle, alertMessage, actionUrl, actionText } = data;

    try {
      const userDoc = await admin.firestore().collection('users').doc(userId).get();
      const userData = userDoc.data();

      if (!userData?.email) {
        throw new functions.https.HttpsError('not-found', 'User email not found');
      }

      const html = alertEmailTemplate({
        userName: userData.displayName || 'User',
        alertTitle,
        alertMessage,
        actionUrl,
        actionText,
      });

      await sendEmail({
        to: userData.email,
        subject: alertTitle,
        html,
      });

      return { success: true, message: 'Alert email sent' };
    } catch (error: any) {
      console.error('Error sending alert email:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

---

## In-App Notifications

### Toast Notifications (React)

```tsx
// components/Toast.tsx
import { useEffect } from 'react';
import { createPortal } from 'react-dom';

interface ToastProps {
  message: string;
  type?: 'success' | 'error' | 'info' | 'warning';
  duration?: number;
  onClose: () => void;
}

export default function Toast({ message, type = 'info', duration = 3000, onClose }: ToastProps) {
  useEffect(() => {
    const timer = setTimeout(onClose, duration);
    return () => clearTimeout(timer);
  }, [duration, onClose]);

  const styles = {
    success: 'bg-green-500',
    error: 'bg-red-500',
    info: 'bg-blue-500',
    warning: 'bg-yellow-500',
  };

  return createPortal(
    <div className="fixed bottom-4 right-4 z-50 animate-slide-up">
      <div className={`${styles[type]} text-white px-6 py-4 rounded-lg shadow-lg flex items-center gap-3`}>
        <span>{message}</span>
        <button onClick={onClose} className="ml-4 text-white hover:text-gray-200">
          ✕
        </button>
      </div>
    </div>,
    document.body
  );
}
```

### Toast Hook (React)

```tsx
// hooks/useToast.tsx
import { useState, useCallback } from 'react';
import Toast from '@/components/Toast';

interface ToastOptions {
  message: string;
  type?: 'success' | 'error' | 'info' | 'warning';
  duration?: number;
}

export function useToast() {
  const [toast, setToast] = useState<ToastOptions | null>(null);

  const showToast = useCallback((options: ToastOptions) => {
    setToast(options);
  }, []);

  const hideToast = useCallback(() => {
    setToast(null);
  }, []);

  const ToastComponent = toast ? (
    <Toast
      message={toast.message}
      type={toast.type}
      duration={toast.duration}
      onClose={hideToast}
    />
  ) : null;

  return {
    showToast,
    hideToast,
    ToastComponent,
  };
}

// Usage:
// const { showToast, ToastComponent } = useToast();
// 
// showToast({ message: 'Email sent successfully!', type: 'success' });
// 
// return (
//   <div>
//     {ToastComponent}
//     <button onClick={() => showToast({ message: 'Hello!', type: 'info' })}>
//       Show Toast
//     </button>
//   </div>
// );
```

### Notification Bell (React)

```tsx
// components/NotificationBell.tsx
import { useState, useEffect } from 'react';
import { collection, query, where, onSnapshot, updateDoc, doc } from 'firebase/firestore';
import { db } from '@/lib/firebase';
import { useAuth } from '@/contexts/AuthContext';

interface Notification {
  id: string;
  title: string;
  message: string;
  read: boolean;
  createdAt: Date;
  actionUrl?: string;
}

export default function NotificationBell() {
  const { user } = useAuth();
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [showDropdown, setShowDropdown] = useState(false);
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    if (!user) return;

    // Subscribe to notifications
    const q = query(
      collection(db, 'notifications'),
      where('userId', '==', user.uid),
      where('read', '==', false)
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const notifs = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data(),
        createdAt: doc.data().createdAt?.toDate(),
      })) as Notification[];

      setNotifications(notifs);
      setUnreadCount(notifs.length);
    });

    return () => unsubscribe();
  }, [user]);

  const markAsRead = async (notificationId: string) => {
    await updateDoc(doc(db, 'notifications', notificationId), {
      read: true,
      readAt: new Date(),
    });
  };

  return (
    <div className="relative">
      <button
        onClick={() => setShowDropdown(!showDropdown)}
        className="relative p-2 rounded-full hover:bg-gray-100"
      >
        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
        </svg>

        {unreadCount > 0 && (
          <span className="absolute top-0 right-0 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">
            {unreadCount}
          </span>
        )}
      </button>

      {showDropdown && (
        <div className="absolute right-0 mt-2 w-80 bg-white rounded-lg shadow-lg border z-50">
          <div className="p-4 border-b">
            <h3 className="font-semibold">Notifications</h3>
          </div>

          <div className="max-h-96 overflow-y-auto">
            {notifications.length === 0 ? (
              <p className="p-4 text-gray-500 text-center">No new notifications</p>
            ) : (
              notifications.map((notif) => (
                <div
                  key={notif.id}
                  className="p-4 border-b hover:bg-gray-50 cursor-pointer"
                  onClick={() => {
                    markAsRead(notif.id);
                    if (notif.actionUrl) {
                      window.location.href = notif.actionUrl;
                    }
                  }}
                >
                  <h4 className="font-semibold text-sm">{notif.title}</h4>
                  <p className="text-sm text-gray-600 mt-1">{notif.message}</p>
                  <p className="text-xs text-gray-400 mt-2">
                    {notif.createdAt?.toLocaleString()}
                  </p>
                </div>
              ))
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Email Verification Flow

### Email Verification Function

```typescript
// functions/src/emails/sendVerificationEmail.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import { sendEmail } from '../services/emailService';
import { welcomeEmailTemplate } from '../templates/welcomeEmail';

interface VerificationData {
  email: string;
}

export const sendVerificationEmail = functions.https.onCall(
  async (data: VerificationData, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
    }

    const { email } = data;

    try {
      // Generate email verification link
      const verificationUrl = await admin.auth().generateEmailVerificationLink(email, {
        url: `${process.env.APP_URL}/dashboard`,
      });

      const userRecord = await admin.auth().getUserByEmail(email);

      const html = welcomeEmailTemplate(
        userRecord.displayName || 'User',
        verificationUrl
      );

      await sendEmail({
        to: email,
        subject: 'Verify Your Email Address',
        html,
      });

      return { success: true, message: 'Verification email sent' };
    } catch (error: any) {
      console.error('Error sending verification email:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

### Client-side Verification Check (React)

```tsx
// components/EmailVerificationBanner.tsx
import { useState } from 'react';
import { useAuth } from '@/contexts/AuthContext';
import { httpsCallable } from 'firebase/functions';
import { functions } from '@/lib/firebase';

export default function EmailVerificationBanner() {
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);
  const [sent, setSent] = useState(false);

  if (!user || user.emailVerified) {
    return null;
  }

  const handleResendVerification = async () => {
    setLoading(true);

    try {
      const sendVerificationEmail = httpsCallable(functions, 'sendVerificationEmail');
      await sendVerificationEmail({ email: user.email });

      setSent(true);
      setTimeout(() => setSent(false), 5000);
    } catch (error) {
      console.error('Error resending verification:', error);
      alert('Failed to send verification email');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="bg-yellow-50 border-l-4 border-yellow-400 p-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center">
          <svg className="w-5 h-5 text-yellow-400 mr-3" fill="currentColor" viewBox="0 0 20 20">
            <path fillRule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clipRule="evenodd" />
          </svg>
          <p className="text-sm text-yellow-700">
            Please verify your email address to access all features.
          </p>
        </div>

        <button
          onClick={handleResendVerification}
          disabled={loading || sent}
          className="ml-4 px-4 py-2 bg-yellow-500 text-white rounded hover:bg-yellow-600 disabled:opacity-50 text-sm"
        >
          {sent ? 'Email Sent!' : loading ? 'Sending...' : 'Resend Verification'}
        </button>
      </div>
    </div>
  );
}
```

---

## Unsubscribe Management

### Unsubscribe Preferences (Firestore)

```typescript
// functions/src/emails/unsubscribe.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

interface UnsubscribeData {
  userId: string;
  emailType: 'marketing' | 'product_updates' | 'transactional';
}

export const updateEmailPreferences = functions.https.onCall(
  async (data: UnsubscribeData, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
    }

    const { userId, emailType } = data;

    try {
      await admin.firestore().collection('users').doc(userId).update({
        [`emailPreferences.${emailType}`]: false,
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      });

      return { success: true, message: 'Email preferences updated' };
    } catch (error: any) {
      console.error('Error updating email preferences:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);

// Check preferences before sending
export async function shouldSendEmail(
  userId: string,
  emailType: 'marketing' | 'product_updates' | 'transactional'
): Promise<boolean> {
  const userDoc = await admin.firestore().collection('users').doc(userId).get();
  const preferences = userDoc.data()?.emailPreferences || {};

  // Transactional emails always sent (required by law)
  if (emailType === 'transactional') {
    return true;
  }

  // Check user preference (default to true if not set)
  return preferences[emailType] !== false;
}
```

---

## Email Deliverability Configuration

### SPF Record (DNS)

Add to your domain's DNS records:

```
Type: TXT
Name: @
Value: v=spf1 include:_spf.google.com ~all
TTL: 3600
```

### DKIM Configuration

1. Go to Gmail Admin Console
2. Apps → Google Workspace → Gmail → Authenticate email
3. Generate new DKIM key
4. Add the provided TXT record to your DNS

```
Type: TXT
Name: google._domainkey
Value: [provided by Google]
TTL: 3600
```

### DMARC Record (DNS)

```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com
TTL: 3600
```

### Custom Domain Setup (Optional)

If using custom domain with Gmail:

1. Verify domain ownership in Google Admin Console
2. Set up MX records
3. Configure SPF, DKIM, DMARC
4. Update `GMAIL_FROM_EMAIL` in environment variables

---

## Best Practices

### Email Sending

- ✅ Use templates for consistency
- ✅ Always include plain text version
- ✅ Test emails before production
- ✅ Log all sent emails
- ✅ Respect user preferences (unsubscribe)
- ✅ Include unsubscribe link in all marketing emails
- ✅ Use Gmail App Password, not regular password
- ✅ Rate limit email sending (avoid spam)

### Email Content

- ✅ Clear subject lines (< 50 characters)
- ✅ Personalize with user name
- ✅ Include clear call-to-action
- ✅ Mobile-responsive design
- ✅ Preheader text for preview
- ✅ Brand consistency (colors, logo)
- ✅ Footer with contact and legal links

### Notifications

- ✅ Don't overwhelm users (batch notifications)
- ✅ Make notifications actionable
- ✅ Allow users to customize preferences
- ✅ Mark as read after interaction
- ✅ Clean up old notifications
- ✅ Use appropriate severity (info, warning, error)

### Security

- ✅ Validate email addresses
- ✅ Rate limit verification emails
- ✅ Expire verification links (1 hour)
- ✅ Don't reveal user existence in errors
- ✅ Sanitize user input in emails
- ✅ Use HTTPS for all links

---

## Testing

### Test Email Sending Locally

```typescript
// functions/src/test/testEmail.ts
import { sendEmail } from '../services/emailService';
import { welcomeEmailTemplate } from '../templates/welcomeEmail';

async function testWelcomeEmail() {
  const html = welcomeEmailTemplate('Test User', 'https://yourapp.com/verify');

  await sendEmail({
    to: 'test@example.com',
    subject: 'Test Welcome Email',
    html,
  });

  console.log('Test email sent!');
}

testWelcomeEmail().catch(console.error);
```

### Test with Gmail Filters

Create a Gmail filter to test without spamming:

1. In Gmail, create filter for "from:noreply@yourapp.com"
2. Apply label "YourApp Test"
3. Skip inbox
4. All test emails go to this label

---

## Troubleshooting

### Emails Not Sending

```typescript
// Check Gmail App Password
// Verify environment variables are set
console.log('GMAIL_USER:', process.env.GMAIL_USER);
console.log('GMAIL_APP_PASSWORD set:', !!process.env.GMAIL_APP_PASSWORD);

// Test transporter
import { verifyEmailConfig } from './services/emailService';
await verifyEmailConfig();
```

### Emails Going to Spam

- Configure SPF, DKIM, DMARC records
- Warm up sending (start with low volume)
- Don't use spammy words in subject
- Include unsubscribe link
- Use authenticated domain

### Gmail Daily Limit

Gmail has sending limits:
- **Free Gmail**: 500 emails/day
- **Google Workspace**: 2000 emails/day

For higher volume, consider:
- SendGrid (12,000 free/month)
- AWS SES (62,000 free/month with EC2)
- Resend (3,000 free/month)

---

## Migration to Professional Email Service (Optional)

If you outgrow Gmail SMTP:

### Resend Example

```typescript
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

await resend.emails.send({
  from: 'noreply@yourapp.com',
  to: 'user@example.com',
  subject: 'Welcome!',
  html: welcomeEmailTemplate('User'),
});
```

### SendGrid Example

```typescript
import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY);

await sgMail.send({
  from: 'noreply@yourapp.com',
  to: 'user@example.com',
  subject: 'Welcome!',
  html: welcomeEmailTemplate('User'),
});
```

---

## Resources

- **Nodemailer Docs**: https://nodemailer.com/
- **Gmail App Passwords**: https://support.google.com/accounts/answer/185833
- **Email Design Best Practices**: https://www.campaignmonitor.com/resources/guides/email-design/
- **SPF/DKIM/DMARC Guide**: https://www.cloudflare.com/learning/dns/dns-records/dns-dmarc-record/

---

**Remember: Always use Gmail App Password, never your regular Google password!**
