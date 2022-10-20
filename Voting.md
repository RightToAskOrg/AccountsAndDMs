# Discussion on EG and Voting Related Issues

## Balancing Votes
There is likely to be more upvotes than downvotes because downvotes are only from explicit dismissals. There has been some discussion on increasing the votes from just Up and Down, to Up2, Up1, Down1, Down2. What they mean remains to be decided, but it could be as follows:
* Up2 - Upvote having read the contents of the question, i.e. some additional engagement has taken place - e.g. retweet on twitter
* Up1 - Upvote from question alone, e.g. a like on twitter
* Down1 - Scrolled past, no engagement
* Down2 - Explicitly dismissed the question

Everything except Down1 requires an explicit user action. Down1 would be automatic, i.e., if you scroll past the question with no interaction an automatic Down1 vote is cast. The challenge with this is that the user may want to come back and upvote that question later, so how can we reverse the previous vote?

### Inverted Vote Submission
If, when submitting a vote, the inverse of the vote is also submitted to the Bulletin Board it will be possible to invert a vote if someone changes their mind later. A submission therefore contains Vote and InverseVote. Only the vote is counted at this point. If a user subsequently replaces their vote, or changes it, then they just make a new submission, again with a Vote and InverseVote. The way this is tallied is that all VOtes and InverseVotes are tallied excluding the last InverseVote, effectively cancelling all previous vote in favour of the most recent. This would require a zero knowledge proof that the Vote and InverseVote sum to zero, but that should be fairly straightforward.

### Vote Sampling
Instead of tallying all votes a random sample of the votes is taken and then the full result extrapolated from it. In this way, individual privacy is protected, assuming that there are at least 2 votes. To hide which votes were counted a vector of 1's or 0's would need to be constructed and mixed so as to hide, even from the server, which votes were included.

## Pagination Attack
Dismissed questions are stored locally, not by the server, with the dismissals applied locally when viewing the file. This presents a synchronisation challenge, but also risks creating a timing based pagination attack in the future. Although how serious such an attack is remains to be seen.

Currently there is no pagination, the entire file of questions is downloaded, so there is no pagination attack. Although it is important that questions cannot contain external content, i.e. images or tracking pixels, otherwise the server or a third-party will be able to detect that a question has been shown. Again this is probably less of an issue in the App, unless remote content is loaded by labels/components automatically.

In time the file of questions will become too big and will require pagination. i.e. download the next 10 questions. The problem with this is that the timing between requests for paginated content could reveal information about whether a user has dismissed a question. At the extreme, if the user has dismissed all questions in the page then the next pagination request will occur almost immediately, instead of taking the appropriate amount of time to scroll through the page of questions.


