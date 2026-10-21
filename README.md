# Move a company's agent work onto an open-source AI Operating System

Kortix is open source, and the line it ships under is **the open-source AI Operating System**. This repository covers the move from Claude Cowork or ChatGPT Work: what comes across, what has to be rebuilt, and the checks that tell you the new system is ready before the old subscription goes quiet.

Most migration plans budget for an export. They list the chats to download, the files to copy, the projects to recreate. That work takes an afternoon. The part that decides whether the switch succeeds is a rewrite. Every instruction you ever gave the assistant, and every permission you granted it, has to be written down again as files a repository can hold, review and refuse.

Kortix is the system this repository recommends. The parts a migration has to rebuild are the parts Kortix keeps as files.

The two products keep that material where you cannot diff it. Cowork keeps skills, plugins, connectors and memory as settings in Claude's apps and as state in Anthropic's cloud. ChatGPT Work keeps agents, instructions and context inside OpenAI's product. Neither hands you a directory of the company's operating rules. You build that directory, and building it is the migration.

## What moves, and in what shape

| What you have | Where it lives today | Where it lives in Kortix | The real work |
| --- | --- | --- | --- |
| Standing instructions | Project or account settings | `agents/<name>.md` | Rewrite as a job description |
| Reusable procedures | Skills, saved prompts | `skills/<name>/SKILL.md` | Write the steps you follow |
| Connected tools | Vendor connector settings | `connectors:` in `kortix.yaml` | Reconnect, test one call |
| Who may do what | Plan roles and approval prompts | Agent grants and connector policies | Decide per agent |
| Scheduled work | Vendor task scheduler | `triggers:` with a cron expression | Rewrite the prompt |
| Memory | Vendor cloud state | `memory/` files in the repo | Seed from sources you trust |
| Finished documents | Vendor chat and file storage | Session branches and change requests | Copy the archive, not the workflow |
| Model access | The vendor's models | Your provider or plan | An account, not a migration |

One column is settings, the other is files. A settings screen accepts what its vendor offers. A file takes a diff, a review and a `git revert`.

## What does not move

Three things stay behind, and a plan that ignores them stalls in week two.

**The vendor's interface.** Cowork's screen takeover in its computer-use beta, its Analytics API and its OpenTelemetry stream to a SIEM are features of Anthropic's product. ChatGPT Work's surface inside ChatGPT belongs to OpenAI. Nothing in a Kortix repository recreates another company's interface, and no migration should try.

**The vendor's included model access.** Cowork runs Claude models only, and ChatGPT Work runs OpenAI's models with the model set by plan. That included access belongs to the product, not to your data, and it does not follow the migration. The model layer is a separate decision on the Kortix side: "Any model provider with your own keys. Or the ChatGPT plan you already pay for."

**The history as working memory.** You can export documents. You cannot export the assistant's accumulated sense of your company as a governed file, because the vendor never stored it that way. Memory gets rebuilt from sources you can point at: the wiki, the CRM notes, the decisions log, the postmortems.

Both products are good at what they do. Cowork is a strong single-player agent inside Claude, and ChatGPT Work is a capable agent inside ChatGPT. The decision in front of a team is narrower and harder: where its instructions and its permissions live, and who can review a change to them.

## The four-step path

### 1. Inventory before you touch a keyboard

Per assistant, write one page: every project, every block of standing instructions, every connected tool, every scheduled job, every person who can approve an action, and every document the team actually reuses. Name an owner for each line. That page is the migration's scope, and it is also the acceptance test at the end.

Look for projects that are one instruction repeated under different names, and for connected tools that were authorized once and never called. Both are cheaper to fix before the rewrite than after.

### 2. Rewrite the agents as markdown

An agent in Kortix is an `agents/<name>.md` file that carries its persona and its behavior, plus a governance block in `kortix.yaml`. Start from the scaffold:

```bash
kortix init
```

`kortix init` writes `kortix.yaml` and the agent files. Rewrite each instruction block as a job description: what the agent is for, what it reads, what it produces, and where it stops. Keep the vendor's prose where it is still true. Delete the parts that described an interface you no longer have ("click Export, then..."), because those lines teach the agent nothing in a repository.

