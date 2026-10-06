# hyperframes-studio

Installs the current **HyperFrames** and **HyperFrames Studio** on a GitHub-hosted
runner and renders a real MP4 with them, so nothing is installed on a local
machine.

## Why this exists

Local was on **0.8.111**. Upstream was on **0.8.136** and publishing several
releases a day — 0.8.127 through 0.8.136 all landed inside about 26 hours, and
most of those carried Studio fixes. A tool moving that fast is one you install
from CI on a version you can point at, not one you upgrade by hand and hope.

## What gets installed

| | |
|---|---|
| [`hyperframes`](https://www.npmjs.com/package/hyperframes) | the CLI — create, preview, render |
| [`@hyperframes/studio`](https://www.npmjs.com/package/@hyperframes/studio) | the browser surface: visual timeline, code editor, live preview |

Upstream is [`heygen-com/hyperframes`](https://github.com/heygen-com/hyperframes),
Apache-2.0. Docs: <https://hyperframes.dev> · Studio: <https://www.hyperframes.dev/studio>

## Run it

Actions → **install and render** → *Run workflow*, and pass a `version` if you want
something other than the current `latest`. It runs on every push, on demand, and
weekly so a new upstream release surfaces as a run rather than as a surprise.

Each run:

1. resolves which version `latest` actually points at, and prints it
2. installs both packages on the runner
3. installs `unzip` **before** `browser ensure` — without it, Chrome downloads and
   then fails to extract, and the error blames Chrome
4. ensures headless Chrome, runs `doctor`
5. scaffolds the bundled `swiss-grid` example and renders 720p with `-w 1`
6. `ffprobe`s the output, because a render exits 0 even when the result is empty
7. uploads the MP4 as an artifact and writes the version into the run summary

## Moving the version

Nothing to edit. Dispatch the workflow with `version: 0.8.136`, or change the
default input to pin it. The packages are published in lockstep, so both are
installed from the same input and a mismatch is impossible.

## Notes that cost time to learn

- **`unzip` is mandatory** before `browser ensure`, or the Chrome extraction fails
  with a misleading error.
- **`-w 1`.** Each render worker launches its own Chrome (~256 MB). The default
  spawns several and dies on memory.
- **Pass `HYPERFRAMES_SKIP_SKILLS=1` to `init`** in CI, or it reaches out to GitHub
  for the agent skills.
- **`init` needs `--non-interactive`**, or it blocks on a prompt.
- **Never trust exit 0.** `ffprobe` the file.
