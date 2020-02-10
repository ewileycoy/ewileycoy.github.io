---
layout: default
title: Smart cards for User Impersonation
By: Karl
---
So I had a crazy idea...
A project at work from the IT department caught my attention: replacing hardware and migrating user data with the absolute least disruption to the end-user.
Fortunately my IT manager colleague was thinking security-first and asked me how best to handle user credentials during their migration. The migration process could take 8-10 hours depending on mailbox size, user profile issues, and giving time for QA. His first thought was to reset user passwords for the day and have the user reset it at the end, however this would cause disruption with a variety of systems tied to their password.

After a little discussion we agreed that this was not all that secure, since it’s still a shared password (not good) and not actually very helpful since it’s disruptive to the user (very not good). There are also certificate escrow devices that the end user can type their password into, then it replays to a device. Unfortunately there’s no way to tell if someone just steals the password by typing it into notepad, for example. Also I don’t want to have to bother the end-user and habituate them to typing their password into random devices.

Casting about for solutions, I remembered we have an enterprise PKI and smart card login capabilities. Currently we use smart cards for administrators, but we’re looking to expand to Windows Hello for Business some time in the future.

So I struck on the idea that we (carefully) issue login certificates to administrators that spoof the end user without resetting their password.
Now there are a number of reasons why this is a terrible idea, but very few seemed worse than a shared password (very bad) or password escrow (complicated and still not great since the password is exposed).

## Requirements
There were a few major areas I wanted to focus on in this scheme—since if it works out, we’ll definitely re-use it in other situations.

1. Token security and admin responsibility
2. Issuance and use auditing of the certificates
3. Limitation of risk if things went badly (e.g. a token is stolen or mis-used)

## Setup
Windows 2012 R2 Active Directory Certificate Services
Taglio PIVKey T600 USB tokens

Some notes:
- I selected the Taglio tokens because they can hold up to 30 certificates using their mini driver. They’re also relatively inexpensive, have a USB form-factor, and are readily available through Amazon.
- By default though, Windows does not see all the certificates and won’t work correctly if you don’t take specific steps to map the certificates.
- The minidriver is pretty simple to distribute as an MSI file to potential windows clients.


## PKI Token security
PKI tokens tend to be pretty secure by design. They have tamper-resistant circuitry and anti-hammering to block the PIN in the event of a brute-force attack. The Taglio tokens come with a pre-set all “0”s pin and admin key, so you must manage them prior to issuance to ensure the cards are secure.

I recommend using Versasec’s vSEC:CMS tool to set the admin key to a random value and securely storing that somewhere. It’s a nice advantage to be able to un-block the card in the event of a PIN lockout.

Importantly the Taglio admin tools treat the admin key differently than vSEC so do not alternate tools when setting or recovering the admin key!

The Javacard operating system does not support changing the PIN policy, so you’re stuck with 6 characters and 10 tries until lockout. For me this is perfectly acceptable since 

## Issuance Restrictions and Auditing

The enrollment agent certificate template specifies restrictions on which admins are permitted to issue certificates as enrollment agents. This way I can create a restricted delegate (e.g. managers) who can issue end-user certificates but not risk them issuing them for administrators.

Unfortunately the enrollment agent can’t be stopped from issuing themselves certificates so there is some separation of duties I can’t quite overcome. Fortunately the agents will be limited to a few trusted managers who hopefully understand the importance of proper policy.

Auditing AD Certificate Services is built-in to the Windows role, but must be activated both in Group Policy (the Advanced Audit settings must audit object access for Certification Services) AND the CA snap-in must be configured to enable auditing of specific certificate events. These events then make their way to the Windows Security event log for easy audit.

Since ADCS lacks a remote issuance capability for signature-required requests[^1], I can force all issuance to be in-person.

[^1]:Ok this is not strictly true, you can use certutil to create and send CSRs, sign requests, and issue the certificates remotely to a smart card, but it’s an enormous pain in the ass.

## Risk limitation
How can I limit the consequences and chances of misuse, abuse, or exploitation of this scheme?

I very much wanted this to be an ephemeral solution that self-cancels, which PKI lends itself to nicely. I created policies on certificate templates that specify expiration time in just 3 days to give admins time to use the logins but not take so long that there’s a risk of continuing access.

Token loss: because the certificates are short-term, I’m not as concerned with loss and mis-use by external parties. One blind spot that I’d like to improve is the correlation of issued certificate to physical token, so I can revoke all the certificates on a lost token.

Collusion and mis-use: I’ve deliberately separated roles of issuers and administrators, so the smart card users cannot issue certificates. Also frequent audits of certificate issuance and spot-checking against the schedule of use (or a ticketing system) will help reveal mis-use by administrators.

Mis-use of smart card logins: I use windows event 4768 to monitor for smart card interactive logins against domain controllers. If this event contains certificate information (CertIssuerName, CertThumbprint, etc) then a smart card login has occurred. Correlating between authorized activity and these login events should show when unauthorized logins occur.

## Limitations
Smart Cards have several drawbacks:
* Password-only systems do not work. Some systems require passwords for authentication, and while we’re working to eliminate those one or two still exist
* Remote access support sessions are more complicated. Passthrough only works in limited situations, like Remote Desktop, but several remote admin tools do not pass-through the smart card. This is a limited issue at the moment, since the cards are intended for local logins.
* Requiring smart card authentication can break stuff, so don’t set it unless you’ve extensively tested in your environment. The 