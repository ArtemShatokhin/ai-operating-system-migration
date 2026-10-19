# The pre-switch checklist

Twelve checks between a plan and a working system. Each one is either a file in the repository or a fact you can verify from the command line. A migration that passes all twelve is ready to run beside the old tool for two weeks, and ready to cancel it after that.

## The repository

- [ ] **1. One owner and one cutover date.** Done when a named person owns the migration and a calendar date exists for the old tool to go read-only. Check: both are written in the repository's first issue, not in a chat.

- [ ] **2. The inventory page is complete.** Done when every project, instruction block, connected tool, scheduled job, approver and reused document has a line and an owner. Check: the page names nothing that lives only in the vendor's interface.

- [ ] **3. The repository exists and validates.** Done when `kortix init` has scaffolded `kortix.yaml` and the agent files, the work is committed, and the project opens from the dashboard or `kortix ship`. Check: `kortix validate` exits 0 with no errors.

- [ ] **4. Every agent is a markdown file.** Done when one agent exists per job in `agents/<name>.md`, and each reads as a job description: goal, inputs, output, stopping condition. Check: no instruction tells the agent where to click.

- [ ] **5. Repeated procedures are skills.** Done when any paragraph two agents would both need lives in `skills/<name>/SKILL.md`, not copied into both prompts. Check: `ls skills/*/SKILL.md` returns the procedures from the inventory.

## The permissions

- [ ] **6. Secret names are declared, values are not.** Done when every name an agent or trigger needs appears under `env`, and every value is in the project's Secrets store. Check: search the repository for the first characters of each value and find nothing.

- [ ] **7. Connectors are declared and tested.** Done when each tool the inventory names has a `connectors:` entry with its provider and auth, and one read call has returned real data. "Connector credentials are brokered server-side and never enter the machine." Check: the connector list in `kortix.yaml` matches the inventory, with no extras.

- [ ] **8. Grants are per agent, never assumed.** Done when each agent block lists its own `connectors`, `secrets` and permissions, and a grant you meant to withhold is absent. Check: `kortix validate --scopes` has been read, and no agent carries `all` unless its job is the general one.

- [ ] **9. Policies are deliberate.** Done when `policy.default_mode` is chosen and each connector that needs a gate has one set to allow, ask or block. Check: the team can say what happens to a write that matches no rule. Gates are off until you set them.

- [ ] **10. One trigger has fired once.** Done when a cron schedule or a signed webhook has started a session with nobody present, and the output was read. Check: `kortix triggers add` wrote the entry, the timezone is an IANA name, and the trigger appears in the repository.

## The proof

- [ ] **11. The review path is agreed.** Done when the team knows who reviews a change request, who answers an ask, and which agents, if any, hold `project.cr.merge`. "Session work reaches `main` through a change request. Merge is default-deny for agents." Check: an agent's merge grant is absent unless an admin added it on purpose.

- [ ] **12. The first change request is merged.** Done when one real job has run end to end, its diff was reviewed as a diff, and the result is on the default branch. Check: `kortix cr ls` shows the request, and `kortix cr diff 1` showed the files before `kortix cr merge 1` ran.

## After the twelfth

Run the old tool read-only for two weeks. Give it the same job once, compare the two outputs, and write down which one a person would have shipped. Cancel on the date from item 1. If the comparison goes the wrong way, the diff tells you which agent file to fix, and the fix is a change request rather than a support ticket.
