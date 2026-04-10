# Rubber Duck

```
>o)
(_>
```

Cross-model second opinion companion for AI coding agents.

Inspired by [GitHub Copilot's Rubber Duck](https://github.blog/ai-and-ml/github-copilot/github-copilot-cli-combines-model-families-for-a-second-opinion/)
— the same idea, built for [Claude Code](https://claude.ai/claude-code)
using [Codex CLI](https://github.com/openai/codex) as the cross-model reviewer.

Like the classic debugging practice — you explain what you did, the duck
listens, and sometimes it spots what you missed.

## What it does

- Reviews the **decisions** your AI agent made — not code style
- Gives a **plain-language** take, like a colleague's quick read
- Ends with an **actionable bottom line** (ship it / fix X / rethink Y)
- Activates **automatically** at high-signal checkpoints or **on demand**

## Example

```
>o)
(_>
```

Took a look at what you just did.

- The retry logic in `fetcher.ts:82` doesn't back off — it'll hammer
  the endpoint on transient failures. This project uses `p-retry`
  elsewhere, probably worth using it here too.

- `user.preferences` can be null but you're accessing `.theme` directly
  on line 112. Added optional chaining.

**Ship it** after switching to `p-retry`.

## Install

### Prerequisites

[Codex CLI](https://github.com/openai/codex) must be installed and authenticated:

```bash
npm install -g @openai/codex
codex login
```

### Via APM (recommended — automatic + on demand)

```bash
apm install chkp-roniz/cc-rubber-duck
apm compile --target claude --local-only
```

APM compiles the trigger criteria into your project's `CLAUDE.md`, so
Claude sees them at session start and invokes the duck automatically at
high-signal checkpoints.

### Via Claude Code plugin (on demand only)

```bash
claude /plugin marketplace add chkp-roniz/cc-rubber-duck
```

Claude sees the skill description (which includes trigger hints), but
automatic activation depends on the agent reading the description and
deciding to invoke. Use `/rubber-duck` to invoke explicitly.

### Manual

Copy `skills/`, `commands/`, and `instructions/` into your project's `.claude/`:

```bash
cp -r skills/ .claude/skills/
cp -r commands/ .claude/commands/
cp -r instructions/ .claude/instructions/
```

## Usage

### On demand

Type `/rubber-duck` in Claude Code, or just say "rubber duck this",
"second opinion", "what am I missing", etc.

### Automatic (APM or manual install)

The duck activates on its own when Claude:

- Finishes a plan touching 3+ files
- Writes 100+ lines of meaningful changes
- Completes a test suite
- Gets stuck retrying the same fix
- Is about to do something destructive

## How it works

1. Shows the duck logo
2. Gathers context (git diff, recent files, or conversation context)
3. Pipes it to `codex exec` with a decision-review prompt
4. Evaluates concerns against full codebase context
5. Presents a short, honest summary with an actionable bottom line

The duck runs in `--sandbox read-only` mode — it can't modify your code.

## Project structure

```
rubber-duck/
├── plugin.json                    # Plugin manifest
├── instructions/
│   └── rubber-duck.instructions.md  # Trigger criteria (compiled into CLAUDE.md by APM)
├── skills/
│   └── rubber-duck/
│       └── SKILL.md               # Full skill definition (behavior + presentation)
├── commands/
│   └── rubber-duck.md             # /rubber-duck slash command
└── README.md
```

## License

MIT
