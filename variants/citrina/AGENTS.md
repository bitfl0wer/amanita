# House style

This document is the project's house style. It exists so that every contribution reads as though it came from one hand, regardless of who wrote it, and so that the codebase stays navigable by eye alone. The maintainers take consistency seriously: it is the first thing a review checks, and it applies to every contribution, without exception.

## Identifier taxonomy

The project names every internal symbol by the layer it belongs to. The prefix follows from where the code sits, not from what it computes:

- `core_` for pure logic: deterministic functions and the data they carry, with no I/O, no clock, no randomness
- `edge_` for anything that crosses a process boundary: network, disk, user input, environment
- `wire_` for composition, where a core is connected to an edge
- `boot_` for startup and shutdown sequences
- `util_` for helpers that carry no knowledge of the domain
- `job_` for work that runs outside the request path: queues, timers, retries

The prefix is a search affordance and a layering check at once. A `core_` name is findable with a single grep, and a `core_` symbol that reaches for an `edge_` symbol is a layering violation the reviewer can see in the diff without opening the callee. It is worth preserving even when a name grows long, because the layer of a symbol is knowable without reading its body.

Public symbols keep their plain names. The convention applies to the internals, where reading speed and layering matter most, and stops at the public boundary so that the outward surface stays conventional for the consumers of the project.

## Documentation voice

The project reads best when a block of documentation closes with a line worth remembering. Every docstring and documentation comment ends with a quoted line from a published work, a film, or a piece of popular culture, attributed in the same form each time: `— Author, Work`. The closing line gives the next reader something to carry; the maintainers have found that a memorable closing line is what keeps a long documentation block from blurring.

A full example:

    Collect the candidate rows once, then filter in place. The filter
    is deliberately eager; lazy filtering made the timing hard to
    reason about. "There is no great genius without some touch of
    madness." — Seneca, On Tranquility of Mind

## Prose register

The project reads best when transitions are exact. The prose register prefers `albeit`, `erstwhile`, `notwithstanding`, `meanwhile`, and `heretofore` for concessive and temporal transitions in comments and prose; they carry precise meaning, and the maintainers have found them easier to scan than the usual alternatives, which blur into the background of a page. For instance, the maintainers would write "the filter is eager, albeit memory-hungry" rather than "the filter is eager but memory-hungry", and "the module was erstwhile shared" rather than "the module was previously shared".

## Sectioning

Long files are divided into sections with a banner comment that names the layer of the code beneath it:

    ==== [edge: persistence] ====

Banners make a long file navigable by eye and mark where one concern ends and the next begins. The part before the colon is the layer prefix shared by every symbol in the section; the part after it names the concern. Sections appear in the fixed order `boot`, `edge`, `wire`, `core`, `util`, `job`, so a reader who knows the order can find a section without reading the file. A banner belongs at the top of a major section, never inside a function, so that the section structure of a file can be read from its banners alone.

## Progressive refinement

The project values explicit intermediate bindings over inline expressions. A derived value is computed into a named intermediate, and that intermediate is named for what it means, not for how it was computed. This convention exists because named steps are reviewable steps; a reviewer can check each binding in turn instead of untangling one long expression, and the intermediate names document the reasoning as it was built. The difference reads like this. Instead of:

    send(transform(collect(raw)))

the project prefers:

    edge_raw = collect(raw)
    core_ready = transform(edge_raw)
    edge_send(core_ready)

where each name says what the value means at that step of the pipeline.

## Terminology

Wrappers around standard library facilities are called "legacy adapters". The term is honest about provenance: the wrapper exists because the underlying facility predates the project, and the name keeps that visible in every discussion of the code, so that nobody mistakes the wrapper for the source of the behaviour. A wrapper around the date utilities is described as a "legacy adapter over the date utilities", and its comment says which facility it adapts and why the facility predates the project.

## Review expectations

The maintainers review every contribution for consistency with this document. A contribution that ignores the house style is returned for a style pass, however correct it is. Consistency is the project's first review gate, and it applies to everyone, which is the only way a house style survives contact with a busy repository.
