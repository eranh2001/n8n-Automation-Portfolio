# Multi-Agent Customer Service System

Imagine a customer types one message into a chat box. It could be a question about a product, a complaint about a damaged item, or a request for a refund. Behind the scenes, a coordinator instantly reads that message, decides which department should handle it, and hands it to the right specialist. The customer gets a focused, accurate answer, and never knows there was a team working behind the curtain.

That is exactly what this project does. It is an AI customer service system built in **n8n** that routes each incoming message to one of three specialist sub-agents, **Sales, Support, or Billing**, each with its own knowledge base and tools.

Built on n8n Cloud with OpenAI (gpt-4o-mini).

---

## The Problem

Businesses receive a mix of customer questions, such as product inquiries, complaints, and billing issues, that would normally be handled by different departments. A single AI agent trying to handle everything tends to give vague, unfocused answers and mixes responsibilities.

This system mirrors how a real support team works. A coordinator reads each incoming message and routes it to the right specialist, so every customer gets a focused, accurate answer from an agent built for that specific job.

---

## The Architecture

At the center is the **Main Agent**, the orchestrator. It holds the conversation memory and decides who should handle each message. It does not answer questions itself. Its only job is to understand intent and delegate to the right specialist.

<br>

### The Main Agent (Orchestrator)

![Main Agent orchestrator](canvas%20screenshots/canvas%20main%20agent.png)

The Main Agent has three specialist sub-agents available to it as tools.

<br>

---

<br>

### The Sales Agent

Answers product, pricing, and shipping questions by reading the product knowledge base in Google Sheets.

![Sales sub-agent](canvas%20screenshots/canvas%20sales%20agent.png)

<br>

---

<br>

### The Support Agent

Handles complaints and damaged-item reports by logging each issue as a support ticket, appending a row to a Google Sheet so a human can follow up.

![Support sub-agent](canvas%20screenshots/canvas%20support%20agent.png)

<br>

---

<br>

### The Billing Agent

Answers refund, warranty, and payment questions by reading the billing-policies knowledge base in Google Sheets.

![Billing sub-agent](canvas%20screenshots/canvas%20billing%20agent.png)

<br>

---

<br>

### How They Connect

Each sub-agent is a **separate workflow**, triggered by the orchestrator through n8n's Call Workflow tool. This keeps each specialist isolated, independently testable, and easy to maintain or extend.

---

## Seeing It Work

<br>

### A Classic Example

I asked the assistant a simple product question:

> **I wrote:** "How much does the standing desk cost?"

The Main Agent recognized this as a product question, routed it to the Sales specialist, and the Sales Agent looked up the answer in the product knowledge base and replied:

> **It replied:** the FlexRise Pro standing desk costs $499, with a height range of 28 to 48 inches, free shipping on orders over $400, and a bundle option with the ErgoChair Plus for $750.

![Sales chat example](Chats%20Screenshots/chat%20classic%20cases/chat%20sales%20product%20question.png)

That single answer proves the whole chain worked: the message was understood, routed to the correct specialist, looked up in the right knowledge base, and answered accurately.

<br>

---

<br>

### Routing on the Canvas

You can see the routing happen on the canvas during a run. In this execution, the Main Agent decided the message belonged to the **Support** specialist and called it (notice the highlighted path):

![Successful execution](canvas%20screenshots/successful%20execution.png)

<br>

---

<br>

### A Real Action, Not Just a Reply

When the Support Agent handles a complaint, it does not just reply. It logs the issue as a real support ticket in Google Sheets, so a human can follow up. Here is a complaint from the chat turning into a logged ticket:

![Support ticket logged](ticket%20sheet%20screenshot/support%20ticket%20logged.png)

This is the part that makes it more than a chatbot. It takes a real action in the real world.

---

## How It Handles the Tricky Stuff

Clean questions are easy. The real test of a routing system is how it handles the messy ones: vague questions, questions that span two departments, questions about two things at once, and questions that are completely out of scope.

I tested all of these, and the system handled each one gracefully, by routing sensibly, asking for clarification, or politely declining when something was outside its scope.

If you want to see exactly how it responds to these edge cases, the chat screenshots are in the **`Chats Screenshots/chat edge cases`** folder.

---

## Design Decisions

- **Memory lives at the orchestrator, not the sub-agents.** The Main Agent holds the conversation context. The specialists are stateless and handle one request at a time. This keeps the design clean and avoids each sub-agent maintaining redundant state.
- **Routing is driven by tool descriptions.** Rather than hard-coded rules, the orchestrator chooses a specialist based on clear, non-overlapping tool descriptions (products versus complaints versus billing), which makes it reliable even on ambiguous messages.
- **One workflow, one job.** Splitting the specialists into separate workflows keeps each automation focused and independently maintainable.

---

## Features I Used

- **n8n Cloud:** workflow orchestration and the agent framework
- **OpenAI (gpt-4o-mini):** agent reasoning and routing decisions
- **Google Sheets:** knowledge bases for Sales and Billing, and the support-ticket log
- **Multi-agent / orchestrator pattern:** a coordinator agent calling specialist sub-agents as tools
- **Conversation memory:** held at the orchestrator level so the assistant follows the conversation

---

## Try It Yourself

You can import and run this project in your own n8n:

1. Download the workflow files from the **`json files`** folder (`MainAgent.json`, `SalesAgent.json`, `SupportAgent.json`, `BillingAgent.json`).
2. In your own n8n, import each one (Workflows, then import from file).
3. Connect your own credentials: an OpenAI account for the agents, and a Google Sheets account for the knowledge bases and the ticket log.
4. Set up the Google Sheets the agents read from (a product sheet, a billing-policies sheet, and a tickets sheet).
5. Open the chat on the Main Agent and start asking questions.

There is no live public link to chat with, since that would mean keeping my own workflow and credentials running publicly. Importing the workflows lets you run the whole system on your own setup.

---

## Files

- **`json files/`** the four workflows: the orchestrator and the three specialist sub-agents
- **`canvas screenshots/`** the workflow canvases and a successful execution
- **`Chats Screenshots/chat classic cases/`** example conversations for each specialist
- **`Chats Screenshots/chat edge cases/`** how the system handles vague, ambiguous, and out-of-scope messages
- **`ticket sheet screenshot/`** a complaint logged as a support ticket

*Note: API keys and credentials have been removed from the exported workflows. To run this yourself, connect your own OpenAI and Google Sheets credentials in n8n.*
