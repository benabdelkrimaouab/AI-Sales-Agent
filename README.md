# AI Sales Agent (n8n)

I built this because I was tired of seeing the same problem everywhere: a lead fills out a form, and then... nothing happens for hours. Sometimes days. By then they've already talked to someone else.

So this is an n8n workflow that handles the whole thing automatically — a lead comes in, an AI decides how good it is, and depending on that, it either books them a call or politely puts them on a follow-up list. No human touches it.

## What it actually does

1. Someone submits a form → hits a webhook
2. An AI agent reads their message and scores it 1-10 (I weighted it: 40% budget clarity, 30% urgency, 30% how legit/decisive they sound)
3. If they score 6+, they're "qualified":
   - AI writes them a personalized reply (not a template)
   - The workflow checks my Google Calendar, finds an actual free slot, books it
   - Sales team gets pinged on Telegram
4. If they don't qualify, they get a friendly "thanks, we'll follow up" email instead
5. Either way, it all lands in an Airtable CRM. Same email twice = updates the record instead of duplicating it
6. If anything breaks, a separate error-handling workflow sends me a Telegram alert with what failed and why

## Why I made some of the choices I did

**AI Agent + structured output instead of a simple classifier** — I needed more than a yes/no. I wanted a score, a reason, and a suggested next step, all as clean JSON I could actually use downstream.

**The scoring rubric is explicit in the prompt** — early on I noticed the AI was basically just scoring on budget size alone, which gave weird results (a $3k lead outscoring a $15k one because of vague "vibes"). Spelling out the weights in the system prompt fixed that.

**Real availability checking, not a hardcoded time slot** — my first version just booked everyone for "tomorrow at 10am," which obviously broke the second two leads came in the same day. Now it pulls existing events and scans for the next actually-free 30-minute slot during business hours.

**Airtable upsert on email** — cheap way to get idempotency without overengineering it. Good enough for this use case.

## Stack

n8n, OpenAI, Google Calendar API, Gmail API, Telegram Bot API, Airtable API.

## Testing

I ran it through three cases: a clearly qualified lead, a clearly unqualified one, and one with missing/broken data to make sure the error workflow actually catches failures instead of just... failing silently. All three worked. Also caught (and fixed) a bug where an empty calendar made the whole thing 500 — n8n stops a branch dead if a node returns zero items, so I had to force the calendar-check node to always output something.

## Setup

You'll need your own credentials for: OpenAI, Google Calendar, Gmail, Telegram, Airtable — all connected as standard n8n credentials. The Airtable base needs a `Leads` table with these columns: Name, Email, Phone, Company, Budget, Message, LeadScore, Qualified, Reason, SuggestedNextStep, Status, AppointmentLink, CreatedAt, LeadId.

## Could extend this with

- Swapping Airtable for a real CRM (HubSpot, Pipedrive)
- WhatsApp or SMS follow-ups
- Tuning the scoring weights per industry

---

## License

This project was built as a freelance portfolio piece. Feel free to reference the architecture; credentials, API keys, and specific business data are not included.
