# UPI and QR payment setup

After a proposal is accepted, both participants see the connection payment page. The page shows the accepted project amount, the applicable Adrox slab fee, the total payable amount, the Admin-configured UPI ID, payee name, and optional QR image. Each participant pays independently and submits the resulting UPI transaction ID. Contacts remain locked while either payment is awaiting verification.

## Admin configuration

1. Sign in with `mdadnan018863@gmail.com` and open `/admin`.
2. In **Fee control**, enter the **UPI ID**, **Payee name**, and an optional publicly accessible **QR image URL**.
3. Add clear payment instructions, then select **Save fees**.
4. For each submitted payment, open **Payment verification** and verify the transaction reference only after checking the payment in the UPI provider account.
5. After both the Business and FDE payments are verified, the trusted Firebase payment transition marks the connection `connected` and copies both private contact cards into the connection. The contact details then appear on both sides.

The QR URL is display-only. Payment verification remains an Admin responsibility until a trusted Razorpay/UPI webhook is configured. Do not place private keys or secrets in the QR URL or frontend code.
