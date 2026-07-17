# FoodExpress — Online Food Ordering Frontend

A React (Vite) frontend for the Online Food Order Processing System (Camunda + ActiveMQ +
Spring Boot microservices backend, per the take-home spec). This is a **frontend-only**
project — no backend code is included. It expects a backend REST API exposing:

- `POST /api/orders` — create a new order (`{ customerName, item, amount }`)
- `GET /api/orders` — list all orders with current status
- `GET /api/orders/{id}` — detailed status of a single order

The order lifecycle matches the Camunda BPMN workflow described in the spec:

```
PLACED → PAYMENT_PROCESSING → KITCHEN_PREP → OUT_FOR_DELIVERY → DELIVERED
                    └────────────→ CANCELLED (on payment failure)
```

If your backend uses different status enum names, update the mapping in
`src/orderStatus.js` — everything else (badges, stepper) reads from that one file.

## Project structure

```
food-ordering-frontend/
├── index.html
├── package.json
├── vite.config.js
├── .env.example
├── public/
│   └── favicon.svg
└── src/
    ├── main.jsx
    ├── App.jsx / App.css
    ├── index.css
    ├── api.js              # axios client: getOrders, getOrderById, createOrder
    ├── orderStatus.js       # canonical lifecycle statuses + helpers
    └── components/
        ├── OrderForm.jsx / .css           # order placement form + validation
        ├── OrderDashboard.jsx / .css      # polling table of all orders (every 2s)
        ├── OrderDetailModal.jsx / .css    # click a row -> live single-order detail (GET /api/orders/{id})
        └── OrderStatusStepper.jsx / .css  # visual Placed→Payment→Kitchen→Delivery→Delivered progress
```

## Setup & running

1. Install dependencies:

   ```bash
   npm install
   ```

2. (Optional) configure your backend URL. Copy `.env.example` to `.env` and set
   `VITE_API_BASE_URL` to point at your backend server (defaults to
   `http://localhost:5000` otherwise):

   ```bash
   cp .env.example .env
   ```

3. Start the dev server:

   ```bash
   npm run dev
   ```

4. Open the printed local URL (typically `http://localhost:5173`) in your browser.

## Notes

- The dashboard polls `GET /api/orders` every 2 seconds to reflect status updates
  in near real-time.
- Clicking any order row opens a detail modal that calls `GET /api/orders/{id}`
  and polls it every 2 seconds, with a visual stepper showing progress through
  Placed → Payment → Kitchen Prep → Out for Delivery → Delivered (or a
  "Cancelled — Payment Failed" state).
- Status badges/stepper expect the backend to return one of: `PLACED`,
  `PAYMENT_PROCESSING`, `KITCHEN_PREP`, `OUT_FOR_DELIVERY`, `DELIVERED`,
  `CANCELLED` (case-insensitive, spaces/hyphens also normalized). Adjust
  `src/orderStatus.js` if your Order Service uses different names.
- Since there's no backend yet, `npm run dev` will load fine but the dashboard will
  show a connection error and the form will fail to submit until a backend is running
  at the configured URL (see `.env.example`).
