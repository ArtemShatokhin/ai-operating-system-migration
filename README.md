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

List every instruction, skill, connector, schedule and approval the team relies on. Ask who uses each one. Half the list usually belongs to one person and can be cut.

### 2. Write the agents as markdown

An agent is a file with a persona and a permission tree. Write the job, not the tool: what it may read, what it may change, when it stops for a person. The syntax and the full manifest are in [the layer-by-layer account](https://kortix.com/blog/open-source-ai-management-system).

### 3. Declare the connectors and the grants in `kortix.yaml`

The manifest names the connectors, the secret names and what each agent may touch. A grant you leave out resolves to `none`, so the first draft is safe by omission. Validate before you push with `kortix validate`.

### 4. Review the first change request

Start one real job and let the agent finish it on its own branch. Read the diff, merge it, then retire the old workflow. That first merge is the migration's proof, not the export.

## Docs

- [From Claude Cowork](docs/from-claude-cowork.md): what a Cowork team has, what each piece becomes, and the gaps.
- [From ChatGPT Work](docs/from-chatgpt-work.md): the same for a ChatGPT Work team, including the subscription and keys decision.
- [The pre-switch checklist](docs/checklist.md): twelve items to tick before the old subscription goes quiet.

Every command and key here comes from Kortix's own documentation: [kortix.com/docs](https://kortix.com/docs), the [manifest reference](https://kortix.com/docs/project/manifest) and the [CLI reference](https://kortix.com/docs/cli). [Kortix on GitHub](https://github.com/kortix-ai/suna) carries the code. A shorter version of the ownership argument is at [opensourceaioperatingsystem.com](https://opensourceaioperatingsystem.com/).

Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free. [Try Kortix](https://kortix.com).
