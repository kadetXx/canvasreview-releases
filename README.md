# canvasreview

A pull request as a canvas.

Open a PR and you get one frame per changed function, wired in the order the logic runs,
with a one-line summary on each and a short list of questions to answer before you
approve. You read it with the arrow keys, frame by frame, instead of scrolling a diff
sorted alphabetically by file.

It runs on your Mac, through the GitHub CLI and the Claude Code you already use. There is
no server of ours, no account, and nothing is stored anywhere but GitHub.

- [Why](#why)
- [Install](#install)
- [Use](#use)
- [What you are looking at](#what-you-are-looking-at)
- [Keys](#keys)
- [How it works](#how-it-works)
- [Where your code goes](#where-your-code-goes)
- [Cost](#cost)
- [Troubleshooting](#troubleshooting)
- [Questions people ask](#questions-people-ask)
- [What is not built yet](#what-is-not-built-yet)

## Why

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

## Install

Apple Silicon only for now.

1. Download `CanvasReview.dmg` from the [latest release](https://github.com/kadetXx/canvasreview-releases/releases/latest).
2. Open it and drag the app to Applications.
3. Open the app. It lives in the menu bar, no Dock icon.

It needs two tools, and the panel tells you if either is missing:

```sh
brew install gh && gh auth login
npm install -g @anthropic-ai/claude-code && claude
```

`gh` fetches the PR and posts your comments and reviews as you, so your company's SSO and
org approvals apply as they already do. Claude Code writes the summaries and the questions
on your own plan. There are no API keys to paste.

## Use

Click the menu bar icon. Paste a PR link, or type `owner/repo#123`, and press Return.
About a minute later the canvas opens in your browser tab. The panel lists what is
running; each canvas has Open, a copy-link button for the local url, and a stop.

Quitting the app stops every canvas it started.

## What you are looking at

**Frames.** One per changed declaration: a function, a component, a type, a config
block. The frame shows the whole declaration as it stands after the change, with added
lines marked `+` and removed lines `-`, and long unchanged stretches folded. A frame is
not a file; a file with four changed functions is four frames.

**Order.** Frames are numbered in the order the logic runs, callers before callees,
starting from the entry point of the change, which carries a `start` badge. Arrows
between frames say how they relate: calls, awaits, imports, returns to. The number
badges are coloured so you can find frame 12 on the map at a glance.

**Story and Code.** Story mode shows only the summaries, one line per frame, so the whole
PR fits on a screen. Code mode shows the diff inside each frame. `s` or `⏎` toggles.
The canvas opens in Story.

**Threads.** When a PR does more than one thing, the frames are grouped into bands: the
one the title is about on top, then the others, labelled. The top bar says "3 threads,
title covers 1". That is the "please split this PR" comment, as a count.

**The rail.** To the right of each frame in Code mode is its rail: up to four questions,
each pointing at a line. They come from a fixed list of concerns (a query inside a loop,
a public contract that changed, a URL that went away, a default that changed, a
subscription that got narrower, and so on) plus questions written for this PR
specifically. Hover one to see the line it means. Click to mark it checked, click again
for issue, again for n/a. Frames with nothing to ask say so.

**Comments.** Hover a line of code and a `+` appears in the gutter; click it to comment
on that line. Press `c` and click anywhere on the canvas for a free comment near a
frame. Both post to the PR on GitHub as you. Existing GitHub review comments show up as
bubbles on their lines. Threads are ordinary GitHub replies. Bubbles you have not opened
yet are highlighted; the show/hide button under the zoom controls hides them all.

**Approve.** The Approve menu offers Approve, Request changes, and Comment only. Each
shows a summary first: how many questions you opened, how many you marked as issues,
with a box for a note, and a confirm. The review posts to GitHub with that summary and
the list of issues in its body.

## Keys

| Key | Does |
| --- | --- |
| `→` `j` | next frame in reading order |
| `←` `k` | previous frame |
| `shift+→` | resume from the frame under the middle of the screen, after panning |
| `⏎` `s` | switch Story and Code, on the frame you are looking at |
| `esc` | overview of the whole PR |
| `c` | place a free comment with the next click |
| `h` | show or hide comments |
| double-click | open that frame |

Drag a frame by its header. Drag anywhere else to pan. Scroll to zoom. Click the minimap
to jump.

## How it works

1. `gh` fetches the PR's metadata and the two commits it spans. A blob-less clone of the
   repo goes in `~/.cache/canvasreview`, so nothing is checked out.
2. Each changed file is parsed with tree-sitter (TypeScript, JavaScript, Go, Python;
   other files become one frame each) and split into top-level declarations. Hunks are
   matched to the declarations they touch. Those are the frames.
3. Symbol references between frames become candidate arrows.
4. One Claude Code call reads all the frames and returns the entry point, the reading
   order, the arrows worth keeping, a summary per frame, and the thread grouping.
5. Claude then grades the frames in parallel batches of eight against the concern list,
   and adds PR-specific questions where one fits. This is separate from the narration
   because one call over a large PR goes quiet: on a 32-frame PR it asked two questions,
   in batches it asked twenty-three.
6. The canvas is served from a local port and opened in your browser. The page talks to
   that local server only; the server runs `gh` for anything that touches GitHub.

## Where your code goes

Two places, both of which it already goes to.

- **GitHub**, through `gh`, to fetch the PR and to post what you write. Same as the
  website.
- **Anthropic**, through your Claude Code session, for the summaries and the questions.
  The frames of the PR are sent as the prompt, the same way they are sent when you use
  Claude Code on that repo. Your Claude Code login, plan and data settings apply.

Nowhere else. There is no canvasreview server. Comments live on the PR. What you have
checked on the rail lives in your browser's local storage, per PR, and is only ever
posted to GitHub as a count inside your review.

## Cost

One PR is one Claude Code call for the narration plus a few small ones for the rail, at
low effort, on your plan. On a 34-frame PR that came to somewhere between a few cents
and half a dollar depending on cache state, and about a minute of waiting. There is no
charge from canvasreview.

## Troubleshooting

**"Needs the GitHub CLI, signed in."** Install `gh` and run `gh auth login`. Then click
Check again. If your org uses SSO, `gh auth login` walks you through it.

**"Claude Code, signed in."** Install it with the command in the panel, then run `claude`
once to sign in.

**"No pull request #123 on owner/repo, or no access to it."** Check the number, and that
`gh auth status` shows an account that can see that repo. Some orgs require an admin to
approve third-party access for the GitHub CLI; the org's settings page says so.

**The canvas built but the questions look off.** They come from a model and will be
wrong sometimes. That is why they are questions. Mark one n/a and move on.

**A very large PR.** Layout has been tested to about 35 frames. Beyond that the overview
gets dense; the arrow keys still work.

**macOS asks about folders or the network.** It should not; the app runs everything from
your home folder. If it does, say so in an issue with what it asked for.

## Questions people ask

**Why not just ask Claude to review the PR?** You can, and you get a page of findings.
This is not that. The summaries put the change in the order it runs so you can read it;
the questions are a checklist you answer, not claims you argue with; and the count of
what you never opened is something a review has never had before.

**Why does it need Claude Code and not an API key?** Because you already have it, it is
already allowed at your company, and its cost is already on your plan. Nothing to set up.

**Can my teammate see the canvas I am looking at?** Not yet as a link. They can run it on
the same PR and get the same comments, because the comments live on GitHub. Sharing is
the next thing.

**Does it work on private repos?** Yes, anything your `gh` can see.

**Intel Macs?** Not yet. The bundled runtime is Apple Silicon.

**Where is the source?** Private for now, while it finds its shape. The app contains what
it runs. Issues and screenshots here are very welcome.

## What is not built yet

- Sharing a canvas by link, and a read-only view of one in the browser.
- Intel builds.
- Languages beyond TypeScript, JavaScript, Go and Python get one frame per file.
- A hosted team layer: review state that persists across people, and a GitHub App that
  puts a canvas on every PR without anyone running anything.

## Feedback

Open an issue here. A screenshot and the PR's size help more than anything.
