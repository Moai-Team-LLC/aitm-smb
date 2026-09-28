# AITM-SMB Repository Maintenance

Framework governance ([`NORMATIVE_INDEX.md`](NORMATIVE_INDEX.md)): rules for changing this repository, not for engagements. Change classes: [`VERSIONING.md`](VERSIONING.md). Contribution process: [`CONTRIBUTING.md`](CONTRIBUTING.md).

## 1. Purpose

AITM-SMB is agent-readable.

Documentation drift is therefore a methodology defect.

---

## 2. Release checks

Every release passes, in order:

```text
1. python3 tools/validate.py                                   0 errors
2. python3 tools/validate.py --engagement examples/compact-scenario-b   0 errors
3. skills/39-audit-framework-integrity/SKILL.md                no unresolved finding
4. orphan check: every validator orphan warning resolved
5. CHANGELOG.md section for the version, and release notes (`releases/<version>.md`) with the migration table for any field rename
6. rubrics/RELEASE_READINESS.md reviewed                       informative
```

Steps 1–2 MUST pass (CI runs them on every change and before a tagged release); steps 3–6 SHOULD be completed.

Step 1 verifies that all normative, skill, and artifact references resolve; skill and artifact contracts are well-formed and listed in their registries; identifier prefixes and gate IDs are registered; deprecated content is marked and does not leak into active files; versions align ([`VERSIONING.md`](VERSIONING.md) §8), including that the MANIFEST version matches the release and CHANGELOG.md is updated; [`REPO_STRUCTURE.md`](REPO_STRUCTURE.md) matches the file tree (regenerate with `python3 tools/validate.py --write-structure`).

Step 3 covers what a script cannot judge: no duplicate canonical definition exists (§3, §4), semantics are consistent, and the §5 audits hold.

Orphan check: a file with no inbound reference may never be loaded under bounded context loading ([`AGENT_CONTEXT_POLICY.md`](AGENT_CONTEXT_POLICY.md)). Reference it from where it is used, or remove it.

Record the result in the release notes ([`releases/`](releases/)). Tagging and publication: [`VERSIONING.md`](VERSIONING.md) §10. The 1.0.0 record: [`releases/1.0.0-release-checklist.md`](releases/1.0.0-release-checklist.md).

A published tag never moves. If a release's distributions must be rebuilt (a failed upload, or a defect in the build rather than in the tagged files), run the release workflow manually with the tag as its input: it rebuilds both distributions from the tagged tree, checks that the unpacked Full distribution validates on its own, and replaces the release assets. Record the rebuild in CHANGELOG.md.

A material methodology change, and every MAJOR change, SHOULD be recorded as an ADR ([`decisions/README.md`](decisions/README.md); template [`decisions/ADR-0000-template.md`](decisions/ADR-0000-template.md)).

---

## 3. Single-definition rule

Each record shape is defined exactly once; [`CANONICAL_CONCEPTS.md`](CANONICAL_CONCEPTS.md) names where.

Modules own meaning and rules; the record contract owns field names.

Files that hold instances reference the record contract and MAY add fields only where they say so explicitly ("extends <record> with: …").

A concept SHOULD have one canonical definition; other files SHOULD reference it. Where each concept is defined: [`CANONICAL_CONCEPTS.md`](CANONICAL_CONCEPTS.md) §1–§2. Example: Capability — semantics and record [`CORE_MODEL.md`](CORE_MODEL.md) §2; instances [`artifacts/capability-map.md`](artifacts/capability-map.md).

---

## 4. Duplication classes

### Acceptable duplication

Short reminders or summaries.

### Dangerous duplication

Two normative definitions that may diverge.

Dangerous duplication SHOULD be removed or replaced with a reference to the canonical source.

---

## 5. Release audit

Before every MINOR or MAJOR release, also perform (skill 39 procedure; step 1 of §2 where automated):

```text
semantic consistency audit     Core, modules, contracts and skills agree (NORMATIVE_INDEX.md)
broken-reference audit         automated
duplicate-definition audit     §3, §4; includes artifact-versus-module field parity
profile completeness audit     every APPLICATION_PROFILES.md item resolves to an artifact
                               contract (MINIMUM_ARTIFACT_SET.md), module, or skill
skill dependency audit         every skill input is produced upstream; every orchestrator's
                               specialists exist; no skill is unreachable
artifact completeness audit    every contract has a producing skill; produced_by, the
                               artifacts/INDEX.md column and the skills' Produces sections
                               name the same set; uniform shape (artifacts/_ARTIFACT_CONTRACT.md)
scenario walk-through          changed methodology still works for every scenario in
                               validation/ABSTRACT_SCENARIOS.md and the worked example
```

---

## 6. Self-contained layout

Every path is relative to the repository root, and every command runs from it. AITM-SMB can therefore be installed as one skill, vendored into another repository as a subfolder, or unpacked from the Full distribution, and [`tools/validate.py`](tools/validate.py) and [`tools/build_dist.py`](tools/build_dist.py) work unchanged in each place. Keep it that way: no link or command may reach outside the root.
