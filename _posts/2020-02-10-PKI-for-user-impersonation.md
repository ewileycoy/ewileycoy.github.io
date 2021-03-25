---
layout: default
title: Smart cards for User Impersonation
By: Karl
---
# Using things the wrong way
As a professional services firm, every hour of downtime for a user costs money, so anything IT can do to reduce user disruption means (potentially) more billable hours for the firm. New hardware means a long tedious migration for the end user (the reasons thereof are beyond this particular article)

For this project, the migration process could take 8-10 hours depending on mailbox size, user profile issues, and giving time for QA. Our IT manager, being security-conscious reached out to me to discuss how to make this least disruptive. In the past, IT just had the user write-down their password so the admin could back up files, perform the upgrade or migrate to new hardware, and run QA checks on the new system to ensure everything works. His first thought was to have the user reset their password before and after the migration.

After a little discussion we agreed that this was not all that secure, since it’s still a shared password (not good) and not actually very helpful since it’s disruptive to the user (very not good).

After looking for other solutions briefly, I decided this is a singular use-case for PKI. Currently we use smart cards for some administrator accounts, but we’re looking to expand to Windows Hello for Business some time in the future, so our infrastructure is in-place.

So I struck on the idea that we (carefully) issue smart card login certificates to administrators that spoof the end user.

Now there are a number of reasons why this is a terrible idea, but very few seemed worse than a shared password (very bad) or password escrow (complicated and still not great since the password is exposed).

## Requirements
There were a few major areas I wanted to focus on in this scheme—since if it works out, we’ll definitely re-use it in other situations.

1. Token security and admin responsibility
2. Issuance and use auditing of the certificates
3. Limitation of risk if things went badly (e.g. a token is stolen or mis-used)

## Setup
We currently utilize Windows 2012 R2 Active Directory  Certificate Services with an Active Directory integrated CA. This is not a tutorial on setting up this infrastructure, it’s got to be done with care and purpose so I’d recommend reading up before setting up your own CA/PKI.

Windows logins with smart cards require a few pre-conditions, like Domain Controller certificates and domain configuration, but work reasonably well out of the box.

