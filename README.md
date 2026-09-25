# canvasreview

A pull request as a canvas.

Open a PR and you get one frame per changed function, wired in the order the logic runs,
with a one-line summary on each and a short list of questions to answer before you
approve. You read it with the arrow keys, frame by frame, instead of scrolling a diff
sorted alphabetically by file.

It runs on your Mac. The code never leaves it.

## Install

Apple Silicon only for now.

1. Download `CanvasReview.dmg` from the [latest release](https://github.com/kadetXx/canvasreview-releases/releases/latest).
2. Open it and drag the app to Applications.
3. Open the app. It lives in the menu bar.

It needs two tools you may already have, and it tells you if either is missing:

```sh
brew install gh && gh auth login
npm install -g @anthropic-ai/claude-code && claude
```

`gh` fetches the PR and posts your comments as you. Claude Code writes the summaries and
the questions, on your own plan. There are no keys to paste and no account to make.

## Use

Click the menu bar icon, paste a PR link or `owner/repo#123`, press Return. About a
minute later the canvas opens in your browser. The panel lists what is running; each
canvas has an Open, a copy-link, and a stop.

### Reading a canvas

- It opens on the whole PR. Press `→` to start reading in order, `←` to go back.
- `⏎` opens the code on the frame you are on. `s` switches between Story and Code.
  `esc` goes back to the overview.
- If you pan around and lose your place, `shift+→` resumes from whatever is under the
  middle of the screen.
- Each frame has a rail of questions. Click one to mark it checked, then issue, then
  n/a. Nothing here is a finding; they are things to look at.
- Hover a line of code for a `+` to comment on it. Press `c` and click anywhere for a
  free comment. Both post to the PR on GitHub, and replies are ordinary GitHub threads.
- Approve shows how many questions you never opened, then posts your review to GitHub
  with that number in it. Request changes and comment-only are in the same menu.

## How it works

1. The PR is fetched with `gh`, without checking out the repo.
2. Every changed file is parsed and split into frames: one per top-level declaration,
   the function or component as it stands after the change, with the diff marked inside.
3. One Claude call reads all the frames and returns the reading order, the arrows between
   frames, a summary per frame, and, when a PR does several things, which frames belong
   to which.
4. Claude then grades the frames in small batches against a fixed list of concerns
   (a query inside a loop, a public contract that changed, a URL that went away, and so
   on) and asks a PR-specific question where one fits.
5. The canvas is served from a local port and opened in your browser.

Comments and reviews go to GitHub through `gh`, as you. The app stores nothing anywhere.
There is no server of ours in the loop.

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
