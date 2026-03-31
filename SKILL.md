---
name: create-teammate
description: "Distill a teammate into an AI Skill. Auto-collect Slack/Teams/GitHub data, generate Work Skill + Persona, with continuous evolution."
argument-hint: "[teammate-name-or-slug]"
version: "1.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
platforms:
  - claude-code
  - openclaw
  - agentskills-compatible
---

> **Language**: This skill auto-detects the user's language from their first message and responds in the same language throughout. Instructions below are in English; the skill will mirror the user's language in all output.

# teammate.skill Creator

## Trigger Conditions

Activate when the user says any of:
- `/create-teammate`
- "Help me create a teammate skill"
- "I want to distill a teammate"
- "New teammate"
- "Make a skill for XX"

Enter evolution mode when the user says:
- "I have new files" / "append" / "add more context"
- "That's wrong" / "They wouldn't do that" / "They should be"
- `/update-teammate {slug}`

List all generated teammates when the user says `/list-teammates`.

---

## Tool Usage Rules

This Skill runs in any AgentSkills-compatible environment (Claude Code, OpenClaw, or other agents) with the following tools:

| Task | Tool |
|------|------|
| Read PDF documents | `Read` tool (native PDF support) |
| Read image screenshots | `Read` tool (native image support) |
| Read MD/TXT/JSON files | `Read` tool |
| Slack export JSON | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/slack_parser.py` |
| Slack auto-collect | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/slack_collector.py` |
| Teams/Outlook export | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/teams_parser.py` |
| Gmail .mbox / .eml | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py` |
| Notion export | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/notion_parser.py` |
| GitHub PR/review data | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/github_collector.py` |
| Linear/JIRA export | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/project_tracker_parser.py` |
| Confluence export | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/confluence_parser.py` |
| Write/update Skill files | `Write` / `Edit` tool |
| Version management | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| List existing Skills | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**Base directory**: Skill files are written to `./teammates/{slug}/` (relative to this project directory).

For platform-specific global install paths:
- Claude Code: `--base-dir ~/.claude/skills/teammates`
- OpenClaw: `--base-dir ~/.openclaw/workspace/skills/teammates`
- Other agents: use your agent's skill directory

---

## Main Flow: Create a New Teammate Skill

### Step 1: Basic Info Collection (3 questions)

Refer to `${CLAUDE_SKILL_DIR}/prompts/intake.md` for the question sequence. Only ask 3 questions:

1. **Name / Alias** (required)
   - Example: `alex-chen` or `Big Mike`
2. **Role info** (one sentence: company, level, title, any detail — all optional)
   - Example: `Google L5 backend engineer` or `Stripe senior frontend` or `Series B startup, lead designer`
3. **Personality profile** (one sentence: MBTI, traits, culture, your impression — all optional)
   - Example: `INTJ, perfectionist, very Google-style, gives brutal CR feedback but is usually right`

Everything except the name can be skipped. Summarize and confirm before moving on.

### Step 2: Source Material Import

Ask the user how they'd like to provide materials:

```
How would you like to provide source materials?

  [A] Slack Auto-Collect (recommended)
      Enter their Slack username — auto-pull messages, threads, reactions

  [B] GitHub Auto-Collect
      Enter their GitHub handle — auto-pull PRs, reviews, comments, commits

  [C] Upload Files
      Slack export JSON / Gmail .mbox / Notion export / Confluence pages /
      PDF / images / Markdown / JIRA CSV / Linear export

  [D] Paste Text
      Copy-paste directly — meeting notes, chat logs, emails, anything

  [E] Provide Links
      Notion pages, Confluence docs, Google Docs (via share link)

Mix and match, or skip entirely (generate from manual info only).
```

#### Option A: Slack Auto-Collect

First-time setup:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/slack_collector.py --setup
```

After setup:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/slack_collector.py \
  --username "{slack_username}" \
  --output-dir ./knowledge/{slug} \
  --msg-limit 1000 \
  --channel-limit 20
```

Auto-collected content:
- All messages sent by them in shared channels (system messages filtered)
- Thread replies and reactions
- Pinned messages and bookmarks they created

After collection, `Read` the output files:
- `knowledge/{slug}/messages.txt` → messages
- `knowledge/{slug}/threads.txt` → thread conversations
- `knowledge/{slug}/collection_summary.json` → collection summary

If collection fails (insufficient permissions / bot not in channels), inform user to:
1. Add the Slack App to relevant channels
2. Or switch to Option C (upload Slack export)

#### Option B: GitHub Auto-Collect

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/github_collector.py \
  --username "{github_handle}" \
  --repos "{repo1,repo2}" \
  --output-dir ./knowledge/{slug} \
  --pr-limit 50 \
  --review-limit 100
```

Auto-collected content:
- PR descriptions and commit messages they authored
- Code review comments they left on others' PRs
- Issue comments and discussions
- README/doc contributions

After collection, `Read`:
- `knowledge/{slug}/prs.txt` → PR descriptions + commit messages
- `knowledge/{slug}/reviews.txt` → code review comments
- `knowledge/{slug}/issues.txt` → issue activity

#### Option C: Upload Files

- **PDF / Images**: `Read` tool directly
- **Slack export JSON**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/slack_parser.py --file {path} --target "{name}" --output /tmp/slack_out.txt
  ```
