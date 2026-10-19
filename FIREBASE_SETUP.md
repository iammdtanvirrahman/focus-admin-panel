# Focus Admin — Firebase setup

Project ID: `focus-603ab`

The admin page uses the Firebase Web SDK from Google's CDN. Its Firebase web config is a client-side project identifier, not an admin secret. Access is controlled by Firebase Authentication plus Firestore rules.

## 1. Add the GitHub Pages domain

In Firebase Console open **Authentication → Settings → Authorized domains** and add:

`iammdtanvirrahman.github.io`

Keep the existing Firebase domains. This is required for Google sign-in on the deployed admin page.

## 2. Enable sign-in providers

Go to **Authentication → Sign-in method** and enable:
- Email/Password
- Google (select a support email)

The login page deliberately has no public sign-up. Create the first account from Authentication → Users → Add user (email/password), or sign in with Google once and use the newly created Firebase Auth UID for the admin profile.

## 3. Set the first admin profile

In Firestore → Data create a collection named `users`. Create a document whose ID is the exact UID shown in **Authentication → Users** for the intended admin.

Fields:
- `role` — string — `admin`
- `active` — boolean — `true`

Do not assume an old UID belongs to the current Firebase project. When the login gate rejects an account, it displays the actual signed-in UID so it can be matched to the intended account.

## 4. Publish initial Firestore security rules

Open Firestore Database → **Rules**. Review and publish the following baseline rules. They let an authenticated user read only their own profile, let an approved admin manage the content collections, and deny all other paths by default.

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function signedIn() {
      return request.auth != null;
    }

    function isAdmin() {
      return signedIn()
        && exists(/databases/$(database)/documents/users/$(request.auth.uid))
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin'
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.active == true;
    }

    match /users/{userId} {
      allow get: if signedIn() && request.auth.uid == userId || isAdmin();
      allow list: if isAdmin();
      allow create, update, delete: if isAdmin();
    }

    match /courses/{courseId} {
      allow read: if isAdmin() || resource.data.status == 'published';
      allow create, update, delete: if isAdmin();
    }

    match /classes/{classId} {
      allow read: if isAdmin() || resource.data.status == 'published';
      allow create, update, delete: if isAdmin();
    }

    match /materials/{materialId} {
      allow read: if isAdmin() || resource.data.status == 'published';
      allow create, update, delete: if isAdmin();
    }

    match /orders/{orderId} {
      allow read: if isAdmin()
        || (signedIn() && resource.data.userId == request.auth.uid);
      // Client-side checkout is not trusted. Add verified server/webhook
      // processing before enabling student purchases.
      allow create, update, delete: if isAdmin();
    }

    match /enrollments/{enrollmentId} {
      allow read: if isAdmin()
        || (signedIn() && resource.data.userId == request.auth.uid);
      allow create, update, delete: if isAdmin();
    }

    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

**Important:** Firestore rules are not query filters. Public catalog queries must constrain published content (for example, `where("status", "==", "published")`) to satisfy the rules. The current admin prototype still stores its course/class demo records in browser localStorage; Firestore syncing is a separate next implementation step.

## 5. Test login

1. Open `https://iammdtanvirrahman.github.io/focus-admin-panel/`.
2. Sign in with the email/password account or Google account.
3. The login gate reads `users/{signed-in-UID}`.
4. Dashboard opens only if the profile exists and has `role == "admin"` and `active == true`.
5. If Firestore permission is denied, ensure the rules above were published. If the user is not authorized, use the exact UID displayed by the page and create/update that user's profile via the Firebase Console.

## Current limits

- This release adds the authentication gate and role check; course/class/material/order forms remain demo data in localStorage.
- Firebase Storage is not enabled because the project console showed it requires a billing-plan upgrade.
- Live video hosting and payment processing are not implemented. Do not treat editable demo order statuses as payments.
- Never paste private service-account credentials or payment secrets into frontend files.
