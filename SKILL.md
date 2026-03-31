---
name: create-teammate
description: "Distill a teammate into an AI Skill. Auto-collect Slack/Teams/GitHub data, generate Work Skill + 5-layer Persona, with continuous evolution. Use when: user wants to capture a colleague's knowledge before they leave, create an AI version of a teammate, distill tribal knowledge into a reusable skill, or says /create-teammate."
user-invocable: true
argument-hint: "[teammate-name-or-slug]"
---

> **Language**: Auto-detect the user's language from their first message and respond in the same language throughout.

# teammate.skill Creator

## Trigger Conditions

Activate when the user says any of:
- `/create-teammate` or `/create-teammate alex-chen`
- "Help me create a teammate skill"
- "I want to distill a teammate"
- "New teammate" / "Make a skill for XX"

If the user provides a name as an argument (e.g. `/create-teammate alex-chen`), skip Q1 in intake and use it directly as the slug.

Enter evolution mode when:
- "I have new files" / "append" / "add more context"
- "That's wrong" / "They wouldn't do that"
- `/update-teammate {slug}`

List teammates: `/list-teammates`

---

## Quick Start Mode

If the user provides everything in one message (e.g. "Create a teammate: Alex Chen, Google L5 backend engineer, INTJ, perfectionist"), skip the 3-question intake entirely:
1. Parse name, role, personality from the message
2. Show confirmation summary
3. Jump directly to Step 2 (Source Material Import)

This makes single-message creation possible — zero back-and-forth when the user already knows what they want.

---

## Platform Detection & Tool Mapping

Detect the runtime environment and use the correct tools:

| Action | Claude Code | OpenClaw | Other AgentSkills |
|--------|------------|----------|-------------------|
| Read files | `Read` tool | `read` tool | `Read` tool |
| Write files | `Write` tool | `write` tool | `Write` tool |
| Edit files | `Edit` tool | `edit` tool | `Edit` tool |
| Run scripts | `Bash` tool | `exec` tool | `Bash` / `exec` |
| Fetch URLs | `Bash` → curl | `web_fetch` tool | `Bash` → curl |

### Path Resolution

All script/prompt paths use `{baseDir}` — the skill's own directory, auto-resolved by the platform.

- **Claude Code**: `{baseDir}` = `${CLAUDE_SKILL_DIR}` (set by AgentSkills runtime)
- **OpenClaw**: `{baseDir}` = skill directory (auto-resolved from SKILL.md location)
- **Other agents**: resolve relative to the SKILL.md parent directory

### Output Directory

Generated teammate files go to `teammates/{slug}/` under the agent's workspace:

| Platform | Default output path |
|----------|-------------------|
| Claude Code | `./teammates/{slug}/` (project-local) |
| OpenClaw | `./teammates/{slug}/` (workspace-local, `~/.openclaw/workspace/teammates/{slug}/`) |
| Other | `./teammates/{slug}/` (current working directory) |

To install the generated skill globally, copy `teammates/{slug}/SKILL.md` to the platform's skill directory.

---

## Tool Reference

| Task | Command |
|------|---------|
| Parse Slack export | `python3 {baseDir}/tools/slack_parser.py --file {path} --target "{name}" --output /tmp/slack_out.txt` |
| Slack auto-collect | `python3 {baseDir}/tools/slack_collector.py --username "{user}" --output-dir ./knowledge/{slug}` |
| Parse Teams/Outlook | `python3 {baseDir}/tools/teams_parser.py --file {path} --target "{name}" --output /tmp/teams_out.txt` |
| Parse Gmail .mbox | `python3 {baseDir}/tools/email_parser.py --file {path} --target "{name}" --output /tmp/email_out.txt` |
| Parse Notion export | `python3 {baseDir}/tools/notion_parser.py --dir {path} --target "{name}" --output /tmp/notion_out.txt` |
| GitHub auto-collect | `python3 {baseDir}/tools/github_collector.py --username "{user}" --repos "{repos}" --output-dir ./knowledge/{slug}` |
| Parse JIRA/Linear | `python3 {baseDir}/tools/project_tracker_parser.py --file {path} --target "{name}" --output /tmp/tracker_out.txt` |
| Parse Confluence | `python3 {baseDir}/tools/confluence_parser.py --file {path} --target "{name}" --output /tmp/confluence_out.txt` |
| Version backup | `python3 {baseDir}/tools/version_manager.py --action backup --slug {slug} --base-dir ./teammates` |
| Version rollback | `python3 {baseDir}/tools/version_manager.py --action rollback --slug {slug} --version {ver} --base-dir ./teammates` |
| List teammates | `python3 {baseDir}/tools/skill_writer.py --action list --base-dir ./teammates` |

