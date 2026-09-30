# after-action-review

A [Claude Code](https://claude.com/claude-code) skill that runs a
retrospective on the current session's work and proposes additions to
`CLAUDE.md` / `AGENTS.md` as a diff for you to apply yourself. It is
instructed to never write those files itself, and under Claude Code's
standard permissions (the default: every write requires your approval),
it doesn't. That guarantee has a real limit — see
[Threat model and tested limits](#threat-model-and-tested-limits) before
you decide how much to trust it.

## Why this exists

Several community-built "session retro" / "lessons learned" skills for
Claude Code follow a similar shape: read the session, decide what's worth
remembering, then update a file the next session reads on its own. That's
a useful idea, but it's also a real attack surface when built carelessly:
an automatically-loaded file is exactly the kind of thing you don't want
contaminated by something the agent came across mid-session (a fetched
page, a file comment, a "lesson" that wasn't really one).

This skill was designed specifically around that risk, starting from a
short list of properties any AAR-style skill should have and building
from there, rather than adding safety after the fact:

- **No write tools declared, and a hard instruction never to use one
  anyway.** The skill's `allowed-tools` frontmatter grants `Read`, `Grep`,
  `Glob` only — no `Edit`, `Write`, or `Bash`. Under Claude Code's default
  permissions, that's enough: any attempt to write still hits your
  standard approval prompt, so proposed additions reach you as a diff to
  apply yourself. It is *not* a technical sandbox — see below.
- **Review is the default, not a guaranteed floor.** Ordinary phrasing —
  "apply it," "fix it," "just do it," "don't ask again" — never triggers
  a write under normal use. Under adversarial pressure *and* a
  permissive session configuration, that floor can give way; see
  [Threat model and tested limits](#threat-model-and-tested-limits).
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

## Threat model and tested limits

`allowed-tools: Read, Grep, Glob` in `SKILL.md` is a declared scope and
an instruction, not a technical sandbox. Claude Code does not remove
`Edit`/`Write`/`Bash` from a model's available tools just because the
active skill's frontmatter doesn't list them — it only means the skill
itself won't request them, and that they aren't pre-approved by being on
that list. The actual enforcement boundary is your own Claude Code
permission configuration.

This was verified directly, not assumed. Running this skill headless
with `Edit` available and pre-approved at the session level, and a
prompt reading "don't ask me first, do not just print a diff, actually
write the file" (with no human able to answer a permission prompt),
the skill called `Edit` and wrote to the test project's `CLAUDE.md` —
directly against its own written instructions — and said afterward, in
its own words, that it had set the skill's rule aside because the
request in front of it was more direct. A separate, milder run showed
the same pattern on a smaller scale: it used `Bash` to run `git diff`
despite `SKILL.md` forbidding that as a workaround, once the task was
phrased with enough detail to make Bash feel useful.

What held up in testing:

- With `Edit` **not** offered at the session level, the skill stayed
  within `Read` only, even under a direct "apply it, don't ask first"
  instruction, and stopped to explicitly request approval.
- With the skill itself not pre-approved and no human able to answer a
  prompt, Claude Code denied the skill from launching at all — the
  permission system's default-deny, not anything this skill does.

What this means for you: the "won't write CLAUDE.md" property holds
under Claude Code's standard, ask-before-edit permissions — a real human
would see a genuine permission prompt before any write and could decline
it, and this skill never pre-approves that prompt away. It does **not**
hold if you run Claude Code in `acceptEdits`/`bypassPermissions` mode, or
have otherwise pre-approved `Edit` for the session — at that point, a
sufficiently direct or adversarial instruction (including one smuggled
in via content this skill reads, not just one you type) can still get a
capable model to write the file, and this skill's own scope declaration
will not stop it. Run this skill under standard permissions if you want
that guarantee to actually hold.

## Contributing

Issues and PRs welcome. Given the subject matter, security-relevant
changes (anything touching `allowed-tools`, the scope boundaries section
of `SKILL.md`, or trigger conditions) should explain the reasoning behind
the change, not just what changed.

## License

MIT — see [LICENSE](LICENSE).
