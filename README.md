# Deepgram Voice Agent Workshop — AWS Workshop Studio content

The AWS Workshop Studio edition of the Deepgram Voice Agent Workshop. This repository holds **content only**.

**The code lives in [deepgram-devs/deepgram-workshop-py](https://github.com/deepgram-devs/deepgram-workshop-py).** Attendees clone that repository and work the `TODO` blocks in `steps/NN-slug/main.py`; the pages here are the instructions. Every code link in `content/` points at that repository, so a change to a step's file layout there needs a matching change here.

## What differs from the source repository

| | Source repo | This edition |
|---|---|---|
| Step 6b (Amazon Bedrock) | Optional detour, skipped in the run of show | Merged into Step 6 as Part 2, required |
| Navigation | Flat `steps/` folders | Three modules, plus an Easy Mode track |
| Check-yourself questions | Question only; answers in `FACILITATOR.md` | Question with the answer in an expandable block |
| Facilitator guide | `FACILITATOR.md` | `FACILITATOR_GUIDE.md`, in Workshop Studio's section format |

Everything else is the source prose, converted to Workshop Studio directives.

## Repo structure

```
.
├── contentspec.yaml       Workshop Studio configuration and AWS account requirements
├── FACILITATOR_GUIDE.md   Parsed at build time and rendered in Workshop Studio
├── static/                Assets referenced from content as /static/...
└── content/
    ├── index.en.md                  Landing page
    ├── introduction/                What you'll build, prerequisites, voice agent concepts
    ├── build-the-agent/             Module 1 — Steps 1 to 5
    ├── make-it-yours/               Module 2 — Steps 6 and 7
    ├── optimize/                    Module 3 — Step 8
    ├── easy-mode/                   Parallel no-install track on the Deepgram Playground
    ├── summary/                     The finished agent and where to go next
    └── cleanup/                     Teardown
```

Every folder needs at least one `index.en.md`. The `title` in each page's front matter becomes its left-nav label, and `weight` sets the order — chapters are multiples of 10, pages inside them count up from there.

## Local preview

Download the preview binary for your platform, make it executable, and point it at this folder:

```bash
curl -o preview_build https://artifacts.us-east-1.prod.workshops.aws/v2/cli/osx/preview_build
chmod +x preview_build
./preview_build -input /path/to/deepgram-workshop
```

The preview serves at <http://localhost:8080> and reloads when files change. Watch the terminal — build failures and directive warnings only appear there. Use `-port 8081` if 8080 is taken.

On macOS the binary is unsigned: Control-click it in Finder and choose **Open** once before running it from the terminal.

## Authoring notes

- **Directives, not fenced code blocks.** Code uses `:::code{language=... showCopyAction=true}`, callouts use `::alert`, collapsibles use `:::expand`, and tabs use `::::tabs` wrapping `:::tab`.
- **Nested containers need more colons than their children.** A `::::tabs` holding `:::tab` holding `::code` is the pattern. Get the count wrong and the page renders literal colons.
- **A line beginning with `:` is parsed as a directive.** Reflow or escape prose and terminal output that starts that way.
- **Images must live in `static/`.** Workshop Studio blocks images loaded from outside the workshop.
- **No mermaid renderer.** Diagrams are either SVG in `static/` or plain text inside a code block.