**Reading files**: PDF, images, markdown, text → use the platform's native read tool directly.

---

## Main Flow: Create a New Teammate Skill

### Step 1: Basic Info Collection (3 questions — or fewer)

Read `{baseDir}/prompts/intake.md` for the full question sequence. Only ask 3 questions:

1. **Name / Alias** (required) — e.g. `alex-chen` or `Big Mike`
2. **Role info** (optional, one sentence) — e.g. `Google L5 backend engineer`
3. **Personality profile** (optional, one sentence) — e.g. `INTJ, perfectionist, Google-style, brutal CR feedback`

Everything except name can be skipped. If the user says "skip" or just gives a name, move on immediately — don't keep asking.

After collecting, show a compact confirmation:
```
👤 alex-chen | Google L5 Backend | INTJ, Perfectionist, Google-style
Looks right? (y / change something)
```
One line, not a multi-line summary. Get confirmation fast.

### Step 2: Source Material Import

Present data source options — but keep it conversational, not a wall of text:

```
Now, do you have any of their work artifacts? (all optional)

  • Slack username → I'll auto-pull their messages
  • GitHub handle → I'll pull PRs and reviews
  • Files to upload → Slack export, Gmail, Notion, Confluence, PDF, screenshots
  • Or just paste text — meeting notes, chat logs, whatever you have

You can also skip this entirely — I'll work with what you gave me above.
```

If the user says "skip", "no", or "none", jump straight to Step 3 and generate from the info in Step 1 only. Don't ask again.

#### Option A: Slack Auto-Collect

First-time setup:
```bash
python3 {baseDir}/tools/slack_collector.py --setup
```

Collect data:
```bash
python3 {baseDir}/tools/slack_collector.py \
  --username "{slack_username}" \
  --output-dir ./knowledge/{slug} \
  --msg-limit 1000 \
  --channel-limit 20
```

Then read the output files: `knowledge/{slug}/messages.txt`, `threads.txt`, `collection_summary.json`.

If collection fails, suggest adding the Slack App to channels or switching to Option C.

#### Option B: GitHub Auto-Collect

```bash
python3 {baseDir}/tools/github_collector.py \
  --username "{github_handle}" \
  --repos "{repo1,repo2}" \
  --output-dir ./knowledge/{slug} \
  --pr-limit 50 \
  --review-limit 100
```

Then read: `knowledge/{slug}/prs.txt`, `reviews.txt`, `issues.txt`.

#### Option C: Upload Files

Use the tool reference table above. For each file type, run the appropriate parser. PDF/images/markdown → read directly with platform read tool.

#### Option D: Paste Text

Use pasted content directly as source material. No tools needed.

#### Option E: Provide Links

- **OpenClaw**: use `web_fetch` tool to retrieve page content
- **Claude Code / Other**: use `Bash` → `curl` or browser tool

If user says "skip", generate from Step 1 info only.

### Step 3: Analyze Source Material

Run dual-track analysis on all collected materials:

**Track A (Work Skill)**: Read `{baseDir}/prompts/work_analyzer.md` for extraction dimensions. Extract: responsible systems, technical standards, workflow habits, output preferences, domain experience.

**Track B (Persona)**: Read `{baseDir}/prompts/persona_analyzer.md` for extraction dimensions. Extract: communication style, decision patterns, interpersonal behavior, cultural tags → concrete behavior rules.

### Step 4: Generate and Preview

Read `{baseDir}/prompts/work_builder.md` to generate Work Skill content.
Read `{baseDir}/prompts/persona_builder.md` to generate Persona content (5-layer structure).

Show a concise preview card (not full content — just the highlights):
```
━━━ Preview: alex-chen ━━━

💼 Work Skill:
  • Owns: Payments Core, webhook pipeline, idempotency layer
  • Stack: Ruby (Sorbet), Go, PostgreSQL, Kafka
  • CR focus: idempotency, error handling, naming, financial precision

🧠 Persona:
  • Style: Short & direct, conclusion-first, zero emoji
  • Decision: Correctness > Clarity > Simplicity > Speed
  • Signature: "What problem are we actually solving?"

━━━━━━━━━━━━━━━━━━━━━━━

Looks right? Or want to tweak something before I write the files?
```
Keep to 10–12 lines max. If user says "yes" / "good" / "ok" / "👍", proceed to write immediately.

