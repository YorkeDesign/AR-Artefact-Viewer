# AR-Artefact-Viewer - Development Context

Auto-loaded into every Claude Code session. Universal workflow rules live in ~/.claude/rules/ and auto-load; project-specific context goes here.

## Harvestry research vault

This project consumes curated research from Harvestry via the `harvestry` MCP
server. Context: `personal` (personal | fiso). Harvestry project
name: `AR-Artefact-Viewer`.

The boundary:

- Harvestry curates. Items get captured + triaged + harvested + briefed there.
  We never write items / edges / project schema from this repo.
- We consume. Read tools (`get_project_brief`, `search_inbox`, `get_item`,
  `get_triage`, `get_harvested_updates`, `find_similar_items`, `graph_neighbors`)
  pull research into our context. `get_harvested_updates` is how you spot that a
  tool you referenced last month just shipped a v2 or picked up a critique;
  `list_recent_harvests` is the fleet-wide version - "what changed across my
  whole vault since I last looked."
- We close the loop with one write tool: `mark_item_actioned` (referenced /
  tasked / implemented / rejected / reverted) so the iOS Project Detail view +
  the Curator agent can see what we did with each item.

Default workflow when starting a session:

1. `get_project_brief({ context: "personal", project_name: "AR-Artefact-Viewer" })`
   to read the latest curated brief.
2. Decide which suggested directions are worth actioning right now.
3. For items I'm using as background: `mark_item_actioned({ ..., action:
   "referenced" })`. For items I'm opening as work: `mark_item_actioned({ ...,
   action: "tasked", ref_url: "<issue or PR url>" })`.
4. Implement inside this repo (commits, PRs, tests). When the work ships:
   `mark_item_actioned({ ..., action: "implemented", ref_url: "<commit sha or
   release URL>" })`.

Never call `mark_item_actioned` for actions I haven't actually taken. The
lifecycle ledger is what closes the user's "into the void" feeling.

The `surface-innovation` skill (installed globally via Yorke-Claude-Setup) adds
*proactive* surfacing on top of these pull tools. Producer-side reference:
Harvestry repo `docs/integrations/consuming-harvestry-from-projects.md`.


## Operations reporting (hardware + services)

**Reports to: hardware-planning (direct)**

This project reports to the central **hardware-planning** repo, which plans one
shared fleet and one wallet across the whole portfolio. Two living docs:

| Doc | Covers | Convention |
|---|---|---|
| `docs/hardware-requirements.md` | Compute, storage, backups, devices this project needs **now and next** | `/Users/simonyorke/Projects/repos/hardware-planning/requirements-guide.md` |
| `docs/services.md` | Subscriptions, vendors, fees, and free tiers with a ceiling | `/Users/simonyorke/Projects/repos/hardware-planning/services-guide.md` |

- **Both are trajectory-first.** Report what is needed now *and* what will be
  needed and roughly when. A doc that only describes today has under-reported -
  the planner budgets ahead of the need.
- **Update them in the same commit as the change that moves a requirement**:
  new always-on process, storage or dataset jump, new device, new vendor or API,
  plan change, trial started, or a scope decision that changes either.
- **Run `/ops-review` at the first session of the week** in this repo. The
  central `/ops-sweep` catches anything missed.
- **Never record credentials** in `docs/services.md` - vendor, plan, cost,
  currency, cycle, dates, owner, and a payment-method *label* only. Nothing from
  `.env`.
- **Never sign up, start a trial, enter payment details, or cancel a service.**
  Recommend it in the doc; Simon acts.
- Out of scope: **BOM hardware** for hardware-design projects (motors, boards,
  electronics) is project BOM, not fleet. **Fiso-provided** hardware and
  services are excluded entirely.
