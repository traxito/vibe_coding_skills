---
name: Firebase Best Practices
description: Configure and use Firebase (Authentication, Firestore, Cloud Functions, Hosting, Storage) following best practices. Includes authentication patterns, optimized queries, deployment workflows, and troubleshooting. Use whenever working with Firebase services.
applyTo: "**/*.{ts,tsx,js,jsx,json}"
---

# Firebase Best Practices Skill

Complete guide for Firebase development with production-ready patterns.

## When to Use This Skill

- Setting up Firebase project configuration
- Implementing authentication flows
- Writing Firestore queries and data models
- Creating Cloud Functions
- Deploying to Firebase Hosting
- Managing Firebase Storage
- Optimizing Firebase usage and costs
- Troubleshooting Firebase issues

---

## Important: PowerShell Users

**If working in PowerShell terminal, always prefix Firebase CLI commands with `npx`:**

```powershell
# ✅ CORRECT in PowerShell
npx firebase login
npx firebase init
npx firebase deploy

# ❌ WRONG in PowerShell (may fail with execution policy errors)
firebase login
firebase init
firebase deploy
```

This prevents PowerShell script execution policy issues.

---

## Firebase Project Setup

### 1. Install Firebase CLI

```bash
# Install globally
npm install -g firebase-tools

# Or use npx (recommended for PowerShell)
npx firebase --version
```

### 2. Initialize Firebase Project

```bash
# Login to Firebase
npx firebase login

# Initialize in your project directory
npx firebase init

# Select services:
# [x] Authentication
# [x] Firestore
# [x] Functions
# [x] Hosting
# [x] Storage
# [x] Emulators
```

### 3. Project Structure

```
your-project/
├── src/
│   ├── lib/
│   │   ├── firebase.ts          # Firebase config & initialization
│   │   ├── firestore.ts         # Firestore utilities
│   │   └── auth.ts              # Auth utilities
│   └── ...
├── functions/
│   ├── src/
│   │   ├── index.ts             # Cloud Functions entry
│   │   ├── auth/                # Auth-related functions
│   │   ├── firestore/           # Firestore triggers
│   │   └── api/                 # HTTP callable functions
│   └── package.json
├── firestore.rules              # Security rules
├── firestore.indexes.json       # Composite indexes
├── storage.rules                # Storage security rules
├── firebase.json                # Firebase config
└── .firebaserc                  # Project aliases
```

---

## Firebase Configuration

### Client-side Configuration (React/TypeScript)

```typescript
// src/lib/firebase.ts
import { initializeApp, getApps, FirebaseApp } from 'firebase/app';
import { getAuth, Auth } from 'firebase/auth';
import { getFirestore, Firestore } from 'firebase/firestore';
import { getStorage, FirebaseStorage } from 'firebase/storage';
import { getFunctions, Functions } from 'firebase/functions';

// Firebase configuration
const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
  measurementId: import.meta.env.VITE_FIREBASE_MEASUREMENT_ID
};

// Initialize Firebase (singleton pattern)
let app: FirebaseApp;
let auth: Auth;
let db: Firestore;
let storage: FirebaseStorage;
let functions: Functions;

if (!getApps().length) {
  app = initializeApp(firebaseConfig);
  auth = getAuth(app);
  db = getFirestore(app);
  storage = getStorage(app);
  functions = getFunctions(app);

  // Enable emulators in development
  if (import.meta.env.DEV) {
    const { connectAuthEmulator } = await import('firebase/auth');
    const { connectFirestoreEmulator } = await import('firebase/firestore');
    const { connectStorageEmulator } = await import('firebase/storage');
    const { connectFunctionsEmulator } = await import('firebase/functions');

    connectAuthEmulator(auth, 'http://localhost:9099', { disableWarnings: true });
    connectFirestoreEmulator(db, 'localhost', 8080);
    connectStorageEmulator(storage, 'localhost', 9199);
    connectFunctionsEmulator(functions, 'localhost', 5001);
  }
} else {
  app = getApps()[0];
  auth = getAuth(app);
  db = getFirestore(app);
  storage = getStorage(app);
  functions = getFunctions(app);
}

export { app, auth, db, storage, functions };
```

