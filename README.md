# Workflow registry

Versioned workflow definitions for [Workflow Console](https://github.com/RyanAmundson/workflow-console).

A workflow is a YAML file describing phases of agents and the prompt each one
receives. This repository is the registry those definitions are published to, so
a run on any machine uses the version that was dispatched rather than whatever
copy that machine happened to have on disk.

## Why a Git repository

A registry is a versioned, addressable, permanent collection of text, and Git is
already that. Git also supplies the thing that matters most here: **review before
anyone runs it.** A workflow instructs an agent that may hold write access to a
checkout, so a human reading the prompt is the correct gate, and a pull request is
that gate.

The alternative considered was a multi-tenant database. It needed an identity
system, a security-rules rewrite, and read quota, to arrive at weaker review.

## Layout

```
index.json                       generated; what a console reads to list and search
workflows/<owner>/<name>.yaml    the definitions
```

`index.json` carries each workflow's name, description, tags, trigger, estimate,
permissions, declared params, cloud spec, and content hash — enough to browse and
filter without fetching every definition.

## Versions are content hashes

A version id is the SHA-256 of the definition's canonical bytes, where canonical
means newlines normalised and trailing whitespace trimmed. Key order, comments,
and indentation are all part of the version, because they are part of what an
agent reads.

Identical bytes therefore produce an identical version anywhere: in this registry,
in a private library, or in the copy bundled with the desktop app. A pinned hash
means one exact set of instructions, and there is no way for it to mean anything
else later.

## Publishing

Definitions are generated, not hand-edited here. From a Workflow Console checkout:

```bash
workflow-console registry build <owner> <path-to-this-repo>
```

Then open a pull request. Regenerating is idempotent: entries are sorted, so a
rebuild diffs cleanly and one real change is not buried in reordering.

Two classes of workflow are refused at build time rather than published broken:

- **Anything that cannot start.** The build runs the console's dry run, which
  renders every prompt and reports unfilled placeholders, artifacts that escape
  the checkout, a phase with no agent, and similar. A registry should not hand
  someone instructions that fail on first use.
- **Anything that describes one machine.** A workflow whose inputs default to an
  absolute or home-relative path means nothing on anyone else's disk, and that is
  also how private work leaks into a shared place.

## What is not here

**Script workflows.** Some Workflow Console workflows are JavaScript rather than
YAML. A YAML workflow is a set of prompts a reviewer can judge; a script is
arbitrary code that runs on the operator's machine. Distributing executable code
through a registry is a supply-chain mechanism, and it needs signing, pinned
hashes, and an explicit consent step before it would be responsible. Scripts stay
local.
