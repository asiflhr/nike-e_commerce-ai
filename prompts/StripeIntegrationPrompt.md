You are a senior full-stack engineer assigned to integrate **Stripe Checkout** into the e-commerce platform.

## Objective

Implement a seamless **Stripe Checkout integration** that supports both **guest sessions** and **authenticated users**. The flow must handle cart persistence, create Stripe checkout sessions securely on the server, and redirect users to Stripe’s hosted checkout page. After successful payment, orders should be stored in the database and linked to the correct user (or merged guest session).

## Structure

- Use **Next.js 14 App Router**, **TypeScript**, **Drizzle ORM**, and **PostgreSQL**.
- Strictly follow theme guidelines
- Follow the file/folder structure:
  ```
  /src
  ├── app/
  │   ├── (root)/
  │   │   ├── cart/
  │   │   │   └── page.tsx              ← cart page with checkout button
  │   │   ├── checkout/
  │   │   │   └── success/page.tsx      ← order success page
  │   │   api/
  │   │   ├── stripe/
  │   │        └── route.ts             ← webhook handler for Stripe events
  │   └── layout.tsx
  │
  ├── components/
  │   ├── CartSummary.tsx               ← cart total + checkout button
  │   └── OrderSuccess.tsx              ← success UI after checkout
  │
  ├── lib/
  │   ├── stripe/
  │   │   └── client.ts                 ← stripe client instance
  │   ├── actions/
  │   │   ├── checkout.ts               ← server action: createStripeCheckoutSession
  │   │   └── orders.ts                 ← server actions: createOrder, getOrder
  │   └── utils/
  │       └── mergeSessions.ts          ← helper to merge guest + user sessions
  ```

## Tasks

### 1. **Setup Stripe**

- Add `stripe` NPM package.
- Create `/src/lib/stripe/client.ts` exporting initialized Stripe instance with `process.env.STRIPE_SECRET_KEY`.
- Store publishable key in `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`.

### 2. **Cart Checkout Button**

- Add a **Checkout button** in `CartSummary.tsx`.
- On click, call **`createStripeCheckoutSession`** (server action).
- Redirect user to Stripe Checkout URL returned from the server.

### 3. **Server Actions**

- **`createStripeCheckoutSession(cartId: string)`**

  - Fetch cart items from DB (merge guest + user if needed).
  - Create a Stripe Checkout session with line items.
  - Return checkout URL.

- **`createOrder(stripeSessionId: string, userId?: string)`**

  - On successful payment (via webhook), insert order in DB with associated cart + user.

- **`getOrder(orderId: string)`**

  - Retrieve order details for success page.

### 4. **Webhook Handling**

- Implement `/app/api/stripe/route.ts`.
- Listen to Stripe events:

  - `checkout.session.completed`: call `createOrder`.
  - `payment_intent.payment_failed`: log failure.

- Ensure webhook is secured using Stripe signature verification.

### 5. **Success Page**

- Create `/checkout/success/page.tsx`.
- Fetch order by session/orderId.
- Show `OrderSuccess.tsx` with purchased items + total amount.

## Output Requirements

- Fully working **Stripe Checkout integration** with server actions.
- Checkout flow must work for **guest sessions** and **authenticated users**.
- Orders must persist in DB with proper user linkage.
- Webhooks must be verified and idempotent.
- UI should use **Lucide icons**, **Tailwind**, and be **responsive**.
- Code must follow **TypeScript strict mode** and Next.js **App Router conventions**.

## Notes

- Place **all server actions inside `/src/lib/actions`**.
- Use strict naming: `createStripeCheckoutSession`, `createOrder`, `getOrder`.
- Cart session merging logic is already in `src/lib/auth/actions.ts`. Use it before creating Stripe session.
- Ensure empty carts cannot initiate checkout.
- Store all monetary values as **integers in smallest currency unit** (e.g., cents).
