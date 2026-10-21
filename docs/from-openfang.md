# From OpenFang to a Kortix repository

A single-binary agent runtime is the right shape for one operator and the wrong shape for five. The agents keep doing the same jobs when a team shares OpenFang's work. What has nowhere to live is the state around them: who owns which agent, which memory is authoritative, and who approved the last permission change. Kortix is the open-source AI Operating System, and it keeps that state in a git repo the team owns.

## What OpenFang is

OpenFang is an [open-source Agent Operating System](https://github.com/RightNow-AI/openfang) written in Rust. It compiles to a single binary of about 32MB, which you install with a script and start with `openfang init` and `openfang start`. The dashboard answers on `localhost:4200`, and nothing else has to be provisioned.

Its unit of work is a **Hand**: a `HAND.toml` manifest that declares tools, settings, requirements and dashboard metrics. A Hand also carries its system prompt, a `SKILL.md`, and guardrails on the actions that need a person. Seven Hands ship inside the binary, compiled in rather than loaded from disk. Memory is a SQLite store with vector embeddings and compaction. The audit trail is a Merkle hash chain. Both sit on the machine that runs the binary.

The current release is [v0.6.9, published 12 May 2026](https://github.com/RightNow-AI/openfang/releases/tag/v0.6.9), a security patch release. The README's badge and stability notice still read v0.5.10. They carry the warning that matters for a migration: OpenFang is pre-1.0, breaking changes can arrive between minor versions, and production should pin a commit until v1.0.

The licence signals disagree too. `Cargo.toml` sets `license = "Apache-2.0 OR MIT"` and both `LICENSE-MIT` and `LICENSE-APACHE` sit at the root, while the README badge and its License section name MIT alone. Get the answer from your own legal read before you fork and pin a commit.

Memory, the clock, the guardrail and the record all living on one disk is why one person can install OpenFang and have scheduled work running the same afternoon. The same arrangement is what a team runs into. Two laptops hold two memory files. The manifest sits where its author put it, so a colleague has to ask. The approval came from whoever had the dashboard open. The hash chain proves the log was not edited, and it says nothing about which of three installs is the real one.

## The mapping, one line at a time

| In OpenFang | In a Kortix repo | What you rewrite |
| --- | --- | --- |
| A Hand | `agents/<name>.md` plus a skill | The prompt, as a job description |
| `HAND.toml` settings | The agent block in `kortix.yaml` | Which connectors and secrets it names |
| The binary's scheduler | `triggers:` with a cron expression | The prompt and the timezone |
| The SQLite memory store | Files under `memory/` | What is worth keeping |
| Guardrails | `policy` and connector policies | The action, and who answers it |
| The hash-chained log | The change request that merged the change | Where the approval is recorded |
| The dashboard on `localhost:4200` | The repo, read as a diff | Where a person looks |

## What travels, what is rebuilt, what is lost

| What you have | Fate | What you do with it |
| --- | --- | --- |
| Provider keys and model choice | Travels | Declare the names as secrets, keep values out of the repo |
| Your `SKILL.md` files | Travels | Move each one into `skills/<name>/SKILL.md` |
| Your own Hand prompts | Rewritten | Goal, inputs, output, stopping condition |
| `HAND.toml` settings | Rewritten | An agent block, with grants written out |
| Hand schedules | Rewritten | A trigger, with an IANA timezone |
| Guardrail rules | Rewritten | allow, ask or block on the calls that matter |
| The SQLite memory | Mostly lost | Keep citable facts, drop embeddings and run logs |
| The seven bundled Hands | Lost as files | They ship inside the binary you installed |

## Step 1: export the Hand manifests

The binary ships a migration path in, `openfang migrate --from openclaw`. The README documents nothing for the way out. So the export is a copy: each Hand's `HAND.toml`, its system prompt text, its `SKILL.md`, and the settings that point at a provider, a channel and a schedule. Copy the state directory too. Private state has lived under `~/.openfang/` since the [0.6.8 split](https://github.com/RightNow-AI/openfang/releases/tag/v0.6.8), and that is where the store and the chain sit.

The first finding usually arrives here. If your team activated the shipped Lead, Researcher or Browser Hand and tuned it from the dashboard, you own no definition to export. Those prompts sit inside the binary. What you have is the output they produced and the corrections you made to it, which is enough to write a new agent file and is not an export. Write down what each Hand did for you before you close the dashboard.

## Step 2: port each Hand to an agent file and a skill

Start with the scaffold, which writes `kortix.yaml` and one agent file per role:

```bash
kortix init
```

Rewrite each Hand's prompt as a job description in `agents/<name>.md`: what the agent is for, what it reads, what it produces, when it stops, and which decisions need a person. Delete sentences that describe a surface, such as which panel to open or which button starts a run. Those lines teach an agent in a repository nothing. Keep the judgment the prompt was built around, usually the part worth having. The lines that decide something are the ones to carry over: what counts as a qualified lead, when a claim needs a second source, when to stop and ask somebody.

A procedure two agents would both need becomes a skill, one directory with a `SKILL.md` at its root. Then declare the agent, so its reach sits beside its behavior:

```yaml
agents:
  researcher:
    file: agents/researcher.md
    connectors: [web-search, drive]
    secrets: [SEARCH_API_KEY]
```

An omitted grant resolves to `none`. The Hand that could browse and the Hand that could publish were separated by a dashboard setting. Here they are separated by a line somebody reviews.

Guardrails move the same way. A Hand that required approval before a purchase becomes a connector policy set to ask on the purchase call, and the project default is a separate decision:

```yaml
policy:
  default_mode: risk
```

Approval gates are off until you set them, so a project with no `policy` block allows every call no rule matches. Choose the default deliberately and put an explicit ask on the action that would spend money, publish something, or write to a system of record.

## Step 3: decide what the memory file keeps

The SQLite store is a binary file. A diff of it shows nothing, which is the problem once more than one person depends on what it knows. Export it to text, read it, and treat the export as an archive rather than something to port.

Keep the facts a new hire would need and the corrections someone made after a wrong answer. Drop the embeddings, the session transcripts, the tool logs and the duplicates compaction left behind. Write what survives into `memory/` as markdown files named for what they hold, such as `memory/ideal-customer.md`. The test for each line is whether you can point at where it came from. A fact nobody can source does not go in.

The payoff shows up the first time the memory is wrong. A correction arrives as a diff against a file, with a name on it and a revert available. In the single-binary setup it was a chat message to whoever ran the install.

## Step 4: set the connector grants

Each service a Hand called becomes a `connectors:` entry, and each chat surface it posted to becomes a channel. Confirm the exact service exists before you put it in the plan, because a Hand's channel list is wider than most teams use.

Reauthorize rather than copy a token. Connector credentials are brokered server-side and never enter the machine, so the migration moves a connection, not a secret. Declare the secret names under `env` and keep the values in the project's secrets store.

Grant per agent, and test with one read call before a prompt depends on the result. The failure this catches is the ordinary one: the connection authenticates and returns nothing, because the account it authorized cannot see the folder the job needs.

## Step 5: run one session end to end

Give one ported agent one real job, and read what comes back as a diff:

```bash
kortix sessions new --prompt "Rebuild last week's lead list from the CRM and open a change request"
kortix cr ls
kortix cr diff 1
kortix cr merge 1
```

Session work reaches `main` through a change request. Merge is default-deny for agents. That review is where the ported prompt proves itself. If the diff is wrong, the fix belongs in the agent file and the next run is the test.

## Step 6: the cutover check

Five checks stand between a working pilot and a safe cutover, and each one is a fact the repository can show.

- Every agent file has a named owner, written in the repo rather than agreed in a chat.
- One memory is authoritative. A warm SQLite copy "just in case" leaves two answers to the same question, and the older one wins by habit.
- The last permission change is a merged change request with a person's name on it, not a dashboard edit someone remembers making.
- The same job has run once in both systems, and someone wrote down which output a person would have shipped.
- The date the old binary goes read-only is in the repo, and so is the person who owns it.

The twelve-check version, with the manifest and scope commands, is in [the pre-switch checklist](checklist.md). Work through it before the old binary goes quiet.

## When staying on OpenFang is the better call

Stay if one person runs the agents and the work is scheduled rather than shared. OpenFang does that with one install and no repository to adopt, and the seven bundled Hands already know a few jobs. Stay as well if the team cannot yet name who would review a change to an agent definition. Moving the work into a repository would only relocate that argument.

The calculation changes when a second person needs to edit an agent. It changes again when a manager asks who widened a permission last month, or when "on Alex's machine" stops being an acceptable answer. At that point the runtime is not the question. Where the shared state lives is, and that answer decides the migration.

For the wider field, [OpenFang and four other products that use the Agent OS name](https://opensourceaioperatingsystem.com/openfang-alternative) compares where each one keeps definitions, memory and permissions. If the vocabulary is the unclear part, [what an agent operating system has to hold](https://opensourceaioperatingsystem.com/agent-os) sets out the four parts and who owns each.

The OpenFang facts come from the project's README, `Cargo.toml`, licence files and release notes, read October 2026. Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free.
