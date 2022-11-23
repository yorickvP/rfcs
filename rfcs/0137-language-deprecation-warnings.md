---
feature: language-deprecation-warnings
start-date: 2022-11-01
author: Yorick van Pelt
co-authors: (find a buddy later to help out with the RFC)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Introduce a way to deprecate constructs in the Nix language, generating
warnings in the Nix CLI.

# Motivation
[motivation]: #motivation

We want to keep Nix evolving, but not cause undue maintenance burden
for anyone using it. This sometimes involves breaking changes.
Possible examples of what we might want to propose are:

- removing url literals (currently implemented via experimental-features)
- removing the old `let { body = ... }` syntax
- disallow leading zeroes in integer literals
- disallow '.x or y' operator on non-attrsets

To do this, we want a mechanism to allow deprecating Nix language constructs.

# Detailed design
[design]: #detailed-design

Introduce two command line flags, controlling the behaviour of deprecation warnings.

```
--no-warn-deprecated="url-literal let-body int-zeroes non-attr-select"
--error-deprecated="url-literal let-body int-zeroes non-attr-select"
```

Then, during in evaluation,
- in non-verbose mode, warn once for each construct.
- In verbose mode, warn for each occurrence.

Print statistics and explanations at the end of the evaluation.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

```
nix eval --json ./test.nix
warning: URL literals are deprecated (url-literal)
         please replace this with a string: "https://google.com"

       at test.nix:1:1:

            1| https://google.com
             | ^

"https://google.com"

warning: The following deprecated features were used:
  - url-literal (nix.dev/c/dXJs), 1 time

  Use `--no-warn-deprecated=<feature>` to disable this warning, and `--verbose` to show all occurrences.
```

# Drawbacks
[drawbacks]: #drawbacks

Introducing breaking changes to the Nix language will cause maintenance
burden for everyone using these old constructs, or evaluating old nixpkgs.

As an example, if we were to deprecate leading zeroes, this would show
a warning on nixpkgs prior to 22.11.

# Alternatives
[alternatives]: #alternatives

- No warnings, just errors
- Keep everything working forever
- Opt-in warnings
- Nix 'editions'
- Use flakes lastModifiedDate to detect code age.

# Unresolved questions
[unresolved]: #unresolved-questions

Which language constructs are we going to deprecate? Should we keep
deprecation warnings around forever?
Should we attempt to be compatible with C-compilers -Wno- and -Werror=?

# Future work
[future]: #future-work

Future work would be deprecating constructs using the proposed mechanism.
