---
layout: default
title: A Measured Response?
By: Karl
---
*I’ve updated this a bit to tone it down, but still there might be bad words below*

[This pile of words from The Hill “opinion writer” and professor of “Information Technology Practice” needs more responses.](https://thehill.com/opinion/technology/553891-our-cybersecurity-industry-best-practices-keep-allowing-breaches)

I am going to give my take on this..thing. Don’t think that I’m just defensive because I work in the industry. Tropes like this place the blame in the wrong places.

> The hacking at Colonial Pipeline is the latest in a series of breaches that have impacted a long-and-growing list of other businesses — all ambushed by some individual or group that managed to hack through cyber security “industry best practices.”

First off, there’s no evidence that Colonial was following “industry best practices”. Quite the opposite, [according to their bizarrely-public auditor](https://apnews.com/article/va-state-wire-technology-business-1f06c091c492c1630471d29a9cf6529d).

> It is only getting worse. Reports surface daily about new incidents involving prominent health care providers, government agencies or retailers hit by hackers — thus releasing millions or billions of pieces of sensitive information all over the dark web.
> Guarding over these critical resources, health care providers and government agencies are a veritable army of information security professionals. 
> They sport impressive credentials and certifications like Certified Information Systems Security Professional (CISSP) and Certified Information System Auditor (CISA). Many even have academic credentials, including bachelors, master and doctoral degrees in information security. All of them embrace the latest "industry best practices."

“Impressive credentials”. So ya know if you’re hiring people just because of their credentials you’re failing.

> These impressively credentialed professionals are skilled in the art of tedium. They know all about audits. They can absolutely push paper. 
> They can painfully examine endless lists of accounts and identify exactly who does, and who does not, need system or service access. They can write impressive 100-page missives justifying a proposed new password policy.

I’m sorry, this is so *ad hominem* I’m embarrassed for this “professor”. 

> They can argue with developers as to why their job really does need to be more difficult.

Good. Developers having an ‘easy’ job typically means they can develop insecure software because they don’t want to incorporate secure practices.

> And for when their security fortress breaks down? They can eventually come up with someone to blame. They can explain what the unaware user, whose computer was exploited in a way the user can't understand, did wrong. They can identify and blame "the vendor" of a piece of equipment for a malfunction.

This is __actually__ a good point that’s buried in a bullshit rant about CISSPs. Users should be able to click links in emails and not get in too much trouble. Security admins should be able to let people do their jobs without worrying about compromising a network.

This isn’t the point he’s making—that security controls should cooperate with *users* and enable *users* to get their jobs done safely. Prevention should be the focus of user-facing controls. Detection and remediation should operate when they fail. The rant he’s on is that *administrators* don’t have sufficient access. This is utterly ridiculous.

> So with all of these impressively credentialed experts, we should be getting better at this "information security" business, right? So, what is wrong?
> The core problem is that "industry best practices" are not.

You are confusing “industry best practice” with “industry *de facto* practice”.

> Not only are "industry best practices" not "best" practices, but they are also dangerous practices
> Industry best practices," for instance, dictate that network administrators should be boxed in administratively. They should not be able to see what is happening on workstations, servers or storage resources. Server administrators, likewise, should be administratively restricted from being able to monitor network information or anything else that is not directly related to one specific niche job function.

Visibility and administration are TWO DIFFERENT THINGS. If you think you need full admin rights to monitor things, you have no idea what you’re talking about and should be ignored. It’s perfectly possible to give someone full access to logs and not need admin rights to investigate anamolies.

I agree that visibility is absolutely essential for a security program. I disagree that you need admin rights to accomplish it. Ransomware (and most other breaches) take advantage of wide-ranging unconstrained admin access to spread quickly. “Boxing-in” administrators means you have just a little bit more time to slow down an attacker before they take over your whole network.

> These practices limit the opportunity for a technically skilled employee to identify anomalies — a key sign that someone may have breached security and be roaming around preparing to launch the next big cyber attack. 

No they do not.

> A network engineer, for instance, does not have the tools or access to investigate the activity occurring on an innocuous sales department workstation at 3 a.m. A server administrator lacks the access to explore why the network throughput seems painfully slow while trying to copy files.

Is this the network engineer’s job to investigate workstations? Are they even looking for anomalies or did they just get a call from a sales guy at 3am? Is it a sysadmin’s job to monitor the network for security anomalies?

Having a good cross-functional incident response team with representation from all of your IT organization is a good idea. Training and running table tops on your IR plans with IT operations is highly recommended by “best practice”. Train them to engage the IR process if they notice something bad going on!

> The "good guys" are administratively prevented from having a holistic view of systems, networks, applications, workstations and other resources — when this holistic view is exactly what is needed to prevent cyber attacks.
> It seems the only person with a truly holistic view of a corporate network and data resources is the hacker. Unfortunately, hackers tend to not comply with corporate information security policy. 

If that’s the case then you are not following “industry best practice” of having central logging and analysis and a solid incident response plan. VISIBILITY != ADMIN RIGHTS.

> What can businesses and industries do right now? 

Patch. Grant least privilege. Get central logging and analysis setup. Develop an IRP. Engage an MSSP if you don’t want to hire in-house security people.

> Implement a "one strike and you are out" hiring policy for information security employees. When they fail, do not let it happen twice.

Wait what?

> Also, never hire an information security employee who has ever worked for a firm that has had a security incident. Their "industry best practices" did not work for the previous employer, why would they work better for the next victim? These former employees bring disaster. 

What the hell? There are two types of firms. Those that have had breaches and those that don’t know that they’ve been breached.

Also, firms have security incidents for all sorts of reasons. [Blaming security employees when management fails to properly address underlying security issues is a well-known blame-shifting tactic](https://www.theverge.com/2017/10/3/16410806/equifax-ceo-blame-breach-patch-congress-testimony).

> As far as "industry best practices," try going against the grain. Return to the practices that were in place before ransomware, breaches and other information security disasters became commonplace.

[Ransomware existed in the 1980s](https://www.vice.com/en/article/nzpwe7/the-worlds-first-ransomware-came-on-a-floppy-disk-in-1989), sorry bub. 

Also the threats and motivations of attackers has radically changed. Data and IT systems are much easier to monetize now either directly (e.g. stealing credit card numbers or committing wire fraud) or indirectly through extortion. Almost every company is under some mandate not to get breached, or at the very least take their systems back under control from an attacker.

> Embrace "holistic" approaches to information security.

Oh boy here we go. [Maybe if we diluted our antivirus hundreds of times by only having one installation in a thousand, we’d have better security results!](https://en.wikipedia.org/wiki/Homeopathy)

> Instead of impressively credentialed, paper-savvy information security professionals, hire competent technically skilled professionals. Encourage collaboration with other technically skilled professionals and give them the tools and access to protect your firm's cyber resources. 

I can’t really argue with this in concept. There are [plenty of charlatans](https://attrition.org/errata/charlatan/) in the industry, but if I were interviewing a security professional and they used “holistic” and claimed that you need admin access to investigate anomalies, they’d be shown the door.

You can’t just hire technical people, though. You need processes and technology to support them. The “boring paper-pushing” exercises like auditing your systems. Even the smartest people forget things, or neglect to finish a small part of a project that could have disastrous consequences if not caught by some kind of QA process. 

> Grant network engineers administrative access to the server cluster. Grant developers access such that network or workstation anomalies can be fully investigated. 

So uh, who administers server clusters in this scenario? The admins should be the ones who have admin access, no one else. What do you expect the network engineer to *do* to a cluster they don’t own?

Are you paying developers to investigate security issues or to develop software? Also investigating anomalies does not mean you need to grant everyone admin rights everywhere. Wake up the admins if you think their cluster is compromised.

> The security approaches that existed before "industry best practices" really do work. Ask the next hacker who breaches security.

Uh what? Security approaches like forcing password changes every 90 days? Leaving data unencrypted on the Internet? Security is empirically better than it was decades ago, but because the Internet connects everything, security flaws are just more obvious and more available to exploit.

> We can only hope the information security industry will undergo a renaissance. Until we realize that "industry best practices" continue to enable legions of hackers, we are doomed to more disruption. 

Hi, wait hello, is this the end of the article? Like where are your ACTUAL SUGGESTIONS for making things better? Other than whining about lack of access to things, I’ve not heard a single constructive suggestion from this guy other than “hire smart people and give them domain admin”.

This entire article is a screed on how the “good guys” (developers and network engineers?) somehow know more about security than people whose livelihood depends on it. “Don’t hire security people who’ve been breached” Well according to this guy just hire a bunch of network engineers and devs and call it a day!

Finally, the role of business management seems to be entirely ignored. Management remains ultimately responsible for their firm’s security. Hiring the wrong people, getting the wrong advice, or using the wrong technology are all business decisions. I’m not saying the security industry is blameless or doesn’t lie about capabilities, but blaming “best practice” on charlatan vendors misses the point by a mile.

We need better adoption of real “best practice” like the [CIS controls](https://www.cisecurity.org/controls/cis-controls-list/). Honest and thorough adoption of well-understood security controls can work! There are a lot of motivated criminals out there, why don’t 100% of all business suffer from ransomware all the time? Because security controls work when applied properly.

Are they perfect? Of course not. Should you rip the seat belts out of your car because they’re restricting your motion? Probably not!