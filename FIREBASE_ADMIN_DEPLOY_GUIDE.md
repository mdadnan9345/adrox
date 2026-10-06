# Adnify administrator activation guide

This guide explains how to grant the Firebase deployment service account enough temporary access, deploy Adnify’s trusted administrator functions and Firestore rules, activate the administrator account, verify `/admin`, and remove the deployment credential afterward.

> The administrator password should be entered only into Firebase Authentication or the Adnify login form. Do not put it in a source file, command history, `.env` file, screenshot, or chat message.

## 1. Identify the deployment service account

Open the service-account JSON you downloaded for the `adnify-f033d` project and copy the value of `client_email`. It usually looks similar to:

```json
"client_email": "firebase-adminsdk-...@adnify-f033d.iam.gserviceaccount.com"
```

Use the exact value from your JSON. Do not share the private key.

## 2. Grant temporary IAM permissions

1. Open [Google Cloud IAM](https://console.cloud.google.com/iam-admin/iam) and select project **adnify-f033d**.
2. Click **Grant access**.
3. In **New principals**, paste the service account email from step 1.
4. Choose one of the following access paths.

### Easiest temporary path

Grant **Project Owner** (`roles/owner`) temporarily. This is the simplest way to avoid missing a required deployment permission, but it is intentionally broad. Remove this role immediately after deployment and verification.

### Least-privilege path

Grant these roles to the service account:

| Role | Why it is needed |
| --- | --- |
| **Service Usage Viewer** (`roles/serviceusage.serviceUsageViewer`) | Lets Firebase CLI check whether required Google APIs are enabled, including `serviceusage.services.get`. |
| **Cloud Functions Admin** (`roles/cloudfunctions.admin`) | Deploys and updates the trusted Firebase Functions. |
| **Service Account User** (`roles/iam.serviceAccountUser`) | Allows the deployment process to act as the runtime service account. |
| **Firebase Rules Admin** (`roles/firebaserules.admin`) | Deploys the Firestore and Storage security rules. |

For the current Adnify deployment, deploy only Functions and Firestore rules first. Storage rules can be added later if the project needs profile-image uploads. Firebase Functions deployment may also ask you to enable billing or required Google APIs in the Firebase/Google Cloud console.

Firebase’s official IAM documentation lists `serviceusage.services.get` and related Service Usage permissions as required for Firebase CLI commands. Cloud Functions deployment is performed with `firebase deploy --only functions`.[1] [2]

## 3. Stage the key locally, outside the project

Do not copy the service-account JSON into `/home/ubuntu/adnify`, `client/`, `client/public/`, or any Git repository. Use a restricted temporary path instead:

```bash
install -m 600 /path/to/your/adnify-f033d-service-account.json /tmp/adnify-firebase-deploy-key.json
export GOOGLE_APPLICATION_CREDENTIALS=/tmp/adnify-firebase-deploy-key.json
```

Replace `/path/to/your/...json` with the local path of your downloaded file.

## 4. No-Blaze alternative: provision the Admin claim locally

If the Firebase project remains on the Spark plan, Cloud Functions cannot be deployed. A project owner can still provision the custom claim locally with the Firebase Admin SDK. This requires a service-account JSON for **adnify-f033d**, but does not require Cloud Functions or billing.

Run this from the project root, keeping the JSON outside the repository:

```bash
cd /home/ubuntu/adnify
install -m 600 /path/to/your/adnify-f033d-service-account.json /tmp/adnify-firebase-admin-key.json
export GOOGLE_APPLICATION_CREDENTIALS=/tmp/adnify-firebase-admin-key.json
npm --prefix firebase/functions install
npm --prefix firebase/functions run set-admin
unset GOOGLE_APPLICATION_CREDENTIALS
rm -f /tmp/adnify-firebase-admin-key.json
```

The script verifies that the service account belongs to `adnify-f033d`, finds `mdadnan018863@gmail.com`, sets `{ admin: true, role: "ADMIN" }`, and writes the matching `roles/{uid}` and `users/{uid}` records. It never reads or stores the administrator password. After it succeeds, sign out of Adrox, sign in again, and open `/admin`.

This local method enables the Admin dashboard and Firestore admin reads/writes only. The marketplace’s automatic connection creation, payment transitions, and refund re-lock trigger still require Cloud Functions and therefore the Blaze plan.

## 5. Deploy the trusted administrator function and Firestore rules

From the Adnify project root, run:

```bash
cd /home/ubuntu/adnify
npm --prefix firebase/functions install
npm --prefix firebase/functions run build
npx --yes firebase-tools@latest deploy \
  --project adnify-f033d \
  --only functions,firestore:rules
```

This deploys the `activateAdministrator` callable, the account-role function, the connection/payment transition functions, and the Firestore rules that restrict administrator actions.

If you intentionally need Storage rules as well, run:

```bash
npx --yes firebase-tools@latest deploy \
  --project adnify-f033d \
  --only functions,firestore:rules,storage
```

A successful deployment should list `activateAdministrator` among the deployed Functions. If the command reports `serviceusage.services.get` or a `403`, return to step 2 and confirm that the role was granted to the exact `client_email` principal from the JSON.

## 6. Sign in and activate the administrator claim

1. Open the Adnify website.
2. Click **Log in**.
3. Enter the designated administrator email and the password you supplied privately.
4. The login succeeds first through Firebase Authentication. The client then invokes `activateAdministrator`.
5. The trusted Function validates the email on the server, creates or updates the `users/{uid}` record, writes `roles/{uid}` with `role: ADMIN`, and sets the Firebase custom claims `{ admin: true, role: ADMIN }`.
6. Sign out and sign in again. This refreshes the Firebase ID token so Firestore rules can recognise the administrator claim.
7. Open `/admin` and confirm that the administrator control center loads.

If the site returns to the homepage with **“Admin activation is not deployed yet”**, the Function is not deployed, the browser is using a different Firebase project, or the callable deployment failed. If it says **“ADMIN claim is not active yet”**, sign out completely and sign in again after confirming the deployment completed.

## 7. Verify administrator powers safely

Inside `/admin`, verify that these sections render without a permission error:

| Area | Safe verification |
| --- | --- |
| Fee control | Confirm the Business and FDE fee fields are visible. Do not change them unless intended. |
| Payment verification | Confirm the pending-payment list loads. |
| Problem moderation | Confirm the moderation list loads. |
| People and safety | Confirm user verification and block controls load. |
| Connections | Confirm connection records can be read by the admin claim. |
| Reports, disputes, refunds | Confirm the status areas render and remain empty when there are no records. |

Do not use destructive actions merely to test access. Use a real payment or problem record only when you are ready to change its status.

## 8. Remove deployment access and the temporary key

After successful verification:

```bash
unset GOOGLE_APPLICATION_CREDENTIALS
rm -f /tmp/adnify-firebase-deploy-key.json
```

Return to Google Cloud IAM and remove the temporary Project Owner role, or remove any deployment-only roles that are no longer needed. If you used the least-privilege path and want to retain automated deployments, keep only the roles required by your deployment process. Never commit the JSON file to Git.

## 9. If the admin panel still does not open

Confirm all of the following:

1. The Firebase Web App configuration in Adnify points to project `adnify-f033d`.
2. The Firebase Authentication account email exactly matches the server-side designated administrator email.
3. `activateAdministrator` appears in the Functions deployment output.
4. The browser was signed out and signed back in after deployment.
5. The service account received permissions in project `adnify-f033d`, not another Google Cloud project.
6. Firestore rules were deployed successfully.

The Adnify login dialog now shows an explicit activation error instead of silently hiding this failure.

## References

[1]: https://firebase.google.com/docs/projects/iam/permissions — Firebase IAM permissions.

[2]: https://firebase.google.com/docs/functions/manage-functions — Manage Cloud Functions for Firebase.
