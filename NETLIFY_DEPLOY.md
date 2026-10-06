# Deploy Adrox to Netlify

Adrox is arranged as a Vite single-page frontend with Firebase handling Authentication, Firestore, Storage, and Cloud Functions. Netlify should deploy only the frontend; the `firebase/` directory is retained for Firebase CLI deployments.

## Netlify settings

Use the repository root as the base directory and configure:

- **Build command:** `pnpm run build:netlify`
- **Publish directory:** `dist/public`
- **Node version:** `22`
- **Package manager:** `pnpm` (detected from `pnpm-lock.yaml`)

These settings are already committed in `netlify.toml`. The SPA fallback is in `client/public/_redirects`, so routes such as `/business`, `/fde`, and `/admin` continue to work after refresh.

## Firebase backend

Do not remove the `firebase/` directory. Netlify does not deploy Firebase Functions or security rules. Deploy those separately from a trusted machine with the Firebase CLI:

```bash
firebase deploy --only functions,firestore:rules,storage
```

The frontend is configured to use the existing Firebase project `adnify-f033d`.

## Razorpay checkout

The connection desk now calculates the requested slab charge and opens Razorpay Checkout when a public test/live key is configured. Add the Razorpay **Key ID only** as a Netlify environment variable:

```text
VITE_RAZORPAY_KEY_ID=rzp_test_...
```

For a ₹57,000 proposal, the UI shows ₹57,000 project amount + ₹5,000 slab fee = ₹62,000 payable. The fee table is implemented in `client/src/lib/fees.ts` and is shared by the public pricing section and connection checkout.

Razorpay success currently records the payment ID as a Firebase payment reference. Contact unlocking must still be performed by trusted server-side verification; never unlock contacts from a browser-only success callback. Fully automatic verification requires the Razorpay secret/webhook handler on a trusted backend, such as deployed Firebase Functions on the Blaze plan.

## Repository layout

```text
client/                  Vite/React frontend source
client/public/assets/    Netlify-served brand and editorial images
client/public/_redirects SPA route fallback
firebase/                Firebase rules and Cloud Functions
netlify.toml             Netlify build and publish configuration
```
