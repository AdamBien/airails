<!--
Capture one system — the whole assembly of capabilities — so that each capability line is a
one-shot input to `/capability-brief` and the mission, language and decisions seed the sbce system
doc. Fill every section; a blank one becomes a question downstream, once per capability. Describe
why the system exists and which capabilities it needs, never how any capability works or how the
system is built: no operations, inputs, screens, types, transports, frameworks or file names. The
stack is not captured here — if none is declared in the repository, say so under Open issues.

How the sections become downstream artifacts:
System           -> the system doc's title, `>` mission line and `## Vision`
Actors           -> the `primary` and `external` actors of each capability brief
Capabilities     -> one `/capability-brief` input per `new` or `extends` line; name and tag
                    become its `name` and `kind`; `existing` lines complete the assembly only;
                    the proposed BC set `/sbce new` confirms
Interactions     -> the system doc's `## Components` wiring, proposed
Language         -> the system doc's `## Ubiquitous language`
Invariants       -> candidates for the system doc's `## System invariants`, written as EARS by sbce
Success measures -> rationale only: justifies the carving and the build order; feeds Vision
Out of scope     -> closes scope questions in every capability brief
Decisions        -> the system doc's `Dn` log with rejected alternatives
Open issues      -> the only questions asked downstream about the system

Use: run `/capability-brief "<one-liner>"` per capability in build order, then `/sbce new`; or
paste the body into the README outside the `sbce:generated` markers as the inception seed.
-->

## System
- name: `<one lowercase word>`
- mission: <one sentence: what the whole assembly promises, from the system's side>
- vision: <one aspirational sentence: the outcome the assembly chases> | none

## Actors
<!-- primary: people or systems that use the assembly, each with what they achieve; external:
systems the assembly relies on but does not own, each with what for -->
- primary: <e.g. "an employee"> — <what they achieve with the system, one line>
- external: <e.g. "the payroll system"> — <what the assembly relies on it for>

## Capabilities
<!-- one line each, in build order; one lowercase word, tagged new, extends or existing; one sentence naming
the primary actor and the outcome the system promises; no operations, inputs, steps or screens —
those belong to the capability's own brief. The first line states why it is first. `new`: a
capability to build; `extends`: an existing capability that gains this outcome, the line
describes the addition; `existing`: reused as it is, no brief needed. -->
1. `<name>` — new | extends | existing — <primary actor> <achieves outcome>; the system <promises …>. _(first: <why>)_
2. `<name>` — new | extends | existing — …

## Interactions
<!-- only between listed capabilities and external systems; direction and what crosses; or
`none known yet` — the briefs will reveal the rest -->
- `<capability>` relies on `<capability>` for <what>
- `<capability>` relies on <external system> for <what>

## Language
<!-- shared nouns: any noun used in two or more actor or capability lines, defined once in one
line; no fields, no types; a second meaning is a decision -->
- <Noun> — <one-line meaning>; owned by `<capability>`

## Invariants
<!-- rules in domain terms that span two or more capabilities; a rule owned by one capability is a
note for that capability's brief instead -->
- <rule, one sentence>

## Success measures
<!-- observable outcomes the stakeholders can check; none is a feature; or `none` -->
- <measure>

## Out of scope
<!-- what was weighed and excluded, each with its reason -->
- <item> — <reason>

## Decisions
<!-- each confirmed choice with the alternatives rejected; else `none` -->
- record: <choice> _(rejected: <alternatives>)_

## Open issues
<!-- only questions nobody could answer yet; these are the only questions asked downstream about
the system -->
- <question>

<!-- ======================================================================================
Filled example: `hrportal`, an HR self-service system
========================================================================================= -->

## System
- name: `hrportal`
- mission: let employees and managers handle leave, expenses and onboarding themselves, with HR involved only where a rule requires it
- vision: no routine HR request ever waits for an email reply

## Actors
- primary: an employee — requests leave, claims expenses, completes their own onboarding
- primary: a manager — decides their team's leave and expense requests, starts onboarding for a new hire
- primary: an HR administrator — maintains the rules and steps in where a request falls outside them
- external: the HR system — the record of employees, their managers and their leave balances
- external: the payroll system — pays approved expenses and applies unpaid leave
- external: the team calendar — shows approved absences to the team

## Capabilities
1. `leave` — new — an employee gets a decided leave request; the system has it decided by the manager and keeps balance and calendar in line. _(first: highest request volume today; the rules are already written down)_
2. `expenses` — new — an employee gets an approved expense claim paid; the system has it decided by the manager and hands it to payroll.
3. `onboarding` — new — a new hire arrives with accounts, equipment and first-week steps in place; the system tracks each step from the manager's start to completion.
4. `directory` — existing — anyone finds a colleague, their manager and their team; the system mirrors the HR system's assignments.

## Interactions
- `leave` relies on `directory` for the employee's manager
- `expenses` relies on `directory` for the employee's manager
- `onboarding` relies on `directory` for the team the new hire joins
- `leave` relies on the HR system for the leave balance, and on the team calendar for showing approved leave
- `expenses` relies on the payroll system for paying an approved claim
- `onboarding` relies on the HR system for the new hire's record

## Language
- Request — something an employee asks for that a manager decides: a leave request or an expense claim; owned by `leave` and `expenses` respectively
- Manager — the person the HR system assigns as deciding for an employee; owned by `directory`
- Balance — the leave days an employee may still take this year; owned by the HR system, read by `leave`

## Invariants
- a request is decided only by the requester's current manager, never by the requester
- nothing the system records contradicts the HR system; where they differ, the HR system is right

## Success measures
- the share of leave and expense requests decided without an email to HR
- days from a new hire's start to the last onboarding step completed

## Out of scope
- payroll calculation — owned by the payroll system
- performance reviews — a different conversation with different rules; a later system
- recruiting — the hire exists before this system sees them

## Decisions
- record: `leave` and `expenses` are separate capabilities _(rejected: one `requests` capability — different rules, different deciders in HR)_
- record: `directory` is reused as it is, not rebuilt _(rejected: each capability reading the HR system directly)_
- record: the HR system stays the source of truth for balances and assignments _(rejected: keeping copies in the portal)_

## Open issues
- does a manager's manager decide when the manager is themselves on leave, or does the request wait?
- stack not declared