### Environment Variables (.env)

```bash
# .env (DO NOT COMMIT - add to .gitignore)
VITE_FIREBASE_API_KEY=AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789012
VITE_FIREBASE_APP_ID=1:123456789012:web:abcdef1234567890
VITE_FIREBASE_MEASUREMENT_ID=G-XXXXXXXXXX
```

---

## Firebase Authentication

### Authentication Utilities

```typescript
// src/lib/auth.ts
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signInWithPopup,
  GoogleAuthProvider,
  signOut,
  sendPasswordResetEmail,
  updateProfile,
  User,
  UserCredential
} from 'firebase/auth';
import { doc, setDoc, getDoc, serverTimestamp } from 'firebase/firestore';
import { auth, db } from './firebase';

// Create user with email/password
export async function signUp(email: string, password: string, displayName: string): Promise<User> {
  try {
    // Create auth user
    const userCredential: UserCredential = await createUserWithEmailAndPassword(
      auth,
      email,
      password
    );

    const user = userCredential.user;

    // Update profile
    await updateProfile(user, { displayName });

    // Create user document in Firestore
    await setDoc(doc(db, 'users', user.uid), {
      uid: user.uid,
      email: user.email,
      displayName,
      photoURL: user.photoURL || null,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp(),
      plan: 'free',
      active: true
    });

    return user;
  } catch (error: any) {
    console.error('Sign up error:', error);
    throw new Error(getAuthErrorMessage(error.code));
  }
}

// Sign in with email/password
export async function signIn(email: string, password: string): Promise<User> {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, password);
    return userCredential.user;
  } catch (error: any) {
    console.error('Sign in error:', error);
    throw new Error(getAuthErrorMessage(error.code));
  }
}

// Sign in with Google
export async function signInWithGoogle(): Promise<User> {
  try {
    const provider = new GoogleAuthProvider();
    const userCredential = await signInWithPopup(auth, provider);
    const user = userCredential.user;

    // Check if user document exists
    const userDoc = await getDoc(doc(db, 'users', user.uid));

    if (!userDoc.exists()) {
      // Create user document for new Google sign-in
      await setDoc(doc(db, 'users', user.uid), {
        uid: user.uid,
        email: user.email,
        displayName: user.displayName,
        photoURL: user.photoURL,
        createdAt: serverTimestamp(),
        updatedAt: serverTimestamp(),
        plan: 'free',
        active: true
      });
    }

    return user;
  } catch (error: any) {
    console.error('Google sign in error:', error);
    throw new Error(getAuthErrorMessage(error.code));
  }
}

// Sign out
export async function logout(): Promise<void> {
  try {
    await signOut(auth);
  } catch (error) {
    console.error('Sign out error:', error);
    throw error;
  }
}

// Reset password
export async function resetPassword(email: string): Promise<void> {
  try {
    await sendPasswordResetEmail(auth, email);
  } catch (error: any) {
    console.error('Password reset error:', error);
    throw new Error(getAuthErrorMessage(error.code));
  }
}

// Get user-friendly error messages
function getAuthErrorMessage(code: string): string {
  switch (code) {
    case 'auth/email-already-in-use':
      return 'This email is already registered';
    case 'auth/invalid-email':
      return 'Invalid email address';
    case 'auth/operation-not-allowed':
      return 'Operation not allowed';
    case 'auth/weak-password':
      return 'Password should be at least 6 characters';
    case 'auth/user-disabled':
      return 'This account has been disabled';
    case 'auth/user-not-found':
      return 'No account found with this email';
    case 'auth/wrong-password':
      return 'Incorrect password';
    case 'auth/too-many-requests':
      return 'Too many failed attempts. Please try again later';
    default:
      return 'An error occurred. Please try again';
  }
}
```

### Auth Context (React)

