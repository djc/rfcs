- Feature Name: crates-io-vulnerabilities
- Start Date: 2025-10-06
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

This RFC proposes that crates.io should provide insight into vulnerabilities and unsound
API surface based on the RustSec advisory database.

# Motivation
[motivation]: #motivation

One of the roles that crates.io serves for Rust developers is as a discovery mechanism for library
packages. As such, it is important that users can quickly assess the quality of a given crate,
including security considerations such as unsound code/API or known vulnerabilities.
The RustSec advisory database is a curated database of security advisories for Rust crates,
which tracks known vulnerabilities, unsound code, and maintenance status of crates.

The Rust ecosystem has a culture of having smaller, focused crates with a clear purpose.
As a result, many Rust projects have a large number of dependencies, which increases the
risk of introducing problems in the final artifact via the supply chain of dependencies.

We've seen an increasing number of security issues via transitive dependencies:

* In 2024, some releases of the popular xz-utils package (written in C) contained
  a [malicious backdoor] affecting OpenSSH servers running on the local system.
* In 2025, a phishing campaign [targeted crates.io users] using the `rustfoundation.dev` domain
  name to impersonate the Rust Foundation and steal maintainer's credentials.

The Open Source Security Foundation (OpenSSF) has enumerated [Principles for Package Repository
Security]; while crates.io already addresses many of these, one of these is:

> The package repository warns of known security vulnerabilities in dependencies in the package
> repository UI.

The RustSec advisory database tooling already supports exporting advisories in the OSV format.

[malicious backdoor]: https://en.wikipedia.org/wiki/XZ_Utils_backdoor
[targeted crates.io users]: https://blog.rust-lang.org/2025/09/12/crates-io-phishing-campaign/
[Principles for Package Repository Security]: https://repos.openssf.org/principles-for-package-repository-security.html

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

The crates.io website will display information about known vulnerabilities and unsound APIs.
This might take the form of a Security tab on a crate's page. If there are known vulnerabilities
for the currently selected version, the tab might be highlighted.

Opening the Security tab for a crate should show a list of advisories that affect the crate,
including a summary of the issue, a list of affected versions, and links to more information.
Care should be taken to ensure that the mere existence of past vulnerabilities does not negatively
impact the perceived quality of a crate; very popular crates are much more likely to have
vulnerabilities reported against them, simply due to their popularity and the amount of scrutiny.

The Security tab should give Rust developers a quick overview of the security status of a crate,
and allow them to make informed decisions about whether to use the crate in their projects.

While this information is available today via the RustSec website (including feeds that can
automatically be consumed by tooling), having this information directly on crates.io would
make it accessible and visible to a wider audience.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

There are two potential ways of implementing this feature:

- Stateless: the RustSec website will expose a per-crate JSON endpoint that lists all advisories
  affecting the crate. The crates.io website will fetch this data on-demand when rendering the
  crate page. This is simpler to implement but might incur extra latency for the user.
- Stateful: the crates.io backend will periodically update a local copy of the RustSec advisory
  database, and will store a mapping of crates to advisories in its own database. This will allow
  for faster rendering of crate pages but requires more maintenance.

In both cases, the RustSec project publishes a number of Rust crates that can be used to parse
and query the advisory database, which can be reused in the crates.io backend.

[advisory-db repository]: https://github.com/RustSec/advisory-db

# Drawbacks
[drawbacks]: #drawbacks

The RustSec project is maintained by an independent team of volunteers, so the crates.io Security
tab will be reflecting data that is maintained by what amounts to a kind of third party.
The Leadership Council has an [ongoing discussion] on governance for the Secure Code WG that
governs the RustSec project, which might be relevant to this proposal.

Rust developers might be scared off of using crates that have known vulnerabilities, even if
those vulnerabilities are not relevant to their use case, or have been fixed in later versions.
This seems like a reasonable trade-off to me -- we should allow informed users to make decisions
that are best for their projects.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

crates.io is the official package repository for the Rust ecosystem, so sharing important security
context via this interface seems like an effective way to make it accessible to a wide audience.

Widely used tools like [cargo-audit] and [cargo-deny] already provide a way to check for
security-sensitive issues in a Rust project's dependencies, but these tools are opt-in and require
users to be aware of them and to run them. They are also more focused on auditing a project's
existing dependencies rather than helping inform users in the discovery phase.

Alternatively, we might make the RustSec advisory database available directly via cargo. This
seems mostly unrelated to what crates.io does, and seems like an interesting future possibility.

[cargo-audit]: https://crates.io/crates/cargo-audit
[cargo-deny]: https://crates.io/crates/cargo-deny

# Prior art
[prior-art]: #prior-art

Neither npm nor PyPI currently seem to provide support for displaying security advisories.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

This seems like a relatively straightforward feature with a limited scope. The main questions
are about the desirability of the feature, the implementation approach, and the governance
of the source data.

# Future possibilities
[future-possibilities]: #future-possibilities

In the future, it would be valuable if lockfile updates exposed open vulnerabilities in a
project's dependency graph in the Cargo CLI, for example on `cargo update` or `cargo check`.
