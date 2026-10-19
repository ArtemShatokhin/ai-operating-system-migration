# From Claude Cowork to a Kortix repository

Kortix is the open-source AI Operating System. A team leaving Claude Cowork faces a rewrite of instructions and permissions. The files are the small part; the rewrite is where the weeks go.

## What a Cowork team has today

Anthropic folded Cowork into Claude on September 16, 2026. What used to be a separate product now starts from any conversation ([release notes](https://support.claude.com/en/articles/12138966-release-notes)). Chats, Cowork tasks, projects, connectors and skills carried over. Pro and Max plans saw the change first, on a gradual rollout across web, desktop and mobile, while enterprise administrators got at least 30 days' notice and [an account that moves does not move back](https://support.claude.com/en/articles/16761823-claude-cowork-and-chat-are-one-claude). It returns finished documents, spreadsheets and presentations, and its scheduled tasks keep running while the laptop is closed.

Three properties of that setup decide the migration.

- **The configuration lives in Anthropic's product.** Skills, plugins, connectors and memory are settings in Claude's apps, and they are state in Anthropic's cloud. There is no repository on your side to version, diff or roll back.
- **The work runs on Anthropic's terms.** Cowork runs in Anthropic's cloud, or in your own Amazon Bedrock, Google Cloud or Microsoft Foundry account, and every task runs on a Claude model. There is [no version your team hosts](https://claude.com/product/cowork), and no bring-your-own-model path.
- **The bill is one pool.** Everything you do with Claude counts against the plan's usage limits. Agentic work draws that pool faster than chat.

None of that is a defect for one person. It becomes the deciding question when a company needs to review its instructions and permissions.

## The mapping, one line at a time

| In Cowork | In a Kortix repo | What you rewrite |
| --- | --- | --- |
| Projects | A project repo with `kortix.yaml` | The scope and owner of each project |
| Standing instructions | `agents/<name>.md` | The prose, as a job description |
| Skills | `skills/<name>/SKILL.md` | The procedure, step by step |
| Plugins | Connectors plus skills | Each bundled tool, reconnected |
| Connectors | `connectors:` in `kortix.yaml` | The auth method and one test call |
| Scheduled tasks | `triggers:` with a cron expression | The prompt and the timezone |
| Memory | `memory/` files in the repo | Seeded from sources you can cite |
| Approval prompts | `policy` and connector policies | Written as allow, ask or block |

## Projects and standing instructions

A Cowork project is a container: a name, a set of files, an instruction block and often a connected tool. In Kortix the container is a git repository, and the instruction block becomes an agent file.

```bash
kortix init
```

`kortix init` scaffolds `kortix.yaml` and an agent file per role. Move each project's instruction block into `agents/<name>.md`, and rewrite it as a job description: the goal, the inputs, the output, the stopping condition, and the actions that need a person. Then declare the agent in `kortix.yaml` so governance sits beside behavior:

```yaml
agents:
  chief-of-staff:
    file: agents/chief-of-staff.md
    connectors: [crm, drive]
    secrets: [CRM_TOKEN]
```

An omitted grant resolves to `none`, so an agent reaches only the connectors and secrets it names. That is the permission model in one line, and it is the part a settings screen never made you write down.

One rewrite rule matters more than the rest. Cowork instructions tend to describe a surface: where to click, which panel to open, what the screen should show. An agent in a repository reads files and calls tools. Every sentence about a surface has to become a sentence about an input, a tool or an output. What survives is judgment: when to stop, what to escalate, what a good result looks like.

## Skills and plugins

A Cowork skill is a reusable procedure. In Kortix a skill is a directory with a `SKILL.md` at its root, so the procedure gets a file, an author and a diff.

Take the skills the team actually uses and leave the defaults where they are. For each, write the steps in the order a careful person performs them, name the tools it needs, and state what a good result looks like. A plugin bundles several tools, knowledge and workflows into one install. On the Kortix side it splits: the tools become connectors, the knowledge becomes a skill or a memory file, and the workflow becomes the agent's prompt.

## Connectors

Reconnect each tool once for the project. Kortix declares connectors in `kortix.yaml`, where a connector's `provider` is one of `composio`, `pipedream`, `mcp`, `openapi`, `postman`, `graphql` or `http`, and an agent's `connectors` grant decides which ones it can call.

"Connector credentials are brokered server-side and never enter the machine." That changes what a migration has to move. You reauthorize the tool, and you do not copy a key out of Cowork.

Test each connector with one read call before you write an agent prompt against it. A connector that authenticates but returns nothing is a prompt bug you will otherwise debug twice. Credentials live in the platform, never in the manifest, so the repository can be shared with the team that reviews it.

## Scheduled tasks

Cowork's scheduled tasks run unattended, and since July 7, 2026 its sessions have run remotely, so the work continues after the laptop closes. In Kortix the equivalent is a trigger, and there are two kinds: a schedule, or a webhook that carries a signing secret.

```yaml
triggers:
  - slug: monday-digest
    type: cron
    agent: chief-of-staff
    cron: "0 0 9 * * 1"
    timezone: Europe/Belgrade
    prompt: |
      Summarize last week's pipeline changes and open one change request.
```

Rewrite the prompt, keep the schedule, and set the timezone explicitly. A cron trigger fires at most once every 60 seconds, and the prompt is a file in the repository, so the overnight job has an author and a history like everything else. A webhook trigger needs a signing secret and a name for it in `env`. A webhook that names no signing secret is rejected at validation.

## Memory

Cowork memory spans chat and Cowork in the cloud. Free, Pro and Max plans have it on by default, while Team and Enterprise organizations start with it off, a setting an administrator can change.

Kortix memory is files under `memory/` in the project repository. Seed it from sources you can point at: the decisions log, the CRM notes, the support macros, the postmortems. Do not try to recreate a year of chat context. Write the facts a new hire needs, and let the agent add to them through change requests, so every addition gets an author and a reviewer.

## Approval behavior

Cowork shows its plan and waits before anything significant, and the merged experience adds a permission setting. Manual, the default, asks before each action. Auto keeps working under automated safety checks.

Kortix gives you two places to express the same intent. Connector policies answer per action with allow, ask or block, and a rule can match the arguments of a call. The project policy answers for everything else:

```yaml
policy:
  default_mode: risk
```

Gates are off until you set them. A project that declares no policy block falls back to allowing the calls no rule matches, so choose the default deliberately and put an explicit ask on the step that matters. A person answering an ask holds the call, and the agent resumes after the decision.

## The gaps, named

- **No configuration export.** Anthropic does not hand you a directory of projects, skills and connectors. There is no shortcut around that rebuild.
- **Chat history is not memory.** You can keep the documents. The accumulated context stays where it was, in a product you are leaving.
- **Hosted features stay hosted.** Enterprise activity streaming to a SIEM over OpenTelemetry, usage analytics and the Analytics API are Anthropic product features. A repository does not recreate them.
- **Model access is separate.** Cowork runs Claude models only. On the Kortix side the choice opens up: "Any model provider with your own keys. Or the ChatGPT plan you already pay for."
- **Hosting becomes a choice.** Cowork has no version your team hosts. Decide where the new system runs before the repository is created, because that choice sets the network and the accounts.

## The order that works

1. Inventory every project, instruction block, skill, connector, scheduled task and approver.
2. Rewrite agents, one per job, and delete instructions that describe a surface you no longer use.
3. Extract the repeated procedures into skills.
4. Reconnect tools, declare secret names, and grant each agent only what its job needs.
5. Move schedules to triggers, with the timezone written down.
6. Set the policy, then run one real job and review the change request.

"Session work reaches `main` through a change request. Merge is default-deny for agents." That review is the last check, and the first one a Cowork team feels: the work arrives as a diff somebody can refuse.

For the checklist version, see [the pre-switch checklist](checklist.md). The Cowork facts come from Anthropic's release notes and product pages, checked October 2026.
