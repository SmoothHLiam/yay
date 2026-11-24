# yay

A small terminal assistant that runs entirely on your Mac. It chats, reads and writes files, runs shell commands, reads images and uses MCP servers, with every bit of thinking done on-device through [LM Studio](https://lmstudio.ai). Nothing is sent anywhere.

It is one Python file with no dependencies beyond the standard library.

```
╭──────────────────────────────────────────────────────────────╮
│ ◆ yay                                                        │
│   local · private · Frontier                                 │
│   ~/code/my-project                                          │
╰──────────────────────────────────────────────────────────────╯
  /model · /think · ctrl-t show thinking · /auto · /mem · /mcp · /new · ctrl-d
```

## What you need

- An Apple Silicon Mac. It was built and measured on an M5 Pro with 24 GB of memory; the three largest brains need about that much.
- LM Studio, with its `lms` command available at `~/.lmstudio/bin/lms`.
- Python 3.9 or newer and `curl` (both come with macOS).

## Install

```bash
git clone https://github.com/SmoothHLiam/yay.git
```

```bash
chmod +x yay/yay.py && mkdir -p ~/.local/bin && ln -s "$PWD/yay/yay.py" ~/.local/bin/yay
```

Make sure `~/.local/bin` is on your `PATH`, then:

```
yay                 chat
yay "question"      one answer, then exit
yay --selftest      check the plumbing without a model
```

## Models download when you ask for them

The repository contains no model files. The first time you pick a brain that isn't on your Mac, yay tells you its size and where it comes from and asks before downloading. Files are fetched from Hugging Face in parallel pieces, checked against a SHA-256 checksum, and placed in LM Studio's own models folder (`~/.lmstudio/models`). An interrupted download resumes when you try again.

`/model` shows the list and marks what you don't have yet:

| # | Brain | Model | Download | Good for |
|---|---|---|---|---|
| 1 | Spark | Qwen3.5 9B | about 6 GB | Quick questions and small edits. Reads images |
| 2 | Sage | Huihui Qwen3.8 27B (abliterated) | 14.3 GB | Careful work on hard problems. Reads images |
| 3 | Somnium | Google Gemma 4 12B | 7.2 GB | Teaching and explaining. Reads images |
| 4 | Cyber | Attestor 4.4 3B (abliterated) | 3.3 GB | Security questions on pasted code and logs. Chat only, no tools |
| 5 | Cyber+ | RavenX CyberAgent 35B (abliterated) | 15.4 GB | Hands-on security work with tools |
| 6 | Frontier | Huihui Qwen3.6 35B-A3B (abliterated) | 15.4 GB | The default all-rounder |

A fresh start tries to load Frontier. If you have less than 24 GB of memory, say no to that download and pick Spark, Somnium or Cyber with `/model` instead.

Spark comes from LM Studio's own catalog (`lms get`); the others come from the Hugging Face repositories listed in the `SOURCES` table near the top of `yay.py`.

These are third-party models with their own licences and behaviour. Several are abliterated, meaning their built-in refusals were removed, and the two security brains are meant for systems you own or are authorised to test.

## Commands

| Command | What it does |
|---|---|
| `/model` | Pick the brain. `/model 3` and `/model sage` also work. |
| `/think` | Turn deep thinking on or off. Off is much faster. |
| ctrl-t or `/thoughts` | Show or hide the model's thinking. |
| `/auto` | Auto mode: safe actions run without asking, risky ones still ask. |
| `/mem <text>` | Save a fact to permanent memory. `/mem` alone prints it. |
| `/mcp` | List the tools that came from MCP servers. |
| `/new` | Start a fresh conversation. |
| ctrl-c | Stop the current answer. |
| ctrl-d | Quit. |

## How it behaves

- **It asks before acting.** Every file write and shell command is shown in full and needs a `y`. In auto mode a guard (the loaded model, plus a fixed list of commands that always ask, such as `rm`, `sudo`, `curl` and `git push`) lets safe actions through. The guard is a model and can be wrong.
- **Roles.** `roles.md` tells each brain what its job is. Edit it freely; it is reread on every message.
- **Memory.** yay keeps what it learns about you in `me.md`, next to `yay.py`. The file is created the first time something is saved and is listed in `.gitignore` so it is never committed.
- **Images.** Put an image's path in your message (dragging the file into the terminal does this) and a brain that reads images gets the picture.
- **MCP.** Servers listed in `~/.lmstudio/mcp.json` (the file LM Studio uses) are started and their tools offered to the model. Only local `command` servers are supported. Tool lists use context on every message, so add servers one at a time.
- **Long chats.** At 85% of the context window the conversation is summarised and carries on.
- **Thinking loops.** If a model's thinking repeats itself or runs away, yay cuts it off and answers without it.

## Adding or changing a brain

Add a line to `MODELS` in `yay.py` (name, LM Studio key from `lms ls`, context size, description), a matching entry in `SOURCES` so it can be downloaded, and a `## Name` section in `roles.md`. Add the name to `VISION` if it reads images, or to `CHAT_ONLY` if it cannot make tool calls. Then run `yay --selftest`.

## Limits

- Tested on one machine (M5 Pro, 24 GB). The speeds in `/model` were measured there.
- Downloading Spark through LM Studio's catalog has not been tested from a clean machine; the Hugging Face downloads have.
- yay expects LM Studio to name a downloaded model after its folder. If a brain downloads but then isn't found, compare `lms ls` with the key in `MODELS`.
- The 35B brains leave little memory for anything else on a 24 GB Mac.

## Licence

MIT, see [LICENSE](LICENSE). The models yay downloads are not part of this repository and carry their own licences.
