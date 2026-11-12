# ADR-056: Consolidate Weekly GitHub Actions Dependabot Version Updates

Date: 2026-11-12

## Status

Accepted

## Decision

Retain weekly Dependabot version updates for the maintained GitHub Actions
ecosystem, including the existing seven-day cooldown, but replace the
`minor-and-patch` group with one wildcard
`actions-version-updates` group for eligible major, minor, and patch
version updates. The group explicitly uses `applies-to: version-updates`
and `patterns: ["*"]`.

Security updates remain independent of the scheduled routine version-update
group. Preserve the existing `dependabot/fetch-metadata` auto-merge
condition: a PR whose highest reported update is
`version-update:semver-major` is not automatically merged by that
workflow. Every grouped PR still requires effective checks and review under
the repository's GitHub branch and repository governance.

Do not close or merge pre-existing Dependabot PRs merely because the new
grouping has been configured. Reconcile them only after observing actual
replacement updates and sufficient CI evidence.

## Context

The research repository currently schedules GitHub Actions version updates
weekly, applies a seven-day cooldown, and groups only minor/patch changes.
Major-version updates therefore appear as separate proposals, contributing
to maintainer review load across several repositories. The newly imported
`coding_standards@v3.2.0` release documents a balanced weekly model with a
broad per-ecosystem routine update group.

This is a security-sensitive certificate-management research repository.
Workflow dependencies may control CI, release verification, artifact
publication, and access to credentials. Grouping a major update with smaller
updates does not reduce its compatibility or supply-chain risk. The existing
project threat model, generated-artifact rules, and release governance remain
binding.

## Assumptions and Constraints

- GitHub Actions is the currently configured Dependabot ecosystem.
- The existing weekly schedule and seven-day cooldown are intentional.
- Existing checks and CODEOWNERS requirements must be enforced by effective
  GitHub settings; repository YAML alone does not prove that they are.
- The existing auto-merge path is restricted to updates whose highest
  SemVer category is not major, and this boundary remains unchanged.
- Dependabot security updates must not be intentionally delayed by the
  routine update schedule or grouped into the routine version-update PR.
- No new GitHub permissions, runtime dependencies, or secret access is
  authorized by this decision.

## Alternatives Considered

1. **Keep the minor/patch-only group.** Rejected as the default because it
   perpetuates separate major-update PR overhead without improving review
   requirements for those major upgrades.
2. **Group all eligible routine versions, including majors.** Chosen for
   fewer routine PRs, while preserving an explicit manual review boundary
   when any major version is present.
3. **Use monthly updates or a longer cooldown.** Rejected for now because
   that would delay regular visibility into dependency changes without
   evidence of a sufficiently low-activity maintenance requirement.
4. **Automatically merge major-containing groups.** Rejected because
   release-tool and GitHub Actions compatibility changes require review of
   migrations, execution evidence, and affected trust boundaries.
5. **Close existing Dependabot PRs immediately.** Rejected because the new
   group does not retroactively establish coverage of earlier updates.

## Consequences and Review Requirements

Routine updates within the GitHub Actions ecosystem may now arrive together.
A major-containing group demands review of each dependency's release notes,
migration behavior, action permissions, and relevant tests, using the
highest-risk component to set review depth. Group failures may require
splitting out an incompatible or unusually privileged dependency.

No guaranteed one-PR-per-week result is asserted. GitHub dependency-graph,
alert, security-update, CODEOWNERS, and required-check settings need direct
verification; this ADR does not claim to have inspected or changed them.
Existing open Dependabot PRs remain subject to their own checks and reviews.

This change is limited to `.github/dependabot.yml` and governing
documentation. It does not change Certbot, Route53, S3, archive formats,
release artifacts, test execution permissions, or published interfaces.

## Expected Outcomes and Verification

The next eligible scheduled Dependabot run should propose routine version
updates in the named wildcard group when resolution permits. Verify that
major-containing updates do not enter the existing non-major auto-merge
path, and compare actual PR counts, dependency coverage, and failures
before concluding that review burden was reduced.

## Related Decisions

ADR-024 governs the verified, released shared coding-standards snapshot.
ADR-015 and ADR-016 govern dependency attack surfaces and threat-model
review. ADR-011 governs release semantics; ADR-009 governs behavioral
test evidence. This ADR supplements those decisions without superseding
their security, release, or runtime contracts.
