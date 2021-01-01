This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [Contact Tracing and Consent (23 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
- [Tracing the Challenges of COVIDSafe (27 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md) by Chris Culnane, Eleanor McMurtry, Robert Merkel and me.
- [The Missing Server Code and why it Matters (14 May)](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)  by Robert Merkel, Eleanor McMurtry and me.
- [Security Analysis of the UK's NHS Contact Tracing App](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-19UKContactTracing.md) by Chris Culnane and me.
- [COVIDSafe's new payload encryption scheme (15 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-15COVIDSafesNewEncryptionScheme.md) by Chris Culnane, Ben Frengley, Eleanor McMurtry, Jim Mussared, Yaakov
 Smith, Alwen Tiu and me.
- [**Issues with COVIDSafe's new encryption scheme (19 June) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md) by the same authors.
- [The current state of COVIDSafe (mid-June 2020) (22 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md) by the same authors.
- [COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2021)](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys (16 Dec)](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)
- 
---------------------------------------

# Issues fixed in COVIDSafe today
## Why every COVIDSafe user should upgrade immediately

Chris Culnane: [stateofit.com](https://stateofit.com) / [@chrisculnane](https://twitter.com/chrisculnane)   
Ben Frengley: ben.frengley [at] gmail.com / [@bgf_nz](https://twitter.com/bgf_nz)  
Eleanor McMurtry: [eleanorm.info](https://eleanorm.info) / [@noneuclideangrl](https://twitter.com/noneuclideangrl)   
Jim Mussared: jim.mussared [at] gmail.com / [@jim_mussared](https://twitter.com/jim_mussared)   
Yaakov Smith: [yaakov.online](https://yaakov.online) / [@yaakov_h](https://twitter.com/yaakov_h)   
Vanessa Teague: [ThinkingCybersecurity Pty Ltd](https://www.thinkingcybersecurity.com) / [@VTeagueAus](https://twitter.com/vteagueaus)   
Alwen Tiu: The Australian National University alwen.tiu [at] anu.edu.au

The new encryption scheme was described in the [previous post](blog/2020-06-15COVIDSafesNewEncryptionScheme.md). Here we explain several issues with its implementation that were corrected in the most recent version.  We'll follow up with a third post, describing remaining issues.  The release of this post has been delayed, because the new version introduced a significant cryptographic bug in the Android app, which has now been corrected in the most recent version (1.0.28).
Today's iPhone update (1.6) also contains critical bug fixes.    

**All COVIDSafe users, both iPhone and Android, should upgrade immediately.**

The main bug fixes are:

1. [A bug in the way COVIDSafe reads Bluetooth messages on iPhones](#iPhoneToiPhoneBug) means that the new, longer, encrypted messages are sometimes garbled. This means that some iPhone-to-iPhone contacts will not be recorded, though it is possible for the same phones to connect again in a different way that does record properly.

1. A patch for CVE-2020-14292, a vulnerability allowing for long-term tracking of Android devices.

1. COVIDSafe on iPhones can now download a new TempID when the phone is locked.

1. Encryption was implemented in a manner that did not prevent [interference between multiple threads](#EncryptionBug). This sometimes crashed the app, and could possibly lead to garbled encryptions or leaked information. This has now been patched.

<a name="iPhoneToiPhoneBug"></a>
## COVIDSafe v2 doesn't log iPhone-to-iPhone contacts as intended
The Bluetooth messages sent by COVIDSafe v2 are much longer than those of v1.  A pre-existing bug causes these to be garbled in some iPhone-to-iPhone transactions, significantly reducing the reliability of contact logging.

This issue was discovered by John Evershed of ProjectComputing; the details are copied here with his permission. The issue affects all iPhone-to-iPhone exchanges, regardless of whether the phone is locked or unlocked. (This should not be confused with the [TempID download problem](https://docs.google.com/document/d/1dsSxC48cJ91X17PoOybpun1U163YDxxL0CDk3kmAHvY/preview) discovered by Richard Nelson, which affects COVIDSafe bluetooth messages that should be sent from a locked iPhone, regardless of the recipient, and has been patched in the most recent version.)

The size of the payload exchanged by COVIDSafe far exceeds the default maximum transmission unit (MTU) used by Bluetooth Low Energy (BLE)'s ATT protocol (23 bytes). However, there are two provisions in BLE to allow for the exchange of larger payloads:

* The central device can initiate negotiation of a higher MTU (up to 512 bytes).

* Reading and writing of a GATT characteristic value can be done in chunks. (This is called a "long" read or write, in which the offset of the chunk is included in the transmitted data.)

The Android version of COVIDSafe will always attempt to negotiate a 512 byte MTU. iOS does not provide an equivalent API and will automatically [negotiate to use 185 bytes on iOS 10.0 and higher](https://github.com/chrisc11/ble-guides/blob/master/ios-mtu.md).

The version 1 payload was slightly smaller than 185 bytes, so "long" reads and writes never needed to be used. However, the new version 2 payload, as described in the previous post, is typically around 350 bytes.

When an iPhone app in peripheral role (i.e. a GATT server) receives a read for a characteristic value, it is given the offset of the read, and is expected to only return the part of the value starting at that offset. This is described in the [Apple documentation](https://developer.apple.com/library/archive/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW6) for CoreBluetooth. However, COVIDSafe for iOS ignores the offset, and always attempts to return the full payload.

The result is that when an iPhone in central role attempts to read this value, it will instead see approximately the first 180 bytes of the payload repeated two or three times, rather than the full payload. The central will then be unable to decode this as a valid JSON message and therefore ignore it.

This issue has been verified in several different ways:

 1. A developer build of the COVIDSafe app (using the v1.5 source code published to GitHub) will log the failure to decode the JSON, and the log message shows the payload containing the repeated fragments.
 
 1. Developer builds of the COVIDSafe app (using the v1.5 source code published to GitHub) have a debug screen that allows scanning or advertising modes to be individually enabled or disabled. Using a developer build on two iPhones, one with only scanning enabled and one with only advertising enabled, the advertising device records the details of the scanning device, but the scanning device does not record the details of the advertising device.
 
 1. A test program running on a PC that attempts to connect to the iPhone, negotiate a 100 byte MTU, and then issue long reads will always see the same response regardless of the offset (the first 100 bytes of the payload). The same test connecting to an Android will see the the correct responses.

The issue does not exist when an Android device is involved, because the Android device in the central role will negotiate a 512 byte MTU, and in the peripheral role it correctly handles the offset reads.

It's still possible, however, that encounters can be logged on iPhone, as the protocol is bi-directional and both phones can perform both roles. As described above, the peripheral-to-central payload will be lost, but the central-to-peripheral payload should still be transmitted, allowing the peripheral phone to record an encounter. If the previously-peripheral phone is subsequently able to connect as a central, then the payload can be successfully exchanged in the opposite direction.

*Summary:* this bug significantly reduces the reliability of encounter logging.

*Fixing the problem:* Fortunately this issue is easy to fix, and the offset can be applied in `PeripheralController.swift`'s `didReceiveRead` method when returning the payload. It is remarkable that this issue was not discovered during development and testing, as every single payload transmitted from a peripheral iPhone to a central iPhone is affected, and every single time the central would have logged an error saying that it could not decode the JSON.

*Status:* This has been corrected in v1.6.

## CVE-2020-12856

COVIDSafe (Android) v1.0.17 and earlier contain a vulnerability, tracked using CVE ID CVE-2020-12856, that could allow an attacker to __bond__ silently with an Android phone running a vulnerable version of the app. 
The bonding process involves exchanges of permanent identifiers of the victim phone: the identity address of the bluetooth device in the phone and a cryptographic key called Identity Resolving Key (IRK). 
Possession of the identity address (sometimes also called, confusingly, the public address, in Bluetooth lexicon) would allow the attacker to detect the presence of the victim's phone, as long as Bluetooth is turned on, without the phone needing to run the COVIDSafe app. It also allows the attacker to launch further attacks leveraging on existing Bluetooth vulnerabilities.

Possession of the IRK would allow an attacker to reverse a (resolvable) random Bluetooth address back to the real identity address, so if the attacker can obtain a record of Bluetooth addresses from a third party (for example, logs captured by Bluetooth traffic monitors), the attacker will be able to track movements of a particular vulnerable device without needing to interact with the device.

This issue was reported to DTA on May 5th, 2020, and has been fixed since v.1.0.18.
Details of the issue are available [here.](https://github.com/alwentiu/COVIDSafe-CVE-2020-12856)


## CVE-2020-14292 

A vulnerability similar to that reported in CVE-2020-12856 has recently been identified for the Android version of COVIDSafe v1.0.21 and earlier. This vulnerability allows an attacker to obtain the Bluetooth identity address, and in some cases, to perform a silent bonding. 
This vulnerability has been reported to DTA on June 2nd, 2020, along with a suggestion for a fix.
That suggestion seems to have been incorporated in version 1.0.28, but we have not yet confirmed whether it fixes the vulnerability. 
This vulnerability is now tracked using CVE ID CVE-2020-14292. Details will be made public once the fix has been confirmed. 

## Locked iPhones cannot receive new TempIDs

This was discovered by Richard Nelson, and is [described in detail here](https://docs.google.com/document/d/1dsSxC48cJ91X17PoOybpun1U163YDxxL0CDk3kmAHvY/preview).

During registration, a JWT (JSON web token) is given to the app by the server. This is the authentication key that the app uses for any subsequent request to the server, such as downloading a new TempID or uploading the encounter database.

This JWT is stored in the device keychain, which is a secure key storage facility provided by iOS. However, for security reasons and to prevent apps using these keys without user interaction, the default behavior on iOS is that an app cannot access the keychain while the device is locked. COVIDSafe does not request this permission, and so if the device attempts to fetch a new TempID while the device is locked it will be unable to access the JWT.

The result is that if a device has been locked for more than 1 hour (on average), then it will be unable to generate new tracing payloads to send to other devices. This means that the user of the phone cannot ever be notified of a contact with an infected individual.

The reverse direction is still possible, i.e. the phone can still receive payloads from other devices, meaning that if this user becomes infected, other users will be notified. However, due to the MTU issue described in the [previous post](Post2.md), this doesn't work reliably for iPhone to iPhone connections.

*Summary:* The bug prevents iPhones from downloading a new TempID when locked, hence preventing them from sending COVIDSafe bluetooth messages to other phones.

*Fixing the problem:* Fortunately there's a simple fix, which is to set this permission on the keychain item to allow it to be accessed when the device is locked. The need for this is explicitly called out in [the documentation](https://github.com/evgenyneu/keychain-swift#keychain-item-access) for KeychainSwift (the library that COVIDSafe uses to access the Keychain), and even includes sample code. It's somewhat remarkable that this issue wasn't discovered during testing as it happens reliably if the TempID fetch happens while the phone is locked, and a log message is printed out.

*Status:* This has been corrected in v1.6.  All iPhone users should upgrade immediately.

<a name="EncryptionBug"></a>
## Critical Concurrency Bug in encryption for COVIDSafe on Android

COVIDSafe's [encryption code](https://github.com/AU-COVIDSafe/mobile-android/blob/05a2ca94a6e858ba751b6069151386038b776943/app/src/main/java/au/gov/health/covidsafe/streetpass/persistence/Encryption.kt) v1.0.18 contains a critical concurrency bug that could result in an exception or possible corrupted encryptions. The root cause is sharing a single Cipher instance across different threads, without appropriate synchronization.
This bug affects Android versions 1.0.18 to 1.0.27, and has been corrected in the current version (1.0.28). It does not affect the iPhone versions, which perform encryption differently (and, as far as we can tell, correctly).

The issue is due to the implementation of the [encryptPayload function](Post1.md#EncryptPayloadFunction). This function is not synchronised and could be called from multiple threads including:

* StreetPassReceiver declared in `app/src/main/java/au/gov/health/covidsafe/services/BluetoothMonitoringService.kt`
* gattServerCallback declared in `app/src/main/java/au/gov/health/covidsafe/bluetooth/gatt/GattServer.kt`
* StreetPassGattCallback declared in `app/src/main/java/au/gov/health/covidsafe/streetpass/StreetPassWorker.kt`
* StreetPassRecordDatabase declared in `app/src/main/java/au/gov/health/covidsafe/streetpass/persistence/StreetPassRecordDatabase.kt`

As a result, the Cipher object (shared between threads) can be either in an invalid state (e.g. uninitialised) or could produce corrupt encryptions if two calls are made to it concurrently. For example, a HMAC could be mismatched with the wrong AES-CBC blocks. We believe this failure could occur silently without the user being alerted, and may only become apparent when invalid encryptions fail decryption by the authorities. This would mean that users exposed to COVID-19 may not be notified, because their TempIDs cannot be decrypted. Corrupted encryptions could also leak information and allow some decryption without the key, but since this would reveal at most the information that had been available in plaintext in version 1, it is hard to argue that this is not a disaster. It could also cause the app to crash -- we are not sure whether it reliably restarts in this situation, or remains non-operational.

The fix for this bug is extremely simple: symCipher should be instantiated inside `encryptPayload`. The computational cost of instantiating it is low in comparison to initialising it, which is performed every time anyway. Alternatively, the `encryptPayload` function could be synchronised, however, this could cause a deterioration in performance as it will force all Bluetooth interactions to synchronise on the `encryptPayload` function.

*Status:* This was disclosed to DTA and ASD in early June. It affected Android versions 1.0.18 to 1.0.27 of COVIDSafe and has been patched in the most recent version.  Android users of COVIDSafe should upgrade to 1.0.28 immediately, which you may need to do manually because automatic updates do not seem to be working reliably.  The iPhone version is not affected by this bug.

## Acknowledgements

We'd like to thank the DTA and the Australian Signals Directorate for their prompt attention to the encryption issue, and encourage the DTA to address the Bluetooth tracking problem and the iPhone logging failure urgently.

We'd also like to thank the large and active community of Australian techies who have examined, discussed, and tried to correct the code.

### Followup and reuse

Comments, edits, suggestions and pull requests are welcome.

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link:
[https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md).
