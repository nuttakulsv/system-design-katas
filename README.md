# system-design-katas

[ภาษาไทย](README.th.md) · Part of the katas series with [staff-katas](https://github.com/nuttakulsv/staff-katas) and [incident-katas](https://github.com/nuttakulsv/incident-katas)

Reading system design guides teaches you the answers. Interviews test whether you can find the question.

system-design-katas turns your AI agent into a system design interviewer. Each prompt is deliberately vague, the way real interviews are, and the interviewer holds hidden requirements that only come out if you ask the right questions. Halfway through, it throws a curveball at your design. At the end you get a debrief calibrated to the level you're aiming for (senior, staff or principal), with quotes, the requirements you missed, a hiring signal, and one good reference design.

## What it feels like

```
You:    Design concert tickets. Level: staff.

Agent:  "A huge concert goes on sale at 10:00 tomorrow. Design the ticketing
        system." Take it away.

You:    How many seats, how many people do we expect, and what went wrong last time?
Agent:  [2 min] 50,000 assigned seats. About a million people at 10:00. Last time
        bots bought most of the seats and the site went down.
  ...
Agent:  [31 min] Okay. Halfway through the sale, the payment provider slows down:
        confirmations now take three minutes. What happens to your seat holds?
  ...
Signal: Hire (staff)
Hidden requirements you missed: PromptPay confirmations arrive by bank callback,
sometimes after the hold expires. Your design would double-sell those seats...
```

## Prompts

| # | Prompt | The hard part (spoiler-free) |
|---|---|---|
| 01 | Concert tickets | A million people at 10:00:00 |
| 02 | Wallet ledger | Money that must never appear or disappear |
| 03 | Short links | It's not the one you've practised |
| 04 | Notifications | Other teams will use it in ways you didn't plan |
| 05 | Ride dispatch | Bangkok traffic, 50k location updates a second |
| 06 | LLM gateway | Twenty teams, five providers, one budget |
| 07 | Shared documents | Fifty people and a phone that's offline |

Prompt files are in [`skills/system-design-katas/prompts/`](skills/system-design-katas/prompts) and include the hidden requirements, so don't read one before you play it.

## Install

Claude Code:

```
/plugin marketplace add nuttakulsv/system-design-katas
/plugin install system-design-katas@system-design-katas
```

Codex, Cursor, or anything else that reads SKILL.md:

```
npx skills add nuttakulsv/system-design-katas
```

Then ask for "a system design kata at staff level" or "prompt 6".

**No coding agent?** Paste [`SKILL.md`](skills/system-design-katas/SKILL.md) and a prompt file into any chat assistant and say "start the interview".

## How the levels differ

The same prompt, judged three ways:

- **Senior:** a working design with correct core mechanics and sensible trade-offs.
- **Staff:** finds the hard part without being told, drives the conversation, and covers failure modes, operations, cost and how the system evolves.
- **Principal:** questions the problem itself, ties it to the business, and sees the organisational and long-term consequences.

## Also useful for

- **Design reviews:** bring your real design and ask the agent to review it as a skeptical staff engineer would, with hidden constraints it invents for your domain.
- **Mentoring:** run a prompt with a junior engineer and talk through the debrief together.
- **Interviewers:** calibrate your own questions and rubrics.

## Write a prompt

Copy [`_template.md`](skills/system-design-katas/prompts/_template.md). The best prompts hide the real difficulty behind a familiar title, and have curveballs that stress the design rather than just add features. PRs welcome.

## License

MIT
