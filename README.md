# canvasreview

<video src="https://github.com/user-attachments/assets/6b09bbdb-22a9-488b-93ba-e9ae5ce7e521" poster="https://raw.githubusercontent.com/kadetXx/canvasreview-releases/main/docs/poster.jpg" autoplay muted loop playsinline controls width="100%"></video>

*Trial one. Thirty seconds of a PR being read on the canvas; the questions get sharper from here.*

A pull request as a canvas.

**[Download for Mac](https://github.com/kadetXx/canvasreview-releases/releases/latest/download/CanvasReview.dmg)** · Apple Silicon, notarised. Free.

Open a PR and you get one frame per changed function, wired in the order the logic runs,
with a one-line summary on each and a short list of questions to answer before you
approve. You read it with the arrow keys, frame by frame, instead of scrolling a diff
sorted alphabetically by file.

It runs on your Mac, through the GitHub CLI (`gh`) and the Claude Code you already use. There is
no server of ours, no account, and nothing is stored anywhere but GitHub.

Which cuts both ways: every comment already on the PR, from a teammate on github.com or
from a bot, shows up on your canvas as a bubble on its line, and you answer it right
there. Your replies go back to the PR. Nobody has to switch tools for you to use this.

## Install

Apple Silicon only for now.

1. [Download `CanvasReview.dmg`](https://github.com/kadetXx/canvasreview-releases/releases/latest/download/CanvasReview.dmg).
2. Open it and drag the app to Applications.
3. Open the app. It lives in the menu bar.

It needs two tools you may already have, and it tells you if either is missing:

```sh
brew install gh && gh auth login
curl -fsSL https://claude.ai/install.sh | bash && claude
```

The GitHub CLI (`gh`) fetches the PR and posts your comments as you. Claude Code writes the summaries and
the questions, on your own plan. There are no keys to paste and no account to make.

## Use

Click the menu bar icon, paste a PR link or `owner/repo#123`, press Return. About a
minute later the canvas opens in your browser. The panel lists what is running; each
canvas has an Open, a copy-link, and a stop. Quitting the app stops them all.

### Reading a canvas

- It opens on the whole PR. Press `→` to start reading in order, `←` to go back.
- `⏎` opens the code on the frame you are on. `s` switches between Story and Code.
  `esc` goes back to the overview.
- If you pan around and lose your place, `shift+→` resumes from whatever is under the
  middle of the screen.
- Each frame has a rail of questions. Click one to mark it checked, then issue, then
  n/a. Nothing here is a finding; they are things to look at.
- Comments already on the PR, whoever left them and wherever they left them, appear as
  bubbles on their lines. Click one to read the thread and reply. Unread ones are marked.
- Hover a line of code for a `+` to comment on it. Press `c` and click anywhere for a
  free comment. Both post to the PR on GitHub, and replies are ordinary GitHub threads.
- Approve shows how many questions you never opened, then posts your review to GitHub
  with that number in it. Request changes and comment-only are in the same menu.

## How it works

1. The PR is fetched with the GitHub CLI, without checking out the repo.
2. Every changed file is parsed and split into frames: one per top-level declaration,
   the function or component as it stands after the change, with the diff marked inside.
3. One Claude call reads all the frames and returns the reading order, the arrows between
   frames, a summary per frame, and, when a PR does several things, which frames belong
   to which.
4. Claude then grades the frames in small batches against a fixed list of concerns
   (a query inside a loop, a public contract that changed, a URL that went away, and so
   on) and asks a PR-specific question where one fits.
5. The canvas is served from a local port and opened in your browser.

## Who does what

```
 you                          your Mac
 ┌────────────────────┐       ┌──────────────────────────────────────────┐
 │ paste a PR link    │──────▶│ the GitHub CLI fetches the PR            │
 └────────────────────┘       │        │                                 │
                              │        ▼                                 │
                              │ Claude Code writes the summaries         │
                              │ and the questions                        │
                              │        │                                 │
                              │        ▼                                 │
                              │ the canvas opens in your browser         │
                              └────────────────────┬─────────────────────┘
                                                   │
                                                   ▼
                              ┌──────────────────────────────────────────┐
                              │ you read it in order, answer the         │
                              │ questions, comment, approve              │
                              └────────────────────┬─────────────────────┘
                                                   │ as you, through the GitHub CLI
                                                   ▼
 ┌────────────────────┐       ┌──────────────────────────────────────────┐       ┌────────────────────┐
 │ anyone else        │──────▶│ GitHub: the PR                           │◀─────▶│ a teammate         │
 │ comments on        │       │ comments, reviews, threads               │       │ opens the same PR  │
 │ github.com, or a   │       └────────────────────┬─────────────────────┘       │ in their app; your │
 │ bot does           │                            │                             │ comments are on    │
 └────────────────────┘                            ▼                             │ their canvas       │
                              ┌──────────────────────────────────────────┐       └────────────────────┘
                              │ it shows on your canvas, on its line,    │
                              │ and you answer it there                  │
                              └──────────────────────────────────────────┘
```

Everything you write lands on the PR as you, through the GitHub CLI. A teammate with the app runs
the same PR and gets their own canvas with your comments already on it, because the
comments never lived anywhere but GitHub.

## Where your code goes

Two places, both of which it already goes to. GitHub, through the GitHub CLI, to fetch the PR and
to post what you write. And Anthropic, through your Claude Code session, which reads the
frames of the PR the same way it reads a repo when you use Claude Code on it. Your
Claude Code login, plan and data settings apply. Nowhere else: there is no canvasreview
server, comments live on the PR, and what you have checked on the rail lives in your
browser.

## Honest notes

- **Cost.** One PR is one Claude Code call plus a few small ones, on your plan. On a
  34-file PR that was under a dollar and about a minute.
- **Quality.** The summaries and questions come from a model. They are usually right and
  sometimes not. That is why they are questions, not verdicts.
- **Early.** This is weeks old. Apple Silicon only. Sharing a canvas with a teammate is
  not built yet; today they run it on the same PR and see the same comments, because
  the comments live on GitHub.
- **Source.** Private for now, while it finds its shape. The app contains what it runs.

## Feedback

Open an issue here. Screenshots help.

---

## In more detail

### Why

Reviewing hasn't changed since we started working with agents. The diff is still a list
of files in alphabetical order, and you still read every line to work out what the change
does and what could go wrong. The tools that add AI to this mostly add findings: comments
that assert something is wrong, on the same alphabetical diff. Reviewers learn to skip
them.

canvasreview changes the reading surface instead. The change is shown in the order it
runs, from the entry point outward, so you read it the way it executes. Each piece has a
sentence saying what it does after the change. And instead of findings, each piece has
questions: things a careful colleague would look at, that you answer, and that you can
mark as an issue if the answer is bad. When you approve, the review says how many of
those questions you never opened. That number is the point.

### What you are looking at

**Frames.** One per changed declaration: a function, a component, a type, a config
block. The frame shows the whole declaration as it stands after the change, with added
lines marked `+` and removed lines `-`, and long unchanged stretches folded. A frame is
not a file; a file with four changed functions is four frames.

**Order.** Frames are numbered in the order the logic runs, callers before callees,
starting from the entry point of the change, which carries a `start` badge. Arrows
between frames say how they relate: calls, awaits, imports, returns to. The number
badges are coloured so you can find frame 12 on the map at a glance.

**Story and Code.** Story mode shows only the summaries, one line per frame, so the whole
PR fits on a screen. Code mode shows the diff inside each frame. The canvas opens in
Story.

**Threads.** When a PR does more than one thing, the frames are grouped into bands: the
one the title is about on top, then the others, labelled. The top bar says "3 threads,
title covers 1". That is the "please split this PR" comment, as a count.

**The rail.** To the right of each frame in Code mode: up to four questions, each
pointing at a line. They come from a fixed list of concerns plus questions written for
this PR specifically. Hover one to see the line it means. Frames with nothing to ask
say so.

**Comments.** Existing GitHub review comments show up as bubbles on their lines. Bubbles
you have not opened yet are highlighted; the button under the zoom controls hides them
all.

### Keys

| Key | Does |
| --- | --- |
| `→` `j` | next frame in reading order |
| `←` `k` | previous frame |
| `shift+→` | resume from the frame under the middle of the screen |
| `⏎` `s` | switch Story and Code, on the frame you are looking at |
| `esc` | overview of the whole PR |
| `c` | place a free comment with the next click |
| `h` | show or hide comments |
| double-click | open that frame |

Drag a frame by its header. Drag anywhere else to pan. Scroll to zoom. Click the minimap
to jump.

### Troubleshooting

**"Needs the GitHub CLI, signed in."** Install `gh` and run `gh auth login`, then Check
again. If your org uses SSO, `gh auth login` walks you through it.

**"Claude Code, signed in."** Install it with the command in the panel (the native
installer; `brew install --cask claude-code` also works), then run `claude` once to sign in.

**"No pull request #123 on owner/repo, or no access to it."** Check the number, and that
`gh auth status` shows an account that can see that repo. Some orgs require an admin to
approve third-party access for the GitHub CLI.

**A very large PR.** Layout has been tested to about 35 frames. Beyond that the overview
gets dense; the arrow keys still work.

**macOS asks about folders or the network.** It should not; the app runs everything from
your home folder. If it does, say so in an issue with what it asked for.

### Questions people ask

**Why not just ask Claude to review the PR?** You can, and you get a page of findings.
This is not that. The summaries put the change in the order it runs so you can read it;
the questions are a checklist you answer, not claims you argue with; and the count of
what you never opened is something a review has never had before.

**Why Claude Code and not an API key?** You already have it, it is already allowed at
your company, and its cost is already on your plan. Nothing to set up.

**Private repos?** Yes, anything your GitHub CLI login can see.

**Intel Macs?** Not yet. The bundled runtime is Apple Silicon.

### What is not built yet

- Sharing a canvas by link, and a read-only view of one in the browser.
- Intel builds.
- Languages beyond TypeScript, JavaScript, Go and Python get one frame per file.
- A hosted team layer: review state that persists across people, and a GitHub App that
  puts a canvas on every PR without anyone running anything.
