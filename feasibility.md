# Technical feasibility: [NEEDS CLARIFICATION: product name - "Aibot" is a repository placeholder]

Date: 2026-09-05
Brief: [.specify/memory/projectbrief.md](.specify/memory/projectbrief.md)
Appetite: one week at 4 h/day (~28 h), extendable to two weeks at 5 h/day if needed
Run-cost ceiling: none defined in the brief — cost is charged on to the business

> **Research limitation, stated up front.** Meta's and Cloudflare's own documentation
> domains are blocked by this environment's egress policy, so nothing below was read
> from a vendor's own page. Every figure comes from third-party sources dated
> 2026-09-05 and is a **lead, not evidence**. The canonical URL is given for each so
> the developer can confirm it in one sitting — that confirmation is the remaining
> half of stage B, and two of the numbers decide the project.

## Hypothesis
A Cloudflare Worker receives WhatsApp Cloud API webhooks, answers from the content the
business uploaded (Vectorize + D1/R2), generates the reply with a language model, and
drives outbound sends with Queues and Durable Objects — inside Meta's limits and at a
cost per conversation below the price charged to the business.

## Stack in play
- **Boring here:** Workers, D1, KV, R2, Durable Objects, Workers AI, Vectorize, Queues —
  all already in production for this developer, failure modes known.
- **New here:** the WhatsApp Business Platform (Cloud API). The whole innovation budget
  goes to one piece, which is the healthiest possible split for a one-week MVP.
- **Ruled out to operate:** a VPS or any self-administered server; a third-party BSP
  platform (Twilio, 360dialog) as the messaging layer.

## Load scenario
A mass outage at the pilot ISP: customers lose their connection and write within minutes,
asking the same question, before the business has posted the incident. Each inbound
message opens a billable exchange and a model call at once. The developer's intended
behaviour: answer honestly, serve repeats of the same question from a cache, notify the
business so it can publish the reason, then answer.
[NEEDS CLARIFICATION: how many customers does the pilot ISP have, and how many would write in the first 10 minutes?]
[NEEDS CLARIFICATION: acceptable time to first reply - under 5 s / under 30 s?]

