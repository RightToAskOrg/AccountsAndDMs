# User Accounts (WP1A)

This report provides a review and recommendations for user management in Right To Ask (RTA). It will cover the following:

* Back-end structure
* Types of accounts and their respective needs
* User registration
  * Including local and federated login
* User Verification
* Account recovery
* Support for E2E DMs
  * Including support for the future implementation of push notifications
* Account Moderation

## Back-end Structure
The current implementation as the following schema for a user:
| Field Name | Field Type | Comments |
| ------------- |:-------------:| :-------------: |
| UID | VARCHAR(30) | PRIMARY KEY NOT NULL
| DisplayName | VARCHAR(60) |  |
| AusState | VARCHAR(3) |  |
| PublicKey | TEXT NOT NULL | For bulletin board access

### Multi-Device Support
Currently the back-end has only partial support for a single user having multiple devices. This is because each user can only have a single PublicKey associated with it. Multiple devices could only be supported if the key pair were synchronised across the devices. This is both difficult to do securely and if the key is eventually stored in a SecureElement will be impossible to do. 

> **Recommendation 1**: The user schema should only contain data that is about the user and should not contain device data. Instead, create a Devices table which is linked to a user via the UID. This allows a one-to-many relationships between the user and their devices. This will also be required to support E2E DMs and Push Notifications, both of which will require device specific identifiers to be stored against a user.

### UID
Currently the UID is equivalent to a username, for example, a handle or email address. Any user selected value should be anticipated to need to change, even if not fully supported on the front-end. For example, if a user selected UID contained a racist term or a swear word. Moderation may looks to remove that from the UID, which would present a problem if that value is being used throughout the database.

> **Recommendation 2**: Change UID to be an internal-only UID that is not displayed to the user. It should be auto-generated to guarantee uniqueness and permanent, i.e. it never changes. As such, UID should be changed to ``INTEGER AUTOINCREMENT``. Any corresponding uses of UID in other tables would also need to be updated.

### Foreign Key Constraints and General Database Structure
The current database schema does not always set primary or foreign key constraints. Such constraints help to maintain both referential integrity, as well as having a possible performance benefit. Specifically, the information provided via key constraints allows the database to optimise linking queries, as well as the searching of tables via the primary key. It also allows the ``ON DELETE CASCADE`` property to be configured. This will automatically delete tables that reference the primary key when deleting the record containing the primary key. For example, delete a user will delete any badges that are associated with that user. This is not automatic, and could be selectively applied. For example, it might not be desirable to delete a question when deleting a user. Although that does need some consideration.

> **For Consideration**: What should occur when a User is deleted or requests to be deleted. Should all of their data be deleted or should only personal information be deleted. Note: this does not impact on whether key constrains should be set, it just impacts on whether a ``ON DELETE CASCADE`` constraint is set.

Whether the primary key should be just an auto-incrementing integer or a UUID is open to some debate.

#### Should a UUID be used as a primary key?
There is some debate about whether UUID's are more desirable than simple integer keys. The advantage of a UUID is that it is globally unique, as such merging databases is possible without the problem of key duplication. Furthermore, if sharding a database across multiple servers UUID will prevent key collisions. Some argue that is offers security benefits, in that a standard auto incrementing integer is susceptible to index attacks, i.e. ID=10 leads to it being obvious that ID's from 1-9 should be tried and 10+. UUID prevent that simple indexing, but their value against brute-force attack are dependent on how they are constructed.

MySQL contains a UUID function that generates a Version 1 UUID, which is based on MAC address and time in nano seconds. As such, there is no 'randomness' to speak of, just a rapidly changing clock. Therefore the indexing attack is not over the entire space, but is in fact bounded by the rate of change caused by the clock. Whilst there is likely to be a security benefit, quantifying that benefit is problematic and therefore should not be relied upon to provide any meaningful protection. If the system is not secure with Integer keys it isn't secure with UUID keys. 

The biggest drawback of using a UUID is that they require considerably more space to store and are potentially difficult to debug and use. For example, a UUID is 16 bytes, whilst an Integer is only 4 bytes. The most efficient way to store a UUID is as a Binary field, however, this requires converting it to characters and back to be able to be human readable, and therefore displayed in any debug logs or used directly in any debug queries.

For a Minimum Viable Product (MVP) there are not sufficient compelling reasons to recommend a UUID. For simplicity a standard auto incrementing integer will suffice.

#### Database Structure
The current database structure is as follows:
![Current DB Schema](./db_schema_current.png )
Note that no relationships are defined between tables, despite them existing within the schema. This can be resolved with correctly configured key constraints.

The following diagrams shows an updated diagram with the relational links added, as well some deduplication and some additional fields in the User table, which will be discussed in further detail later in this report. Additionally, there are placeholders for a devices table and the the PublicKey has been shifted to the device record. 

![New DB Schema](./db_schema_current_update.png )

Whilst this may look like a lot of changes, most of it is the addition of foreign key constraints. Additionally, linking tables for badges and electorates have been added. Previously each Badge would duplicate the content for each user, when in fact the badge is static and is just awarded to the user. Similar to electorates, there is only one electorate that each user should reference, instead of each user having an entry for that electorate. 

> **For Consideration**: Is there a one-to-one relationship between question and answer? If so, is there a one-to-one relationships between an answer and an MP? Or could an answer have more than one MP answer it? Alternatively could each question have multiple answers? If the latter is true, the answer table will need its own primary key, probably an auto increment integer called answer_id, and the questionid set as a foreign key constraint only and removed from the primary key of answer.

### User Table
Users has been modified to convert the UID to an auto increment integer. A new field has been added to hold the email address, irrespective or whether it is added manually or retrieved from a federated login. A password_sha256 field has been added to handle local logins. This should contain a 32 byte verification value calculated using the password_salt. Whether the salt is returned to the user for local calculation or the salted hashing is performed on the server remains to be decided. It is possible that these fields are blank depending on how registration is performed. 

The verification_code and verification_code_expiry contain the last randomly generated verification code sent to the user. The expiry should be a time, for example, 30 minutes after the code was generated. When checking the code, if the current time exceeds this expiry the code should be rejected. [_**Note**: implementing this requires careful consideration of daylight saving time if the clock being used for comparison changes with daylight saving time. It is generally easier to use UTC instead of a local timezone for such comparisons._]

### Sessions
Currently there is no concept of sessions within the back-end or the client app. Traditionally, apps would establish a session with the server though logging in, which would return some form of bearer-token that could be used to authenticate all subsequent requests until the session expires. By contrast, RTA uses the signing of messages as a means for authentication. This is sufficient when dealing with a very limited API, for example, adding a question, accepting an answer etc., however, it presents a problem for user account management. 

For example, if a user wants to change their display name or email address is will be necessary to authenticate both the request to view the current data, and any follow-up requests to change it. It is possible that these requests could be signed as well, but that will make the processing of such requests more computationally expensive due to the signature checking required. 

One option for establishing sessions would be to use Mutual Authentication of the TLS connection. The server could act as an internal Certificate Authority that signed certificate requests from the client. As such, when a user registers and generates their key pair they also generate a certificate signing request which is signed to create a certificate including their UID and their device_id.



