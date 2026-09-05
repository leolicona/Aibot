# Technical feasibility: [NEEDS CLARIFICATION: product name - "Aibot" is a repository placeholder]

Date: 2026-09-05
Brief: [.specify/memory/projectbrief.md](.specify/memory/projectbrief.md)
Appetite: one week at 4 h/day (~28 h), extendable to two weeks at 5 h/day if needed
Run-cost ceiling: none defined in the brief — cost is charged on to the business

> **Sources.** Meta's and Cloudflare's documentation domains are blocked by this
> environment's egress policy. Meta's pricing page was supplied by the developer and is
> treated as vendor evidence; everything sourced to a third-party page is marked as a
> **lead, not evidence**, and still needs confirmation. An earlier draft of this document
> claimed, on the strength of several third-party posts, that WhatsApp service messages
> become billable on 2026-10-01. **Meta's own page refutes that** — see assumption 2.

## Hypothesis
A Cloudflare Worker receives WhatsApp Cloud API webhooks, answers from the content the
business uploaded (Vectorize + D1/R2), generates the reply with a language model, and
drives outbound sends with Queues and Durable Objects — inside Meta's limits and at a
cost per business below the price charged for the product.

## Stack in play
- **Boring here:** Workers, D1, KV, R2, Durable Objects, Workers AI, Vectorize, Queues —
  all already in production for this developer, failure modes known.
- **New here:** the WhatsApp Business Platform (Cloud API). The whole innovation budget
  goes to one piece, which is the healthiest possible split for a one-week MVP.
- **Ruled out to operate:** a VPS or any self-administered server; a third-party BSP
  platform (Twilio, 360dialog) as the messaging layer.

## Load scenario
A mass outage at the pilot ISP: customers lose their connection and write within minutes,
asking the same question, before the business has posted the incident. The developer's
intended behaviour: answer honestly, serve repeats of the same question from a cache,
notify the business so it can publish the reason, then answer. Each inbound message is
free and opens a 24-hour customer service window; the cost of the surge is model calls and
tier headroom, not Meta message charges.
[NEEDS CLARIFICATION: how many customers does the pilot ISP have, and how many would write in the first 10 minutes?]
[NEEDS CLARIFICATION: acceptable time to first reply - under 5 s / under 30 s?]

