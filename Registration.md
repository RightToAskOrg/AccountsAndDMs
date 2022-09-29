# User Account Types

The following user account types will be defined:
1. **Basic** - The initial account type, no registration is required but extremely limited in functionality
3. **Registered** - Intermediate account type that has no greater permissions, but has started the registration process by providing their email address
5. **Verified** - Verified account, their email account has been verified and they now have full standard user account privileges 
7. **Verified Secondary** - Extended standard user account that has been linked to an MP's account, either via email verification or direct approval TBD
9. **Verified Primary** - Verified as a parliamentary MP account via email

Accounts effectively have escalating permissions with type. i.e. Basic can view but that is it, registered is an internal only status to handle the transfer from registered through to verified. 


## Permissions
| Account Type | Read Posts | Create Posts  |  Upvote/Dismiss  |  Flag Posts  |  DM  | Authorise Delegate Account | Act as Delegate Account 
| ------------- |:-------------:| :-------------: |:-------------: |:-------------:|:-------------:| :-----:|:-----:|
| Basic      | :ballot_box_with_check: | :negative_squared_cross_mark: | :negative_squared_cross_mark: |:negative_squared_cross_mark: | :negative_squared_cross_mark: |  :negative_squared_cross_mark: |  :negative_squared_cross_mark: |
| Registered  | :ballot_box_with_check: | :negative_squared_cross_mark: | :negative_squared_cross_mark: |:ballot_box_with_check: | :negative_squared_cross_mark: |  :negative_squared_cross_mark: |  :negative_squared_cross_mark: |
| Verified | :ballot_box_with_check: | :ballot_box_with_check: | :ballot_box_with_check: |:ballot_box_with_check: | :ballot_box_with_check: |  :negative_squared_cross_mark: |  :negative_squared_cross_mark: |
| Verified Secondary | :ballot_box_with_check: | :ballot_box_with_check: | :ballot_box_with_check: |:ballot_box_with_check: | :ballot_box_with_check: |  :negative_squared_cross_mark: | :ballot_box_with_check: |
| Verified Primary |:ballot_box_with_check: | :ballot_box_with_check: |:ballot_box_with_check: |:ballot_box_with_check: | :ballot_box_with_check: | :ballot_box_with_check: | :negative_squared_cross_mark: |

## Pathway 
How a user moves from the initial **Basic** account through to verified will be a matter for UI design. One option is to request registration at the start to encourage engagement, but with the downside of increased friction. The alternative is to allow browsing but require registration before interaction (posting/voting/flagging). This allows rapid access but creates friction when trying to act for the first time. There is an argument for saying that when installing there is an expectation of registration/friction, when there might not be when first trying to post.

It might be that a middle ground is best, in which registration/verification is nudged during install/first-run, but that it is possible to by-pass and do later if desired.

When the app is installed and first started it defaults to the **Basic** state. This does not require registration, but it would make sense to perform the client side key generation at this point. In effect, the account only has read-only permissions. There is a debate to be had as to whether it should be allowed to flag posts, my feeling is no, since that could allow false flagging with no control options.

How the registration state is nudged/enforced is a matter for UI design. I would suggest nudging during first-run and then enforced on interaction through a dialog. Luckily registration is extremely simple, it just requires providing an email address. Consideration should be given for allowing the user to complete their action before verification, for example, allow them to finish writing their post/submit it, and then hold it for 24 hours until they have completed verification. If this is too difficult then verification will need to be completed prior to completion of their actions. The app should gracefully handle verification.

The Registered Account has no additional permissions, it is only there to distinguish between an account that is awaiting verification having provided an email address and an account that hasn't provided an email address yet. In the registered state an account should be able to resubmit an alternative email address. In doing so any previous verification emails should become invalid. Some limit on the number of tries could also be considered.

One the verification link is clicked, either in the app or directly on the website, the full account functionality is available.

