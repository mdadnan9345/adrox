# Spark-compatible payment workflow

Adrox no longer needs Cloud Functions for the manual UPI payment path. After an accepted proposal, both participants see the Admin-configured UPI/QR payment page and submit transaction IDs. In `/admin`, verify the Business payment and the FDE payment. Each verification runs a Firestore transaction from the Admin browser. The transaction marks that payment verified, updates the connection payment status, and after the second approval copies both private contact cards into the connection and sets `contactUnlocked: true`.

This path requires the Admin account to have the trusted `admin` custom claim and the updated Firestore rules. It does not automatically confirm that money was received; the Admin must check the UPI provider before selecting Verify. Cloud Functions remain optional for future automation such as webhook verification and automatic connection creation.
