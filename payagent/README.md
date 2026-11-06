# PayAgent AI

An invoicing agent for freelancers and small shops, built for the PayPal AI Hackathon 2026.

Type one sentence, such as *"Invoice Ravi ₹4,000 for logo design, due Friday"*. The agent (Google Gemini) turns it into a structured invoice, you confirm it, and a real **PayPal sandbox invoice** is created and published with a payer link and QR code. A dashboard tracks paid and unpaid invoices. You can also ask *"Who hasn't paid yet?"* or say *"Remind Ravi"*.

## What it does
- **AI agent:** Gemini converts free text into JSON (client, line items, currency, due date, intent). Relative dates like "Friday" are resolved to real dates. Intents: create invoice, list unpaid, send reminder.
- **Human in the loop:** every invoice is shown for review before anything is sent to PayPal.
- **PayPal Invoicing API v2 (sandbox):** generate invoice number, create, send (publish), fetch payer link, check status, record payment, send reminder.
- **Dashboard:** paid/unpaid status, outstanding totals, payer link and QR code, refresh from PayPal.
- **Modes are labelled in the UI:** "PayPal: sandbox live / simulated" and "AI: gemini / fallback". Without keys the app still runs, but it says so.

## Run locally
```bash
cd payagent
python3 -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env      # then fill in your keys
uvicorn main:app --reload --port 8000
```
Open http://localhost:8000

### Keys you need
1. **PayPal sandbox:** https://developer.paypal.com > Apps & Credentials > Sandbox > create an app > copy Client ID and Secret into `.env`.
2. **Gemini:** a free key from https://aistudio.google.com > put it in `GEMINI_API_KEY`. If the model name is rejected, change `GEMINI_MODEL`.

## Deploy (Render, free tier)
1. Push this folder to a public GitHub repo (the `LICENSE` file is included).
2. Render > New > Web Service > connect the repo.
3. Build command: `pip install -r requirements.txt`. Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`.
4. Add the environment variables from `.env.example`.

The free tier sleeps when idle and loses memory on restart. The app rebuilds the invoice list from PayPal on the first load, so the dashboard is not empty for judges.

## Demo flow
1. Type *Invoice Ravi ₹4,000 for logo design, due Friday* and review the draft.
2. Create the invoice, then open the payer link or QR code.
3. Pay with a PayPal sandbox buyer account, then click **Refresh** and watch the status turn PAID. (**Mark paid** records an offline payment as a shortcut.)
4. Ask *Who hasn't paid yet?* and *Remind Ravi*.

## Notes and limits
- PayPal sandbox invoices are billed in USD here. INR amounts are converted at `INR_PER_USD` and the original INR amount is written in the invoice note.
- Invoices are created with `send_to_recipient: false`, so no emails go to the (fake) client address.
- Data is held in memory. A production version would add a database and PayPal webhooks.

## Tech
Python, FastAPI, PayPal Invoicing API v2, Google Gemini API, Tailwind CSS, QRCode.js.

## License
MIT
