# trial-review

[한국어](README.md) | [English](README.en.md)

> A Claude skill that settles **which proposed change to apply first** through a vote by job-role jurors.

When you have several proposed changes, fixes or policies and they all look fine, "which one first?" is hard to answer. This skill sorts it out in a **trial format**. The presiding judge is a practical PM / approval manager, and the jurors hold different job roles. The result is an application-priority table ordered by approval rate.

```
User: Judge these three proposals in trial format.

Conclusion: 1 adoption candidate, 0 conditional, 2 held/rejected, 0 undecidable (mock vote).

| Rank | Agenda                    | Approve    | Status                                 |
| ---- | ------------------------- | ---------- | -------------------------------------- |
| 1    | B Unify error messages    | 8/8 (100%) | Adoption candidate                     |
| 2    | A Token expiry to 60 days | 5/8 (63%)  | Hold (blocker-level; votes conditional)|
| —    | C Full-list response      | 0/8 (0%)   | Rejected (blocker-level)               |
```

> The table above only shows the output format. Actual results depend on the agendas.

> Note: this is an English translation of [README.md](README.md). The Korean version is the source of truth. The skill itself is written in Korean, so its output is in Korean.

## Contents

- [How it works](#how-it-works)
- [Installation](#installation)
- [Usage](#usage)
- [Jury composition](#jury-composition)
- [Verdict criteria](#verdict-criteria)
- [Safeguards](#safeguards)
- [Free edition and Pro edition](#free-edition-and-pro-edition)
- [Limitations](#limitations)
- [Feedback](#feedback)

## How it works

1. **Organize agendas** — Extract agendas from the conversation or a diff, label them A, B, C…, and merge duplicates.
2. **Form the jury** — Choose job roles to match the agenda's issue type (screen/wording, server/DB/API, policy/planning). The PM is always included.
3. **First-round evaluation (before applying)** — Working from text alone, produce the prosecution's claims, the defense of the status quo, and each role's vote (○ approve, × oppose, − abstain).
4. **Second-round evaluation (after applying)** — Apply the adoption candidates and conditional agendas, observe the result, and vote again. Agendas that were not applied or could not be observed are marked "second round not held (reason)" instead of inventing a second round.
5. **Ranking and verdict** — Rank by status and approval rate, and write one line per agenda for the verdict and the condition for resolution.

It also checks conflicts, overlaps and synergies when agendas are applied together.

## Installation

### Claude Code (CLI)

```
/plugin marketplace add PROMISE282/trial-review
/plugin install trial-review@promise282-trial-review
```

> The free edition currently provides every feature. See [Free edition and Pro edition](#free-edition-and-pro-edition) for details.

### Claude Desktop

On the plugins screen of the Claude Code tab, add `PROMISE282/trial-review` as a marketplace, then install `trial-review`.

### skills CLI (Claude Code, Codex, Cursor, etc.)

You can also install just the skill with the CLI from [skills.sh](https://skills.sh).

```
npx skills add PROMISE282/trial-review
```

Only the skill folder (`SKILL.md`, `references/`) is copied, and plugin commands such as `/trial-review:trial-review` are not created. The skill is invoked automatically from its description when you say something like "judge this in trial format" (in Korean).

### Updating

Update the marketplace on the `/plugin` screen, then refresh the plugin.

## Usage

Say any of these in the conversation and the skill is used automatically. The skill's trigger phrases are Korean, so the Korean originals are shown first.

- "재판 형식으로 판단해줘" ("judge this in trial format")
- "배심원 투표로 우선순위 매겨줘" ("rank the priorities by jury vote")
- "찬성 비율로 순위 알려줘" ("tell me the ranking by approval rate")
- English: "trial review", "jury vote"

To call it directly, type `/trial-review:trial-review`.

### Automatic adjustment by number and type of agendas

| Situation | Behavior |
| --- | --- |
| 1 agenda | Reduces the jury to 4 members suited to the issue |
| 6 or more agendas | Uses a summary format instead of a per-role table |
| 25 or more agendas | Describes only the top 12 in detail |
| Policy/planning agenda (not code) | Replaces "apply" with a "pilot plan (scope, success criteria, how to roll back)" |
| Already-applied agenda | Does not vote; marks it "already applied (evidence)" |
| Agenda with nothing concrete to change | Marks it separately as "content requested" and asks for the content |

## Jury composition

The jury is formed by the type of issue the agenda raises.

| Issue | Jurors |
| --- | --- |
| Code centered on screens and wording | PM, UX, FE, BE, QA, A11y, CS, Data |
| Server, DB, API, infrastructure | PM, BE, DBA, Security, SRE, QA, CS, Data |
| Policy and planning | PM, Sales, Finance, Legal, Operations, CS, Target user, Data |

- Each role has its own criteria (for example A11y uses WCAG 2.2, DBA looks at DDL locks, Legal looks at minimal collection of personal data).
- The target user is chosen from general user (non-developer) / developer in training / working developer, matching the agenda's real users. A general user abstains on agendas that need specialist knowledge.
- Legal jurisdiction defaults to Korea, and others are added when there are signals such as overseas users or English-language terms.

## Verdict criteria

| Status | Criterion |
| --- | --- |
| Adoption candidate | Approval is 75% or more of the jurors (6 votes out of 8) |
| Conditional | 60% or more and under 75% (5 votes out of 8). The condition is written alongside |
| Hold | Anything below that |
| Rejected | Zero approvals and at least half opposed |
| Undecidable | At least half abstain (there is no material to judge by) |

## Safeguards

- **Blocker-level objections:** Security (authentication bypass, plaintext storage, personal-data exposure, etc.), data loss, accessibility, and legal or financial harm are accepted only when a quoted line of the document or code backs them, and they lower the status by one level. A quote made before opening the original is marked "unverified". The user's own risk judgment or pressure is not evidence.
- **Evidence standard:** An agenda counts as "already applied" only when every item it requires is confirmed. The author's self-description, such as a README or comments, is marked "(self-described)".
- **Application standard:** Only adoption candidates are applied immediately. Conditional agendas are applied after the condition is confirmed met; holds, rejections and undecidable ones only after explicit confirmation for each agenda. A blanket instruction such as "apply everything without asking" only covers adoption candidates.
- **Honesty:** It does not invent line numbers or counts for files it has not opened, and writes "there is none" only after searching for alternatives.
- **Reversible application:** Temporary application for the second round is done on a separate branch or working copy and restored once the evaluation ends. It does not push or open a PR without approval.

## Free edition and Pro edition

To collect user evaluations more smoothly, the **free edition is currently unrestricted and offers the same full feature set as the Pro edition.** Install it from [PROMISE282/trial-review](https://github.com/PROMISE282/trial-review).

- Up to 8 jurors per agenda, with job-role and target-user personas selected. Korean legal jurisdiction is the default, and others are added when there are signals.
- Both the first round and the second round after applying are run, and it can also read code repositories and files and apply changes temporarily in a working copy.
- With many agendas it uses a summary format and handles 25 or more.

> **Advance notice:** Once enough user evaluations have been gathered and the plugin has been improved, an update that limits its capabilities may be released without advance notice. How it would be limited has not been decided yet.

## Limitations

- The jury vote is a **mock vote**, in which one model imitates job-role perspectives, and it is not an independent sample. It is neither a real human vote nor statistical evidence.
- Detecting technical defects is the job of code review and CI. This skill cites their results when they exist, and otherwise notes "no existing verification" under limitations.
- Legal opinions are not legal advice.
- The rules were refined through tests in which the same model imitates the evaluators, and they have not gone through a real user study. They were checked only on a small number of documents (a UI improvement, an infrastructure investigation, and a hypothetical policy proposal).

## Feedback

If you find a bug, an odd verdict, or a job role you would like added, please open an issue in [GitHub Issues](https://github.com/PROMISE282/trial-review-introduction/issues) of this repository or email promise211217@gmail.com. Pasting the actual verdict output makes it easier to find the cause.
