# E2E Direct Messaging

The E2E direct messaging will be based on Signal, specifically the signal library. Whilst documentation for this is limited, it is safer to use an existing implementation that attempt to re-implement the double-ratchet protocol. 

## Multi-Device
Multi-device presents some challenges as it is not a use-case that exists in any of the test classes or documentation and information provided on the forums is extremely limited. The Sesame protocol does not make explicit how multi-device should be handled, in particular, whether each device has its own IdentityKey - with some form of cross-signing, or whether each device shares the same IdentityKey, with the private key being transferred during linking.

It appears the Signal has opted for the [latter](https://community.signalusers.org/t/should-my-identity-key-be-the-same-across-all-my-devices/1170/9). It appears that a linked device generates a temporary key pair which is used to encrypt the private identity key and it is then sent to the new device. Although it could be that the send is actually a QRCode scan, which is not something we want. Once in possession of the IdentityKey the device can create its own PreKeyBundle, with a different DeviceId to the original and upload these to the server as normal. 

The current development server doesn't record deviceId, primarily because the deviceId is packaged within the address in the library. However, it clearly needs to be stored separately. This is because if Alice wants to send a message to Bob, and Bob has more than one device, Alice needs to be able to request all PreKeyBundles from the server for all of Bob's devices, and as such, the deviceId cannot be concatenated with the address. There isn't an example implementation of a server, so the data structures need to be interpreted from the code.

When sending a message the process seems to be get an instance of the session for each device and then encrypt the message for that device and send it. See [here](https://github.com/signalapp/Signal-Android/blob/7ffdf91ce524ec510884a015424328327043faef/libsignal/service/src/main/java/org/whispersystems/signalservice/api/SignalServiceMessageSender.java#L2182) for example code.

The code handling of multiple devices appears to be implemented in the library, although the calling of the appropriate methods to support it is an implementation task, but there are no concrete examples, only what can be inferred from the various client source code. However, the syncing functionality does not appear to be part of the library, and will require implementation. 

### Additional Syncing Functionality 
Sending a message to multiple devices is in some ways the easy part. The challenge comes in how to synchronise actions across the devices and signal to a sender when multiple devices need to be sent to. It is likely that some control messages will need to be defined to handle this functionality. Such messages will be sent silently in the background, but using the same encryption functionality. It might be that this functionality can be extended to allow generic synchronisation messages to be sent by the app, providing a safe way of synchronising between devices without having to rely on the server.

#### Message Syncing
Signal doesn't support syncing history, but it does synchronise messages across devices. The synchronising of incoming messages is effectively handle automatically as the sender sends to all devices. However, outgoing messages need to be manually synced. This is vital to keep chat history consistent across all devices. This also needs to be done in a timely fashion to ensure message ordering is preserved. Note, the messages don't have to be read by the device, they just need to be added into the queue on the server in a timely fashion. It's likely that bundling messages that all relate to the same plaintext message will be the best way to achieve consistency and efficiency. As such, the client should prepare all the encrypted messages:

* Messages to all target devices
* Control Messages to all linked devices


bundle these messages into a single data structure and send that to the server to be processed.

#### Contact Syncing
In addition to syncing messages contact may also need to be synced. This will be to both ensure consistent naming across devices, but also to handle past message threads to be continued, even if the message history is not. For example, if on Device A Alice had started a conversation with Bob and then subsequently linked Device B to her account, the previous messages would not be synced, but there should be an entry for Bob so that Alice doesn't need to rediscover the contact details again and check the security key.

#### How Many Devices?
How does Alice know how many devices Bob has registered if Bob has registered devices after Alice has established a session? According to the Sesame protocol the sender sends their messages in a bundle with the corresponding deviceIds and compares it to the known list on the server. If different it returns an error with new deviceIds and the client updates its records and tries again. This isn't necessarily the most efficient approach, since some of the encryptions could have been valid, but it is necessary to maintain consistency. If some messages were accepted whilst others were rejected or requested it could lead to those devices getting out of order messages. There is probably some consideration needed in terms of concurrency on the device, it will be necessary to process those error messages in order and to ensure that the update to the local records and the recreating of the messages is performed together and before other send operations. Otherwise, there could again be out of order messages. 

#### Linking Devices
This code must exist somewhere within the client application, but doesn't appear easy to find, although it would be advantageous to know how the encryption is being performed. It is unlikely to be based on a standard instance of the double ratchet, since the identityKeys won't have been shared at this point. Worst case is just a fairly standard ephemeral Diffie-Hellman to create a key to send to the shared public key. This will need authorising somehow by the user. In Signal they appear to use the QRCode scan as this authorisation step. However, if we don't want to use QRCodes we will need some way of requesting the authorisation from the current device. We will obviously be somewhat better protected from fake requests since the new device will need to be able to log in to the current account, but some consideration of UI and security of that authorisation is needed - particularly given that it will transfer the long-term private identity key.

It appears this implemented in the library using the [PrimaryProvisioningCipher](https://github.com/signalapp/Signal-Android/blob/main/libsignal/service/src/main/java/org/whispersystems/signalservice/internal/crypto/PrimaryProvisioningCipher.java) for creating and the [SecondaryProvisioningCipher]() for receiving. The contents of message is defined in the [Proto](https://github.com/signalapp/libsignal-service-java/blob/master/protobuf/Provisioning.proto) object.

#### Read Syncing
Signal doesn't create notifications for messages already read. For example, if on the Desktop and it is open the read notification does not go off on your phone. However, if the message is not read on the Desktop for a period of time then the notification is sent to the device. I'm assuming this is being performed via some form of delayed notification syncing. In that the device currently active is somehow able to prevent other devices from receiving notifications. This isn't essential, and may not be desirable, but will require further consideration.

## Core Additional Functionality
Whilst the library handles a lot of functionality the client performs a lot of management on the underlying key/sessions stores that will need to be implemented. For example, if a device switches to a new preKey the other devices will need to re-initialise their sessions. This requires that the old session is placed in stale state. It cannot be immediately deleted as there could be delayed messages still in transit. These stale sessions are held around for an unspecified amount of time, after which the client is free to start deleting them.

This functionality is managed by the DeviceRecord, but there isn't an obvious reference implementation of DeviceRecord.

## Staffer Accounts
The general design permits staffers to act on behalf of MPs, would that functionality need to extend to DMs, i.e. would a staff need to be able to read the MPs DMs (not necessarily by default, but as an option)? If the answer is yes, we might want to consider using an alternative approach for multi-device, in that the signing of sub-keys by the primary key would permit potentially shared access.

## Crypto of Multiple Devices
There are essentially two options for having multiple devices:
1. Share the identity key material across all devices
2. Each device retains a unique identity key and they are somehow linked

Signal adopts option 1. However, this presents a number of problems:
1. The private key has to be transferred to another device
2. A single device cannot be revoked without changing the identity key across all devices, i.e., a staff couldn't be given access and then have access revoked without reinitialising all devices
3. It is not clear how well the safety number checks work in the setting

The last point is of particular interest. In a one-to-one conversation there is no trust in the server if you have verified your safety number correctly. However, that may not be the case when re-initialising a new identity key in a multi-device setting. For example, let's imagine a scenario where Alice has DeviceA and Device B. DeviceB is compromised by a malicious party who has also compromised the platform. Alice realises DeviceB has been lost and as precaution creates a new identity on DeviceA. However, Alice is entirely dependent on the platform behaving honestly to notify this change to her contacts. If the platform is compromised it can keep DeviceB active and as far as any contacts will know nothing has changed. Simultaneously the platform blocks Alice from contact any exist contacts, thereby preventing alerts about her safety number changing being shown to them. In effect, the compromised DeviceB can continue to impersonate Alice for as long as the platform allows. 

It is not clear that this issue can be easily resolved, but it is worth noting in order to correctly evaluate alternatives.

### Alternative Approach
Instead of doing what signal does, we adopt the authorisation mechanism instead, with devices having unique identity keys. This would work using a similar approach to TLS certificates. If Alice has Primary Device A and wishes to authorise Device B to have DM access, Alice would perform the following procedure:

1. DeviceB (having already registered and having its own identity key) creates a certificate signing request (CSR) that can be sent to DeviceA. This CSR is a standard x.509 CSR, which includes a public key certificate for DeviceB's identity key.
2. DeviceB displays the CSR (CSR-B) as a QRCode. DeviceA scans the QRCode and reviews it.
3. If DeviceA and the user authorise the request DeviceA creates a new subkey identity key pair (subKeyA), constructs a CSR (CSR-A) for that key and sends it to itself.
4. DeviceA signs CSR-A with its primary identity key and store the result
5. DeviceA uses subKeyA to sign CSR-B and return the Public Key Certificate to DeviceB.
6. DeviceA sends subKeyA Public Key Certificate to the platform to be recorded in its record of active subKeyCerts. (For additional security a certificate transparency service could be used to record all subkeys generated)
7. DeviceA sends the signed Public Key certificate of DeviceB to the platform to be recorded as a linked device.

When a device wishes to send to DeviceA it performs the following: (VT: DeviceB? Or just Alice generally?)

1. Sesame protocol as before, i.e. construct messages for any known devices and send to the platform. If the platform contains a different list of devices it will reject the message and return the full list of deviceIds and their corresponding public key certificates.
2. The device examines each device certificate and checks that there is a corresponding subKeyCert available, signed by the primary device identity key, and not revoked. If no such certificate path can be found the sending is canceled and an error is shown (cannot send due to unauthorised devices).
3. If a certificate path exists to the same identity key that it has previously seen it does not need to alert the user that a new device has been added or that the safety number has changed. It only need do that if the primary identity key changes. 

If a device wishes to revoke access it should create a Signed Revocation Request which it sends to the platform. It should also delete the subKeyCert entry thereby preventing that device from being authorised in the future, and remove it from its device list. Thereby cutting off the linked device without the need to revoke the identity key. 

### Advantages
The biggest advantage of this approach is that it allows a single device to be linked to multiple accounts. i.e. if a staffer works across multiple offices they can have a single account that is linked to multiple other accounts. The other advantage is the ability to add and remove access without compromising the secrecy of the identity key.

### Disadvantages
It diverges from the signal implementation, however, it is still consistent with the Sesame standard. It is also not clear that the signal implementation actually delivers what is required (see above). The same issue of a compromised device with a compromised platform exists, in that it can still hide the revocation. Although one option would be to send the revocation notification directly to all contacts via an internal system message and receive confirmation to bypass the trust assumption on the platform to broadcast that information. 