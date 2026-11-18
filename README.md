# Tube Coffee

**Capstone project proposal and requirements**

Tube Coffee is an online coffee shop where customers can browse the menu, order drinks for pickup, pay through ABA PayWay, and track their orders. Staff use an admin dashboard to manage the menu, website content, orders, and customer messages. An AI assistant answers common questions over chat and can take a pickup request over the phone, read it back to the customer, and send the details to staff for review.

This README describes what the team plans to build.
![Tube Coffee system architecture](docs/images/https://res.cloudinary.com/t6wyjwqt/image/upload/v1794985137/tube-coffee-realtime-architecture.png)

## 1. Problem and objectives

During busy hours, staff may miss calls or messages. Customers may have to wait for answers about prices, availability, or pickup times. Staff may also record incomplete orders if requests arrive through several channels.

Tube Coffee gives customers a self-service ordering website and lets staff run the shop from one dashboard. The storefront, chat assistant, and voice assistant use the same menu and order data. If an admin changes a price or marks a drink sold out, every channel should use the updated information.

## 2. First release

The proposed first release supports one shop, pickup orders, ABA PayWay sandbox payments, website chat, and a browser-based voice demo. Test the assistants in Khmer and English. Connect a real shop number once the team confirms access to a compatible phone or SIP provider.

Delivery, multiple branches, loyalty points, and production bank payments can follow after the core capstone demo works.

## 3. Technology stack

| Part | Technology | Purpose |
| --- | --- | --- |
| Frontend | React, Vite, React Router, JavaScript, Tailwind CSS | Customer website and admin dashboard |
| Backend | Node.js and Express | REST API, authentication, order logic, payment verification, and AI integration |
| Database | PostgreSQL and Prisma | Store menu, accounts, orders, payments, and conversations |
| Payments | ABA PayWay sandbox | Test checkout and verify payment results |
| AI chat | Language model API with tool calling | Answer shop questions and route conversations to staff |
| AI voice | Realtime voice API and browser microphone | Talk with customers and collect pickup requests |
| Live calls | SIP provider, subject to availability | Route a real shop phone number to the voice assistant |
| Team tools | GitHub, VS Code, Postman | Collaboration, development, and API testing |

Node.js runs the backend JavaScript. Express runs on Node.js and provides the routes that React calls. Use one React app for both the customer website and `/admin` pages. Use one Express backend with separate modules for authentication, menu, orders, payments, chat, voice, and site content.

## 4. Users and permissions

| User | Permissions |
| --- | --- |
| Visitor | Browse the menu, view shop information, and ask general questions |
| Customer | Place orders, pay, view their own order history, and message support |
| Phone caller | Ask about the shop and leave a pickup request |
| Staff | Prepare orders, respond to customers, and review voice requests |
| Admin | Manage menu, prices, availability, site content, staff access, and sales reports |

Express must enforce permissions. Customers can access only their own orders and conversations. Staff and admin functions require a valid account and the appropriate role.

## 5. Customer website requirements

- Show shop hours, location, contact information, menu categories, drink photos, descriptions, sizes, prices, and availability.
- Let customers choose drink options such as sugar or ice when offered. Let them change quantities and remove items from the cart.
- Collect a contact name, phone number, and pickup time. Check that pickup is available at the selected time.
- Recheck prices and drink availability before checkout. Ask the customer to review the cart if either changes.
- Show the payment status, order reference, pickup details, and preparation progress.
- Give customers a chat button and a clear way to request a staff member.
- Support both mobile and desktop layouts.

## 6. Admin dashboard requirements

- Show incoming orders with separate payment and preparation statuses.
- Let staff search orders and update their preparation status.
- Let admins add, edit, archive, and mark menu items sold out. Support categories, photos, prices, sizes, and drink options.
- Let admins edit supported website content such as the homepage banner, hours, contact details, and FAQs.
- Give staff one inbox for customer messages, AI handoffs, and incoming voice requests.
- Show verified paid order counts and sales totals by date.
- Record who made important changes, including order status and menu price updates.

## 7. Order and payment workflow

1. A customer adds drinks to the cart and selects a pickup time.
2. Express validates availability and calculates the order total from current menu prices. It saves each item's name, quantity, options, and price on the order so later menu changes preserve the original order details.
3. Express creates a unique payment attempt and opens ABA PayWay hosted checkout.
4. When PayWay returns a result or sends a callback, Express verifies the transaction with PayWay. It checks that the transaction reference, amount, and currency match the saved attempt before marking it paid.
5. The paid order appears in the kitchen queue. Staff move it from `RECEIVED` to `PREPARING`, then `READY` and `COMPLETED`.
6. If payment is pending or fails, show the status to the customer. Allow a retry on the same order. Repeated callbacks must produce only one successful payment update.

Keep payment states such as `PENDING`, `PAID`, `FAILED`, and `EXPIRED` separate from preparation states.

[ABA PayWay developer documentation](https://developer.payway.com.kh/)

## 8. AI chat requirements

The chat assistant identifies itself as automated. It handles common questions when staff are busy or offline and should understand common requests in Khmer and English.

| Customer question | Expected behavior |
| --- | --- |
| How much is a large iced latte? | Look up the current price from the menu service |
| Can I pick up my order now? | Check the signed-in customer's order and report its current status |
| What time do you close? | Read the shop hours stored by staff |
| Can I get a refund? | Create a staff handoff with the conversation context |
| Does this drink contain peanuts? | Use approved ingredient information or ask staff to confirm |

Save conversations so staff can see what the customer already asked. Pause AI responses when staff take over. If the AI service is unavailable, let customers leave a message for staff.

Start with a hosted model, instructions, approved FAQs, and backend tools. Evaluate the assistant on Khmer and English conversations. If it repeatedly struggles with tone or common conversation patterns, the team can test fine-tuning with reviewed examples that exclude personal and payment data. Current prices, availability, and order status always come from Express.

## 9. AI phone and voice workflow

1. The assistant tells the caller it is Tube Coffee's automated assistant.
2. It asks what drinks the customer wants, including size, quantity, options, and preferred pickup time.
3. It checks the menu and asks follow-up questions when a drink or option is unclear.
4. It reads back the request and asks the caller to confirm.
5. Express saves the items, pickup time, callback details, confirmation status, and a short summary in the staff dashboard.
6. Staff review the request. They can contact the customer, decline it, or accept it and create one linked order.
7. If the call ends before confirmation, label the request `UNCONFIRMED`. If online payment is required, use the normal checkout flow.

Example staff note:

![Tube Coffee system architecture](docs/images/https://res.cloudinary.com/t6wyjwqt/image/upload/v1794985098/mermaid-diagram.png)

> Two large iced lattes, one with 25% sugar. Preferred pickup at 3:30 p.m. The caller confirmed the details. Awaiting staff review.

Test the workflow with a browser microphone first. Then connect a shop number if the carrier and SIP provider support routing calls to the voice service. Test background noise, mixed Khmer and English drink names, order corrections, and dropped calls.



## 10. Database and API plan

| Data | What to store |
| --- | --- |
| Users | Name, contact, password hash, role |
| Categories, products, variants | Category, name, description, photo, size, options, price, currency, availability |
| Orders and order items | Customer, pickup time, item snapshots, total, payment status, preparation status |
| Payment attempts | Order ID, unique transaction reference, amount, currency, status, verification time |
| Shop settings and FAQs | Hours, location, contact details, banner, staff-approved answers |
| Conversations and messages | Participants, staff assignment, message history, conversation state |
| Call requests | Structured drinks, callback details, pickup time, confirmation, summary, linked order |
| Audit events | Who changed what and when |

| Proposed API | Purpose |
| --- | --- |
| `GET /api/menu` | Read the current menu |
| `POST /api/auth/login` | Sign in |
| `POST /api/orders` | Submit a pickup order |
| `GET /api/orders/:id` | Read an order the customer owns |
| `POST /api/orders/:id/payments` | Start or retry checkout |
| `POST /api/webhooks/payway` | Receive payment events for backend verification |
| `POST /api/conversations/:id/messages` | Send a chat message |
| `PATCH /api/admin/menu/products/:id` | Update a menu item as an admin |
| `PATCH /api/admin/orders/:id/status` | Update preparation status as staff |
| `GET /api/admin/call-requests` | List voice requests for staff review |
| `POST /api/admin/call-requests/:id/accept` | Accept a voice request and create its linked order |

## 11. Security, privacy, and reliability

- Keep database, payment, and AI credentials on the backend. Commit only `.env.example` placeholders.
- Validate input in Express. Hash passwords, protect sessions, and use HTTPS for deployment.
- Limit AI tools to permitted actions. The backend checks access before returning customer data.
- Keep raw call audio storage disabled by default. Save only the contact details and conversation data the workflow needs.
- Never ask for bank passwords, PINs, or one-time codes in chat or by phone.
- Prevent duplicate payment updates and duplicate orders from a single voice request.
- Show recoverable errors when the bank or AI service is unavailable.

## 12. Implementation phases

| Phase | Build | Completion check |
| --- | --- | --- |
| 1. Menu foundation | Set up React, Express, PostgreSQL, admin login, public menu, and menu editing | Staff update a drink and customers see the change |
| 2. Orders | Cart, pickup form, order history, and kitchen dashboard | Staff can move a test order to ready |
| 3. Payments | ABA PayWay sandbox checkout and transaction verification | One payment updates one order, even when callbacks repeat |
| 4. Staff chat | Website messaging and staff inbox | Customer and staff can exchange messages and reload the history |
| 5. AI chat | Menu tools, FAQs, and human handoff | AI gives the updated price and routes a refund question to staff |
| 6. Voice orders | Browser voice demo, order read-back, and staff review | A confirmed spoken order appears with the correct drinks and quantities |
| 7. Release | Editable content, sales reports, deployment, and demo testing | A teammate can run the full demo from the project instructions |

## 13. Team responsibilities and first sprint

Suggested workstreams are customer frontend, admin frontend, backend and database, payment and QA, and AI chat and voice. Assign an owner and reviewer to each workstream. Use GitHub issues for requirements and pull requests for review. Include what changed and how another teammate can verify it.

Use `frontend/` for React and `backend/` for Express and Prisma. Add `.env.example` files for configuration. Record the exact install, database setup, and startup commands once the team has run them successfully.

**First sprint:** Seed three drinks with prices, build `GET /api/menu`, and display it in React. Then add an admin-only price editor and sold-out toggle. The sprint is complete when an admin changes the iced latte and the public menu reflects the change.

## 14. Final capstone demo

1. Change a drink price in the admin dashboard and show it update on the customer menu.
2. Place a pickup order, pay in ABA sandbox, and move the order to ready.
3. Ask AI chat for the updated price, then request a refund to demonstrate staff handoff.
4. Speak an order, correct a quantity before confirming, and show the structured request in the staff dashboard.
5. Demonstrate one payment retry, one unauthorized order lookup, and one call that ends before confirmation.