```typescript
// src/contexts/AuthContext.tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';
import { User, onAuthStateChanged } from 'firebase/auth';
import { doc, onSnapshot } from 'firebase/firestore';
import { auth, db } from '@/lib/firebase';

interface UserData {
  uid: string;
  email: string | null;
  displayName: string | null;
  photoURL: string | null;
  plan: 'free' | 'pro' | 'enterprise';
  active: boolean;
}

interface AuthContextType {
  user: User | null;
  userData: UserData | null;
  loading: boolean;
}

const AuthContext = createContext<AuthContextType>({
  user: null,
  userData: null,
  loading: true
});

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [userData, setUserData] = useState<UserData | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Subscribe to auth state changes
    const unsubscribeAuth = onAuthStateChanged(auth, (user) => {
      setUser(user);

      if (!user) {
        setUserData(null);
        setLoading(false);
      }
    });

    return () => unsubscribeAuth();
  }, []);

  useEffect(() => {
    if (!user) return;

    // Subscribe to user document changes
    const unsubscribeUser = onSnapshot(
      doc(db, 'users', user.uid),
      (doc) => {
        if (doc.exists()) {
          setUserData(doc.data() as UserData);
        }
        setLoading(false);
      },
      (error) => {
        console.error('Error fetching user data:', error);
        setLoading(false);
      }
    );

    return () => unsubscribeUser();
  }, [user]);

  return (
    <AuthContext.Provider value={{ user, userData, loading }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

## Firestore Best Practices

### Data Model Patterns

```typescript
// src/types/firestore.ts