### Handling Verification
Email verification can be done via an email link that needs to be clicked to approve the registration. Some consideration should be given for using [App Links](https://developer.android.com/studio/write/app-link-indexing) on Android to allow the verification to be performed using the app rather than the browser. Either way, the UI needs to gracefully handle the verification. Including:
* Remember where the user was and what they were doing
  * Remembering that the user is likely to switch away from the app to their email client so restore needs to be correctly implemented
* Provide either a Push notification to the app so that it automatically moves on from the verification challenge or a button that allows the user to trigger a check that verification has been completed and move on 
* Some form of UI symbol should be shown within the app to signify verified status
* For Verified Secondary the process is probably no different, I don't think we can require a parliament address for delegated assistant accounts as the staff may be private staff in the constituency. 
* For Verified Primary accounts enforcement of registered MPs email addresses needs to in place

### Delegating to an Account
A Verified Primary account can delegate to a Verified Secondary account through some form of approval process, mostly likely digitally signing a request. It is probably best that such requests come from the Secondary account so there isn't a burden on the MP to perform too many management functions. Once delegated to the Verified Secondary account should have similar or possible the same functionality as the primary. Currently it does not have permissions to delegate to further secondary accounts, which seems like a reasonable limitation.

**Open Questions**
* What permissions should Primary and Secondary accounts have in terms of voting and posting? Are they equivalent to normal accounts, or are they only permitted to respond?
* Should delegate accounts show they are delegate accounts, i.e. Alice on Behalf of MP Bob when being viewed, possibly with some form of icon to show they are a primary/secondary account? (Side Note: Icons should not be permitted within usernames to avoid impersonation, this is somewhat challenging as you can't block unicode characters because some names need the extended character set.)

### Account Moderation
Accounts can be placed into a moderation state by an admin if for example they have posted a series of flagged posts. This could be automatic and would just constitute a pre-moderation setting. For example, if someone has three posts flagged by three different other users that account is put into pre-mod status. The user is free to continue using the site, but their posts will be queued until they are manually moderated.

The workload from this should be fairly minimal, in that someone who has posted three items that are truly flagged is likely to have their account banned in any case, so any further posts don't need moderation.

The moderation flag should be stored separate to the account status flag so accounts can move into and out of moderation without resetting their verification status. 

Accounts that are banned should be kept but have a moderation flag that signifies they are banned and cannot log in. This not only helps with handling appeals, i.e. the account is never truly deleted so any mistakes can be undone, but would also act as a way of blocking email addresses. I.e. an email address on a banned account cannot be used to register a new account. One word of caution on this, the automatic ban of the email address should only happen if the account was verified. Otherwise someone could effectively get someone else's email address banned by using their email address and then getting banned. 

A list of banned email addresses should be kept in addition to any automatic list. This should be a regex based list, since it will allow more flexible blocking, for example, email addresses from Russia or from particular domains. Consideration should be given to blocking temporary use email accounts like mailinator's free disposable email addresses.

### User Account Structure
* User Name (free text, no emoji, unique)
* ID - internal random userID - not changeable
* AccountType - int (basic, registered, verified etc)
* ModStatus - default 0 (1 premod, 2 banned)


Username needs to be unique, so people can for example establish a DM exchange using username as an index to the internal ID. It is likely that changing the username will be a requirement, although hopefully a rare one. The question is how to handle that:
* Should usernames be available for reuse if someone no longer uses it, i.e. they changed their username, does their old username become available for use?
* If old usernames are made available should there be a time delay before they are available for use? If so, how long?
* If not, should all usernames associated with an account be permanently associated with it, i.e. if someone tries to initiate a DM key exchange with an old username should it work? I can see problems with this in that it would reveal old usernames. That might be an issue depending on why the username was changed. Let's imagine it was an offensive username or at least accidently offensive, history of that might what to be forgotten. Alternatively are old usernames simply retired?
* If retiring or permanently associating usernames should there be a limit on the number of changes? I.e. 3 changes to prevent username squatting.

If a user changes their email address their verification status is reset. They should be warned about this before starting to make the change.

Are email addresses unique? I.e. only one user per email address?

