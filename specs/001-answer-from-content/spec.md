# Feature Specification: Answer Customer Questions from Business Content

**Feature Branch**: `001-answer-from-content`

**Created**: 2026-09-05

**Status**: Draft

**Input**: User description: "A customer of a business writes to that business's WhatsApp number. The bot answers in natural language using only the content the business has uploaded through its interface. When the uploaded content does not contain the answer, the bot tells the customer honestly that it does not know and notifies the business so it can supply that answer; once the business supplies it, the bot can answer. Repeated identical questions during a surge (for example a mass outage at an ISP) are answered from a cache instead of generating each reply again. Every answer must be traceable to stored content. This is the walking skeleton for the pilot: inbound customer question → answer from business content → honest unknown plus gap notification. Out of scope for this feature: sending payment links, scheduled reminders and notifications, promotional messages, and the interface the business uses to upload content (assume content already exists)."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - A customer gets an answer from the business's own content (Priority: P1)

A customer of a business sends a question to that business's WhatsApp number — "¿Cuál es el horario de atención?", "¿Cuánto cuesta el plan de 100 megas?", "¿Cuándo sale el tour a Bacalar?". The bot reads the content the business has already uploaded, finds the part that answers the question, and replies in natural language, in the customer's language, within seconds. The customer gets the same answer a well-briefed employee would have given, at any hour, without anyone at the business touching their phone.

**Why this priority**: This is the product. Without it there is nothing to show the pilot business, and every other story only makes sense once this one works.

**Independent Test**: Load a small set of content for one business (hours, prices, a policy), send five questions those items answer from a customer's phone, and check that each reply is correct, arrives within the time target, and names the content item it came from.

**Acceptance Scenarios**:

1. **Given** the business has uploaded content that states its opening hours, **When** a customer asks what time the business opens, **Then** the customer receives a reply stating those hours, and the reply is recorded together with the content item it was drawn from.
2. **Given** the business has uploaded a price list, **When** a customer asks the price of a listed plan, **Then** the reply states that plan's price exactly as uploaded, with no rounding, no estimate and no addition.
3. **Given** a customer writes in Spanish, **When** the bot replies, **Then** the reply is in Spanish; **Given** a customer writes in English, **Then** the reply is in English.
4. **Given** the customer has asked a follow-up that only makes sense with the previous message ("¿y los sábados?" after asking about hours), **When** the bot replies, **Then** the reply reflects the earlier context of that conversation.
5. **Given** two businesses each with their own number and their own content, **When** a customer writes to one of them, **Then** the reply draws only from that business's content, never from the other's.

---

### User Story 2 - The bot admits what it does not know, and the business fills the gap (Priority: P2)

A customer asks something the uploaded content does not cover — "¿Aceptan pago con tarjeta American Express?" when the content never mentions payment methods. The bot does not guess. It tells the customer, in plain words, that it does not have that information yet. At the same time the business is notified: *a customer asked this, and you have not given me an answer.* When the business supplies the answer, the next customer who asks gets it. If a hundred customers ask the same unknown thing during an outage, the business is told once — with a count — not a hundred times.

**Why this priority**: This is the trust mechanism. A single invented answer about an invoice or an outage is the failure the pilot business will remember. Honesty plus a gap report turns every unknown into content, so the bot gets better every day it runs.

**Independent Test**: Ask a question no content item answers, confirm the reply says it does not know and offers nothing invented, confirm the business receives exactly one notification, then add content that answers it and ask again from a second phone to confirm the new answer.

**Acceptance Scenarios**:

1. **Given** no uploaded content addresses a customer's question, **When** the customer asks it, **Then** the reply says the bot does not have that information, contains no invented fact, and does not present general knowledge as the business's answer.
2. **Given** the bot has just replied "I don't know" to a question, **When** that reply is sent, **Then** the business receives one notification naming the question and the number of customers who asked it.
3. **Given** the same unknown question arrives from many customers before the business has answered, **When** those messages are processed, **Then** each customer receives the honest-unknown reply, and the business receives no additional notification — the existing one's count rises instead.
4. **Given** the business supplies content that answers a previously reported gap, **When** a customer asks that question afterwards, **Then** the reply comes from the new content and the gap is marked as answered.
5. **Given** the business supplies the answer to a gap, **When** the customer who originally asked is considered, **Then** [NEEDS CLARIFICATION: what does the original customer receive once the gap is filled - nothing, the bot answers the next question only / the bot sends the answer to every customer who asked, if their conversation window is still open / the business contacts the customer directly, outside the bot?]

