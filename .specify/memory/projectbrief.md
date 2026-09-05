# Project brief: [NEEDS CLARIFICATION: product name - "Aibot" is a repository placeholder, what is the real name?]

Status: draft
Date: 2026-09-05

## Summary
A WhatsApp bot that answers a business's own customers in natural language using
only that business's information, and automates its outbound messages — payment
links, notifications and scheduled reminders — through generic API and webhook
integrations with the systems the business already runs.

## Problem
Customers ask the same questions over and over: about the business, about a
product or service, about their invoice. Today someone at the business answers
each message by hand from a phone, retyping the same replies, missing messages
outside working hours, and doing the outbound work manually too — sending a
payment link, chasing a reminder. The cost is the operator's time and the
customer's wait.

## Audience
- **End customers of the business**, who write on WhatsApp to resolve something
  with that business: receive and pay a payment link, ask when a service will be
  restored, ask about a product, a booking or an invoice.
- **The business operating the bot**: uploads its own information through an
  interface, defines the notifications and reminders, and answers the gaps the bot
  reports back when it did not know something. It may also send promotional
  messages to customers who have consented.

The first two pilot businesses are an ISP and a local tourism agency. The product
is not built for those verticals — any business is a candidate.

## Success criteria
Measurable and technology-agnostic — an outcome, never a feature.

| # | Criterion | How we measure it | Target |
| --- | --- | --- | --- |
| 1 | Customer conversations are resolved without a human stepping in | Share of conversations that end without the bot reporting a gap to the business. [NEEDS CLARIFICATION: confirm this as the measure, or use manual review of a conversation sample?] Measured on real pilot traffic, after the build week | 9 of every 10 |
| 2 | Scheduled outbound messages (notifications, reminders, payment links) leave without anyone sending them by hand | Share of scheduled sends completed automatically vs. sent manually | 100% |
| 3 | Onboarding a new business requires no code written for that business | Second business goes live through configuration plus API/webhook setup only | Yes / no |

## Out of scope
- Validating the transfer or payment itself — that stays with the external
  payment platform; the bot sends the link and relays the outcome.
- Any channel other than WhatsApp — no Instagram, Messenger or web chat.
- Answering anything outside the business's own information.

## Appetite, capacity and cost
- **Appetite:** one week of development with AI agents. It buys a pilot working
  end to end with one business, not a measured 90% resolution rate.
  [NEEDS CLARIFICATION: is there a committed external date with a pilot business - yes/no?]
- **Capacity:** one developer, 4 hours a day (~28 h/week). The agents carry the
  bulk of the implementation; the developer decides scope and reviews.
- **Run cost:** WhatsApp API, a language model, and Cloudflare. Charged on to the
  business as part of the product.
  [NEEDS CLARIFICATION: monthly run-cost ceiling per business - no number defined?]

## Timeline
| Milestone | Target date | What it proves |
| --- | --- | --- |
| Pilot live with one business | End of the appetite week [NEEDS CLARIFICATION: start date?] | The whole loop works: inbound question answered, payment link sent, reminder fired |
| First real traffic measured | After the pilot has run [NEEDS CLARIFICATION: how long?] | Criteria 1 and 2 hold with real customers |
| Second business onboarded | [NEEDS CLARIFICATION: when?] | Criterion 3: no business-specific code |

## Risks and dependencies
| Risk or dependency | Impact | Mitigation |
| --- | --- | --- |
| Meta does not approve the number or the business account | Fatal — no channel, no product; entirely outside the developer's control and can consume the whole appetite week | Start verification before writing code; keep the build testable without a production number |
| The business does not keep its uploaded information current | Answers go stale or the bot reports gaps constantly; criterion 1 becomes unreachable | Make the gap notification the prompt to update; a business that ignores it sees its own resolution rate fall |
| Time-sensitive events are not in the uploaded content | The flagship case — a customer without Internet asking when service returns — has no answer unless the business posts the outage and its estimate | The business posts the event through the same interface; otherwise the bot answers honestly and reports the gap |
| The bot answers from information it does not have | The business loses trust and switches it off | Defined behaviour: only the business's own information, an honest "I don't know", and a notification so the business can feed that answer |
| Criterion 1 cannot be measured inside the appetite | The week ends with no evidence either way | Accepted: the week proves the pilot works, measurement comes with real traffic |

## Assumptions and decisions
| Date | Assumption or decision | Why | Source |
| --- | --- | --- | --- |
| 2026-09-05 | The appetite week buys a working pilot with one business; criteria 1 and 2 are measured on real traffic afterwards, criterion 3 when the second business is onboarded | 28 hours cannot produce a measured 90% resolution rate, and Meta's approval is not in the developer's hands | agent, confirmed by user |
| 2026-09-05 | The bot's knowledge is what the business uploads through an interface, not a live read of the business's systems | Keeps every business onboardable without an integration project | user |
| 2026-09-05 | When the bot does not know, it says so and notifies the business to feed that answer | Honesty over invention, and each gap improves the next answer | user |
| 2026-09-05 | Sending the payment link is in scope; validating the transfer is not | The payment platform already validates and notifies | user |
| 2026-09-05 | Integration with external systems is described as a generic API + webhook capability, not tied to any named service | It has to be a global capability of the product, not a per-business feature | user |
| 2026-09-05 | The product is not ISP-specific | The ISP is simply the first customer | user |
| 2026-09-05 | Criterion 3 restated as an outcome ("no per-business code") instead of "each business has its bot and an API" | The original wording described architecture, not a result that can fail | agent, confirmed by user |
| 2026-09-05 | Brief stored next to the constitution in `.specify/memory/` | Keeps Spec Kit's project-level context together | agent |

## Open questions
- [ ] [NEEDS CLARIFICATION: product name - "Aibot" is a repository placeholder, what is the real name?]
- [ ] [NEEDS CLARIFICATION: how is the 9-in-10 resolution rate measured - share of conversations with no gap reported / manual review of a sample?]
- [ ] [NEEDS CLARIFICATION: monthly run-cost ceiling per business - a number, or purely pass-through?]
- [ ] [NEEDS CLARIFICATION: what does the customer get while a gap is being filled - the business answers them directly / the bot replies once the answer is loaded / nothing in the pilot?]
- [ ] [NEEDS CLARIFICATION: does invoice information also come from uploaded content, or does it need per-customer data the business holds elsewhere?]
- [ ] [NEEDS CLARIFICATION: how is the customer's consent for promotional messages captured and revoked - in the chat / by the business / not in the pilot?]
- [ ] [NEEDS CLARIFICATION: start date of the appetite week, and whether a pilot business has been promised a date?]

## Next steps
1. `/speckit-constitution` — turn appetite, capacity and run cost into project
   principles and constraints.
2. `/speckit-specify` — feed it Summary, Problem, Audience, Success criteria and
   Out of scope; it writes `specs/###-feature/spec.md`.
3. `/speckit-clarify` — works through the open questions above.
4. `tech-feasibility` skill — check the idea is buildable in one week with what
   exists, before `/speckit-plan`.
