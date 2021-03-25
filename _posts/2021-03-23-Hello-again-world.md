---
layout: default
title: The Pain of GRC
By: Karl
---

As a security director with a large professional services firm, we get a *lot* of requests from clients to fill-out The Spreadsheets. You know the ones, the insane multi-tab macro-enabled monstrosities or garbage platforms like RSA Archer (slowly being replaced by silly startups like Whistic). All of which are just trying to glean the same basic information: Are you a complete moron who’s not going to lie to us? Seriously, though, I’m protecting data. I have regulatory requirements, and already Bad Things will happen if I screw that up.

Look, I get that vendor management is really important, just ask Target, TJX, and dozens of other companies. I have vendors to manage myself. We all have problems communicating about security.

I’ve had the pleasure of working with clients who do GRC[^Global Risk and Compliance: the catch-all term for vendor/risk management] well. We got on a call with their blueteam, talked through controls, discussed our approach, and they tried a few ‘gotcha’ questions that indicated our maturity. This is ~subjective~ but I think far more effective than just a ~~dumb~~ spreadsheet.

AICPA’s SOC2 and SOC3 make the case for being the current independent audit standard. It’s got a broad approach, but doesn’t solve all the issues. Point-in-time and self-defined controls don’t answer all of your questions as a client, and are not particularly satisfying as a security manager. I like the ability to select controls that make sense to me and my organization, but other than a pass/fail followed by a management response, it’s still too basic.

Industry-wide maturity-based certifications (like HITRUST or CMMC) are a step in the right direction, but just like CMMI in the development world, I don’t see them catching on broadly. Likely they’ll continue to exist in niche areas like DoD and healthcare, but wide-spread use is not going to happen. 

In my experience, I’ve found HITRUST extremely arbitrary and inflexible. Despite being maturity-based it has no ability to re-prioritize or select controls based on a use-case (outside of their “factors” selections). All the auditors I’ve spoken to recommend doing the bare minimum to “pass” since there’s no benefit to going beyond the “implemented” milestone. What’s worse, after all the pain and effort of a HITRUST certification, the majority of my clients have never heard of it.

CMMC makes sense to have overly-specific requirements, since it’s literally developed for one client: the US federal government. No one else is likely to care.

Overall, I think maturity models can work but I think there need to be some better standards:
- Open standard: the standard itself should not be owned by a company. Some open standards body (something like OWASP) or government agency (like DHS) should own and promote the standard which should be free to anyone who wants to use it.
- Customizable: the standard should support risk-based customization of controls in an obvious way. The standard should incorporate certain baseline minimums (akin to the SANS Top 20 ‘quick wins’ list), without compromising the flexibility
- Clear: The HITRUST scoring matrix drives me insane. I understand why it exists, but it’s *too much*. Maturity does not have to be complicated to measure, depending on how we define maturity. It’s also very difficult to explain to someone not already familiar with HITRUST what all the different levels, percentages, and plans mean. A good, clear definition of security maturity given certain controls would improve a model significantly.
- Dependency-aware: Currently all controls are measured independently, ignoring all others. In the real world you implement controls within the context of a system and other controls that have their own advantages, coverage, and depth. For instance, context-aware authentication could let you remove multi-factor authentication in some situations, or change re-authentication intervals based on other controls in effect. Currently you have to define compensating controls in a complicated, one-off manner (like in PCI), or like in HITRUST there are no compensating controls.
- Self-certification: organizations should be able to self-certify, period. Independent audit is mostly a veneer over spreadsheets. Effective third-party testing (like penetration testing) can be an option or even a requirement, but certification against a standard should rest with the organization.

Obviously getting everyone to agree on some standard is incredibly difficult (hell, we can’t even agree on what good passwords are or what actually constitutes multi-factor authentication)[^this is not true: NIST 800-63r3 has already fixed both of these issues, but no one believes them] But I think it’s worthwhile to try. OWASP and SANS top 10 are great example of effective, consensus-driven security controls frameworks that don’t have to be impossible to implement.

There is no perfect system, but by exercising a clear, lowest-common-denominator approach, I think we can at least start to describe comparative operational security. I should be able to hand over one report and that be the end of the conversation. If it’s not adequate for an organization’s third-party risk management, then the report itself should be updated, so the additional controls or attestations can be captured and re-used. I mostly just want to stop filling-out the same spreadsheet over and over.