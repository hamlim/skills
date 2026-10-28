# Skills

A Git repository for skills shared by Codex and Claude Code. Each skill lives in `skills/<name>/` and has a `SKILL.md` file. Add supporting files inside that directory when needed.

## First computer

1. Create a private remote repository if these skills contain private information.
2. Add the remote and push this repository:

   ```sh
   git remote add origin <remote-url>
   git add .
   git commit -m 'Set up shared skills'
   git push -u origin main
   ```

## Another computer

```sh
git clone <remote-url> ~/dev/skills
cd ~/dev/skills
./scripts/sync
```

`sync` pulls the current branch with `git pull --ff-only`, then links each skill into `~/.agents/skills`, `~/.codex/skills`, and `~/.claude/skills`. It never pushes. It stops before pulling if the working tree has changes. If there is no upstream, it links local skills without pulling. Run `./scripts/sync --link-only` to link without pulling, including when you have local changes.

Existing files, directories, and links with the same name are left untouched and reported as conflicts. Move or remove a conflicting item yourself before running sync again. Links to deleted or renamed skills are not removed automatically; remove those links yourself. A moved clone also requires removing its old links before syncing again.

To use different locations, set `AGENTS_SKILLS_DIR`, `CODEX_SKILLS_DIR`, or `CLAUDE_SKILLS_DIR` when running sync. `CODEX_HOME` changes the default Codex location. The script requires Bash and Git.

## Create a skill

```sh
./scripts/new-skill example-skill
```

Edit `skills/example-skill/SKILL.md`, then run `./scripts/sync --link-only`. Commit and push the changes. On another computer, run `./scripts/sync` to pull and link them. Changes to a linked skill are visible immediately on that computer. Restart the agent or start a new session if it does not discover the skill.
