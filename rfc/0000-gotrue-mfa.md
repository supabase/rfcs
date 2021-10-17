- Feature Name: `gotrue-mfa`
- Start Date: 2021-10-17
- RFC PR: [supabase/rfcs#0003](https://github.com/supabase/rfcs/pull/3)
- Supabase Issue: [supabase/supabase#0000](https://github.com/supabase/supabase/issues/0000)

# Summary
[summary]: #summary

The Auth Server currently dosen't support MFA - [TOTP](https://datatracker.ietf.org/doc/html/rfc6238), [HTOP](https://datatracker.ietf.org/doc/html/rfc4226), [FIDO UTF](https://fidoalliance.org/specs/fido-u2f-v1.2-ps-20170411/fido-u2f-README-v1.2-ps-20170411.txt). This RFC explores the ability to add support for this and possibly contribute it upstream. 

# Motivation
[motivation]: #motivation

MFA is very important for security of accounts. Many platforms already support it and some platforms require it as a security measure. It would be a positive improvement to Supabase to have support for people/companies that have that as an internal requirement for a platform. This is also a feature that is offered by limited services so may give Supabase a better standing compared to them.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Explain the proposal as if it was already included in the platform and you were teaching it to another Supabase developers. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Supabase developers should *think* about the feature, and how it should impact the way they use Supabase. It should explain the impact as concretely as possible.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Make users implement it themselves
- Make users use another service
  - This service may already exist or might need to be build.

# Prior art
[prior-art]: #prior-art

- It looks like Auth0 has support for [MFA](https://auth0.com/docs/login/mfa). The Auth0 implementations seams to offer [limited customization](https://auth0.com/docs/login/mfa/customize-mfa-user-pages) only allowing for the supported methods of MFa to be turned on or off.

- Firebase doesn't have 1st hand support for MFA. Using other google services you can support [OTP via SMS](https://stackoverflow.com/a/52906804/14918759). However, that is an extra cost and requires extra setup as shown [here](https://cloud.google.com/identity-platform/docs/web/mfa).

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Would an extra endpoint be added e.g. `/mfa`
  - How would we identify which user?
  - How would we manage latency and codes expiring?

# Future possibilities
[future-possibilities]: #future-possibilities

Think about what the natural extension and evolution of your proposal would
be and how it would affect the language and project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project and language in your proposal.
Also consider how this all fits into the roadmap for the project
and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
