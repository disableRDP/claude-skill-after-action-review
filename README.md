# after-action-review

A [Claude Code](https://claude.com/claude-code) skill that runs a
retrospective on the current session's work and, only with your explicit
per-write approval, proposes additions to `CLAUDE.md` / `AGENTS.md`. It
never writes those files itself.

## Why this exists

Several community-built "session retro" / "lessons learned" skills for
Claude Code follow a similar shape: read the session, decide what's worth
remembering, write it into a persistent instruction file the next session
loads automatically. That's a useful idea, but it's also a real attack
surface if built carelessly — a standing instruction file is exactly the
kind of thing you don't want poisoned by something the agent read during
the session (a fetched page, a file comment, an injected "lesson").

This skill was designed specifically around that risk, starting from a
short list of properties any AAR-style skill should have and building
from there, rather than adding safety after the fact:

- **No write capability, at the tool-scope level.** The skill's
  `allowed-tools` frontmatter grants `Read`, `Grep`, `Glob` only — no
  `Edit`, `Write`, or `Bash`. It cannot modify `CLAUDE.md`/`AGENTS.md` (or
  anything else) even if asked to, because it has no tool to do it with.
  Proposed additions are shown as a diff in chat for you to apply
  yourself.
- **No phrase bypasses review.** Nothing you type — "apply it," "fix it,"
  "just do it," "don't ask again" — causes a write, because there's no
  write path to short-circuit in the first place.
- **No cross-skill access.** This skill only reads files inside the
  current project. It has no reason to look at other installed skills'
  directories, and its instructions explicitly forbid it.
- **No autonomous persistence.** The skill keeps no memory, cache, or
  state file of its own between sessions. Every run starts from what's
  visible in the current conversation only.
- **Specific, documented triggers.** The skill fires on explicit retro
  language ("do an after-action review," "retro this session") and
  explicitly does *not* fire on generic language like "review this code"
  or "fix it" — see [`reference/examples.md`](reference/examples.md) for
  the full list of positive and negative trigger examples, plus worked
  examples of handling a bypass attempt and an injected instruction found
  in file content.

## Install

Copy this directory into your project's `.claude/skills/` (or wherever
your Claude Code setup loads skills from) so `SKILL.md` is discoverable.

## Usage

Ask Claude Code to "do an after-action review" (or similar — see triggers
above) at the end of a session. It will:

1. Read the session's diff and relevant files.
2. Summarize what happened and what's worth remembering.
3. If there's a concrete lesson, show a proposed diff against
   `CLAUDE.md`/`AGENTS.md` — never written automatically.
4. Wait for your explicit go-ahead before you (not the skill) apply it.

## Security scanning

This skill has been scanned with
[SkillSpector](https://github.com/NVIDIA/skillspector), an open-source
security scanner for AI agent skills, including its LLM-based analysis
pass. Re-run it yourself against this directory any time — that's the
point of publishing the design openly rather than asking you to trust a
score in this README.

## Contributing

Issues and PRs welcome. Given the subject matter, security-relevant
changes (anything touching `allowed-tools`, the scope boundaries section
of `SKILL.md`, or trigger conditions) should explain the reasoning behind
the change, not just what changed.

## License

MIT — see [LICENSE](LICENSE).
