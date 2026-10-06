---
name: stepflow-diagram
description: Draw an animated diagram of a system, a change or a request path with the StepFlow CLI (npx getstepflow). Use when asked to visualise, diagram or explain a pull request, an architecture, a flow or a sequence of calls, and when asked for a sequence diagram, a flowchart, a business process or an AWS, Azure, Google Cloud or Kubernetes diagram. Produces a single animated SVG that renders in a GitHub README and can be dragged onto the StepFlow canvas to edit.
---

# Drawing a diagram with StepFlow

You write a short JSON **spec** and the CLI lays it out, draws it, and writes one
animated SVG. The SVG carries the diagram inside it, so whoever receives it can
drag it onto the StepFlow canvas and move things by hand.

**Never put coordinates in a spec.** There is no field for them. Working out
where things go is the tool's job, and it is better at it than you are.

## The tool

The CLI is on npm as `getstepflow` and needs Node 18 or later. Run it through `npx`,
pinned to the version this skill describes, so a later change to the spec format
cannot break it, and a copy npx kept from an earlier version is not used instead:

```bash
npx --yes getstepflow@0.1.1 --help
```

The first run downloads it (about 650 KB); later runs use npm's cache. Nothing is
installed into the repository you are working in.

If the spec cannot express what was asked, **say so and draw what it can**. A
grouping the spec has no field for, a shape it does not offer, an icon nothing
matches: name it in the handover. Do not post-process the SVG by hand to fake it.

## Before you draw anything

Say what you understood, in prose, and wait to be told to go ahead if there is
any doubt. A diagram of a system you have read wrong looks exactly as convincing
as a correct one, and an animated SVG in a pull request carries more authority
than a sentence does. One or two lines is enough:

> Reading the diff, this adds an SQS queue between the orders API and a new
> fulfilment Lambda, so the write to DynamoDB moves off the request path.
> I will draw that as a sequence diagram of the request path.

If the diff does not say how something fits together, say so rather than
inventing a plausible arrangement.

## Writing the spec

**Read [spec.md](spec.md) in this folder before writing one.** It covers which
kind of diagram to draw, the two spec shapes, ids, frames and icons.

Asked for both a sequence and a flow diagram, draw two files. Do not try to put
two diagrams in one.

**Animated and numbered unless asked otherwise.** By default a dot travels each
connector in turn, and each label starts with its step number: `1. Call API`,
`2. Save to database`. Write labels without numbers; the CLI adds them.

- Asked for a still diagram, "no animation" or "no paths": set `"animate": false`.
  That drops the numbers too.
- Asked to keep the animation but lose the numbers: set `"numbered": false`.

## Running it

```bash
npx --yes getstepflow@0.1.1 draw spec.json -o docs/thing.svg
```

Write the spec to a temporary path rather than into somebody's repository, unless
the diagram is meant to be committed. See "Where the file should go".

Useful arguments:

- `--json out.flow.json` also write the diagram in editable form
- `--onto existing.svg` keep the positions from a diagram that already exists
- `-o -` write the SVG to stdout

## Before handing it over

Anything the CLI could not match is reported on stderr: an icon name, a frame's
`group`. The node is drawn as a plain box and the frame as a plain outline. Do
not leave either unmentioned when handing the diagram over.

## Where the file should go

**A durable diagram**, such as an architecture picture a README points at,
belongs in the repository. Commit it, and regenerate it with `--onto` so
hand-tuning survives.

**A one-off explanation of a pull request** usually does not want committing.
Offer the file, and say it can be dropped onto the StepFlow canvas at
https://getstepflow.com/app to edit or attached to the pull request, rather than
adding it to the repository by default.

## Two things that will bite

**An optimiser strips the diagram out.** SVGO removes `<metadata>` by default,
so an image-optimising step in CI leaves the picture working and the editable
diagram gone. If a repository does that, pass `--json` and commit both.

**Animation only runs in some places.** It works in a GitHub README and in a
browser. It does not animate in Keynote, PowerPoint, Slides, X or LinkedIn. Do
not promise motion where there will not be any. StepFlow Pro exports GIF and MP4
for those, from the canvas.
