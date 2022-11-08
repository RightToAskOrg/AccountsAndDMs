# Server State
There is a need to maintain some state on the server. Whilst local storage is beneficial to privacy, there is an issue when a user logs into a new device or adds a second device. In particular, the record of what questions a user has dismissed is not currently stored on the server and therefore would need to be synced between the devices or in the case of a new device, where the old device is no longer available, the state would be lost. 

This is not good from a user experience perspective, and may present some challenges with having to reject many double votes as the user dismisses questions again. Therefore some consideration needs to be given to how state can be maintained on the server in a manner that still maximises the users privacy. 

## What the server currently knows
* which questions you wrote, answered or otherwise edited (e.g. by
adding new tags),
* which questions you voted on, though not whether you up- or down-
voted,
* your electorates, if you made them part of your public profile,
* your badges, i.e. MP/staffer verification

### Question Flagging
Whilst not currently implemented there will be functionality to allow for a question to be flagged. In doing so the user will be informed that the fact they are flagging the question, and any reasons given for flagging the question, will be known to the server and RTA.

## Planned/Possible Local Information
Currently when a question is dismissed the record of its dismissal is stored locally. A dismissal vote may also be submitted to the server - depending on final voting setup. Future developments may also allow the pinning of questions so that the user can quickly see any updates.

@VJT - Should there be an option to register for notifications to receive push notifications when a question is updated in some way, similar to GitHub issues?
Ans: Excellent idea, prob a 2.0 Feature. Will ask UI experts for advice on how to do it.

Current profile information that is currently only stored locally is as follows:
* your electorates, if you found what they were but didn't make them part of your public profile,
* your list of up-voted questions
* your list of dismissed questions
* your saved/favourite searches (this isn't implemented yet but will be)

## Planned/Possible Server Information
When downvoting or dismissing a question an option could be given to allow the user to provide some feedback as to why they are downvoting of dismissing the question. Such information will need to be stored on the server to be off any use. How this implemented, or even if it is to be implemented, remains an open question.

One option would be to submit the information in the clear, with suitable warnings to the user that such information will be known by the server. An other option, particularly if only a limited number of options is available, would be to use the same voting system. i.e., select one of the following reasons for dismissing the question. Although it is likely this is over-the-top in terms of privacy protection, and that the average user will not mind sharing such information with the server in any case. Particularly since this will be optional.

@VJT - some thought may be required to the UX of this functionality. I would find it quite frustrating if the dismiss process required multiple clicks, i.e., dismiss followed by an option to either provide feedback or not. Even if the feedback option auto-hides if not interacted with it shouldn't break the flow of potentially dismissing multiple questions quickly. It could be that dismissing the question causes the existing space reserved for the question to be replaced with an option to provide feedback widget - note it needs to be the same size so as to not impact on scrolling. This presents a bit of a challenge for how to subsequently remove it, it could be that it is dismissed when scrolled off the screen. Caution should be used when removing a component at a time after the user swiped it away or dismissed it. The UI should not scroll under the user's finger, i.e. out of sync with user input.
Ans: CJC's suggestion: when a user swipes to dismiss, replace with a panel giving them the option to send feedback. If they ignore it once and it scrolls of the screen, it goes away. If they choose to send feedback, we give them some options. Maybe one of the options is 'report as inappropriate' in which case they get the list of reasons, or maybe if that's the way they feel we enourage them to flag instead. Note that we need to manage expectations - there's an expectation that flagged/reported questions will go to moderation, no such expectation with 'give feedback.' This is intended to give people meaningful feedback on the quality of their question. Probably a 2.0 feature.


## Moving Forward
The loss of some local information is not catastrophic, e.g., electorate or favourite/saved searches are frustrating to lose, but don't cause fundamental problems. Loss of dismissed votes is a problem as it will be extremely frustrating for the user. Some information can be inferred from the public information on the server, like the fact the user has voted on a question, even if the vote is not known. This could be used to prevent attempts a double voting, but wouldn't help in terms of knowing which questions to hide. Similarly, information about which questions have been flagged could be synced from the server to prevent them being shown. 

As such, there are three options currently on the table for how to approach the issues:
1. Set up the keys for, but don't actually implement, encrypted server-
side storage of all this. This would impact how the password handled, in particular, using a locally generated password key via a PBKDF function so the password can be used for authentication and encryption. Even though the encryption part would not be immediately implemented, it is necessary to prepare for it in how the password is implemented. As an MVP, and to handle the interim period, the server could tell the reinstalled app what the user has voted on (though it obviously doesn't know which way) and the other things it already knows. Newly installed apps wouldn't know what they'd up-voted, but at least they wouldn't try to clobber previous votes.
2. Same as (1), but with the 'tell us about this question...' feature.
3. Do the whole thing properly. Worth noting that the Apple universe probably makes it a lot easier because we can stick an AES key in their keychain 
    @VJT - this is true but that requires someone to stick to always Apple, which I know is common, but doesn't help someone going from Apple to Android or Android to Apple. I think the storage would need to always be independent and therefore on an RTA server.