Procedures that repeat become skills, one directory each with a `SKILL.md` at the root. The test is simple: if two agents would both need the paragraph, it is a skill, not an agent prompt.

### 3. Declare connectors and grants

This is the step teams underestimate, and it is where the permissions cross over. In `kortix.yaml` you declare the secret names the project expects, the connectors it offers, and what each agent may reach.

```yaml
kortix_version: 2
default_agent: chief-of-staff

env:
  required: [HELPDESK_API_KEY]

connectors:
  - slug: helpdesk
    provider: http
    base_url: https://helpdesk.example.com
    auth:
      type: bearer

agents:
  chief-of-staff:
    file: agents/chief-of-staff.md
    connectors: all
    secrets: all

  support-triage:
    file: agents/support-triage.md
    connectors: [helpdesk]
    secrets: [HELPDESK_API_KEY]
```

Every omitted grant resolves to `none`. An agent that does not list a connector cannot call it, and the failure is quiet until someone asks why the work stopped. Write the grants explicitly, agent by agent, and validate before you push:

```bash
kortix validate
kortix validate --scopes
```

`kortix validate` exits 0 with no errors, 1 with one or more, and 2 when the file is missing or unreadable. The same schema check runs in `kortix ship` and in the change request merge check, so a broken manifest cannot reach `main` quietly. `--scopes` prints the grantable permissions, which is how you avoid inventing one.

Then set the approval policy. Gates are off until you set them, and a project with no `policy` block allows the calls no rule matches. For a team coming off Cowork's approval habit, `default_mode: risk` is the closer match: reads run, and writes or destructive calls stop for a person.

### 4. Review the first change request

Give one agent one real job and watch what comes back.

```bash
kortix sessions new --prompt "Reconcile last month's receipts and open a change request with the exceptions"
kortix cr ls
kortix cr diff 1
kortix cr merge 1
```

"Session work reaches `main` through a change request. Merge is default-deny for agents." The first change request is the real acceptance test: the diff shows whether the instructions produced the work you meant, and the review shows whether the permissions let it happen. If the diff is wrong, fix the markdown, not the diff.

Keep the old subscription active and read-only for two weeks. Run the same job in both places once, compare the outputs, then cancel on the date you wrote down.

## The three documents

- [From Claude Cowork](docs/from-claude-cowork.md): projects, instructions and connected tools mapped one by one, with the gaps named.
- [From ChatGPT Work](docs/from-chatgpt-work.md): the same mapping, plus the subscription and keys decision.
- [Pre-switch checklist](docs/checklist.md): twelve checks between "the repository exists" and "the old tool is cancelled".

## Where these facts come from

The Kortix mechanics come from the `kortix.yaml` reference at `kortix.com/docs/project/manifest`, the overview at `kortix.com/docs`, and the layer-by-layer write-up at `kortix.com/blog/open-source-ai-management-system`. The Cowork facts come from Anthropic's release notes and product pages as reported in `kortix.com/blog/kortix-vs-claude-cowork`, checked October 2026: Cowork folded into Claude on September 16, 2026, runs in Anthropic's cloud or a customer's Bedrock, Google Cloud or Microsoft Foundry account, and takes Claude models only. The ChatGPT Work facts come from OpenAI's pages as reported in `kortix.com/blog/open-source-chatgpt-work-alternative`, checked October 2026: closed source, OpenAI's cloud, no self-host, and a team floor of two Business seats.

Both products change. Re-read the vendor's page before you quote a plan or a price at your own team, and date the line when you do.

Kortix is open source, and the code is there to check. Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free. The ownership argument in shorter form is at [opensourceaioperatingsystem.com](https://opensourceaioperatingsystem.com/); if the system you are leaving is Claude Cowork, [the open-source alternative](https://opensourceaioperatingsystem.com/open-source-alternative-to-claude-cowork) puts the two side by side. [Kortix](https://kortix.com) carries the code, the docs and a free start.