## Assumptions
| # | Assumption | State | Evidence (URL + date) | Spike |
| --- | --- | --- | --- | --- |
| 1 | Meta approves the number and the business in time to matter | unresolved | Business verification and display-name review gate production sending and the messaging tier; no vendor-stated turnaround could be read. [360dialog](https://docs.360dialog.com/docs/waba-management/capacity-quality-rating-and-messaging-limits), 2026-09-05 | none — a lead-time item, not a measurement |
| 2 | Cost per conversation stays below the price charged | **unresolved, and the state changes within weeks** | Per-message billing since 2025-07-01. **From 2026-10-01 service messages — the free-form replies inside the 24 h window, which is exactly what this bot sends — become billable at the same per-country rate as utility templates, with no volume discount.** Rates published by 2026-09-01. [SendPulse](https://sendpulse.com/blog/whatsapp-service-message-pricing), [Wati](https://www.wati.io/en/blog/whatsapp-service-message-pricing/), [ChakraHQ](https://chakrahq.com/article/whatsapp-api-pricing-update-service-messages-october-2026/), 2026-09-05. Canonical: [Meta pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing) (blocked here) | none — read the rate card |
| 3 | Charging the business for messaging is legal and allowed | unresolved | Providers routinely pass through or mark up Meta's charges, but this developer is not a BSP. Sidestepped entirely if each business owns its own WABA and pays Meta directly (see Capabilities). Canonical: [Meta terms](https://www.whatsapp.com/legal/meta-terms-whatsapp-business) (blocked here) | none — read the terms |
| 4 | The outage peak fits inside Meta's and Cloudflare's limits | confirmed on paper, Cloudflare side | Cloudflare: 5 min CPU per request on paid (30 s default), 10,000 subrequests default, raisable to 10M; waiting on `fetch` does not count as CPU. [Workers limits](https://developers.cloudflare.com/workers/platform/limits/), [subrequests changelog](https://developers.cloudflare.com/changelog/post/2026-02-11-subrequests-limit/), 2026-09-05. Meta: a new number starts at ~250 conversations/24 h, ~1,000 once verified, then 10k/100k tiers; ~80 messages/s standard throughput. [360dialog](https://docs.360dialog.com/docs/waba-management/capacity-quality-rating-and-messaging-limits), [Chatarmin](https://chatarmin.com/en/blog/whats-app-messaging-limits), 2026-09-05 | 001 |
| 5 | The 24 h window supports "I don't know → the business feeds the answer → I reply" | unresolved | Free-form replies are only allowed inside the open 24 h service window. If the business fills the gap later than that, the reply must go as an approved template, which is a different build and a different price | 001 |

## Capabilities
| Capability (vendor-neutral) | Decision | Choice | Why | Cost at MVP | Limit to watch |
| --- | --- | --- | --- | --- | --- |
| Receive and verify inbound messages | adopt | WhatsApp Cloud API webhooks → Worker | Direct to Meta, no BSP markup or lock-in | per message, see Cost | signature verification; fast ACK, work off the request path |
| Send outbound messages | adopt | WhatsApp Cloud API | same | per message | 24 h window; template approval outside it |
| Own the WhatsApp account and its bill | **recommend: the business, not you** | one WABA per business, paying Meta directly | Removes assumption 3 entirely, keeps messaging cost off your P&L, and makes the product a software fee rather than a resale | none to you | each business must complete its own verification |
| Store and retrieve business content | reuse | Vectorize + D1/R2 | Already in production here | included in Workers paid | Vectorize index and dimension limits |
| Generate the reply | adopt | Workers AI, or an external model API | Workers AI: 10,000 neurons/day free, then $0.011 per 1,000 neurons | see Cost | neuron cost per model varies sharply; no verified token→neuron table |
| Schedule notifications and reminders | reuse | Queues + Durable Object alarms | Already in production here | included | per-object alarm semantics |
| Deduplicate a surge of identical questions | build | cache keyed by normalised question per business | The differentiator, and the cost control during an outage | — | a cached answer must still be sent as a message, and each send bills |
| Gap notification to the business | build | the product's own loop | The honesty mechanism from the brief | — | see assumption 5 |

## Rejected alternatives
| Option | Considered for | Why not |
| --- | --- | --- |
| VPS (Hetzner, DigitalOcean) | hosting | One developer, one week: administration is time not spent on the product |
| BSP platform (Twilio, 360dialog) | messaging layer | Adds a per-message markup and a second vendor; the developer is capable of Cloud API directly. Keep as the fallback if Cloud API approval stalls |
| n8n / Chatwoot | the whole bot | Would not give the multi-business, no-code-per-business property that is success criterion 3 |

## Cost
| Item | At MVP scale | At 10x | Source |
| --- | --- | --- | --- |
| WhatsApp messages | `R × (bot messages per conversation)`, where `R` is Meta's per-message rate for the recipient's country. Free today inside the 24 h window; **billable from 2026-10-01** | linear, no volume discount on service messages | [SendPulse](https://sendpulse.com/blog/whatsapp-service-message-pricing), 2026-09-05 |
| Language model — Workers AI | 10,000 neurons/day free, then $0.011 / 1,000 neurons | linear | [Workers AI pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/) via [pricepertoken](https://pricepertoken.com/endpoints/cloudflare/free), 2026-09-05 |
| Language model — external comparison | ~$0.003 per reply at 2,000 input + 200 output tokens on Claude Haiku 4.5 ($1 / $5 per MTok); ~$0.015 for a five-reply conversation | linear | bundled `claude-api` skill, cached 2026-06-24 |
| Cloudflare Workers paid | $5/month base, shared across all businesses | negligible per business | Workers pricing, 2026-09-05 |

Total at MVP: **not computable** — `R` is unverified and no selling price has been set.
What the shape already tells you: after 2026-10-01 the messaging charge is per **message**,
so **every extra bubble the bot sends costs money**. One dense answer beats three chatty
ones, and that is an economic constraint on the product, not a style preference.

## Verdict
- **Works** — yes on the half you control. Every Cloudflare piece is already in production
  for you, its limits are far from this workload, and the single new technology is the
  WhatsApp API, which is the one new piece a one-week MVP can absorb. Unproven: the
  end-to-end slice, which spike 001 measures.
- **Affordable** — **unresolved, and this is the gate.** Your own abandon criterion is
  "cost per conversation above revenue", and the free service window that makes support
  bots cheap ends on 2026-10-01, weeks after your pilot. Until you read Meta's rate card
  for your country and set a price, this project has no verified economics.
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
lost, and the gap-notification path exercised once.

## Open questions
- [ ] [NEEDS CLARIFICATION: Meta's per-message rate for the pilot country after 2026-10-01 - read the rate card?]
- [ ] [NEEDS CLARIFICATION: what will a business be charged - a monthly software fee / per conversation / both?]
- [ ] [NEEDS CLARIFICATION: who owns the WhatsApp account and pays Meta - the business directly / you as intermediary?]
- [ ] [NEEDS CLARIFICATION: how many customers does the pilot ISP have, and the expected 10-minute surge?]
- [ ] [NEEDS CLARIFICATION: acceptable time to first reply - under 5 s / under 30 s?]
- [ ] [NEEDS CLARIFICATION: what happens when the business fills a gap after the 24 h window closes - approved template / drop the reply?]
- [ ] [NEEDS CLARIFICATION: which model generates the reply - Workers AI / an external API?]

## Next step
`/speckit-plan` — hand it the capability decisions, the limits and the load scenario as
Technical Context. Before that: read Meta's rate card and terms, and start business
verification. Both are reading and paperwork, and both outrank writing code this week.
