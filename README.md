# AccountsAndDMs
Cryptographic design for authentication and direct messaging - very preliminary ideas and questions.


VT sketch notes based on Thursday conversation - edits welcome.

It's prob fine to leave DMs to 2.0, but it'd be nice to have a clear plan so we can set up the keys appropriately when we need them.

- Android enclave doesn't do ED25519. (Does iPhone?)

- Signal uses ED25519 to sign initial msg of ephemeral exchange. No permanent public encryption key; just permanent signing key.

- Potentially best tradeoff for login with spam protection is that people either use 3rd Party OAUTH (Google/Twitter/Github/etc) or give their email to us and we send them a link or PIN. This is nice because there will be some who, for privacy reasons, refuse to use Google and others who, for privacy reasons, don't want to send their email address to us.

- Still no answer from MS re keystore reliability and whether they accidentally have repeated Google's accidental-deletion in the case of temporary unavailablility error. I think we should ask the question on a public forum.

- CJC will look at exactly how Signal deals with multiple devices. Looks like you can't have more than one phone because it's all about phone number.

- We'd need to think about  (a) key refresh/recovery and (b) multiple current keys for one account.

Chris suggests using an extra layer of indirection for uid. Have a 'handle' that people make up themselves, and which has to be unique, and have a 'real' uid that's basically hidden from public view and which can't change, and is used on the server. Then people can change their handles AND their displaynames. Handles would have to be unique, but could be mutable; display names not necessarily. uids both unique and immutable. Possibly row number in the database - maybe this is already there by default?

## UIDs
The issue of UIDs can be a difficult one. Intuitively one can use a user provided UID, for example, a handle or email address. However, there is a general rule that any data provided by a user will at some point need to change, or at least the user will wish for it to change. Email address is an obvious one, as people move providers or employers and as such email is not suitable as a UID. Choosen screen names or handles can have a long lives, potentially for the entire rest of the persons life. However, there are numerous scenarios where a user may legitimately wish to change their screen name or handle. For example:
* Following marriage or divorce if they used their surname in their screen name/handle
* Change of work, i.e. a staffer for an MP may include the MP in their screen name/handle
* Mistakes during registration, i.e. typos or unexpected concatenation that result in misunderstanding. For example, Experts Exchange which it is claimed changed its domain to add a hyphen Experts-Exchange after it became apparent people were reading it as ExpertSexChange. Something similar is likely to happen with screen names/handles
* Moderation - if someone registers a name deemed inappropriate the moderation may occur after they have started posting/voting. Would that require their account to be deleted and all their posts? 

Using a server generated UID, for example, ROWID if it is built-in, or an auto-incrementing primary key is advisable. This ID can then be used by DMs to ensure that even if people change their screen name/handle/email address the messages are consistently sent to the same account.

### Uses of UID
When recording whether a user has voted rather than storing the public key directly it would be better to store the UID and possible a KeyIndex if multiple keys are supported. In this way, an immediate check can be performed as to whether a user is entitled to still vote on a question. This would ensure in a multi-device setting a user cannot vote multiple times from different devices. The same would be true for flags. 

Without using UID the public key will need to be checked against both the listed public key keys and then each account of each listed public key to ensure it is not from an account that has already voted. This would become prohibitively expensive as more people vote on a question. 

Public Keys within a user object should be set as either Active or Inactive. The server should not accept posts from Inactive keys, this allowing a user to disable future posts from a key on a lost or stolen device without invalidating previous posts. Ideally a user should be able to set a key as active to inactive and back to active, to handle temporary loss - although this would be a more advanced feature.

What if people lose their keys? Maybe post new keys and active/inactive requests on the BB. It's not actually clear that we win much by making verification check the timestamps against the pub key changes. If you're trusting the server, nothing should be accepted from old keys; if you're not trusting the server then it could have delayed, faked the replacement key, or otherwise messed around.

*** How are we going to sync upvote/dismiss votes across devices??
## Syncing upvote/dismiss data
The current design has upvote/dismiss data stored only locally. This presents a challenge in two scenarios:
1. Loss of device
2. Multiple devices linked to one account

The two challenges present fundamentally different problems. A loss of device can only be recovered if the data is stored off the device, i.e. backed-up somewhere else. Storing off-device presents a security problem; how can the data be encrypted in a way that prevents the storage provider, whether that be RightToAsk or someone else, from viewing the contents and breaking the privacy of the voting system. In any case an encryption key is going to be required. The difference between the two scenarios is how that encryption key is generated and used. Once an encryption key has been generated the encrypted data can be stored on the RightToAsk server. In the case of a lost device this would not strictly need to be on a RightToAsk server, it could use platform backup services instead. However, in the case of multiple devices it will need to be on RightToAsk servers because the data needs to be constantly synchronised between devices. This is relevant as some features may not be released immediately, and there may be a development path that doesn't require all the features to be implemented at the start. 

