---
title: "AI Agents are shipping code faster than we can test"
---

Something shifted in January. Teams came back from the holidays and started using agents for real work. Claude Code, Cursor, Codex. Assigning tickets, letting them run in the background, reviewing the PR when it's done.

The code is good. Often better than what a human would write. But everyone's hitting the same wall: the verification process hasn't caught up.

## The accountability gap

When a human submits a PR, there's an implicit expectation: they ran the code. They saw it work. If the change touches something user-facing, most teams expect a screenshot or video. Not just as proof of what changed, but as proof that someone was in the loop. They might have noticed something off in testing and fixed it before submitting. They thought about edge cases. Tried different states.

There's accountability baked in. If something breaks, someone should have caught it.

With agents, that layer is gone. Not because agents are bad at code. They're often better. But no one was in the loop. The agent did exactly what you asked. It just didn't think to check what else might have been affected.

You need more verification with agent PRs, not less. You want to see everything that changed, not just what the agent intended to change. You want to know the code ran and worked. And because there's no one else accountable, you're on the hook. If something breaks, it's on you. So you check it yourself.

But the volume makes that impossible. You could carefully verify a few human PRs a day. When you're getting 10+ from agents, something has to give.

## The ceiling

Teams want to give agents bigger tasks. Refactors across multiple files. Design system updates. The kind of broad changes that are tedious for humans but perfect for agents.

But without verification, they can't. The risk is too high. One engineer I talked to put it this way: they're hitting a ceiling on how much they can use agents. They want to ramp up, not pull back. But the verification problem is blocking them.

So they're stuck giving agents small, safe tasks. Change this copy. Fix this one component. Things where the blast radius is limited and you can eyeball the diff.

The promise of agents is right there. Verification is what's stopping teams from grabbing it.

## What I'm building

I've been working on qckfx, a regression testing tool for mobile. Record a flow, replay it deterministically, see exactly what changed.

Right now it runs locally. When you're working with Claude Code or Cursor, verification should happen in the development loop, not after. Agent makes changes, runs the test, catches regressions before they ever hit review.

A few things make it deterministic and trustworthy:

- Network responses recorded and replayed. Tests don't flake on API timing or non-deterministic data like search results, recommendations, or AI outputs.
- Dates stubbed, so time-dependent views don't break every run.
- Disk state copied, so you're not fighting login flows and test data setup.

Your agent runs tests through MCP and sees what actually changed: screenshots diffed against baseline, network requests compared, logs captured.

It's free. [Download for Mac](https://qckfx.com)

CI and team sync are on the roadmap. But local is the foundation. That's where you catch things before they become someone else's problem.

## How are you solving this?

I'm building qckfx because I couldn't find a better way. If you've found something that works, I want to hear about it.

Are you doing manual QA on agent PRs? Writing more tests? Slowing down the agent volume?

If you want to try qckfx, I'm around to help: chris.wood@qckfx.com.
