# my-story

A web-novel writing project using the [oh-story-claudecode](https://github.com/worldwonderer/oh-story-claudecode)
agent-skill pack (13 skills for scanning rankings, deconstructing, writing, de-AI-ifying, and cover generation).

## Cursor Cloud specific instructions

### What is installed
- The `oh-story-claudecode` skill pack is **vendored into the repo** (copied, not symlinked) at:
  - `.agents/skills/<skill>/` — read by Cursor
  - `.claude/skills/<skill>/` — read by Claude Code
  - `skills-lock.json` at the repo root tracks the pinned source/hash of each skill.
- These are agent-driven skills (Markdown `SKILL.md` + `references/` + a few Node scripts). There is **no app server, build, or test pipeline** in this repo — the "product" is the skills themselves, invoked by the coding agent via slash commands (e.g. `/story`, `/story-setup`, `/story-deslop`) or natural language ("帮我开书", "这篇太AI了").

### Running / verifying the skills' scripts
- The only executable code is a handful of Node scripts under `.agents/skills/story-*/scripts/` (and the `.claude` copies). They use **only Node built-ins — no `npm install` needed** (system Node works; verified on v22).
- The `story-deslop` de-AI ("去AI味") scripts are the quickest end-to-end check:
  ```bash
  cd .agents/skills/story-deslop
  node scripts/check-ai-patterns.js --check <file.md>     # report AI patterns (exit 1 if found)
  node scripts/check-degeneration.js --check <file.md>    # report model degeneration
  node scripts/normalize-punctuation.js <file.md>         # deterministic punctuation cleanup (rewrites file)
  ```
- `check-ai-patterns.js` is report-only for semantic issues (e.g. `不是…而是…`); the actual de-AI rewrite is done by the agent following `SKILL.md`. `normalize-punctuation.js` is the deterministic part (`——`→`，`, `……`→`。`) and edits files in place.
- Syntax-check all bundled scripts: `find .agents/skills -name '*.js' -exec node --check {} \;`

### Updating / reinstalling the pack
- Update to latest upstream: `npx skills update` (needs network; rewrites the vendored files).
- Full reinstall (idempotent): `npx skills add worldwonderer/oh-story-claudecode --skill '*' -a cursor -a claude-code --copy -y`
- The `skills` CLI is fetched on demand via `npx --yes skills ...`; nothing is preinstalled.

### Optional / gated skills
- `story-cover` needs an image API key (`GPT_IMAGE_API_KEY`) to actually generate covers.
- `story-*-scan` and `browser-cdp` drive a real Chrome via CDP to scrape ranking sites; these need a browser + logged-in sessions and network access, so they can't be fully exercised headless-offline.
