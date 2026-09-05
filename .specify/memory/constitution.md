<!--
SYNC IMPACT REPORT
Version change: none (unfilled template) → 1.0.0
Bump rationale: initial ratification. The template carried no principles, so
this is the first governing version rather than an amendment.

Modified principles: none — all five are new:
  - I. Only the Business's Own Information (NON-NEGOTIABLE)
  - II. One Codebase, Many Businesses
  - III. Adopt What Exists, Build Only the Differentiator
  - IV. Every Billable Message Is Counted
  - V. Scope Bends to the Appetite

Added sections:
  - Technical Constraints (replaces [SECTION_2_NAME])
  - Development Workflow (replaces [SECTION_3_NAME])
  - Governance

Removed sections: none.

Sources: .specify/memory/projectbrief.md, feasibility.md.

Deferred TODOs: none. RATIFICATION_DATE is today — the project has no earlier
adoption date, its first commit is 2026-09-04.
-->

# WhatsApp Business Bot Constitution

<!-- The product name is still open: "Aibot" is a repository placeholder.
     Renaming the product does not require a version bump of this document. -->

## Core Principles

### I. Only the Business's Own Information (NON-NEGOTIABLE)

The bot MUST answer only from content the operating business supplied. It MUST NOT
answer from general knowledge, and it MUST NOT infer, extrapolate or invent a fact
about a business, a product, a service or an invoice.

When the uploaded content does not contain the answer, the bot MUST say so plainly to
the customer and MUST notify the business so it can supply that answer. An honest
"I don't know" plus a gap report is a correct outcome, not a failure.

Every answer MUST be traceable to stored content. An answer that cannot name its
source is a defect, whatever its quality.

*Rationale: the product's value is that a customer can trust what it says. One
invented invoice or outage estimate and the business switches it off — that is the
failure mode this principle exists to prevent, and it outranks coverage.*

### II. One Codebase, Many Businesses

Onboarding a business MUST be configuration plus API and webhook setup. No branch,
file, module, environment variable or deployment may be named after a customer, and
no conditional may test which customer is being served.

Anything a business needs to change about its own behaviour MUST be data it can edit,
not code someone deploys.

*Rationale: this is success criterion 3 of the brief stated as a rule. A product that
needs an engineer per customer is a consultancy, and it cannot reach the second
business inside any workable appetite.*

### III. Adopt What Exists, Build Only the Differentiator

Messaging transport, storage, retrieval, queueing and scheduling MUST be adopted from
the platform rather than written. What is built here is the answer loop over the
business's content, the gap-notification mechanism, and the surge cache.

Before any capability is built, the feasibility record MUST name what already solves
it and why that was rejected.

*Rationale: an MVP affords roughly one new technology, and it is already spent on the
WhatsApp Business Platform. Every additional invention is drawn from the same week.*

### IV. Every Billable Message Is Counted

Outbound sends fall into two classes and the system MUST always know which: free
non-template messages inside an open customer service window, and billable template
messages. Meta's `pricing` object in the status webhook is the source of truth, not
the code's own belief about what it sent.

No feature that can send a billable message may ship without recording the send, its
category, and the business it belongs to. Cost per business MUST be answerable from
stored data at any time, without a support ticket to the vendor.

*Rationale: the developer's own abandon criterion is cost per conversation exceeding
revenue. A criterion that cannot be measured cannot be met, and cost passed on to a
customer must be explainable to that customer.*

### V. Scope Bends to the Appetite

The appetite is a budget, not an estimate. When the work does not fit, scope is cut —
never the appetite extended by default — and what was cut MUST be recorded.

Something whole and small ships over something broad and unfinished. A spike is
throwaway, and that decision is made before it works, never after.

*Rationale: one developer at four hours a day has no slack to recover a week spent
half-finishing three things.*

## Technical Constraints

- **Runtime:** Cloudflare only — Workers, D1, KV, R2, Durable Objects, Queues,
  Vectorize, Workers AI. No self-administered server is operated.
- **Messaging:** WhatsApp Cloud API directly, without a BSP intermediary. The
  transport MUST sit behind a thin interface so that adopting a BSP stays a swap
  rather than a rewrite.
- **The 24-hour customer service window is a design constraint, not a detail.** Any
  flow that may need to reply after the window closes MUST be designed with an
  approved template, and that template's cost MUST be accounted for.
- **First market: Mexico.** Rates, and any regulatory policy that applies to a market,
  MUST be re-checked before a business in a new country is onboarded.
- **Vendor facts come from vendor documentation.** A limit, price or policy recorded
  from a third-party page is a lead, and MUST carry its URL, the date checked, and its
  status as unverified until confirmed on the vendor's own page.

## Development Workflow

- Work flows brief → feasibility → `/speckit-specify` → `/speckit-plan` →
  `/speckit-tasks` → `/speckit-implement`. Each artifact feeds the next; none replaces
  another.
- **Unknowns are marked, never invented.** Anything unsettled MUST be written as
  `[NEEDS CLARIFICATION: what is missing - optionA/optionB?]` so `/speckit-clarify`
  and `/speckit-analyze` can find it. A confident paragraph over a gap is a defect.
- A finding that contradicts an earlier document MUST correct that document, and the
  correction MUST say what was wrong. Documents that quietly change lose their value
  as a record.
- Every change reaches `main` through a pull request whose description states what it
  changes and what stayed open.

## Governance

This constitution supersedes other practices in this repository. Where a principle and
a convenience conflict, the principle wins or the constitution is amended — not
ignored.

**Amendment procedure.** Amendments are made through `/speckit-constitution`, take
effect when merged to `main`, and MUST carry a Sync Impact Report recording what
changed and why. The developer is the sole decision-maker; no external approval is
required or implied.

**Versioning policy.** Semantic versioning of governance:
- **MAJOR** — a principle is removed or redefined in a way that invalidates work done
  under the previous reading.
- **MINOR** — a principle or section is added, or its guidance materially expanded.
- **PATCH** — clarification, wording, or a correction that does not change what is
  required.

**Compliance review.** Every pull request is checked against these principles, and
added complexity MUST be justified in its description. A violation that is deliberate
MUST be recorded as such rather than left for a reader to discover.

**Version**: 1.0.0 | **Ratified**: 2026-09-05 | **Last Amended**: 2026-09-05
