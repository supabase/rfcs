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

When you are wanting to enable 2FA you should go into your auth settings and enable 2FA. You should then see more options for 2FA e.g. Enabled methods, Settings for each method, MFA rules, etc. Once you enable a method such as SMS OTPs you can then edit the message that is sent out using the same templates your used too with email templates. You can also set rules for MFA such as all accounts need it but only require when the login is suspicious, all accounts need it and must verify each time or no accounts need it. Once you have everything configured your then read to start using MFA.

When you now use the `supabase.auth.signIn()` you can recieve either the normal response that your used too or recieve a response telling you MFA is required. If you are required to get MFA you can then use `supabase.auth.mfa()` which takes the token from the `signIn()` response and the 2FA code the user inputed. The `mfa()` method will then either return a positive response (the response your used too from `signIn()`) or a negative response (tbd -  the negative response has a new token to be used or there is another method e.g. `mfaRetry()` that can be used to get a new code upto x time). 

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

When implementing MFA we need to somehow get an identify from the server e.g. on login instead of returning the tokens we return:
```json
{
    "type": 0 | 1, // If its just a login or needs mfa
    // ...rest of the data
}
```
For the the MFA type we could return data such as below:
```json
{
    "type" : 1,
    "mfa_token": "aaa.bbb.ccc"
}
```
That MFA token could then be used to identify the requesting user for the `/mfa` endpoint. The MFA token could include data such as enabled/requested MFA type, User ID and could be set to expire at a relevant time based on the MFA method(s). The `/mfa` ednpoint could then accept the data in just JSON e.g.
```json
{
    "mfa_token": "aaa.bbb.ccc",
    "code": "123456" 
}
```
Or it could use a mix of headers and json e.g. X-MFA-Token with the token and then just the code in the JSON body. Once MFA is completed there are a couple methods of then being used. We could then return a token that can be used with the login endpoint or it could just return the login data.


# Drawbacks
[drawbacks]: #drawbacks

- Lots to implement
- Many MFA methods
  - Different people have different requirements/wants.

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
- What would be needed in the UI?
  - Config for what MFA should be enabled
  - OTP SMS Templates

# Future possibilities
[future-possibilities]: #future-possibilities

- Read the RFCs for [TOTP](https://datatracker.ietf.org/doc/html/rfc6238) & [HTOP](https://datatracker.ietf.org/doc/html/rfc4226)
