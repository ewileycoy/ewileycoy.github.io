---
layout: default
title: Hello! Finally a Cloud I can Trust
By: Karl
---

## Introduction
[I've written about Windows Hello for Business before](/2021/04/15/Hello-is-it-me.html) along with the promises and confusion it brings. To Microsoft's credit they've been maturing and improving incrementally, adding features and pushing their "passwordless" initative consistently. With the advent of Windows 10 21H2 Microsoft introduced a new way to onboard and authenticate devices using a "Cloud Trust" model also called "Cloud Kerberos Trust". I think this is a great step forward and between Microsoft, Google, and Apple, hopefully we can continue to displace passwords.

To take a step back, Microsoft is leaning-in to a "passwordless" strategy that involves the use of trusted cryptographic devices that can attest to an account, rather than a memorized secret.

[In a discussion on Passkeys, ](https://securitycryptographywhatever.buzzsprout.com/1822302/11122508)  Google's Adam Langley discusses why we may need to re-think our relationship to 'multifactor' because of how bad passwords are:
> [M]ultifactor was sort of a sensible concept in the world where the first factor of the password was so bad. Uh, but if you do want to think in terms of multifactor, right security keys can be several factors.

In other words most of the 'second factors' we've added to authentications have really been bandaids for just how insecure passwords are. Since crypto-backed authenticators are fundamentally different from passwords, I think it's right that we can treat them with a different attitude as security professionals, _if_ certain conditions are met.

## Windows Hello for Business trust models

Microsoft provides two major approaches to "passwordless" credentials in WHFB: *key trust* and *certificate trust*. Certificate trust requires a Public Key Infrastructure (PKI) that you administer to issue certificates to end-users and devices, typically this consists of Active Directory Certificate Services with Active Directory Federation Services deployed to issue certificates. There are some scenarios that require certificate sign-on such as Remote Desktop Protocol (RDP) or Citrix's cloud-based Virtual Desktop Infrastructure gateway.

Because running on-premise PKI is complex and [fraught with possible security misconfigurations](https://www.youtube.com/watch?v=ejmAIgxFRgM), Microsoft encourages key trust deployments. Key trust utilizes a FIDO-type device container to generate private keys on a device in order to link the credential to a user.

Previously, WHFB's key trust deployment separated the credential completely from on-premise AD by issuing separate certificates to devices as part of a hybrid join process. Because this is effectively a secondary (but linked) credential to an on-prem identity, organizations had to carefully replicate and synchronize on-premise AD and cloud Azure AD objects, especially computer objects.

With cloud Kerberos trust, Azure is strongly linked to on-premise infrastructure through the use of a dummy Read Only Domain Controller (RODC) object. This Azure AD Kerberos Server delegates the ability to issue Kerberos tickets to Azure AD. With this link in place, the user and device can authenticate against Azure resources without any line-of-sight to on-premise resources.

## Deployment

A huge advantage to the cloud Kerberos trust is simplicity of deployment. If Windows 10 clients are at least 21H2, activating cloud Kerberos trust results from [deploying a server with the Azure AD Kerberos agent installed](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-security-key-on-premises#install-the-azure-ad-kerberos-powershell-module) and [enabling the appropriate WHFB Group Policy or Intune policy](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-cloud-kerberos-trust?tabs=gpo#configure-windows-hello-for-business-policy).

Then as long as a user can achieve line-of-sight to a domain controller, they will enroll a PIN (and other biometrics if they're allowed through WHFB policy) and are then enabled for WHFB login. Accessing on-prem AD-secured resources still requires line-of-sight to a DC, though it's likely enabled already.

Given the limited pre-requisites, I believe WHFB cloud Kerberos trust can help many organizations launch their own 'passwordless' initiatives without large infrastructure changes. This approach might also improve cloud security through the use of conditional access controls that could require the use of a strong WHFB credential as opposed to a password/MFA combination.

## Is it MFA?

Recalling the discussion on how *terrible* passwords function as authenticators, in my opinion WHFB cloud trust does constitute multifactor authentication. A protected, non-exportable private key that has to be actively unlocked by a gesture, PIN, or other biometric certainly qualifies. Some auditors might balk that the 'what you have' is included in the device accessing the requested resource, I'd argue the vast increase in credential strength offsets this more minor threat models.

Unless an organization's individual users are targeted physically for theft or non-trivial attacks (like SIM swapping), WHFB cloud Kerberos trust definitely improves resistance to phishing and credential theft to the level of more traditional MFA.
