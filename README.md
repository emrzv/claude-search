# claude-search

Search your local [Claude Code](https://claude.com/claude-code) sessions across **all projects** at once, with results grouped by chat. Handy when you remember a conversation but not which project you had it in.

```
$ claude-search flaky build --since 2w
JpsProjectImportPerformanceTests flaky failures since September  65 hits · 2026-10-28 14:20
  ~/Projects/myapp (master)
       you Why did this test start sporadically failing in September? …
    claude Fetching the build logs was denied, so I'm getting the failure reason from …
           … 63 more
  ↳ cd ~/Projects/myapp && claude --resume c9eb91b7-…
```

Each result shows the chat title, project directory, git branch, last activity time, highlighted snippets, and the command to resume it.

## Install

Requires Python 3.9+. [fzf](https://github.com/junegunn/fzf) is optional (for `-i`).

```sh
curl -o ~/.local/bin/claude-search https://raw.githubusercontent.com/emrzv/claude-search/main/claude-search
chmod +x ~/.local/bin/claude-search
```

## Usage

```sh
claude-search foo bar              # chats containing both "foo" and "bar" (anywhere in the chat)
claude-search "foo bar"            # exact phrase
claude-search -e 'PROJ-\d+'        # regular expression
claude-search -i docker            # pick a chat with fzf and resume it in its project directory
claude-search -p myapp --since 7d  # only projects whose path contains "myapp", active in the last 7 days
claude-search -n 0 gradle          # titles only, no snippets
claude-search --json gradle        # machine-readable output
```

By default only your prompts, Claude's replies and chat titles are searched; matching is case-insensitive.

| Option | |
|---|---|
| `-t`, `--tools` | also search tool calls and tool output |
| `-a`, `--all` | search everything: tools, thinking, slash-command output |
| `--subagents` | also search subagent transcripts (hits count toward the parent chat) |
| `-s` | case-sensitive |
| `--since` | `12h`, `7d`, `2w`, `3m` or a date like `2026-10-01` |
| `--sort hits` | most matches first (default: most recent) |
| `-l N` | show at most N chats |
| `-w N` | characters of context around each match |

See `claude-search -h` for the full list.

## How it works

Claude Code stores each session as a JSONL file under `~/.claude/projects/<project>/<session-id>.jsonl` (or `$CLAUDE_CONFIG_DIR/projects`). `claude-search` scans these files, prefiltering raw lines before parsing JSON, so a few hundred MB of history takes about a second. Titles come from `/rename`, Claude's auto-generated title, or the first prompt.

Sessions can only be resumed from their original project directory, which is why the resume hint includes a `cd`.
