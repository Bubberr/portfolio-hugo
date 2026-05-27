---
title: "Customer Case: What a Christmas Market Taught Me About System Design"
categories: ["Projects", "AI"]
tags: ["customer-case", "system-design", "ai", "automation", "email"]
date: 2026-05-11
draft: false
showauthor: true
authors:
  - Bubberr
---

## Background

As part of our semester, our entire class had a visit from **E.G.** — a large estate with a wide range of activities and events. The visit gave us a chance to talk directly with the people running the place and get a feel for how they operate day-to-day.

One of the things that caught my attention was their annual Christmas market, where external vendors rent stalls and sell everything from handmade crafts to holiday decorations. It sounds straightforward — but behind the scenes, there is a surprising amount of coordination involved.

---

## The Problem: An Inbox Full of Stall Requests

Right now, anyone who wants to rent a stall at the Christmas market has to contact the owner directly by email. She then reads through every single enquiry and replies individually — confirming or declining each one by hand.

When the market is popular, that means dozens of back-and-forth emails to manage, no central overview of who has applied, and a lot of time spent on something that could largely run itself.

---

## The Idea: A Vendor Application Form

The fix does not need to be complicated. A simple form on E.G.'s existing website would let vendors fill in their details — name, contact info, what they sell, any practical requirements — and submit their application in one go.

On the owner's side, instead of an inbox full of emails, she gets a clean list of all applicants where she can confirm or decline each one with a single click. The system then sends the appropriate response automatically.

The core flow would look like this:

```
Vendor fills out form on the website
        ↓
Application added to owner's dashboard
        ↓
Owner reviews and confirms or declines
        ↓
Vendor receives automatic confirmation or rejection email
```

No inbox archaeology. No manually written replies. Just a list to work through.

---

## Where AI Fits In

The interesting part is that AI can add real value at several layers of a system like this — not just as a gimmick, but as something that actually solves problems.

### 1. Smarter Confirmation Emails

Once a vendor is confirmed, a language model can use the data they submitted — name, product type, any special requirements — to generate a confirmation email that feels personal rather than templated:

> *"Hi Maren, your application has been approved! We're looking forward to seeing your homemade Christmas wreaths at this year's market. You'll find all the practical details below..."*

The owner clicks confirm once. The vendor gets a message that feels like it was written for them.

### 2. Answering Vendor Questions

A simple chat assistant on E.G.'s website — trained on their rules, FAQ, and practical info — could handle 80% of the questions vendors ask, without anyone at E.G. needing to respond manually.

### 3. AI as a Tool for the Developer

And this is what actually surprised me most when thinking through the E.G. case: AI is not only useful *inside* the system. It is also useful *during development*.

When mapping out the form fields and the owner's dashboard, I used Claude to:
- Ask the questions I had not thought to ask myself
- Draft template emails based on my description of the audience
- Suggest edge cases ("what happens if a vendor cancels two days before the market?", "should declined applicants be able to reapply?")

It accelerated the design phase significantly. Instead of sitting alone trying to think things through, I had a sounding board that could quickly generate alternatives and point out gaps.

---

## Other Systems E.G. Could Benefit From

Beyond the application form, there are other areas where structured thinking and simple tooling could make a real difference:

| Area | Current problem | Possible solution |
|---|---|---|
| Booking management | Manual handling of enquiries | Simple booking app with calendar integration |
| Payment overview | Unclear who has paid what | Automated invoicing with a status dashboard |
| Internal communication | No overview across events | A lightweight internal notification system |
| Vendor feedback | No structured post-event evaluation | Automatic follow-up email with a feedback form |

None of these require advanced technology — but they do require someone to think them through and build them properly.

---

## What I Took Away

Visiting E.G. as a class was my first real encounter with a customer who does not know exactly what they are missing — only that something feels laborious. That is a fundamentally different problem than building to a specification.

It taught me that system design starts with listening, and that the best solutions are often the ones that remove work rather than add features.

And it reinforced something I keep coming back to: AI is not a plugin you bolt on at the end. It can sit at the table from the very beginning — from idea through to implementation.
