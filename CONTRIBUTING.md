# Contributing

Simple rules so four people can work in parallel without stepping on each other.

## Branches and pull requests

- `main` is always the submittable version. Do not commit directly to it.
- Create a branch per piece of work: `adr/model-provider-abstraction`, `diagram/animal-monitoring`, `docs/overview`.
- Open a pull request; one other teammate reviews and merges. Small PRs, merged often.

## Writing ADRs

- Copy `docs/adrs/000-template.md` to `docs/adrs/NNN-short-title.md` (next number in sequence).
- Keep each ADR to 1–2 pages: Title, Status, Context, Decision, Consequences (trade-offs).
- List the ADR in `docs/adrs/README.md`.

## Diagrams

- Keep diagrams simple: boxes, arrows, words. If shapes mean different things, include a key.
- Put the source (Mermaid `.mmd`, draw.io `.drawio`, etc.) next to the exported `.png`/`.svg` so anyone can edit it.
- Reference every diagram from `docs/overview.md` or an ADR; the judges only see what the README links to.

## Before the Sept 16 deadline

- `README.md` links to every deliverable and nothing is orphaned.
- All diagrams have keys and are referenced.
- Every ADR has a Status of `Accepted`.
