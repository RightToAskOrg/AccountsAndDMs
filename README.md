# AccountsAndDMs
Cryptographic design for authentication and direct messaging

VT sketch notes based on Thursday conversation - edits welcome.

It's prob fine to leave DMs to 2.0, but it'd be nice to have a clear plan so we can set up the keys appropriately when we need them.

- Android enclave doesn't do ED25519. (Does iPhone?)

- Signal uses ED25519 to sign initial msg of ephemeral exchange. No permanent public encryption key; just permanent signing key.

- Potentially best tradeoff for login with spam protection is that people either use 3rd Party OAUTH (Google/Twitter/Github/etc) or give their email to us and we send them a link or PIN. This is nice because there will be some who, for privacy reasons, refuse to use Google and others who, for privacy reasons, don't want to send their email address to us.

- Still no answer from MS re keystore reliability and whether they accidentally have repeated Google's accidental-deletion in the case of temporary unavailablility error. I think we should ask the question on a public forum.

- CJC will look at exactly how Signal deals with multiple devices. Looks like you can't have more than one phone because it's all about phone number.

- We'd need to think about  (a) key refresh/recovery and (b) multiple current keys for one account.

Chris suggests using an extra layer of indirection for uid. Have a 'handle' that people make up themselves, and which has to be unique, and have a 'real' uid that's basically hidden from public view and which can't change, and is used on the server. Then people can change their handles AND their displaynames. Handles would have to be unique, but could be mutable; display names not necessarily. uids both unique and immutable.