// ✅ CORRECT: Flat structure with references
interface User {
  uid: string;
  email: string;
  displayName: string;
  plan: 'free' | 'pro' | 'enterprise';
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

interface Project {
  id: string;
  name: string;
  description: string;
  ownerId: string;              // Reference to user
  memberIds: string[];          // Array of user IDs
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

interface Task {
  id: string;
  projectId: string;            // Reference to project
  assignedToId: string;         // Reference to user
  title: string;
  status: 'todo' | 'in-progress' | 'done';
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

// ❌ WRONG: Deep nesting (avoid this)
interface UserWrong {
  uid: string;
  email: string;
  projects: {                   // Don't nest entire objects
    [projectId: string]: {
      name: string;
      tasks: {
        [taskId: string]: {
          title: string;
        };
      };
    };
  };
}
```

### Firestore Utilities

```typescript
// src/lib/firestore.ts
import {
  collection,
  doc,
  getDoc,
  getDocs,
  addDoc,
  updateDoc,
  deleteDoc,
  query,
  where,
  orderBy,
  limit,
  startAfter,
  DocumentData,
  QueryConstraint,
  serverTimestamp,
  Timestamp
} from 'firebase/firestore';
import { db } from './firebase';

// Generic CRUD operations

// Create document
export async function createDocument<T extends DocumentData>(
  collectionName: string,
  data: Omit<T, 'id' | 'createdAt' | 'updatedAt'>
): Promise<string> {
  try {
    const docRef = await addDoc(collection(db, collectionName), {
      ...data,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp()
    });
    return docRef.id;
  } catch (error) {
    console.error(`Error creating document in ${collectionName}:`, error);
    throw error;
  }
}

// Read document by ID
export async function getDocument<T>(
  collectionName: string,
  documentId: string
): Promise<T | null> {
  try {
    const docRef = doc(db, collectionName, documentId);
    const docSnap = await getDoc(docRef);

    if (docSnap.exists()) {
      return { id: docSnap.id, ...docSnap.data() } as T;
    }
    return null;
  } catch (error) {
    console.error(`Error getting document from ${collectionName}:`, error);
    throw error;
  }
}

// Update document
export async function updateDocument(
  collectionName: string,
  documentId: string,
  data: Partial<DocumentData>
): Promise<void> {
  try {
    const docRef = doc(db, collectionName, documentId);
    await updateDoc(docRef, {
      ...data,
      updatedAt: serverTimestamp()
    });
  } catch (error) {
    console.error(`Error updating document in ${collectionName}:`, error);
    throw error;
  }
}

// Delete document
export async function deleteDocument(
  collectionName: string,
  documentId: string
): Promise<void> {
  try {
    await deleteDoc(doc(db, collectionName, documentId));
  } catch (error) {
    console.error(`Error deleting document from ${collectionName}:`, error);
    throw error;
  }
}

// Query documents
export async function queryDocuments<T>(
  collectionName: string,
  ...constraints: QueryConstraint[]
): Promise<T[]> {
  try {
    const q = query(collection(db, collectionName), ...constraints);
    const querySnapshot = await getDocs(q);

    return querySnapshot.docs.map(doc => ({
      id: doc.id,
      ...doc.data()
    })) as T[];
  } catch (error) {
    console.error(`Error querying ${collectionName}:`, error);
    throw error;
  }
}

// Paginated query
export async function getPaginatedDocuments<T>(
  collectionName: string,
  pageSize: number = 10,
  lastDoc?: any
): Promise<{ docs: T[], hasMore: boolean, lastDoc: any }> {
  try {
    const constraints: QueryConstraint[] = [
      orderBy('createdAt', 'desc'),
      limit(pageSize + 1)
    ];

    if (lastDoc) {
      constraints.push(startAfter(lastDoc));
    }

    const q = query(collection(db, collectionName), ...constraints);
    const querySnapshot = await getDocs(q);

    const docs = querySnapshot.docs.slice(0, pageSize).map(doc => ({
      id: doc.id,
      ...doc.data()
    })) as T[];

    const hasMore = querySnapshot.docs.length > pageSize;
    const newLastDoc = querySnapshot.docs[pageSize - 1];

    return { docs, hasMore, lastDoc: newLastDoc };
  } catch (error) {
    console.error(`Error paginating ${collectionName}:`, error);
    throw error;
  }
}
```

### Optimized Query Examples

```typescript
// src/services/projectService.ts
import { query, where, orderBy, limit } from 'firebase/firestore';
import { queryDocuments } from '@/lib/firestore';

interface Project {
  id: string;
  name: string;
  ownerId: string;
  memberIds: string[];
  status: 'active' | 'archived';
  createdAt: Timestamp;
}

// ✅ CORRECT: Indexed query with proper constraints
export async function getUserProjects(userId: string): Promise<Project[]> {
  return queryDocuments<Project>(
    'projects',
    where('memberIds', 'array-contains', userId),
    where('status', '==', 'active'),
    orderBy('createdAt', 'desc'),
    limit(50)
  );
}

// ✅ CORRECT: Query with compound index (requires firestore.indexes.json)
export async function getActiveProjectsByOwner(ownerId: string): Promise<Project[]> {
  return queryDocuments<Project>(
    'projects',
    where('ownerId', '==', ownerId),
    where('status', '==', 'active'),
    orderBy('createdAt', 'desc')
  );
}

// ❌ WRONG: Inefficient query (fetches all then filters)
export async function getUserProjectsWrong(userId: string): Promise<Project[]> {
  const allProjects = await queryDocuments<Project>('projects');
  return allProjects.filter(p => p.memberIds.includes(userId));
}
```

### Composite Indexes Configuration

```json
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "projects",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "ownerId", "order": "ASCENDING" },
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "tasks",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "projectId", "order": "ASCENDING" },
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ],
  "fieldOverrides": []
}
```

---

## Firestore Security Rules

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }

    function hasRole(role) {
      return isAuthenticated() && 
             get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == role;
    }

    // Users collection
    match /users/{userId} {
      // Anyone authenticated can read user profiles
      allow read: if isAuthenticated();

      // Only the user can create/update their own document
      allow create: if isOwner(userId);
      allow update: if isOwner(userId) && 
                       request.resource.data.uid == userId &&
                       request.resource.data.email == resource.data.email; // Prevent email change

      // Only admins can delete users
      allow delete: if hasRole('admin');
    }

    // Projects collection
    match /projects/{projectId} {
      // Members can read
      allow read: if isAuthenticated() && 
                     request.auth.uid in resource.data.memberIds;

      // Owner can create
      allow create: if isAuthenticated() && 
                       request.resource.data.ownerId == request.auth.uid;

      // Owner or members can update
      allow update: if isAuthenticated() && 
                       (request.auth.uid == resource.data.ownerId ||
                        request.auth.uid in resource.data.memberIds);

      // Only owner can delete
      allow delete: if isAuthenticated() && 
                       request.auth.uid == resource.data.ownerId;
    }

    // Tasks subcollection
    match /projects/{projectId}/tasks/{taskId} {
      // Inherit project permissions
      allow read, write: if isAuthenticated() && 
                            request.auth.uid in get(/databases/$(database)/documents/projects/$(projectId)).data.memberIds;
    }

    // Prevent all other access
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

### Test Security Rules

```bash
# PowerShell users: use npx
npx firebase emulators:start --only firestore

