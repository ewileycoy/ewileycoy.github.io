---
layout: default
title: (Windows) Hello, is it me you’re looking for?
By: Karl
---
![Lionel Richie says Hello](/images/hello.PNG)

Microsoft has an irritating habit of naming products in confusing ways. Skype vs. Skype for Business; Forefront products became Defender products; Zune became... [They’ve been doing this forever](https://www.techrepublic.com/blog/windows-and-office/earth-to-microsoft-please-stop-changing-your-product-names/)

They don’t stop with OS components either: Windows Hello! is theoretically an authentication mechanism or suite of controls around [enabling Microsoft’s password-less strategy.](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-deployment) I love the idea of getting rid of passwords! They’re frequently the cause of breaches and I’ve personally seen them play a key role in red team wins, pentest failures, and account compromises. The bolt-on of multi factor/multi step/2FA doesn’t always fix the problem since there are ways to side-step or intercept the second factor.

This Tweet:
<div class="center"><blockquote class="twitter-tweet"><p lang="en" dir="ltr">Did they tell you &quot;no one will know your password if you do not type it&quot;? They lied.<br>Winlogon.exe will know it anyway. 😈 <a href="https://t.co/ZL14VZWhcc">pic.twitter.com/ZL14VZWhcc</a></p>&mdash; Grzegorz Tworek (@0gtweet) <a href="https://twitter.com/0gtweet/status/1382188338870910977?ref_src=twsrc%5Etfw">April 14, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

generated some discussion that made me realize just how confusing (and bad for security) Hello can be.

Here is the skinny:

**Windows Hello** is a set of features built-in to Windows 10 called “convenience unlock” which stores a copy of your password in memory in order to unlock your device with a picture, PIN, or biometric. You still have to log in with a password! This password is *not* stored very securely[^Ok it’s better than storing your password in the registry, but not by much]. This feature is similar to primitive Android gesture unlocks and I assume was targeted mostly at mobile Windows.

**Windows Hello for Business (WHFB)**, formerly Passport for Business, formerly .Net Passport for Business, is an actual security framework that enables key-based (i.e FIDO2) or certificate-based authentication combining a user account and a device to provide true multi factor authentication without the need for a password[^this is actually a lie, but I’ll talk about that later]

WHFB requires an Azure AD or Active Directory on-premises infrastructure. Both the device and user must be joined in some way to the directory (this is where the ‘trust’ comes from). The user registers on the device using a multi factor prompt and the directory issues a key to both the device and the user. This key can *optionally* be stored in a trusted platform module (TPM), if it’s available and meets the TPM 1.2 standard. The TPM-stored key provides genuine “multi factor cryptographic device” (as defined in NIST 800-63) and is a nice way to fulfill “unprivileged user multi factor network access” for those of you that have upcoming CMMC mandates.

The logon experience is somewhat similar to Hello: the device unlocks the key in several ways to provide authentication. PIN, which is default and required, biometrics (such as compliant cameras with heat sensors and fingerprint readers), or an associated Bluetooth device (unlock only) are some of the built-in options. When stored in the TPM, the device itself provides some hammering resistance and can even wipe the key material if too many failed authentications are tried.

+++

So here is the enormous and annoying caveat: you still have a password with WHFB. Until and unless you move your whole authentication infrastructure to Azure AD (or Microsoft makes massive changes to on-prem Active Directory), you are stuck with a legacy Windows account with all the associated steal-able password hashes.

You can check the user box “smart card required for interactive authentication” BUT this only applies to *interactive* logins, not network logins AND you need to have setup the enterprise certificate-trust model of WHFB.

I won’t go into the pitfalls of Windows network security, suffice it to say that Windows servers and networks still have problems that WHFB doesn’t fix (pass-the-hash, bronze/silver/gold tickets, etc). WHFB at least addresses multi factor during the initial login and finally makes smart cards/FIDO a viable option for regular system administrators to use.