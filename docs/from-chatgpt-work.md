# From ChatGPT Work to a Kortix repository

Kortix is the open-source AI Operating System. A team leaving ChatGPT Work faces a rebuild of its instructions and permissions, and one question it does not face when leaving a desktop tool: what happens to the subscription it already pays for.

## What ChatGPT Work is

OpenAI launched ChatGPT Work on July 9, 2026 as a mode inside ChatGPT ([announcement](https://openai.com/index/chatgpt-for-your-most-ambitious-work)). It takes action across apps and files, stays with a project for hours, and turns a goal into a document, spreadsheet or presentation. It shares ChatGPT's login and billing, and the model and the agent ship together, with the model set by the plan.

For a person working inside ChatGPT, that is a good deal. For a company, five properties decide the migration:

- closed source, so you take the behavior OpenAI ships;
- OpenAI's cloud only, with no self-host;
- OpenAI's models, with the model gated by plan;
- agents, instructions and context inside OpenAI's product, with no repo to diff or roll back;
- metered access, where usage draws from the plan's allowance and credits buy more.

## The mapping, one line at a time

| In ChatGPT Work | In a Kortix repo | What you rewrite |
| --- | --- | --- |
| The Work mode inside ChatGPT | A project repository | The job, outside the chat |
| Instructions and context | `agents/<name>.md` | Standing context as a role |
| Connected apps and files | `connectors:` in `kortix.yaml` | Each app, reconnected and tested |
| Long-running work | A session on its own branch | The goal, as a repeatable prompt |
| Deliverables | Change request diffs and files | Where the output lands |
| Model access | Your provider or plan | The account and the model choice |
| Access and billing | Project credits and your keys | Who pays for what |
| History | An archive you copy | What is worth keeping |

## Instructions and context become agent files

ChatGPT Work keeps agents, instructions and context inside OpenAI's product. You can read them there, and you cannot version them, which means no diff shows what changed the week an agent started behaving differently.

In Kortix the standing context becomes a file. Start with the scaffold:

```bash
kortix init
```

Move each job into `agents/<name>.md` and write it as a role: what the agent is for, what it reads, what it produces, and when it stops. Then declare it in `kortix.yaml` so the governance sits beside the behavior:

```yaml
agents:
  board-reporter:
    file: agents/board-reporter.md
    connectors: [finance, drive]
    secrets: [FINANCE_TOKEN]
```

An omitted grant resolves to `none`. That is the part a ChatGPT team has never had to write: the agent reaches only the connectors and secrets its block names, and every change to that list arrives as a diff somebody reviews.

## Connected apps become connectors

Reconnect each app once for the project. Kortix declares connectors in `kortix.yaml`; a connector's `provider` is one of `composio`, `pipedream`, `mcp`, `openapi`, `postman`, `graphql` or `http`, and an agent's `connectors` grant decides which ones it can call.

"Connector credentials are brokered server-side and never enter the machine." You reauthorize the app, and you do not copy a key out of ChatGPT. Credentials live in the platform, never in the manifest, so the repository can be read by everyone who reviews it.

Test each connector with one read call before an agent prompt depends on it. The failure this catches is the common one: an app that authenticates and still returns an empty result because the account it authorized cannot see the folder the job needs.

## Long-running work becomes a session with a branch

ChatGPT Work stays with a project for hours. A Kortix session works on its own branch of the project repository, and what it produces comes back for review.

```bash
kortix sessions new --prompt "Rebuild the Q3 board pack from the finance folder and open a change request"
kortix cr ls
kortix cr diff 1
```

"Session work reaches `main` through a change request. Merge is default-deny for agents." That is the structural difference a ChatGPT team feels first. In Work the deliverable returns to the chat. In Kortix the deliverable arrives as a diff against the repository, so a person can read what changed before it becomes the company's record.

## The subscription and keys question

This is where teams expect a hard cutover and usually get an easier answer.

"Any model provider with your own keys. Or the ChatGPT plan you already pay for." The subscription is a way to reach models, and Kortix can use the one the team already has. What does not travel is Work itself: its included usage, its plan-gated features, and the configuration inside OpenAI's product.

Keep the two decisions apart. Where the agents run is a deployment decision, and it moves to the repository. Which account pays for model calls is a billing decision, and it can stay with OpenAI if that is the arrangement you prefer, or move to a provider key you control. Whether the seats stay is a third decision, and it depends on what the team still uses ChatGPT for.

One caution belongs here. A subscription is a commercial arrangement with its own terms, and those terms can change. Treat the ChatGPT plan as a path that works today. Keep the provider key option open, and a change to the plan stays a billing detail instead of becoming a second migration.

## What does not move

- **Work's instructions and context.** There is no repository on the OpenAI side to clone. The mapping is the rebuild.
- **The deliverable history.** Documents can be downloaded and stored. The thread and the working context stay in ChatGPT.
- **The interface and plan features.** ChatGPT's surface belongs to OpenAI, and a repository does not recreate it.
- **Included usage.** Work's allowance is part of the subscription, not part of the data.
- **Hosting.** ChatGPT Work runs in OpenAI's cloud with no self-host. On the Kortix side, "Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free."

## The cost shape

ChatGPT Business is the plan that carries Work for a team, and every Business workspace needs at least two seats ([OpenAI's overview](https://help.openai.com/en/articles/8792828-chatgpt-business-overview)). Standard seats cost $20 a seat a month billed annually, or $25 month to month, and premium seats cost $100 and $125, [checked October 2026](https://openai.com/business/pricing). The smallest real bill is two Standard seats, $40 a month billed annually before a single agent runs, and usage sits on top of every seat.

That shape is worth writing into the migration plan, because it prices the old system by the seat and the new one by the work. Do not replace one number with another in the plan. Measure both for two weeks, then decide.

## The order that works

1. Inventory every job, instruction block, connected app, approver and deliverable the team actually reuses.
2. Rewrite the jobs as agents, one file each, and delete instructions that describe a surface you no longer use.
3. Extract the repeated procedures into skills.
4. Reconnect apps, declare secret names, and grant each agent only what its job needs.
5. Choose the model path: the ChatGPT plan or a provider key.
6. Run one real job and review the change request before you move a second one.

For the checklist version, see [the pre-switch checklist](checklist.md). The ChatGPT Work facts come from OpenAI's product and pricing pages, checked October 2026.
