# Adnify Firebase Deployment

Adnify uses Firebase Authentication, Cloud Firestore, Cloud Storage, and Cloud Functions as its only backend. The React app contains only the public Firebase Web App configuration. The password supplied for the administrator account is intentionally not written into this repository or any frontend environment file.

## One-time Firebase Console setup

In the **adnify-f033d** Firebase project, enable **Email/Password** in Authentication and create a **Cloud Firestore** database in production mode. Also enable Cloud Storage if profile-image uploads are required. Cloud Functions deployment requires a Firebase project with billing configured according to Firebase’s current requirements.

## Deploy the trusted backend

Install the Firebase CLI locally, authenticate with the Google account that owns the project, and run the following commands from the project root.

```bash
npm install --global firebase-tools
firebase login
cd firebase/functions && npm install && npm run build && cd ../..
firebase deploy --only functions,firestore:rules,storage
```

If you provide a service-account JSON instead of using interactive Firebase login, set `GOOGLE_APPLICATION_CREDENTIALS` to its local path and run `bash scripts/deploy-firebase.sh`. The helper deliberately checks for the credential at runtime and never copies it into the repository.

## Create the administrator account safely

After the function is deployed, use the Adnify registration dialog to create an account with the designated administrator email and the desired password. The `assignAccountRole` Cloud Function compares the authenticated email to the server-side administrator email, writes the `ADMIN` role, and sets the `admin` custom claim. It never accepts administrator status from the browser.

After changing the designated administrator email, redeploy Firebase Functions first so the trusted `ADMIN_EMAIL` value and `activateAdministrator` callable are live. On its next Adnify login, the designated Firebase account automatically invokes the server-validated callable, receives the `ADMIN` custom claim, and is routed to `/admin`. If it was already signed in, sign out and sign in again. Do not put the password in a source file, browser environment variable, or Firebase configuration.

## Connection-fee operations

An accepted Business proposal creates a Firebase `connections` record through a trusted Cloud Function. The record snapshots the current Business and FDE fee from `platform_config/default` and begins in the `awaiting_payments` state. Both parties can submit a payment reference, but only an administrator can verify, reject, or record a refund against the submission.

When the administrator verifies both payments, the trusted function copies each party’s private contact card into that connection record and sets `contactUnlocked` to `true`. Until then, each participant sees only locked contact placeholders. The platform currently supports **manual payment-reference verification**, which is appropriate when the administrator is verifying UPI, bank-transfer, or offline payment records. It does not initiate a card, wallet, or bank payment in the app.

## Administrator operating sequence

Use the Adnify `/admin` workspace after the administrator token has refreshed. The control center lets an administrator verify or block users, approve or reject submitted problems, set current connection fees and payment instructions, review payment references, record refunds, flag or cancel a connection, and resolve reports. A report can be submitted by either connection participant and is visible only to that participant and the administrator.

> The account must sign out and sign in again once Firebase Functions are deployed to receive the `ADMIN` custom claim. Do not rely on a browser-supplied role field.

## Security boundary

The Firestore rules prevent a browser from assigning itself an administrator role. Public profile documents are separate from user and role records, while connection records are accessible only to their two participants or a verified administrator. Payment-state changes, contact unlocking, fee snapshotting, and account suspension are executed by trusted Cloud Functions rather than client-side code.