The tokens themselves needed to be easy to use and hold a number of certificates. I’ve used the [Taglio PIVKey T600 USB token](https://pivkey.com/) in the past and it seemed like a good choice for this project. USB form factor and large key storage were my main targets.

The Taglio token can hold up to 30 certificates using their minidriver. They’re relatively inexpensive, have a standard USB form-factor, and are readily available through Amazon. By default though, Windows does not see all the certificates and won’t work correctly if you don’t take specific steps to map the certificates. Fortunately the PIVKey minidriver is pretty simple to distribute as an MSI file to windows clients.

## PKI Token security
The Taglio tokens are not the most flexible smart cards out there. The Java Card software they run comes with some default limitations around PIN requirements and lockout thresholds that cannot be changed. The tokens have a pre-set all “0”s pin and admin key, so you must initialize them prior to issuance to ensure the tokens are secure.

I recommend using Versasec’s vSEC:CMS tool to set the admin key to a random value and securely storing that somewhere. It’s a nice advantage to be able to un-block the card in the event of a PIN lockout.

Be sure to delete the default self-signed certificate on the token while you’re initializing them since you’ll be issuing your own.

Older versions of the Taglio command-line admin tools treat the admin key differently than vSEC so only use one or the other when setting or recovering the admin key! Worse yet, the admin key has no way to reset so blocking the admin key essentially makes PIN reset impossible.

The Javacard operating system does not support changing the PIN policy, so you’re stuck with 6-8 characters and 10 tries until lockout. For me this is acceptable due to the manual issuance and limited use of the tokens.

## Issuance Restrictions and Auditing
Since Windows 2008, the ADCS role has supported restricted enrollment agents. I highly recommend taking advantage of this; by default an enrollment agent can enroll a smart card for *any* user (including domain admins). Limiting what users can have certificates issued on their behalf significantly limits risk.

Unfortunately as far as I can tell, the enrollment agent can’t be stopped from issuing certificates to themselves, so there is some separation of duties I can’t quite overcome. Fortunately the agents will be limited to a few trusted managers who hopefully understand the importance of proper policy.

Auditing AD Certificate Services is built-in to the Windows role, but must be activated both in Group Policy (the Advanced Audit settings must audit object access for Certification Services) AND the CA snap-in must be configured to enable auditing of specific certificate events. These events then make their way to the Windows Security event log for easy audit.

Windows event 4887 logs the Subject and Requester so we can track who issued a certificate and when. To make sure your templates are being used the CA issues event ID [???] indicating it loaded a template for the request.

Since ADCS lacks a remote issuance capability for signature-required requests[^1], I can force all issuance to be in-person.

[^1]:Ok this is not strictly true, you can use certutil to create and send CSRs, sign requests, and issue the certificates remotely to a smart card, but it’s an enormous pain in the ass.

## Risk limitation
How can I limit the consequences and chances of misuse, abuse, or exploitation of this scheme?

* **Certificate lifetimes**: I very much wanted this to be an ephemeral solution that self-cancels, which PKI lends itself to nicely. I created policies on certificate templates that specify expiration time in just 3 days to give admins time to use the logins but not take so long that there’s a risk of continuing access. Also, make sure to uncheck ‘publish certificate in AD’ on the template. Since these are temp certificates there’s no need to publish them.

* **Revocation**: We do have the ability to revoke the certificate from the CA to disable it. This requires forcing CRL checking in Windows login [citation needed] so the certificate can’t be used if it’s revoked. Also a robust configuration using published CRL and OSCP helps close the gap quickly, since these revocation lists are not published in real-time.

* **Token loss**: because the certificates are short-term, I’m not as concerned with loss and mis-use by external parties, but we do have the option of revocation. One blind spot that I’d like to improve is creating an automatic correlation of issued certificate to physical token, so I can positively revoke all the certificates on a lost token.

* **Collusion and mis-use of certificates**: I’ve deliberately separated roles of issuers and users, so the admins themselves cannot issue certificates. Frequent audits of certificate issuance and spot-checking against the schedule of use (or a ticketing system) will help reveal mis-use by administrators.

* **Mis-use of smart card logins**: Windows event 4768 indicates when smart card interactive logins against domain controllers occur. When a smart card  is used for login, this event contains certificate information (CertIssuerName, CertThumbprint, etc). Correlating between authorized activity and these login events should show when unauthorized logins occur.

## Limitations
Smart Cards have several drawbacks:
* **Password-only integrations do not work**. Some systems require passwords for authentication, and while we’re working to eliminate those one or two still exist and can’t be tested without the user’s password. Fortunately most services are Windows-integrated so Kerberos allows the login and doesn’t usually care if it’s password or smart card-based.
* **Remote access support sessions are more complicated**. Smart card passthrough only works in limited situations, like Remote Desktop, but several remote admin tools do not pass-through the smart card. This is a limited issue at the moment, since the cards are intended for local logins.
* **Offline logins may not work** (i.e. cached credentials) for smart cards since Windows tries to check the CRL or OCSP location to ensure the certificate is not compromised. Because our internal CA is not exposed to the Internet, a workstation needs to establish VPN if it’s off premises.

# Conclusion
Hopefully this illustrates a slightly different use-case from the usual PKI/smart card login situation. It’s not typical to want to ‘spoof’ users in most situations, but with proper controls and management approval we achieve a better outcome for the end users.

This also seems like an awful lot of work just to prevent a little user disruption, but when the average bill rate is upwards of 250$ per hour, saving even 30 minutes per user due to password disruption for 1200 users is saving the firm potentially 150,000$ for just one project. The token cost and time spent setting this up is trivial compared to that for just one migration project. In addition we can re-use this use-case to help speed other admin tasks while maintaining “do not share your password” and “keep user credentials secure” goals.

Personally I try to consider PKI solutions whenever possible, using them in as many situations as possible both improves my knowledge but also encourages vendors and other administrators to use them as well.