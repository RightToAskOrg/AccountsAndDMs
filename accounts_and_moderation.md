# Accounts and Moderation

This document provides an initial consideration of the need for accounts, how they could be realised and the purposes they should serve. It will initially outline why there is a need for accounts and then establish the properties that would be desirable from an account, with consideration for potential attack vectors. 

## Actions to follow up
* Define an acceptable use policy
  * Including a staged suspension policy for accounts that are in breach
* Define a complaints procedure
  * Implement complaints handling back-end
* Determine moderation strategy
  * Implement moderation functionality


## Need for accounts
Unfortunately one of the few things you can guarantee with the internet is that if you allow people to post content, someone will post content they should not. Whether that is illegal content, harassment, spam, distribution of malware, or one of the many other possibilities, the ability to reach an audience will eventually be noticed by miscreants and subsequently targeted. As such, a completely open system is likely to become unmanageable quickly, and in turn may detract from the true purpose of the site and discourage genuine users. That said, the opposite is also a problem, if the creation of accounts is too arduous, it may cause too much friction and result in prospective users not engaging with the site. What is required is a minimum viable account system, whereby the minimum is determine by the moderation requirements of the site.

It should be noted that this method of determining the minimum viable account system is specific to this context. In other contexts there may be additional requirements, for example, in finance there will be Know Your Customer requirements, in online shopping there might be delivery or payment requirements. However, in this context the primary objective is moderation and as such will be the factor we use to determine the minimum level of account system.

## Moderation Requirements
Before deciding what level of moderation will be required some notion of the sites acceptable use policies will need to be established. Some things should be fairly obvious, i.e. posting illegal content is not permitted. However, in a free speech setting care must be taken in determining the exact wording of the policy. The policy needs to be sufficiently robust that it will permit the banning or exclusion of an individual from the site should they breach said policy. In particular, sections detailing harassment, foul language, threatening behaviour should be clearly defined and explained.

Once a policy is determined there are three core requirements for moderation to be effective
* Items in breach of policy can be quickly and easily removed
* Accounts deemed in breach are prevented from posting in the future, either permanently or temporarily
* Direct messaging can be flagged by the recipient

### Items in breach of policy can be quickly and easily removed
This may seem simple, but can be difficult to deliver as the site scales. There are a number of strategies that can be adopted:
* Pre-moderation
* Reactive Moderation
* Community Flagged Moderation
* Partially Automatic Community Flagged Moderation
* Community Driven Moderation

Whilst these strategies can be used in isolation, it is likely that a combination of different strategies may be more appropriate, possibly with distinct strategies used for different accounts based on past behaviours.

One common feature of them all is the ability to uniquely identify posts and the user who posted it. This exists in the current design due to the signing of posts. However, there is an overall assumption that accounts are largely restricted to one per person and that a single individual cannot have many accounts. If that were  not to occur these moderation strategies, with the exception of pre-moderation, are likely to fail due to the inability to attribute breaches to an individual. If an individual can have many accounts banning their accounts will have no impact and they will be able to spam and potentially overwhelm the site with relative ease.

#### **Pre-moderation**
The most extreme variant of moderation in which content is not posted until it has passed moderation. Such a moderation strategy takes the most time and therefore comes at the highest cost. Furthermore, unless content posting is relatively low in volume this approach will rapidly become infeasible as the site grows. As an ongoing site-wide strategy this should be avoided, however, consideration should be given to being able to apply such moderation to specific accounts, for examples, accounts with a certain number of past breaches could be placed in this state for a period of time to try to modify behaviour of the user without outright exclusion.

It should also be considered whether a site-wide setting is implemented for emergency use. One should consider what would happen if the site came under a sustained and co-ordinated attack that attempted to spam it. This could happen were there to be an oversight in some aspect of authentication or overall moderation strategy. Having the option to enable a site-wide pre-moderation flag could be a useful last resort to prevent the site becoming overwhelmed.

