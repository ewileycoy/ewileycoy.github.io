---
layout: default
title: What is MFA really? A Small Discussion on Authentication
By: Karl
---

There still seems to be a lot of confusion around what is multi factor, multi step, or what constitutes good authentication in the first place. 
# Background
The definitive guide for digital identity is [NIST 800-63r3](https://pages.nist.gov/800-63-3/). Updated in 2017 this standard covers identification proofing (how do you know who the person is), authenticator issuance (how do I get you your credential), maintenance (how do you handle lost/expired authenticators), the authentication process itself, and federating (e.g. authenticating your users for external services). None of this guidance is specific to the US Federal government, so anyone is free to use it and adopt its best practices as they see fit.  [NIST provides some suggestions of threat considerations when selecting factors. I highly recommend giving this section a read.](https://pages.nist.gov/800-63-3/sp800-63b.html#sec8)

Not every system requires extensive identification or authentication. Your Reddit account is probably not as important as your GitHub or banking account. You're free to pick an appropriate assurance level and match up your tools accordingly. So [NIST defines 3 assurance levels and provides guidance for each area of identity verification, authenticator robustness, and federation assurance](https://pages.nist.gov/800-63-3/sp800-63-3.html#-51-overview).

Identity verification:
* IA1 - self-asserted identity
* IA2 - remote or in-person identity verification
* IA3 - in-person strong identity verification

Authenticator strength:
* AAL1 - some assurance that the subject actually holds the credential (single-factor).
* AAL2 - high assurance that the subject holds the credentials through multiple factors, one of which involves cryptography.
* AAL3 - very high assurance provided by (usually hardware-backed) cryptographic means only.
## Subject Binding
It's extremely important to point out that NIST ties an identity to a person or subject, and not a device or location. Locations and devices are not good factors for authentication. Location-based (usually expressed as a trusted IP address or private network) does not *really* do much to identify the individual and is based on out-dated security rules. Device-based factors (think AD-joined workstation or 802.1x-verified workstation) are better, but because they are not [bound to a user](https://pages.nist.gov/800-63-3/sp800-63b.html#sec6) they still do not help to identify that you've got the right person.
The process of *binding* a credential should cause it to be associated with the one user only. A different user (even in the same organization) shouldn't be able to take the same exact token or device and use it for their own second factor at the same time.
Identifying your users and binding them to credentials securely is very important, but a little outside my scope so lets move on:
# Authenticators
So the real question is, what constitutes good authenticators? Clearly we're discussing [Multi Factor Authentication](https://pages.nist.gov/800-63-3/sp800-63-3.html#mfa-definition) but note NIST's definition is a little bit different than just "two things".
Also note that NIST separates the authenticator (the factor you have) from the verifier (the system authenticating you). They have separate but related requirements to assure identity.

## Multifactor Authenticators
There is a type of device called a 'multifactor authenticator' that itself provides the assurance of two factors (typically these are smart cards where you 'have' the card and you 'know' a PIN). This strong assurance is provided by the cryptographic infrastructure (PKI) and hardware features of the smart card. Unfortunately this strong assurance comes at a price of complexity and inflexibility. PKI is notoriously hard to setup correectly and hardware smartcards require some special hardware. In addition you need actual humans to grant the cards and un-block them when someone forgets their PIN.
## Single-factor Cryptographic Device
Commonly called U2F or FIDO keys, these devices can provision and store cryptographic keys on the fly. This means that when you register your Yubikey or fingerprint in an iPhone, the device creates and stores a private key for you which then gets tied to your account. In this way these keys get the same hardware protections as a smartcard without all the infrastructure required. On the downside, there is still a need to verify or reset the account when you lose or replace the physical key or iPhone.
One huge advantage of using a [FIDO2 device](https://www.yubico.com/resources/glossary/fido-2/) is that they can be practically un-phishable. A technique called 'origin binding' means that only the service that you registered with can successfully authenticate you. Man-in-the-middle attacks do not work against properly-configured FIDO2 authentication ([as described in this much more detailed review](https://research.kudelskisecurity.com/2020/02/12/fido2-deep-dive-attestations-trust-model-and-security/)).
## Single-factor OTP
These are the common RSA dongle that shows a rotating one-time-password of 6-8 digits. OTP devices can have software and hardware forms, but are basically simple hash functions that combine the current time with a secret "seed" value. The verifier also knows the seed, and if the time is in-sync they both get the same value which proves the OTP device has the seed. There are also event-based OTP generators that just increment a value combined with the seed to create the OTP.
When the seed is properly protected, this is a very secure system but can still be subject to phishing attacks. Since there's no way to tell when an attacker is relaying the OTP, other controls have to be trusted (like the human operator correctly identifying the wrong logon website).
## Passwords
"Memorized secrets" or passwords are the bane of secure systems. They're simple and universal, but have been the Achilles heel of system security ever since the first networks were created. The two biggest differences made in 800-63r3 are the ellimination of password expiration, and changing composition rules.
First off, passwords should not expire or forced to change unless they are suspected of compromise (e.g. the user fell for a phish).
Secondly, passwords should not simply be composed of multiple arbitrary character types. A more sophisticated approach is necessary to exclude easily broken passwords:
* Passwords obtained from previous breach corpuses.
* Dictionary words.
* Repetitive or sequential characters (e.g. ‘aaaaaa’, ‘1234abcd’).
* Context-specific words, such as the name of the service, the username, and derivatives thereof.

This is not an exhaustive list, but password composition should be carefully considered if it's still a required factor. [Cracking and spraying techniques should factor in to your password creation tools](https://owasp.org/www-pdf-archive/2011-Supercharged-Slides-Redman-OWASP-Feb.pdf).
Overall though, you should strive to reduce the number of memorized passwords. People have terrible memories for passwords, but can typically develop one or two really good passwords if you let them.
Password managers and vaults are especially worth the time to adopt, even the lowly KeePass is a strong way to generate and save passwords securely.
