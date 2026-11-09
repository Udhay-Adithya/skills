---
name: security-review
description: Find vulnerabilities that cross a real trust boundary and produce a concrete result, with source evidence, a severity anchored to demonstrated impact, and the smallest fix that enforces the invariant. Use for security questions, a focused review of auth or permissions or tenancy, a vulnerability investigation, or an audit of a codebase before it ships.
---

# Security review

Find the vulnerabilities that violate a real trust boundary, then hand over the source
evidence, the priority and the smallest effective fix. Source-first and defensive.

**A candidate without a concrete affected principal, resource or security outcome is not a
finding.** That single rule removes most of what passes for security review.

## Modes

**Guidance**, the default. A security question, a focused review of one area, triage of a
specific finding. Use the relevant parts of this skill. Do not create an output directory
or write audit artifacts.

**Audit.** Only when an audit, a pen test, a comprehensive review, or a report artifact is
explicitly asked for. Then work through every surface systematically and write the report.

Where the request could mean either, ask one question before creating any files.

## Execution safety

Source inspection is read-only, and that covers most of this work.

Run anything the target controls, a build, a test, a fuzz case, a fixture, only where the
environment can enforce no external network, a scratch-local home and temp directory, a
read-only target, and explicit CPU, memory, file-size and wall-clock limits. Do not
install dependencies or let a build fetch them.

**Never probe a deployed endpoint, a shared service, production identity, another user's
data, or a live control plane.** Do not test availability against anything shared. Do not
spend paid quota. Stop at the minimum local effect that establishes the defect: a wrong
return value, an unauthorised record against a dummy tenant, a sanitiser firing. Do not go
on to produce persistence or concealment material.

Use dummy principals, fixtures and secrets throughout.

Where the decisive fact lives outside the source and outside a sandboxed fixture, it is
**needs-validation**, not a finding. Say exactly which fact is missing and give a safe way
for the owner to check it.

## The four principles

### Require a boundary and a result

For every candidate, name all six:

1. The lower-trust principal.
2. The input or action they control.
3. The control that was meant to stop it.
4. The boundary crossed.
5. The principal or resource affected.
6. The concrete result, observed or observable by the owner.

A missing best practice is not a finding. A guess about deployment behaviour is not a
finding. A generic parser crash is not a finding. Something that only affects the actor
themselves is not a finding.

### Use bounded local evidence

Static analysis establishes the path. A sandboxed local test resolves the behaviour: a
minimal harness, an existing unit test, a small fixture, a dummy-tenant integration test,
a locally rendered config.

Stop at the minimum effect. Extending a proof past the point where the boundary violation
is established adds risk and no information.

### Respect what the source can and cannot show

Deployment configuration, proxy behaviour, provider settings, browser headers, identity
policy and topology are all real controls. Where one is required and absent from the
repository, **do not assume either its presence or its absence.** That is a
needs-validation item with the exact missing fact named.

### Recommend the smallest effective fix

Identify the invariant the code must enforce, then the narrowest change that enforces it
at the last trusted decision point. A specific repository-relative change plus a
regression test, not a paragraph of generic hardening. The review describes the fix; it
does not rewrite the target.

## Severity, separated from certainty

Only a confirmed finding gets a severity, and the severity cannot exceed the demonstrated
impact.

| Severity | Means |
|---|---|
| **Critical** | An unauthenticated actor gains code execution, full data-store access, or takeover of arbitrary accounts |
| **High** | An actor fully defeats an explicit control with real consequences: auth bypass, cross-tenant read or write, stored script execution affecting others, authenticated code execution |
| **Medium** | A real boundary violation with limited blast radius, uncommon preconditions, or a narrow affected resource set |
| **Low** | Disclosure of non-secret internals, or an effect needing sustained effort for minimal gain |
| **Informational** | Confirmed, minimal impact, useful mainly as a step inside a larger finding |

**The high against medium discriminator:** does the demonstrated result fully defeat the
control, or only weaken it? If the concrete damage cannot be stated, the severity is lower
than it feels.

**Needs-validation has no severity.** It means a specific source-grounded hypothesis is
blocked on a fact, not that there is a low-confidence vulnerability.

## Where to look

Work the trust boundaries rather than a checklist. In rough order of how often each one
actually yields something:

**Authorisation, not authentication.** Authentication is usually present and usually
right. Authorisation is where the bugs are: an endpoint that checks a valid session and
never checks whether this user owns the row. For anything multi-tenant, every query that
reaches tenant data needs the tenant predicate, and the question for each one is whether
it comes from the request or from the session.

**Object references.** Any identifier arriving from a client is attacker-controlled. An
endpoint taking an id and returning the record without an ownership check is the most
common real finding in any application.

**The boundary between the framework's validation and your logic.** A validated shape is
not a validated value. A string that passed a type check can still be a path, a URL, a
template, or a query fragment.

**Secrets.** In the source tree, in the history, in logs, in error responses, in a client
bundle, in a mobile binary. Anything shipped to a client is public, including a key
compiled into an app.

**Tokens and sessions.** Lifetime, rotation, revocation, where they are stored and what
can read them. Browser storage is readable by any script on the origin, so it is not an
access control. A refresh token that moves into script-readable storage for convenience
has given up the property it existed for.

**Trust in your own services.** A response from your own API is untrusted input to the
client. A message from a queue, a webhook, a row written by another service: all
untrusted.

**Output encoding.** Where the data lands decides the encoding: HTML, an attribute, a URL,
a shell argument, SQL, a log line. One sanitising pass does not cover all of them.

**Dependencies and the supply chain.** What is pinned, what is not, what runs at install
time.

**Resource limits.** Unbounded pagination, unbounded upload, unbounded recursion, a regex
that backtracks. Treat these as findings only where there is a real unauthenticated path
to them.

## Anti-patterns

1. A checklist deviation presented as a vulnerability.
2. Defence-in-depth advice with no reachable boundary violation behind it.
3. Testing against live or shared infrastructure where local evidence would have done.
4. Guessing proxy, browser, provider or deployment behaviour that is not in the source.
5. Treating intended same-principal authority, or self-impact, as crossing a boundary.
6. Reporting an effect stronger than the one actually observed.
7. Assigning a severity to a needs-validation item.
8. Writing the report before verifying the claims in it.

## The report

For each confirmed finding: the boundary and the result as the six elements above, the
source evidence as `path/to/file:line`, the reproduction if there is a safe one, the
severity with the reasoning for it, and the smallest fix.

For each needs-validation item: the hypothesis, the source trace supporting it, the exact
missing fact, and a safe way to establish it.

Then the coverage statement. **Say what was not looked at.** No single pass is complete,
and a report that implies otherwise is worse than one that admits its scope, because the
gap gets treated as cleared.

## Beyond this skill

This is the distilled, single-reviewer version. Cloudflare's
[security-audit-skill](https://github.com/cloudflare/security-audit-skill) carries the full
multi-agent workflow: a deterministic coverage ledger, isolated hunter waves, independent
record verification, a findings schema with validators, and the write-isolation procedure
for promoting evidence out of a sandbox. Reach for that where a real audit of a large
target is wanted, rather than rebuilding it here.
