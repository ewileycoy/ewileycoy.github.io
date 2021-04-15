---
layout: default
title: In defense of Security through Obscurity
By: Karl
---

Ok, Hear Me Out.

I’m not advocating obscurity or complexity should ever be used as a *sole* method for securing data. AWS buckets with long random-named files can still be fuzzed and found, running services on non-standard ports can be scanned, etc. etc. In most cases obscurity and complexity probably make you less secure since it’s harder to understand your systems and harder to spot issues.

HOWEVER, *in certain cases*, obscure systems or non-default ways of accessing systems can delay or stop some attackers enough that you’re left responding to a minor incident rather than a full-blown account compromise. As long as you’re looking at the places attackers actually target with bulk, automated campaigns you can really get some value out of being a bit out-of-the-ordinary.

Take Azure/Office365 logins: Most phishing kits are designed to capture username and password and maybe the Microsoft Authenticator and get a valid login. In these cases, using the default (even multi-factor) login can still be successfully phished. But, if you run ADFS federation with Azure AD and a non-normal (Okta, for instance) MFA provider, phishing kits largely break down and stop working. Often just capturing the password isn’t enough to authenticate and it just hangs.

The attacker might still be smart enough to attempt an auth against it manually, but that delay is probably enough time for the user to suspect something is up. Besides, most MFA is only good for 90 seconds or so.

Even better, if you can construct a login that separates the username and password sequence (most SSO is doing this now), you have to make the phishers jump through more hoops that are custom to your environment. For instance, I can set a cookie that the browser has to re-use across the session, making it much harder to implement a proxy for the attack.

This is all predicated on configuring multi factor authentication in your environment [which really does solve a huge percent of phishing attacks by itself](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/your-pa-word-doesn-t-matter/ba-p/731984). Going that extra step in a mature organization can help your CISO sleep better and keep your blueteam from responding unnecessarily to email incidents.

Also, don’t forget to include any authentication providers in your scope for penetration tests, DR planning, and security audits. Think of how they could be bypassed or misconfigured. How can they fail? What happens?