# Run security rules tests
npm run test:rules
```

```typescript
// tests/firestore.rules.test.ts
import { initializeTestEnvironment, assertSucceeds, assertFails } from '@firebase/rules-unit-testing';

let testEnv: any;

beforeAll(async () => {
  testEnv = await initializeTestEnvironment({
    projectId: 'demo-project',
    firestore: {
      rules: fs.readFileSync('firestore.rules', 'utf8'),
    },
  });
});

test('User can read their own document', async () => {
  const alice = testEnv.authenticatedContext('alice');
  await assertSucceeds(alice.firestore().doc('users/alice').get());
});

test('User cannot read another user document', async () => {
  const alice = testEnv.authenticatedContext('alice');
  await assertFails(alice.firestore().doc('users/bob').get());
});
```

---

## Cloud Functions

### Functions Structure

```typescript
// functions/src/index.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

// Initialize Admin SDK
admin.initializeApp();

// Export all functions
export { onUserCreated, onUserDeleted } from './auth/userTriggers';
export { onProjectCreated, onProjectUpdated } from './firestore/projectTriggers';
export { sendEmail, processPayment } from './api/callableFunctions';
export { cleanupOldData } from './scheduled/cleanupJobs';
```

### Auth Triggers

```typescript
// functions/src/auth/userTriggers.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

// Trigger when user is created
export const onUserCreated = functions.auth.user().onCreate(async (user) => {
  try {
    // Create user document if it doesn't exist
    const userDoc = await admin.firestore().collection('users').doc(user.uid).get();

    if (!userDoc.exists) {
      await admin.firestore().collection('users').doc(user.uid).set({
        uid: user.uid,
        email: user.email,
        displayName: user.displayName || null,
        photoURL: user.photoURL || null,
        createdAt: admin.firestore.FieldValue.serverTimestamp(),
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
        plan: 'free',
        active: true
      });

      console.log(`User document created for ${user.uid}`);
    }

    // Send welcome email (implement your email service)
    // await sendWelcomeEmail(user.email, user.displayName);

  } catch (error) {
    console.error('Error in onUserCreated:', error);
  }
});

// Trigger when user is deleted
export const onUserDeleted = functions.auth.user().onDelete(async (user) => {
  try {
    // Delete user document
    await admin.firestore().collection('users').doc(user.uid).delete();

    // Delete user's projects
    const projectsSnapshot = await admin.firestore()
      .collection('projects')
      .where('ownerId', '==', user.uid)
      .get();

    const batch = admin.firestore().batch();
    projectsSnapshot.docs.forEach(doc => {
      batch.delete(doc.ref);
    });
    await batch.commit();

    console.log(`User ${user.uid} and related data deleted`);
  } catch (error) {
    console.error('Error in onUserDeleted:', error);
  }
});
```

### Firestore Triggers

```typescript
// functions/src/firestore/projectTriggers.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

// Trigger when project is created
export const onProjectCreated = functions.firestore
  .document('projects/{projectId}')
  .onCreate(async (snapshot, context) => {
    const project = snapshot.data();
    const projectId = context.params.projectId;

    try {
      // Create default tasks for new project
      const tasksRef = admin.firestore().collection('tasks');

      await tasksRef.add({
        projectId,
        title: 'Welcome to your new project!',
        description: 'Get started by creating your first task',
        status: 'todo',
        assignedToId: project.ownerId,
        createdAt: admin.firestore.FieldValue.serverTimestamp(),
        updatedAt: admin.firestore.FieldValue.serverTimestamp()
      });

      console.log(`Default task created for project ${projectId}`);
    } catch (error) {
      console.error('Error in onProjectCreated:', error);
    }
  });