---

### User Story 3 - A surge of the same question is answered once and delivered to everyone (Priority: P3)

The ISP's network goes down in one neighbourhood. Within ten minutes, hundreds of customers write the same thing: "¿No hay internet?", "se cayó el servicio", "cuándo vuelve". The business has (or has not yet) posted the incident. The bot recognises these as the same question, produces the answer once, and delivers it to every customer who asks — fast, and without paying to generate the same reply hundreds of times. When the business updates the content ("service restored at 18:30"), the cached answer is discarded and the new one takes its place.

**Why this priority**: This is the moment of highest stress for the product, and the one the pilot ISP will judge it by. It is third only because Stories 1 and 2 must exist before there is anything to cache.

**Independent Test**: Send the same question from 50 phones within one minute, confirm every phone gets a reply with the same content, confirm the answer was produced once, then change the relevant content and confirm the next reply reflects the change.

**Acceptance Scenarios**:

1. **Given** many customers send the same question in a short period, **When** the bot answers them, **Then** every customer receives a reply, all replies carry the same answer, and the answer was produced once for the whole group.
2. **Given** an answer is being served from cache, **When** the business changes the content that answer was drawn from, **Then** the cached answer is no longer used and the next reply reflects the changed content.
3. **Given** a surge of an *unknown* question, **When** customers ask it, **Then** each gets the honest-unknown reply and the business receives a single notification with a rising count (as in Story 2).

---

### Edge Cases

- A customer sends a greeting, a sticker, an image, a voice note or a document rather than a text question. The bot replies that it can currently help with written questions and says what it can help with.
- A customer puts two questions in one message. Both are answered, or the reply says which one it could not answer.
- A customer asks about their own account — their invoice, their balance, their ticket. Business-wide content cannot answer that; the bot treats it as a gap unless the content covers it, and never guesses at a customer's personal data.
- The same inbound message is delivered twice by the messaging platform. The customer receives one reply, not two.
- The business supplies the answer to a gap after the customer's conversation window has closed. The bot does not send a free-form reply outside the window; what happens instead depends on the clarification in Story 2.
- The customer writes in a language the content is not in. The bot answers in the customer's language using the content's meaning.
- The bot cannot reach its content or its answer generation for a moment. The customer receives an honest "I can't answer right now, please try again shortly" — never silence, never an invented answer.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST identify which business an inbound message belongs to from the number it was sent to, and MUST use only that business's content to answer it.
- **FR-002**: The system MUST answer a customer's question using only content the business has uploaded; it MUST NOT answer from general knowledge or infer facts the content does not state.
- **FR-003**: Every reply that states a fact MUST be recorded together with the content item(s) it was drawn from, so that any answer can be traced to its source after the fact.
- **FR-004**: When no uploaded content answers a question, the system MUST reply that it does not have that information, and MUST NOT include any guessed or general-knowledge answer in that reply.
- **FR-005**: When the system replies that it does not know, it MUST record a gap for that business containing the question, when it was first asked, and how many customers have asked it.
- **FR-006**: The system MUST notify the business of each new gap exactly once; subsequent customers asking the same unknown question MUST increase that gap's count rather than produce a new notification. Notifications are delivered via [NEEDS CLARIFICATION: notification channel - a WhatsApp message to a number the business designates / an email to a business address / a list the business reads in its own interface?]
- **FR-007**: When the business supplies content that answers an open gap, the system MUST use that content for subsequent questions and MUST mark the gap as answered.
- **FR-008**: The system MUST recognise when a question is the same as one it has answered recently for the same business, and MUST serve the same answer without producing it again, for as long as the underlying content is unchanged.
- **FR-009**: When content a cached answer was drawn from changes, the system MUST stop serving that cached answer.
- **FR-010**: The system MUST reply in the language the customer wrote in.
- **FR-011**: Within one customer's conversation, the system MUST take the recent preceding messages into account so that follow-up questions are understood in context.
- **FR-012**: The system MUST reply to a customer's message at most once, even if the messaging platform delivers the same message more than once.
- **FR-013**: For a message that is not a written question (greeting, media, voice), the system MUST reply that it can currently help with written questions and MUST NOT attempt to answer content it cannot read.
- **FR-014**: If the system cannot produce an answer because of a temporary fault, it MUST tell the customer it cannot answer right now, MUST NOT send an invented answer, and MUST record the fault.
- **FR-015**: The system MUST record, for every message it sends, whether that message was free or billable as reported by the messaging platform, and which business it belongs to.
- **FR-016**: The system MUST only send a free-form reply to a customer while that customer's conversation window is open; it MUST NOT attempt one after the window has closed.
- **FR-017**: Adding a second business MUST require only configuration and content — no change to the system's behaviour that is specific to that business.
- **FR-018**: The system MUST store only what it needs about a customer: their number and the messages of the current conversation, kept for [assumed 90 days, see Assumptions] and no longer.

