- Feature Name: `gotrue_unfork`
- Start Date: 2021-10-07
- RFC PR: [supabase/rfcs#0000](https://github.com/supabase/rfcs/pull/0000)
- Supabase Issue: [supabase/supabase#0000](https://github.com/supabase/supabase/issues/0000)

# Summary
[summary]: #summary

Our [Auth server](https://github.com/supabase/gotrue) is currently a fork of [GoTrue by Netlify](https://github.com/netlify/gotrue). This RFC explores the possibility of "unforking".

# Motivation
[motivation]: #motivation

There are several drawbacks to the current fork:

- *Search*: Due to the way GitHub operates, if you search in our repo it will instead search in the fork. This makes it difficult to find issues/code/PRs.
- *Divergence*: The core codebase is diverging a lot:
  - Supabase uses Postgres, Netlify uses MySQL
  - Netlify uses some methods for managing admin routes which we have removed. Specifically the require a "user" to be marked as an admin, whereas we have implemented Key Auth instead.
- *Different goals*: our fork is becoming one of our key products, and as such we'd like to refactor it (a lot), to enable new features which aren't going to be included in the Netlify Identity product.

# Drawbacks
[drawbacks]: #drawbacks

- We lose on collaboration. Ideally we share updates & security fixes.
- Not the collaborative approach we'd prefer.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Don't fork, let the fork diverge.
- Don't fork, work with the Netlify team to build a unified product which accommodates for Supabase + Netlify goals.
- Build a committee between the two companies where the repo can live with it's own goals.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What is the GitHub process for breaking the fork (ideally while maintaining our issues/PRs)?