### Step 5: Write Files

After confirmation, create the teammate:

**1. Create directories:**
```bash
mkdir -p teammates/{slug}/versions
mkdir -p teammates/{slug}/knowledge/docs
mkdir -p teammates/{slug}/knowledge/messages
mkdir -p teammates/{slug}/knowledge/emails
```

**2. Write `teammates/{slug}/work.md`** — full work skill content

**3. Write `teammates/{slug}/persona.md`** — full persona content (5-layer)

**4. Write `teammates/{slug}/meta.json`:**
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO_timestamp}",
  "updated_at": "{ISO_timestamp}",
  "version": "v1",
  "profile": { "company": "", "level": "", "role": "", "mbti": "" },
  "tags": { "personality": [], "culture": [] },
  "impression": "",
  "knowledge_sources": [],
  "corrections_count": 0
}
```

**5. Write `teammates/{slug}/SKILL.md`** (the generated teammate skill):
```markdown
---
name: teammate-{slug}
description: "{name} — {company} {level} {role}. Invoke to get responses in their voice and style."
user-invocable: true
---

# {name}

{company} {level} {role}

---

## PART A: Work Capabilities

{full work.md content}

---

## PART B: Persona

{full persona.md content}

---

## Execution Rules

1. PART B decides first: what attitude to take on this task?
2. PART A executes: use technical skills to complete the task
3. Always maintain PART B's communication style in output
4. PART B Layer 0 rules have highest priority — never violate
```

**5b. Auto-install the generated skill:**

After writing the files, automatically copy the generated SKILL.md to the platform's skill directory so the user can invoke `/{slug}` immediately without manual setup:

```bash
# OpenClaw
mkdir -p ~/.openclaw/workspace/skills/teammate-{slug}
cp teammates/{slug}/SKILL.md ~/.openclaw/workspace/skills/teammate-{slug}/SKILL.md

# Claude Code (global)
mkdir -p ~/.claude/skills/teammate-{slug}
cp teammates/{slug}/SKILL.md ~/.claude/skills/teammate-{slug}/SKILL.md
```

Detect platform and run the appropriate command. If auto-install fails, show manual instructions instead.

**6. Confirm to user with a live test:**
```
✅ alex-chen created!

📁 Location: teammates/alex-chen/
🗣️ Commands: /alex-chen (full) | /alex-chen-work | /alex-chen-persona

Let me give you a quick demo — ask alex-chen anything:
```

Then immediately switch into the generated skill's persona and respond to whatever the user says next as the teammate. This makes the skill feel real from second one — no "go try it yourself" dead end.

If the user doesn't ask anything, prompt with a sample:
```
Try it: "Alex, should we use MongoDB for this new service?"
```

---

## Evolution Mode: Append Files

When user provides new materials:

1. Parse new content using Step 2 methods
2. Read existing `teammates/{slug}/work.md` and `persona.md`
3. Read `{baseDir}/prompts/merger.md` for incremental analysis rules
4. Backup current version:
   ```bash
   python3 {baseDir}/tools/version_manager.py --action backup --slug {slug} --base-dir ./teammates
   ```
5. Edit files with incremental updates
6. Regenerate `teammates/{slug}/SKILL.md`
7. Update `meta.json` version and timestamp

---

## Evolution Mode: Conversation Correction

When user says "that's wrong" / "they wouldn't do that":

1. Read `{baseDir}/prompts/correction_handler.md`
2. Determine if correction applies to Work or Persona
3. Generate correction record
4. Append to `## Correction Log` section
5. Regenerate `teammates/{slug}/SKILL.md`

---

## Management Commands

| Command | Action |
|---------|--------|
| `/list-teammates` | `python3 {baseDir}/tools/skill_writer.py --action list --base-dir ./teammates` |
| `/teammate-rollback {slug} {ver}` | `python3 {baseDir}/tools/version_manager.py --action rollback --slug {slug} --version {ver} --base-dir ./teammates` |
| `/delete-teammate {slug}` | Confirm, then `rm -rf teammates/{slug}` |
