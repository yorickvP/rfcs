---
feature: improve-number-syntax
start-date: 2022-10-24
author: Yorick van Pelt
co-authors: (find a buddy later to help out with the RFC)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Change float and int literal syntax to disallow some things causing parsing ambiguity.

# Motivation
[motivation]: #motivation

The primary motivation is the horrors we discovered during the creation of `nixfmt`.

```
nix-repl> let e="e";in[001.2e01e.30.4]
[ 1 2 "e" 0.3 0.4 ]
```

Starting and ending floats with `.` leads to some ambiguity, and removing this feature would be an
obvious improvement.

Currently, leading 0's in ints are ignored, which causes confusing when things like `UMask = 0027` seem to work.

Removing leading 0's from ints opens the door for binary, octal and hexadecimal literals in the future.

Allowing integers to contain `_` is nice while we're at it. See [PEP-515](https://peps.python.org/pep-0515) for more details.

# Detailed design
[design]: #detailed-design


## Change floats to:
- cannot start with dot
- cannot end with dot

## Change ints to:
- cannot have leading zeroes
- can contain (but not start with) `_`


## How?


Change the float regex from
`(([1-9][0-9]*\.[0-9]*)|(0?\.[0-9]+))([Ee][+-]?[0-9]+)?`
to
`(([1-9][0-9]*\.[0-9]*)|(0\.[0-9]+))([Ee][+-]?[0-9]+)?`

And change the int regex from
`[0-9]+`
to
`0|[1-9][0-9]*`
todo insert regex here.

Change the lexer.i and parser.y files. Update the tests.

# Drawbacks
[drawbacks]: #drawbacks

The float change will break some code, but floats are not used very often. 
The int change will break some currently confusing code, mainly
in systemd units where masks get converted back to a string and then reinterpreted
as octal.

# Alternatives
[alternatives]: #alternatives

An alternative would be to require floats to be separated by whitespace or another token.
This resolves some syntactial ambiguity, but is harder to implement.

# Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD or unknowns?

# Future work
[future]: #future-work

Binary (0b111), octal (0o03), hex (0x10) integer literals could be useful.
Disallowing leading zeroes in ints helps with this.
