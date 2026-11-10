# claude-search

Search your local [Claude Code](https://claude.com/claude-code) sessions across **all projects** at once, with results grouped by chat. Handy when you remember a conversation but not which project you had it in.

In a terminal, results open in an [fzf](https://github.com/junegunn/fzf) picker: move with the arrow keys, type to narrow down by title or project (exact substrings; several words must all match), press Enter to resume the chat in its project directory. The preview shows the matching snippets. With `--print` (or when output is piped) you get grouped results instead:

```
$ claude-search --print flaky build --since 2w
JpsProjectImportPerformanceTests flaky failures since September  65 hits · 2026-10-28 14:20
  ~/Projects/myapp (master)
       you Why did this test start sporadically failing in September? …
    claude Fetching the build logs was denied, so I'm getting the failure reason from …
           … 63 more
  ↳ cd ~/Projects/myapp && claude --resume c9eb91b7-…
```

Each result shows the chat title, project directory, git branch, last activity time, model, cost, highlighted snippets, and the command to resume it. The model is the one used most recently (the preview lists all of them if the chat switched models). The cost is the session total Claude Code records locally; it's left blank for sessions without one (older ones or ones still open).

Run it **without search terms** to get a switcher: all your chats across projects, most recent first, with a preview of the first and last prompt.

## Install

Requires Python 3.9+. [fzf](https://github.com/junegunn/fzf) is optional: without it, results are always printed.

```sh
curl -o ~/.local/bin/claude-search https://raw.githubusercontent.com/emrzv/claude-search/main/claude-search
chmod +x ~/.local/bin/claude-search
```

## Usage

```sh
claude-search                      # switcher: pick a recent chat and resume it
claude-search -p myapp --since 7d  # only chats in projects whose path contains "myapp", active in the last 7 days
claude-search foo bar              # chats containing both "foo" and "bar" (anywhere in the chat)
claude-search "foo bar"            # exact phrase
claude-search -e 'PROJ-\d+'        # regular expression
claude-search --print docker       # print grouped results with snippets instead of the picker
claude-search --print -n 0 gradle  # titles only, no snippets
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
| `--print` | print results instead of opening the picker |
| `--no-pager` | don't page output |

See `claude-search -h` for the full list.

## Paging and colors

When writing to a terminal, output goes through a pager (`$CLAUDE_SEARCH_PAGER`, `$PAGER` or `less`; `LESS=FRX` by default, like git), so the most recent chats are on screen first and short output is printed directly.

The default colors work on both light and dark terminals. fzf is started with `--color light` when the terminal reports a light background via `COLORFGBG` (unless `FZF_DEFAULT_OPTS` sets colors). To customize, set any of these keys to an SGR code; `NO_COLOR` disables colors:

```sh
export CLAUDE_SEARCH_COLORS="hl=1;31:title=1:path=32:you=35:claude=2:dim=2"
```

## How it works

Claude Code stores each session as a JSONL file under `~/.claude/projects/<project>/<session-id>.jsonl` (or `$CLAUDE_CONFIG_DIR/projects`). `claude-search` scans these files, prefiltering raw lines before parsing JSON, so a few hundred MB of history takes about a second. Titles come from `/rename`, Claude's auto-generated title, or the first prompt.

Sessions can only be resumed from their original project directory, which is why the resume hint includes a `cd`.
