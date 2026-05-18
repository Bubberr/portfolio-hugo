---
title: "E.G. Vendor System: From Customer Case to Actual Project"
categories: ["Projects", "AI"]
tags: ["system-design", "project-planning", "next.js", "ai", "scope", "vendor-system"]
date: 2026-05-18
draft: false
showauthor: true
authors:
  - Bubberr
---

## Why this project?

In my post on the E.G. customer case, I described a problem they face every year: the Christmas market season brings an inbox full of chaos, with the owner manually handling every single enquiry from vendors wanting to rent a stall.

I described a possible solution — but actually building it is a different thing entirely. That is what I am going to do now.

It is a good project to take on because it:

- Is scoped tightly enough to be completable
- Addresses a real frustration for a real user
- Leaves room to use AI in a way that makes sense — not just bolted on as a gimmick
- Aligns with what we have been working on this semester: spec-driven development, LLM integration, and agentic workflows

---

## What the system needs to do

The core flow is simple:

```
Vendor fills out application form on the website
        ↓
Application is stored and appears in the owner's dashboard
        ↓
Owner approves or declines with a single click
        ↓
Vendor receives an automatic confirmation or rejection email
```

That is the MVP. Everything else is scope creep unless it solves a concrete problem.

---

## Technology considerations

The first question is always: *what is the right tool for this?* I have thought through it at three levels.

### Frontend

| Option | Advantages | Disadvantages |
|---|---|---|
| **Next.js** | Full-stack in one framework, easy deploy on Vercel, great for forms and server actions | Could be overkill for a simple flow |
| React + separate backend | Flexible | Two things to manage and deploy |
| Vanilla HTML + fetch | Minimal, quick to set up | Scales poorly, no component model |

I am going with **Next.js**. It gives me forms, server-side logic, and database access in a single project. And when the AI layer needs to be wired in, adding API routes is straightforward.

### Database

| Option | Advantages | Disadvantages |
|---|---|---|
| **PostgreSQL via Supabase** | Hosted, free tier, solid SQL support, real-time updates | External dependency |
| SQLite | Simple, no setup | Not suited for production, hard to scale |
| MongoDB | Flexible schema | Unnecessary complexity for structured data |

Application data is structured and relational — SQL makes sense. Supabase gives me PostgreSQL without managing a server, and Row Level Security means the dashboard can be locked down at the database level.

### Email

I am planning to use [Resend](https://resend.com) for sending automated emails. It has a generous free tier, clean integration with Next.js, and lets you build email templates in React — which makes it easy to inject dynamic content into each message.

### The AI layer

This is the interesting part. I will use the Claude API to generate personalised confirmation emails based on the vendor's application data. Instead of:

> *"Your application has been approved."*

the vendor receives:

> *"Hi Maren — your application has been approved! We're looking forward to seeing your homemade Christmas wreaths at this year's market. All the practical details are below..."*

The implementation is not complicated: receive the approval action → send vendor data to the LLM → return personalised email text → send via Resend.

I am also considering building a simple RAG-based FAQ assistant for the website, but that depends on whether there is time after the core system is done.

---

## Scope considerations

The easiest mistake in a project like this is trying to build everything at once. I have deliberately split it into clear layers:

**MVP — what must work:**
- Application form (name, contact info, what they sell, any special requirements)
- Admin dashboard with a list of applications
- Approve / decline per application
- Automatic email sent on action

**Layer 2 — if the MVP is solid and there is time:**
- AI-generated emails on approval
- Filtering and search in the dashboard
- Status tracking (applied, approved, declined)

**Out of scope — deliberate exclusions:**
- Payment handling
- Role-based user management (there is only one owner)
- Mobile app
- Full FAQ chatbot

Saying no to features is a design decision, not a shortcoming. The system should solve the owner's problem — not every conceivable problem some future version of the system might face.

---

## GDPR and data security

The system stores contact details from vendors. That means I need to:

- Inform applicants about what is stored and when it will be deleted
- Restrict access to the dashboard (simple authentication is enough for MVP)
- Send only the minimum personal data needed to the LLM when generating the email
- Ensure data is deleted or anonymised after the market season ends

None of this is complicated — but it has to be designed in from the start, not glued on afterwards.

---

## What I expect to learn

This is not an experiment in cutting-edge AI. It is an exercise in building something real, for a real user, under real constraints.

I plan to use spec-driven development actively: writing acceptance criteria, letting them drive implementation, and using Claude Code as an agent — not as autocomplete. That is the approach I will be evaluating myself against in the reflections along the way.

And if the system at some point actually replaces an inbox full of emails at E.G., that would not be a bad outcome.