### Key Entities

- **Business**: An operator of the bot. Has a WhatsApp number customers write to, a set of uploaded content, and a way of being notified. Everything else in this feature belongs to exactly one business.
- **Content Item**: A unit of information the business uploaded — a paragraph, a price entry, a policy, an incident notice. Has a business, a body, and a last-changed time. It is the only source an answer may cite.
- **Conversation**: The exchange between one customer and one business, identified by the customer's number and the business's number. Has an open/closed window state and the recent messages.
- **Message**: One inbound or outbound message in a conversation. Outbound messages record which content items they were drawn from and whether the platform billed them.
- **Gap**: A question the content could not answer, for one business. Has the question as first asked, first-asked time, a count of customers who asked, a notified flag, and a state of open or answered.
- **Cached Answer**: An answer produced for a question, kept for reuse. Tied to the business, the question, the content items it drew from, and invalidated when any of those items changes.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Of customer questions the uploaded content does answer, 9 in every 10 receive a correct reply with no human involvement, measured over the pilot's first four weeks of real traffic by review of a weekly sample of 50 conversations.
- **SC-002**: 95% of customer messages receive a reply within 10 seconds of being sent, at any hour.
- **SC-003**: 100% of replies that state a fact can be traced to at least one content item; a reviewer checking a sample of 50 replies finds none without a source.
- **SC-004**: Of questions the content does not answer, 100% receive an honest-unknown reply, and each distinct unknown question produces exactly one business notification, regardless of how many customers asked.
- **SC-005**: When the same question is asked by 100 customers within 10 minutes, all 100 receive a reply, the replies carry the same answer, and the answer was produced once.
- **SC-006**: After the business supplies the answer to a reported gap, the next customer to ask receives it — the gap-to-answer path works end to end in 100% of tested cases.
- **SC-007**: A second business can be brought live using only configuration and content, with no change to the system, demonstrated once during the pilot.
- **SC-008**: Every outbound message has a recorded free/billable status; the count of billable messages per business per month can be produced from stored data alone.

## Assumptions

- **Content already exists.** The interface the business uses to upload and edit content is a separate feature. This feature assumes each business has content available to read, and that the system can tell when a content item changed.
- **One number, one business.** A WhatsApp number belongs to exactly one business, and that mapping is how inbound messages are routed.
- **Customers write in Spanish, mostly.** The pilot businesses are in Mexico. The bot answers in whatever language the customer used; content may be in Spanish only.
- **Business-wide content only.** Questions about a specific customer's own account (invoice, balance, ticket) are out of scope for this feature and are handled as gaps; per-customer data is a later feature.
- **"Same question" means the same meaning.** Two questions are treated as the same when they ask the same thing in the same or substantially the same words; minor differences in punctuation, accents, capitalisation and filler do not make them different.
- **Reply time target: 10 seconds.** No target was set in the brief or feasibility record; 10 seconds is adopted as what a customer waiting in a chat tolerates before assuming nobody is there.
- **Conversation context: the current conversation window.** The bot takes into account messages exchanged in the current 24-hour window; it does not recall earlier conversations.
- **Data retention: 90 days.** Conversation messages and customer numbers are kept for 90 days for review and traceability, then removed. Gaps and their counts are kept while the business is active.
- **No live human handover in this feature.** The gap notification is the escalation path; a person taking over a live conversation is a later feature.
- **The bot does not reply outside a conversation window.** Replies after the window closes would need an approved template, which is out of scope here.
- **Constitution applies.** Principle I (only the business's content, honest unknown), Principle II (no per-business code), and Principle IV (every billable message counted) are treated as requirements of this feature, not as guidance.