#### **Reactive Moderation**
Relies only on moderating content when someone files a complaint. A complaints procedure should be defined, with some consideration for appeal or right to response for the accused account. A decision as to whether to immediately hide a post that has received a complaint should be considered, and if implemented should be detailed in the acceptable use and complaints policy. Some limit on the number of upheld complaints that are permitted and the staged consequences should be made clear to users.

A Reactive Moderation strategy will likely be fairly lightweight, and would hopefully only be required in exceptional situations. However, there is a risk that some undesirable behaviour will go unreported and could lead to genuine users disengaging rather than filing complaints. The risk of this happening will increase the more arduous the complaints procedure is. However, if the complaints procedure is too easy, i.e. just clicking a button, there is a risk of an increase in spurious complaints and the corresponding workload associated with them. If an extremely easy complaints procedure is implemented this becomes more akin to one of the *Community Flagged* variants described below.

#### **Community Flagged Moderation**
A Community Flagged Moderation strategy relies on the community as a whole flagging content as potentially inappropriate. Flagging is intended to be simple and quick, often just a button press. Reactions to flagging can be adaptive, i.e. order items by the number of flags and address those items with the most flags first or above a pre-determined threshold (Note the exact threshold should not necessarily declared, but the usage of a threshold should be). 

The ease of flagging does run the risk that spurious flags will be raised, leading to time wasted reviewing and addressing them. It would be possible to have a consequence for flagging content that is subsequently deemed acceptable, but caution should be taken around applying such measures. If there is a perceived cost to flagging content genuine users may be more likely to not engage in the flagging process. This could result in genuine cases to be lost in the noise from any spurious or politically motivated flagging. 

#### **Partially Automatic Community Flagged Moderation**
This is adds some degree of automatic response to the Community Flagged Moderation through automatically hiding content or suspending an account when certain thresholds are reached. A right to appeal should be made available for those whose posts have been hidden or accounts suspended. In such a strategy the moderation is potentially largely automated, with moderators only getting involved in the most serious cases where additional complaints are filed, or where there are appeals against moderation. 

For example, a threshold of 10 flags could be defined, after which a post will be hidden. The user whose post was moderated should be notified with details of the appeal process - which should be relatively easy to complete, for example, a simple form with an textarea to explain why they believe the moderation to be incorrect.

This could be expanded to automatically suspend or place an account into pre-moderation once a certain number of posts have been moderated within a given period, say 3 months. As such, if a user has 3 posts moderated within a rolling 3 month period their account will enter pre-moderation until they drop back below the threshold, i.e. after a past indiscretion falls outside the 3 month period. This transfer to and from pre-moderation can be automatic, reducing workload. However, this would increase workload in terms of pre-moderation. 

Furthermore, there should be a further step available to moderators. For example, posts that fail pre-moderation should be added to the users count of moderated posts and at a certain threshold the account should be permanently or temporarily suspend outright. This is important to provide a limit on the amount of work a problem account can create in terms of moderation. 

#### **Community Driven Moderation**
Community Driven Moderation is similar to that found of sites such a Wikipedia, in which the moderation is largely performed by the assigned editors or moderators. Some form of community appeal process can be defined and the administrators of the site are largely hands-off except in exception circumstances. Whilst this provides the least work, and potentially scales with the site. It is dependent on having a sufficiently large enough community, with diverse opinions and backgrounds to provide a balanced moderation group. It is possible for administrators to lose control of moderation if community moderation is formally defined. In such scenarios a bias in the moderation team could lead to undesirable results. Heavy handed administrators overriding or interfering with moderation could also cause moderators to withdraw. As such, whilst this is a desirable end goal in terms of sustainability, it should only be evaluated once a sufficiently size community has established and the capability of the community to behave in a balanced and fair way has been established.

That said, if this is the long-term goal it may be worth considering how the moderation functionality is implemented in the back-end to create some notion of a distinction between a moderator and an administrator, even though initially they may be the same. This would facilitate easier transition to the this approach later on, without re-engineering the user system.