// Trigger when project is updated
export const onProjectUpdated = functions.firestore
  .document('projects/{projectId}')
  .onUpdate(async (change, context) => {
    const before = change.before.data();
    const after = change.after.data();

    // Check if status changed to archived
    if (before.status !== 'archived' && after.status === 'archived') {
      try {
        // Archive all tasks in the project
        const tasksSnapshot = await admin.firestore()
          .collection('tasks')
          .where('projectId', '==', context.params.projectId)
          .get();

        const batch = admin.firestore().batch();
        tasksSnapshot.docs.forEach(doc => {
          batch.update(doc.ref, { 
            status: 'archived',
            updatedAt: admin.firestore.FieldValue.serverTimestamp()
          });
        });
        await batch.commit();

        console.log(`Archived ${tasksSnapshot.size} tasks for project ${context.params.projectId}`);
      } catch (error) {
        console.error('Error archiving tasks:', error);
      }
    }
  });
```

### Callable Functions (API)

```typescript
// functions/src/api/callableFunctions.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

// Send email function
export const sendEmail = functions.https.onCall(async (data, context) => {
  // Verify authentication
  if (!context.auth) {
    throw new functions.https.HttpsError(
      'unauthenticated',
      'User must be authenticated'
    );
  }

  const { to, subject, body } = data;

  // Validate input
  if (!to || !subject || !body) {
    throw new functions.https.HttpsError(
      'invalid-argument',
      'Missing required fields: to, subject, body'
    );
  }

  try {
    // Implement your email service (SendGrid, AWS SES, etc.)
    // await emailService.send({ to, subject, body });

    console.log(`Email sent to ${to}`);
    return { success: true, message: 'Email sent successfully' };
  } catch (error) {
    console.error('Error sending email:', error);
    throw new functions.https.HttpsError('internal', 'Failed to send email');
  }
});

// Process payment function
export const processPayment = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
  }

  const { amount, currency, paymentMethodId } = data;

  try {
    // Implement payment processing (Stripe, PayPal, etc.)
    // const paymentIntent = await stripe.paymentIntents.create({
    //   amount,
    //   currency,
    //   payment_method: paymentMethodId,
    //   confirm: true
    // });

    // Update user plan in Firestore
    await admin.firestore().collection('users').doc(context.auth.uid).update({
      plan: 'pro',
      subscriptionId: 'payment-id',
      updatedAt: admin.firestore.FieldValue.serverTimestamp()
    });

    return { success: true, message: 'Payment processed successfully' };
  } catch (error) {
    console.error('Error processing payment:', error);
    throw new functions.https.HttpsError('internal', 'Payment processing failed');
  }
});
```

### Scheduled Functions

```typescript
// functions/src/scheduled/cleanupJobs.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

// Run every day at 2 AM
export const cleanupOldData = functions.pubsub
  .schedule('0 2 * * *')
  .timeZone('America/New_York')
  .onRun(async (context) => {
    try {
      const thirtyDaysAgo = new Date();
      thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

      // Delete old archived projects
      const snapshot = await admin.firestore()
        .collection('projects')
        .where('status', '==', 'archived')
        .where('updatedAt', '<', thirtyDaysAgo)
        .get();

      const batch = admin.firestore().batch();
      snapshot.docs.forEach(doc => {
        batch.delete(doc.ref);
      });
      await batch.commit();

      console.log(`Deleted ${snapshot.size} old archived projects`);
    } catch (error) {
      console.error('Error in cleanupOldData:', error);
    }
  });
```

### Client-side Callable Function Usage

```typescript
// src/services/emailService.ts
import { httpsCallable } from 'firebase/functions';
import { functions } from '@/lib/firebase';

