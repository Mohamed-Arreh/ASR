# Auto-Shop Reactivation SaaS

A multi-tenant B2B platform that helps independent auto repair shops recover **declined repair work** — the jobs a customer was quoted but never came back for. The app surfaces that lost revenue, scores which customers are most worth re-engaging, and uses an LLM to draft personalized SMS outreach in each shop's own voice, gated behind a human approval step before anything is sent.

Built solo with Next.js, TypeScript, Supabase (PostgreSQL + Row-Level Security), and the Claude API.

> **Status:** Core product is built and working — auth, multi-tenant data isolation, CSV import, lead scoring, AI message generation, and the approval workflow are all functional. SMS delivery (Twilio) and billing are the next integration phases.

---

## The problem it solves

Repair shops routinely recommend work that customers decline in the moment ("I'll come back for those brake pads"). Most of that work is never followed up on — it's lost revenue sitting in the shop's records. This app turns that backlog into a recovery pipeline: import the data, see the pile, and send the right customer the right nudge at the right time.

---

## Key features

- **Multi-tenant architecture** — a single PostgreSQL database serves many shops, with **Row-Level Security (RLS)** policies enforcing strict data isolation so each shop only ever sees its own customers and records.
- **Authentication & protected routes** — email/password auth via Supabase, with a profile trigger that provisions each new user's shop on signup.
- **CSV import pipeline** — shops upload their repair-order data; the app parses it and populates the declined-work queue.
- **Priority scoring engine** — ranks declined work by recovery likelihood (service type, ticket value, recency) so shops focus on the highest-value follow-ups first.
- **AI message generation** — uses the **Claude API** to draft customer-specific outreach referencing the exact customer, vehicle, declined service, and price — written in the shop's configured brand voice and tone.
- **Shop voice configuration** — each shop sets sample messages, tone keywords, and a signature; the config is versioned so shops can iterate on how their AI-drafted messages sound.
- **Approval workflow** — every AI draft lands in an approvals queue for a human to review and approve before sending, keeping a person in the loop.

---

## Tech stack

| Layer | Tech |
|---|---|
| Framework | Next.js (App Router), TypeScript |
| Styling | Tailwind CSS |
| Database | PostgreSQL via Supabase |
| Data isolation | Row-Level Security (RLS) policies |
| Auth | Supabase Auth |
| AI | Claude API (Anthropic) |
| Messaging | Twilio (SMS — integration in progress) |
| Hosting | Vercel |

---

## Architecture overview

1. A shop signs up → a DB trigger provisions its tenant profile.
2. The shop imports repair-order data via CSV → rows land in the declined-work queue, isolated to that tenant by RLS.
3. The scoring engine ranks each declined item by recovery potential.
4. For a chosen item, the app sends the customer/vehicle/service context plus the shop's voice config to the Claude API, which returns a personalized draft message.
5. The draft enters the approvals queue; a human reviews and approves it before it's queued for SMS delivery.

Relevant files: `schema_ddl.sql` (database schema), `rls_policies.sql` (tenant isolation), `profiles_trigger.sql` (signup provisioning), `sample-declined-work.csv` (example import data).

---

## Running locally

```bash
# 1. Install dependencies
npm install

# 2. Add environment variables in .env.local
#    NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
#    NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
#    ANTHROPIC_API_KEY=your-claude-api-key

# 3. Run the dev server
npm run dev
```

Then open [http://localhost:3000](http://localhost:3000). The SQL files in the repo (`schema_ddl.sql`, `rls_policies.sql`, `profiles_trigger.sql`) set up the database schema and policies in a fresh Supabase project.

---

## License

MIT — see [LICENSE](LICENSE).
