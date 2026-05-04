---
title: Spec-Driven Development in an AI-Driven World
categories: ["AI", "Method"]
tags: ["spec", "ai", "code-agents", "ethics", "gdpr", "software-development"]
date: 2026-05-04
draft: false
showauthor: true
authors:
  - Bubberr
---

## What is a spec, and why does it matter now?

A specification is a precise description of *what* a system should do — not *how*. It captures requirements, flows, acceptance criteria, and constraints in a single governing document. In traditional development, specs are used to communicate between stakeholders and developers. In AI-driven development, they take on a new role: they are **instructions to an agent**.

That is the critical difference. When I work with a code agent like Claude Code, having a loose idea of what I want is not enough. The agent acts on what I say — and nothing more. A vague task produces a vague result.

---

## The spec as a governing artefact

Spec-driven development means letting the specification drive the entire process: requirements → design → implementation → review. Instead of improvising along the way, you start by defining *what must be true* when a feature is done.

That requires thinking in terms of:

| Element | Example |
|---|---|
| Requirement | "The system must validate input against the rubric before sending it to the LLM" |
| Flow | "User submits text → controller validates → service builds prompt → LLM responds → frontend renders" |
| Acceptance criteria | "Reject with 400 if `assignmentText` is empty or > 50,000 characters" |
| Constraints | "No user data may be persisted — the system is stateless" |

These elements are not just documentation. They are the contract a code agent works from.

---

## Peter Naur and the theory behind the code

In 1985, Peter Naur described programming as *theory building* — the idea that the most important product of software development is not the code, but the mental model (*the theory*) the developer builds around the problem and its solution. The code is merely a deposit of that theory.

That is a provocative thought at a time when code agents can produce code faster than we can read it. If the theory lives in the developer's head but the code is written by an agent — who owns the understanding?

My answer: the theory must live in the specification. The spec is the place where human understanding is formalised and made accessible — both for agents that implement, and for future developers who take over. A good spec is a written-down theory.

---

## Legal and ethical considerations

When AI is part of development, questions arise that are not purely technical.

**GDPR and data handling** — if a code agent has access to files containing personal data, you need to consider what gets sent to an external API. In the LLM API project, we addressed this with a stateless design: no user data is stored, and only the assignment text (without personal identifiers) is passed on. That is a design decision driven by a constraint in the spec.

**Bias and fairness** — an LLM that assesses student reports may have systematic biases. If the model consistently scores certain writing patterns lower, that is not just a technical problem — it is a fairness problem. The specification should include a requirement that output is reviewed by a human, and that the model is never the final judge.

**Accountability** — who is responsible if a code agent introduces a security vulnerability? The person who prompted it. Specs are a way of documenting the *intent* behind the code, so it becomes possible to tell whether a bug is a faulty implementation of a correct spec, or a flaw in the spec itself.

---

## How I plan to use specs going forward

I have learned to prompt on the fly — giving the agent context in the moment. It works, but it does not scale. The next time I start a project I will:

1. **Write a feature spec** before giving the agent a task. Not necessarily a long document — five to ten lines defining requirements, flow, and acceptance criteria is often enough.

2. **Use the spec as a review baseline.** When the agent delivers a solution, I hold it up against the spec. That is faster than guessing whether the result is "good enough".

3. **Document constraints explicitly.** Things like "no user data is persisted" or "output is always reviewed by a human" belong in the spec — not as a verbal agreement.

4. **Let the spec drive acceptance tests.** Each acceptance criterion is a test case. The agent can help write the tests, but the criteria must come from me.

---

## Reflection

Spec-driven development is not a bureaucratic exercise. It is a way of honouring what Naur called *theory building* — ensuring that human understanding is not lost in the speed that AI-driven development gives us.

A code agent is powerful but uncritical. It does what it is told. That means the quality of my output is directly proportional to the quality of my spec. That is a discipline, not a limitation — and one I intend to practise.
