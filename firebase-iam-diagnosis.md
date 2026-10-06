# Firebase IAM diagnosis

Firebase’s official IAM documentation says predefined Firebase roles include the `serviceusage.services.get` and `serviceusage.services.list` permissions needed to check API state and run Firebase CLI commands. The same documentation says Cloud Functions deployment can be delegated to a project Owner, or to a member with Cloud Functions Admin (`roles/cloudfunctions.admin`) and Service Account User (`roles/iam.serviceAccountUser`).

Sources:
- https://firebase.google.com/docs/projects/iam/permissions — Firebase IAM permissions
- https://firebase.google.com/docs/functions/manage-functions — Manage functions

Observed deployment errors for `adnify-f033d`:
- `403 Permission denied to get service [firebasestorage.googleapis.com]`
- `403 Permission denied to get service [firestore.googleapis.com]`
- `403 Permission denied to get service [cloudfunctions.googleapis.com]`

Therefore the Firebase service account can authenticate but does not have sufficient project-level IAM permissions to deploy the administrator callable, Functions, or Firestore rules. The local client now surfaces an explicit administrator-activation error instead of silently returning to the homepage.