### How does signal do it?
Signal backs up configuration data using a combination of a PIN, a secure enclave on the server, and limit on the number of incorrect tries. Whilst a neat solution it is clearly beyond the scope of what would be possibly for RightToAsk at this time. (Secure Value Recovery)[https://signal.org/blog/secure-value-recovery/]

### Generate a Key from a PIN/Password
The simple approach is to generate a key from a PIN/Password using PBKDF, encrypt the configuration file and store it on the RightToAsk server. Whilst simple, the security of this is limited due to the relatively small amount of entropy. An offline attack from the server is going to be difficult to prevent, particularly in the case of a PIN. If a password were used it may be confusing to users to understand that the password is not recoverable and that if they lose it they will lose access to their backup/configuration. As such, whilst simple to implement, it may undermine the security goals to such an extent to make it undersirable.

### Password Usage
If a password is used as part of the normal registration process then this problem is resolved. There is also scope to use the password to generate both an authentication key/password as well as an encryption key. The authentication portion could be used for account access and recovery, whilst the password could be used for syncing as well as possible web access - see below. 

A password need not be stored directly on the server, it could processed locally, either to generate a PKDF a random string to store as a proxy for the password, or it could be used within a secure protocol, for example, [https://en.wikipedia.org/wiki/Secure_Remote_Password_protocol](https://en.wikipedia.org/wiki/Secure_Remote_Password_protocol), which seek to provide authentication without the server holding the secret. There are various protocols proposed, with varying support.

### Web Version
If a password is used it would be possible to create a web version of the app. In this approach a key pair could be locally encrypted using a key derived from the password locally and then stored on the server. When a user logs into the website the encrypted key is downloaded, decrypted and then used for submissions. This doesn't prevent the risk of brute force attacks against weak passwords, however, if it was combined with email based 2FA, and the key was marked as being only usable during 2FA approved sessions the risk of compromise is reduced. It is not eliminated entirely, because there would be a window for a corrupt server to make submissions during an approved session, but greatly reduces online attacks. 

### Loss of device
In the case of a lost device one option would be to rely on platform backups. For example, Android provides an (Auto Backup Feature)[https://developer.android.com/guide/topics/data/autobackup] which is claims is end-to-end encrypted using the PIN/Passphrase of the device. In this scenario provided that the files are stored in the app directory (which they should be) they will be backed-up and restored. However, this restore will only work if it is a Android-to-Android. i.e. if someone had an Android phone, lost it, then decided to switch to an iPhone there is no restoration path. Similarly, iOS provides platform back-ups or one could use CloudKit to manually configure cloud storage and recovery. Note, recovery might only be possible if iCloud backup has been enabled, otherwise the key required to recover the data might be lost.

A similar problem can occur on Android. If EncryptedSharedPreferences are used I believe the key is not backed up and therefore the contents are not recoverable. This is sometimes discussed in errors with EncryptedSharedPreferences where the app has to disable auto-backup in order to get it to work. 

However, in theory, the current setup would allow device recovery, assuming voting data is not stored in an EncryptedSharedPreferences/SecureStorage location on the device.

### Multiple devices linked to one account
This presents a fundamentally different problem as there is an ongoing synchronisation problem. However, by specifying this as a distinct problem it could be added as feature in the future and use a different methodology to the loss of device recovery. This functionality could be provided by requiring devices to encrypt their data files under the public keys of the other devices registered for their account and then storing that data on the RightToAsk server, so that each devices downloads the latest file. An easier approach might to provide this as an extension of the Direct Messaging functionality. Have a device send its updated configuration file to all other devices registered using the DM functionality, albeit with a flag to indicate it is a system message and therefore not shown to the user. This would save implementing additional crypto and would remove the synchronisation problem associated with multiple devices potentially updating their configuration files at the same time. In this method they would just be distinct messages and all the recipient needs to do is merge all the received files. (Merging should be fine as the operations are add only, i.e. the server won't allow a device to have both upvoted and dismissed a question - **note this means that the config file should only be appended after an operation is accepted by the server, so if two devices request different options at the same time only one will succeed**.

## Development Plan
* Rely on platform backups initially to allow recovery after device loss.
  * This should not require immediate changes unless EncryptedSharedPreferences are being used
  * Some verification of Apple iOS backup is required
* When implementing DMs provide functionality for sending encrypted system messages.
* When implementing Multiple Devices use the DM functionality to send configuration file changes after each update
  * Implement configuration file synchronisation
  * Multiple device implementation may also require signing/approval of new devices by existing devices prior to synchronisation