export async function sendEmailToUser(to: string, subject: string, body: string) {
  const sendEmail = httpsCallable(functions, 'sendEmail');

  try {
    const result = await sendEmail({ to, subject, body });
    return result.data;
  } catch (error: any) {
    console.error('Error calling sendEmail:', error);
    throw new Error(error.message);
  }
}
```

---

## Firebase Storage

### Storage Utilities

```typescript
// src/lib/storage.ts
import {
  ref,
  uploadBytes,
  uploadBytesResumable,
  getDownloadURL,
  deleteObject,
  listAll,
  UploadTask
} from 'firebase/storage';
import { storage } from './firebase';

// Upload file
export async function uploadFile(
  file: File,
  path: string,
  onProgress?: (progress: number) => void
): Promise<string> {
  try {
    const storageRef = ref(storage, path);

    if (onProgress) {
      // Upload with progress tracking
      const uploadTask: UploadTask = uploadBytesResumable(storageRef, file);

      return new Promise((resolve, reject) => {
        uploadTask.on(
          'state_changed',
          (snapshot) => {
            const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
            onProgress(progress);
          },
          (error) => {
            console.error('Upload error:', error);
            reject(error);
          },
          async () => {
            const downloadURL = await getDownloadURL(uploadTask.snapshot.ref);
            resolve(downloadURL);
          }
        );
      });
    } else {
      // Simple upload without progress
      await uploadBytes(storageRef, file);
      return await getDownloadURL(storageRef);
    }
  } catch (error) {
    console.error('Error uploading file:', error);
    throw error;
  }
}

// Delete file
export async function deleteFile(path: string): Promise<void> {
  try {
    const storageRef = ref(storage, path);
    await deleteObject(storageRef);
  } catch (error) {
    console.error('Error deleting file:', error);
    throw error;
  }
}

// List files in directory
export async function listFiles(path: string): Promise<string[]> {
  try {
    const storageRef = ref(storage, path);
    const result = await listAll(storageRef);

    const urls = await Promise.all(
      result.items.map(item => getDownloadURL(item))
    );

    return urls;
  } catch (error) {
    console.error('Error listing files:', error);
    throw error;
  }
}
```

### Storage Security Rules

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(userId) {
      return request.auth.uid == userId;
    }

    function isImage() {
      return request.resource.contentType.matches('image/.*');
    }

    function isValidSize(maxSize) {
      return request.resource.size < maxSize;
    }

    // User profile images
    match /users/{userId}/profile/{fileName} {
      // Only owner can upload their profile image
      allow write: if isOwner(userId) && 
                      isImage() && 
                      isValidSize(5 * 1024 * 1024); // 5MB max

      // Anyone authenticated can read
      allow read: if isAuthenticated();
    }

    // Project files
    match /projects/{projectId}/{fileName} {
      // Check if user is project member (requires Firestore lookup)
      allow read, write: if isAuthenticated() && 
                            firestore.get(/databases/(default)/documents/projects/$(projectId)).data.memberIds.hasAny([request.auth.uid]);
    }

    // Deny all other access
    match /{allPaths=**} {
      allow read, write: if false;
    }
  }
}
```

---

## Deployment

### Deploy Commands (PowerShell Compatible)

```bash
# Deploy everything
npx firebase deploy

# Deploy only Firestore rules
npx firebase deploy --only firestore:rules

# Deploy only Firestore indexes
npx firebase deploy --only firestore:indexes

# Deploy only Cloud Functions
npx firebase deploy --only functions

# Deploy specific function
npx firebase deploy --only functions:onUserCreated

# Deploy only Hosting
npx firebase deploy --only hosting

# Deploy only Storage rules
npx firebase deploy --only storage

# Deploy to specific project (if using multiple environments)
npx firebase use production
npx firebase deploy
```

### CI/CD with GitHub Actions

```yaml
# .github/workflows/firebase-deploy.yml
name: Deploy to Firebase

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to Firebase
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: '${{ secrets.GITHUB_TOKEN }}'
          firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
          projectId: your-project-id
          channelId: live
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. PowerShell Execution Policy Error

```powershell
# Error: "firebase : File cannot be loaded because running scripts is disabled"