## Assumptions
| # | Assumption | State | Evidence (URL + date) | Spike |
| --- | --- | --- | --- | --- |
| 1 | Meta approves the number and the business in time to matter | unresolved | Business verification and display-name review gate production sending and the messaging tier; no vendor-stated turnaround could be read. [360dialog](https://docs.360dialog.com/docs/waba-management/capacity-quality-rating-and-messaging-limits), 2026-09-05 — lead | none — a lead-time item, not a measurement |
| 2 | Answering a customer costs no Meta message charge | **confirmed on paper** | Charges apply only when a **template** message is delivered. All non-template messages are free and can only be sent inside an open 24-hour customer service window; utility templates delivered inside an open window are also free; inbound messages are never charged. Service conversations have been free since 2024-11-01. The 2026-10-01 rate-card update moves specific markets out of "Rest of" regions — it does not make service replies billable. [Meta pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing), supplied 2026-09-05 | 001 |
| 3 | The AI-provider pricing policy does not apply | **confirmed on paper — settled, on two independent grounds** | The policy covers "AI Providers" as defined in the Business Solution Terms of 2026-01-15: providers of LLMs, generative-AI platforms or **general-purpose AI assistants**. This product is the opposite — a bot answering only from one business's own content. The page states it explicitly: *"This does NOT change how or what Meta charges all other businesses… This includes not being charged for non-template messages sent in an open customer service window."* And by market: it applies to Brazil from 2026-03-11, and applied to EU/EEA countries and Italy only until 2026-05-12. **Mexico is in neither list.** [Meta AI-provider pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/ai-providers), supplied 2026-09-05 | 001 verifies it empirically: a reclassified message would arrive as `category: "general_purpose_ai"`, `billable: true` |
| 7 | Mexico's rates are stable through the pilot | confirmed on paper | Mexico is a standalone market on the rate card (calling code 52, ISO MX), not part of "Rest of Latin America", and MXN is an available billing currency. It is **not** among the markets moving out of a "Rest of" region on 2026-10-01. Meta may change rates only on the first day of a quarter, with at least one month's notice. Mexico's marketing rates were *lowered* on 2025-10-01. [Meta pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing), supplied 2026-09-05 | none |
| 4 | Charging the business for the product is legal and allowed | unresolved, and largely avoidable | Sidestepped if each business owns its WABA and pays Meta directly, leaving you a software fee. Canonical: [Meta terms](https://www.whatsapp.com/legal/meta-terms-whatsapp-business) (blocked here) | none — read the terms |
| 5 | The outage peak fits inside Meta's and Cloudflare's limits | confirmed on paper, Cloudflare side | Cloudflare: 5 min CPU per request on paid (30 s default), 10,000 subrequests default, raisable to 10M; waiting on `fetch` does not count as CPU. [Workers limits](https://developers.cloudflare.com/workers/platform/limits/), [subrequests changelog](https://developers.cloudflare.com/changelog/post/2026-02-11-subrequests-limit/), 2026-09-05 — leads. Meta: a new number starts around 250 conversations/24 h, ~1,000 once verified, then higher tiers; ~80 messages/s standard throughput. [360dialog](https://docs.360dialog.com/docs/waba-management/capacity-quality-rating-and-messaging-limits), 2026-09-05 — lead | 001 |
| 6 | The gap loop ("I don't know" → business feeds the answer → reply) is free | confirmed on paper **if it closes within 24 h** | The reply is a non-template message, free inside the open window. Past 24 hours the window closes and only a template can be sent — an approved utility template, charged when sent outside a window. So the cost of this feature is a function of **how fast the business answers** | 001 |

## Capabilities
| Capability (vendor-neutral) | Decision | Choice | Why | Cost at MVP | Limit to watch |
| --- | --- | --- | --- | --- | --- |
| Receive and verify inbound messages | adopt | WhatsApp Cloud API webhooks → Worker | Direct to Meta, no BSP markup or lock-in | free | signature verification; fast ACK, work off the request path |
| Answer inside the service window | adopt | non-template messages | Free, and covers the whole support use case | free | the 24-hour window is the constraint, not the price |
| Send outbound reminders and payment links | adopt | utility templates | The paid surface. Free if a window happens to be open, charged if not | utility rate × sends outside a window | template approval and category assignment |
| Send promotions with consent | adopt | marketing templates | Always charged; volume tiers do not apply to marketing | marketing rate × sends | consent, and a Click-to-WhatsApp entry opens a 72 h free window |
| Own the WhatsApp account and its bill | **recommend: the business, not you** | one WABA per business, paying Meta directly | Removes assumption 4, keeps messaging cost off your P&L, makes the product a software fee | none to you | each business completes its own verification |
| Store and retrieve business content | reuse | Vectorize + D1/R2 | Already in production here | included in Workers paid | Vectorize index and dimension limits |
| Generate the reply | adopt | Workers AI, or an external model API | Workers AI: 10,000 neurons/day free, then $0.011 per 1,000 neurons | see Cost | neuron cost per model varies sharply; no verified token→neuron table |
| Schedule notifications and reminders | reuse | Queues + Durable Object alarms | Already in production here | included | per-object alarm semantics |
| Deduplicate a surge of identical questions | build | cache keyed by normalised question per business | The differentiator, and the model-cost control during an outage | — | cache staleness during a live incident |
| Gap notification to the business | build | the product's own loop | The honesty mechanism from the brief | free inside 24 h, a template after | see assumption 6 |

## Rejected alternatives
| Option | Considered for | Why not |
| --- | --- | --- |
| VPS (Hetzner, DigitalOcean) | hosting | One developer, one week: administration is time not spent on the product |
| BSP platform (Twilio, 360dialog) | messaging layer | Adds a per-message markup and a second vendor; Cloud API direct is within this developer's reach. Keep as the fallback if approval stalls |
| n8n / Chatwoot | the whole bot | Would not give the multi-business, no-code-per-business property that is success criterion 3 |

## Cost
| Item | At MVP scale | At 10x | Source |
| --- | --- | --- | --- |
| Inbound messages and in-window replies | **free** | free | [Meta pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing), supplied 2026-09-05 |
| Utility templates sent outside a window (reminders, payment links) | Mexico utility rate × sends | volume tiers lower the utility rate as monthly volume grows | same |
| Marketing templates (promotions) | marketing rate × sends; no volume tiers | linear | same |
| Language model — Workers AI | 10,000 neurons/day free, then $0.011 / 1,000 neurons | linear | [Workers AI pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/) via [pricepertoken](https://pricepertoken.com/endpoints/cloudflare/free), 2026-09-05 — lead |
| Language model — external comparison | ~$0.003 per reply at 2,000 input + 200 output tokens on Claude Haiku 4.5 ($1 / $5 per MTok); ~$0.015 for a five-reply conversation | linear | bundled `claude-api` skill, cached 2026-06-24 |
| Cloudflare Workers paid | $5/month base, shared across all businesses | negligible per business | Workers pricing, 2026-09-05 — lead |

**Shape of the bill.** A customer-initiated support conversation costs the model call and
nothing else — on the order of **cents per conversation**, and the surge in the load
scenario is a model-cost and tier-headroom event, not a messaging bill. Money is spent on
the *proactive* half: reminders and payment links that go out with no window open, and
promotions. That is the line to price against, and it is countable per business per month.
Meta may change rates only on the first day of a quarter, with at least one month's notice
— a predictability worth having when the cost is passed on.

Total at MVP: **not computable** — no selling price has been set and the AI-provider policy
is unread. But the earlier reading, that the support half was about to become expensive,
was wrong: it is free.

## Verdict
- **Works** — yes on the half you control. Every Cloudflare piece is already in production
  for you, its limits are far from this workload, and the single new technology is the
  WhatsApp API, the one new piece a one-week MVP can absorb. Unproven: the end-to-end
  slice, which spike 001 measures.
- **Affordable** — **yes, structurally.** Support conversations carry no Meta message
  charge, the AI-provider policy does not reach this product or this market, and Mexico's
  rates cannot move before the next quarter boundary with a month's notice. The paid
  surface is the outbound half — utility templates sent with no open window, and
  marketing — which is countable per business per month. One number is missing to close
  the arithmetic: Mexico's utility and marketing per-message rates, from the rate card.
  And one decision: the price you charge.
- **Maintainable** — yes. One runtime you already operate, one vendor you don't. Exit cost
  sits on the WhatsApp side; keep the transport behind a thin interface and a BSP stays a
  swap rather than a rewrite.

**Residual risk accepted rather than measured:** Meta's approval lead time. It is outside
your control, it is your own premortem's first cause of death, and no spike shortens it —
only starting it today, before writing code, does.

## Proposed spike 001 — walking skeleton
One falsifiable question: *can a message from a real WhatsApp number reach a Worker, be
answered from uploaded content, and get a reply back inside the service window?*
Time box: 4 hours, spent from the appetite. Shape: a thin end-to-end slice, not a component
probe — feasibility dies at the seams. Throwaway by decision, made before it works.
Pass criterion: [NEEDS CLARIFICATION: reply within how many seconds?] with zero messages
lost, the gap-notification path exercised once, and the `pricing` object in the status
webhook confirming `billable: false` for an in-window reply.

## Open questions
- [ ] [NEEDS CLARIFICATION: Mexico's per-message utility and marketing rates - read the MXN or USD rate card?]
- [ ] [NEEDS CLARIFICATION: what will a business be charged - a monthly software fee / per conversation / both?]
- [ ] [NEEDS CLARIFICATION: who owns the WhatsApp account and pays Meta - the business directly / you as intermediary?]
- [ ] [NEEDS CLARIFICATION: how many customers does the pilot ISP have, and the expected 10-minute surge?]
- [ ] [NEEDS CLARIFICATION: acceptable time to first reply - under 5 s / under 30 s?]
- [ ] [NEEDS CLARIFICATION: how many reminders and payment links go out per business per month with no open window - this is the whole variable bill?]
- [ ] [NEEDS CLARIFICATION: which model generates the reply - Workers AI / an external API?]

## Next step
`/speckit-plan` — hand it the capability decisions, the limits and the load scenario as
Technical Context. Before that: read the AI-provider pricing policy, and start business
verification. Both are reading and paperwork, and both outrank writing code this week.
