# HermesX

**A local-first AI workspace hub that lets models go beyond answering, and actually get the work done.**

HermesX understands your goals, gathers scattered context, and autonomously breaks down tasks, orchestrates tools, and reads and writes files, surfacing every step of progress in plain sight so ideas, information, and results flow together in one space.

Just describe the outcome you want. HermesX brings order to your conversations, files, and tasks, converging complex work into a process that is trackable, controllable, and ready to deliver. You set the direction; it drives execution and lands the result. What you get back is not a single answer, but a complete deliverable you can open, locate, and review. Data and execution stay local, so safety and control remain firmly on your side.

> Download: [hermesx.jackcloud.online](https://hermesx.jackcloud.online)

---

## Changelog

### v1.0.2 (2026-06-23)

- PPT generation: Agent can now directly produce presentations (HTML Deck + PPTX), from brief to final output in one go
- Plan proposals: complex tasks now produce a plan for review before execution
- Provider rate limiting: Rate Limit Gate prevents API call overload
- RAG engine upgrade: more precise keyword retrieval
- Tool Executor refactor: Tool Registry + Safety Guards for more robust execution

### v1.0.1 (2026-06-22)

From "read-only analysis" to "hands-on action" — this release gives the agent the ability to see and operate your desktop:

- Screen vision & desktop control: capture screenshots, use mouse and keyboard to operate any desktop app or browser — open pages, click buttons, fill forms, launch software, working on-site just like a human
- Parallel sub-agent collaboration: multiple sub-agents working in parallel around the same context, streaming progress back in real time
- Windows accessibility hardening: more stable UIA element location and window traversal
- Deeper browser capture: always-on Console / Network event monitoring for more precise web interaction
- Permission panel redesign: clearer, more controllable operation permissions

Letting the agent not just "talk", but truly "do".

### v1.0.0 · Public Beta (2026-06-21)

Polished all the way from 0.x to 1.0.0, HermesX now enters public beta. This release is all about making the agent more autonomous and secure:

- Sub-agent orchestration: automatically decomposes complex tasks and spawns parallel sub-agents
- Browser tab control: locates elements via CSS/Role/Text, auto-waits for render, screenshot-to-analysis
- Persistent shell sessions: directory and environment carry across commands
- Project context awareness: reads HERMESX.md to inject instructions on the fly
- Disaster protection: critical deletes are undoable — no more panic
- Install integrity checks + network safety hardening
- MCP ecosystem: GitHub / PostgreSQL and more plug-and-play tools

Give it a try and tell us what you think. Every piece of feedback makes the full release better.
