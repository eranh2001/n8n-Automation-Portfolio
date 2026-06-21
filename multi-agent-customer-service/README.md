# Multi-Agent Customer Service System

An AI customer service system built in **n8n** that routes incoming customer messages to specialized sub-agents, **Sales, Support, and Billing**, each with its own knowledge base and tools. A central orchestrator decides who should handle each message, delegates it, and relays the answer back to the customer.

Built on n8n Cloud with OpenAI (gpt-4o-mini).

---

## The Problem

Businesses receive a mix of customer questions, such as product inquiries, complaints, and billing issues, that would normally be handled by different departments. A single AI agent trying to handle everything tends to give vague, unfocused answers and mixes responsibilities.

This system mirrors how a real support team works: a coordinator reads each incoming message and routes it to the right specialist, so every customer gets a focused, accurate answer from an agent built for that specific job.

---

## How It Works

```
Customer message
       │
       ▼
  MAIN AGENT  (orchestrator + conversation memory)
       │  decides who should handle this
       ├──────────────┬──────────────┐
       ▼              ▼              ▼
   Sales Agent    Support Agent   Billing Agent
   (reads product (logs a ticket  (reads billing
    knowledge base) for follow-up)  knowledge base)
```

**Main Agent (orchestrator).** Receives the customer message through a chat trigger, keeps conversation memory, and routes each message to the correct specialist using the description of each routing tool. It does not answer questions itself. Its only job is to understand intent and delegate.

**Sales Agent.** Answers product, pricing, and shipping questions by reading the product knowledge base (Google Sheets).

**Support Agent.** Handles complaints and damaged-item reports by logging each issue as a support ticket (appends a row to a Google Sheet) so a human can follow up.

**Billing Agent.** Answers refund, warranty, and payment questions by reading the billing-policies knowledge base (Google Sheets).

Each sub-agent is a **separate workflow** triggered by the orchestrator via n8n's Call Workflow tool. This keeps each specialist isolated, independently testable, and easy to maintain or extend.

---

## Design Decisions

- **Memory lives at the orchestrator, not the sub-agents.** The Main Agent holds the conversation context. The specialists are stateless and handle one request at a time. This keeps the design clean and avoids each sub-agent maintaining redundant state.
- **Routing is driven by tool descriptions.** Rather than hard-coded rules, the orchestrator chooses a specialist based on clear, non-overlapping tool descriptions (products versus complaints versus billing), which makes it reliable even on ambiguous messages.
- **One workflow = one job.** Splitting the specialists into separate workflows follows the principle of keeping each automation focused and independently maintainable.

---

## A Problem I Debugged

While wiring the orchestrator to the sub-agents, the sub-agents kept failing with a "No prompt specified" error. By inspecting the actual execution data flowing between the workflows, I found the orchestrator was passing the customer's question under a field named `query_`, while the sub-agent was reading it from `query`. Matching the field names fixed the routing.

---

## Tech Used

- **n8n Cloud:** workflow orchestration and agent framework
- **OpenAI (gpt-4o-mini):** agent reasoning and routing decisions
- **Google Sheets:** knowledge bases for Sales and Billing, and the support-ticket log
- **Multi-agent / orchestrator pattern:** a coordinator agent calling specialist sub-agents as tools

---

## Demo

The screenshots in this folder show:

- **Architecture:** the Main Agent (orchestrator) and the three specialist sub-agents, each labeled.
- **Routing in action:** the chat handling a product question (Sales), a complaint (Support), and a refund question (Billing), each routed to the correct specialist.
- **Real-world result:** a complaint submitted in chat is automatically logged as a support ticket in Google Sheets for human follow-up.

---

## Files

- `MainAgent.json`: the orchestrator workflow
- `SalesAgent.json`, `SupportAgent.json`, `BillingAgent.json`: the three specialist sub-agents
- Screenshots: workflow canvases, chat demos, and the logged ticket

*Note: API keys and credentials have been removed from the exported workflows. To run this yourself, you would connect your own OpenAI and Google Sheets credentials in n8n.*