- **Gmail .mbox / .eml**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py --file {path} --target "{name}" --output /tmp/email_out.txt
  ```
- **Notion export (HTML/MD)**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/notion_parser.py --dir {path} --target "{name}" --output /tmp/notion_out.txt
  ```
- **Confluence export**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/confluence_parser.py --file {path} --target "{name}" --output /tmp/confluence_out.txt
  ```
- **JIRA CSV / Linear export**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/project_tracker_parser.py --file {path} --target "{name}" --output /tmp/tracker_out.txt
  ```
- **Markdown / TXT**: `Read` tool directly

#### Option D: Paste Text

User-pasted content is used directly as text material. No tools needed.

#### Option E: Provide Links

When the user provides URLs, attempt to fetch content:
- Notion public pages → fetch and parse
- Google Docs (shared) → fetch and parse
- Confluence (if auth configured) → fetch via API
- Other URLs → attempt web fetch, fall back to asking for export

If the user says "no files" or "skip", generate Skill from Step 1 manual info only.

### Step 3: Analyze Source Material

Combine all collected materials and user-provided info, analyze along two tracks:

**Track A (Work Skill)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/work_analyzer.md` for extraction dimensions
- Extract: responsible systems, technical standards, workflow, output preferences, experience
- Emphasize different aspects by role type (backend/frontend/ML/product/design/data/DevOps)

**Track B (Persona)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` for extraction dimensions
- Translate user-provided tags into concrete behavior rules (see tag translation table)
- Extract from materials: communication style, decision patterns, interpersonal behavior

### Step 4: Generate and Preview

Use `${CLAUDE_SKILL_DIR}/prompts/work_builder.md` to generate Work Skill content.
Use `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` to generate Persona content (5-layer structure).

Show the user a summary (5-8 lines each):
```
Work Skill Summary:
  - Responsible for: {xxx}
  - Tech stack: {xxx}
  - CR focus: {xxx}
  ...

Persona Summary:
  - Core personality: {xxx}
  - Communication style: {xxx}
  - Decision pattern: {xxx}
  ...

Confirm generation? Or need adjustments?
```

### Step 5: Write Files

After user confirmation, execute:

**1. Create directory structure** (Bash):
```bash
mkdir -p teammates/{slug}/versions
mkdir -p teammates/{slug}/knowledge/docs
mkdir -p teammates/{slug}/knowledge/messages
mkdir -p teammates/{slug}/knowledge/emails
```

**2. Write work.md** (Write tool):
Path: `teammates/{slug}/work.md`

**3. Write persona.md** (Write tool):
Path: `teammates/{slug}/persona.md`

**4. Write meta.json** (Write tool):
Path: `teammates/{slug}/meta.json`
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO_timestamp}",
  "updated_at": "{ISO_timestamp}",
  "version": "v1",
  "profile": {
    "company": "{company}",
    "level": "{level}",
    "role": "{role}",
    "mbti": "{mbti}"
  },
  "tags": {
    "personality": [...],
    "culture": [...]
  },
  "impression": "{impression}",
  "knowledge_sources": [...],
  "corrections_count": 0
}
```

**5. Generate full SKILL.md** (Write tool):
Path: `teammates/{slug}/SKILL.md`

```markdown
---
name: teammate-{slug}
description: {name}, {company} {level} {role}
user-invocable: true
---

# {name}

{company} {level} {role}{append details if available}

---

## PART A: Work Capabilities

{full work.md content}

---

## PART B: Persona

{full persona.md content}

---

## Execution Rules

1. PART B decides first: what attitude to take on this task?
2. PART A executes: use your technical skills to complete the task
3. Always maintain PART B's communication style in output
4. PART B Layer 0 rules have the highest priority and must never be violated
```

Inform user:
```
✅ Teammate Skill created!

Location: teammates/{slug}/
Commands: /{slug} (full version)
          /{slug}-work (work capabilities only)
          /{slug}-persona (persona only)

If something feels off, just say "they wouldn't do that" and I'll update it.
```

---

## Evolution Mode: Append Files

When user provides new files or text:

1. Read new content using Step 2 methods
2. `Read` existing `teammates/{slug}/work.md` and `persona.md`
3. Refer to `${CLAUDE_SKILL_DIR}/prompts/merger.md` for incremental analysis
4. Archive current version:
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./teammates
   ```
5. Use `Edit` tool to append incremental content
6. Regenerate `SKILL.md` (merge latest work.md + persona.md)
7. Update `meta.json` version and updated_at

---

## Evolution Mode: Conversation Correction

When user expresses "that's wrong" / "they wouldn't do that":

1. Refer to `${CLAUDE_SKILL_DIR}/prompts/correction_handler.md`
2. Determine if it belongs to Work or Persona
3. Generate correction record
4. Use `Edit` to append to `## Correction Log` section
5. Regenerate `SKILL.md`

---

## Management Commands

`/list-teammates`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./teammates
```

`/teammate-rollback {slug} {version}`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./teammates
```

`/delete-teammate {slug}`:
After confirmation:
```bash
rm -rf teammates/{slug}
```