# ✅ Solution: Use npx
npx firebase deploy

# OR: Change execution policy (admin required)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### 2. Firestore Permission Denied

```typescript
// Error: "Missing or insufficient permissions"

// ✅ Check security rules
// ✅ Verify user is authenticated
// ✅ Ensure indexes are deployed

// Test with emulator
npx firebase emulators:start
```

#### 3. Function Deployment Timeout

```bash
# Error: "Deployment timed out"

# ✅ Increase timeout
npx firebase deploy --only functions --force

# ✅ Deploy functions individually
npx firebase deploy --only functions:functionName
```

#### 4. Composite Index Required

```bash
# Error: "The query requires an index"

# ✅ Firebase will provide a link to create the index
# OR manually add to firestore.indexes.json and deploy
npx firebase deploy --only firestore:indexes
```

#### 5. Storage CORS Issues

```bash
# Error: "CORS policy blocked"

# ✅ Configure CORS for Storage bucket
echo '[{"origin": ["*"], "method": ["GET"], "maxAgeSeconds": 3600}]' > cors.json
gsutil cors set cors.json gs://your-bucket.appspot.com
```

---

## Performance Optimization

### 1. Firestore Read/Write Optimization

```typescript
// ✅ GOOD: Batch writes (single write charge)
const batch = writeBatch(db);
batch.set(doc(db, 'users', 'user1'), { name: 'Alice' });
batch.update(doc(db, 'users', 'user2'), { name: 'Bob' });
batch.delete(doc(db, 'users', 'user3'));
await batch.commit();

// ❌ BAD: Multiple individual writes (3 write charges)
await setDoc(doc(db, 'users', 'user1'), { name: 'Alice' });
await updateDoc(doc(db, 'users', 'user2'), { name: 'Bob' });
await deleteDoc(doc(db, 'users', 'user3'));
```

### 2. Use Offline Persistence

```typescript
// Enable offline persistence
import { enableIndexedDbPersistence } from 'firebase/firestore';

try {
  await enableIndexedDbPersistence(db);
} catch (err: any) {
  if (err.code === 'failed-precondition') {
    console.log('Multiple tabs open, persistence only enabled in one tab');
  } else if (err.code === 'unimplemented') {
    console.log('Browser doesn't support persistence');
  }
}
```

### 3. Optimize Function Cold Starts

```typescript
// functions/src/index.ts

// ✅ GOOD: Keep imports minimal
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

// ❌ BAD: Heavy imports slow cold starts
// import * as _ from 'lodash'; // Only import if absolutely needed
```

---

## Best Practices Summary

### Security
- ✅ Always use Security Rules on Firestore and Storage
- ✅ Validate data on both client and server (Cloud Functions)
- ✅ Never expose sensitive data in client-side code
- ✅ Use environment variables for API keys

### Performance
- ✅ Use composite indexes for complex queries
- ✅ Enable offline persistence
- ✅ Implement pagination for large lists
- ✅ Use batch operations for multiple writes

### Cost Optimization
- ✅ Minimize document reads (use real-time listeners sparingly)
- ✅ Clean up unused data with scheduled functions
- ✅ Use Firebase Emulators for development
- ✅ Monitor usage in Firebase Console

### Development
- ✅ Use TypeScript for type safety
- ✅ Test security rules before deploying
- ✅ Use emulators during development
- ✅ Implement proper error handling

---

## Resources

- **Firebase Documentation**: https://firebase.google.com/docs
- **Firestore Data Modeling**: https://firebase.google.com/docs/firestore/manage-data/structure-data
- **Security Rules Guide**: https://firebase.google.com/docs/rules
- **Cloud Functions Samples**: https://github.com/firebase/functions-samples
- **Firebase CLI Reference**: https://firebase.google.com/docs/cli

---

**Remember: When using PowerShell, always prefix commands with `npx`!**

```powershell
npx firebase login
npx firebase init
npx firebase deploy
npx firebase emulators:start
```
