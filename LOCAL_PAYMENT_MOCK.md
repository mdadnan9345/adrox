# Local payment mock

This mock is for QA only. It connects the Adrox client to the Firebase Emulator Suite and refuses to run in deployed Firebase Functions. It does not process money and cannot unlock contacts in the live `adnify-f033d` project.

## Start the local stack

1. Install Firebase CLI if needed: `npm install -g firebase-tools`.
2. From the project root, start the emulators:

```bash
npx firebase emulators:start --only auth,firestore,functions
```

3. In a second terminal, start the frontend with the emulator flag:

```bash
VITE_USE_FIREBASE_EMULATORS=true pnpm run dev
```

4. Register two local accounts, one Business and one FDE. Create or seed an accepted connection and save a private contact card for both participants.
5. Submit a payment reference from each side. The payment card will show **Mock verify locally**.
6. Click the button once for the Business payment and once for the FDE payment. The emulator-only callable marks each submitted payment as verified. After the second verification, the connection becomes `connected` and both contact cards are copied into the connection for display.

Never set `VITE_USE_FIREBASE_EMULATORS=true` in Netlify or another production deployment. The callable also checks `FUNCTIONS_EMULATOR` and rejects all deployed invocations.