### Accounts deemed in breach are prevented from posting in the future, either permanently or temporarily
Fundamentally this is where the real challenge and trade-off begins. Ideally an account should be tied to an individual, and therefore synonymous with each other. Restricting an account should equate to restricting an individual. If it does not, then the individual can just create a new account and circumvent any restrictions. In doing so they render the moderation strategy considerably weaker, possibly undermining it entirely, and lead the site vulnerable to being overwhelmed in terms of moderation workload or inappropriate content. 

Establishing identity and linking it to an account is a challenge. At its most extreme it would involve using digital ID services to establish ID to a very high degree of certainty. However, such measures are likely to be deemed too expensive to operators and too invasive to users. Therefore a proxy for identity needs to be found.

#### **Proxies for Identity**
A proxy for identity needs to be something that is typically tied to just one individual, and for which there is some limit or cost in acquiring. No proxy is going to be perfect, there is likely to always be a way around any limits imposed by the proxy given enough time and money. Some possible proxies for identity:

* Email address
* Phone number
* Credit card
* Device ID

##### *Email Address*
Email address is one of the most common and obvious choices. Most email addresses are tied to an individual and many providers, including Google and Outlook, impose some degree of restriction on the number of email accounts a person can create. This is often through identity checking when opening a new account. However, there are plenty of anonymous mailbox services (e.g. mailinator) or someone could use their own domain. As such, some statistics and alerts should be set to detect multiple accounts coming from the same domain. That could be innocent but some oversight might be necessary. Blocking certain temporary email account services might also be a sensible limitation.

Consider some rate limiting based on domain. e.g. unlimited for outlook, gmail, .edu.au, .gov.au, but perhaps rate limit new accounts from other domains, or at least raise an alert if they happen. Put in place some options for detecting how many people are registering from the same domain, how many flags have been raised about them, how many questions they're posting, etc. One indication of trouble is posting lots of questions, or sending lots of DMs, but not upvoting. Or just rate limit the number of questions each account can ask. Maybe send admins an alert if rate limits are imposed. The rate limits should probably be public.

An email address can only work as a proxy if the email address is verified. Otherwise someone could use an email address that does not belong to them. Limiting functionality until the email address has been verified would be one way of addressing this requirement without adding too much additional friction. For example, DMs and posting can only be made from an account with a verified email address.

As a further note of caution, Gmail potentially allows an unlimited number of email addresses per user through the use of "+" addressing. This works by allowing anything to placed after the genuine address with a "+" symbol. For example, if the email account is joebloggs@gmail.com, then joebloggs+temp123@gmail.com, joebloggs+temp124@gmail.com and joebloggs+facebook@gmail.com are all legitimate aliases. As such, when checking for uniqueness of an email address it is important to consider such "+" addressing for Gmail and other providers.

#### *Phone Number*
Phone number, in particular a mobile phone number, which can be verified via sending a one-time code, provides a high degree of identity verification. This is particularly so in Australia where there are strict identity checks on mobile phones. Apps like Signal use phone number as an identifier for individuals. There is a cost involved in sending verification messages and there could be some hesitation from individuals to share their phone number with a website. 

#### *Credit Card*
Whilst a credit card is a strong identity check it is not practical for a setting which does not involve purchasing. 

#### *Device ID*
Given the mobile nature of the service, in that it is currently only accessible via an app, it would be possible to use a device identifier. For example, `Settings.Secure.ANDROID_ID` is a device/application specific identifier. It remains the same between uninstall and reinstall if the package stays the same and the same signing key is used on the APK. The ID will reset during a factory reset. If combined with Android SafetyNet it would be possible to ensure that the device is not an emulator and is a genuine Android device. This would prevent automated generation of fake devices. It would require Google Play Services to be installed, but so would Firebase Cloud Messaging, so that might already be a requirement. 

This may require an app permsission update if imposed later.

A further downside would be if future expansion includes a web based version there would be no equivalent ID value. It would also not assist in account recovery.
