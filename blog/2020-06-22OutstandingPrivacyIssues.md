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
- [Issues with COVIDSafe's new encryption scheme (19 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md) by the same authors.
- [**The current state of COVIDSafe (mid-June 2020) (22 June) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md) by the same authors.
- [COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2021) - this post](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys (16 Dec)](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)
- 
---------------------------------------

Chris Culnane: [stateofit.com](https://stateofit.com) / [@chrisculnane](https://twitter.com/chrisculnane)   
Ben Frengley: ben.frengley [at] gmail.com / [@bgf_nz](https://twitter.com/bgf_nz)   
Eleanor McMurtry: [eleanorm.info](https://eleanorm.info) / [@noneuclideangrl](https://twitter.com/noneuclideangrl)   
Jim Mussared: jim.mussared [at] gmail.com / [@jim_mussared](https://twitter.com/jim_mussared)   
Yaakov Smith: [yaakov.online](https://yaakov.online) / [@yaakov_h](https://twitter.com/yaakov_h)   
Vanessa Teague: [ThinkingCybersecurity Pty Ltd](https://www.thinkingcybersecurity.com) / [@VTeagueAus](https://twitter.com/vteagueaus)   
Alwen Tiu: The Australian National University alwen.tiu [at] anu.edu.au


# The current state of COVIDSafe (mid-June 2020): Outstanding privacy issues

The new encryption scheme and major bug-fixes were described in the previous posts. This post attempts to summarise the known outstanding issues with COVIDSafe and provides some recommendations for users on understanding the risks, then some recommendations for government on where to go from here.

We know of four main remaining issues. Users can do something about the first two (see below).

1. The app does not seem to automatically update correctly.

1. Users are encouraged to [reset their phone name](#PhoneName) to something that is not unique, but there is no guidance, so many users are more likely to end up with a unique name than if they had stuck with the default. Furthermore this only applies to new users, installing the app for the first time.

1. Although the app's ephemeral public key changes frequently, it is possible to observe this change while connected to the phone, which allows for [long-term continuous tracking](#PhoneTracking) despite the change.

1. Successive uses of the same key within the 7.5 minute window are distinguished by a counter sent in plaintext. [Reading this counter leaks some information about the user's recent past](#Counter). For example, if it isn't zero then they haven't been alone.

These problems are probably not show-stoppers for most Australians in most circumstances. However, users should be aware of them when they consider whether to install the app and whether to leave Bluetooth on in particular situations.

## The app does not automatically update

It has been our experience that the app does not automatically update on Android, and many people's phones are running the first version that they installed. We have reported this to the DTA 10 days ago and they have confirmed that they have also observed this. *This means that users are likely not receiving the security updates to fix the issues that have been described in these posts.*

The phones we tested this on do have auto-updates enabled in the Play Store App, and there was a long list of other apps that had updated recently, including some in the past 24 hours. COVIDSafe shows up in the list of available updates, but had never been auto-updated since the first install. On the first phone we first noticed this on, the version running was over 5 weeks old.

The phones had update-over-WiFi enabled, regularly connected to WiFi at home, running stock Android 10.0, and had successfully auto-updated other apps, so this is likely something specific to COVIDSafe.

A log from a phone as it detected the 1.0.21 update  is shown below. Note that "Finsky" is the code 
 for the Google Play Store app.

```txt
06-13 19:29:42.634 29735 29735 I Finsky  : [2] acbh.b(21): AU: package au.gov.health.covidsafe has available update because at least one of the following constraints met: isGmsCoreInstall=false, isIsotopeInstall=false, isAr
Install=false, canUpdateApp=true
06-13 19:29:43.230 29735 29735 I Finsky  : [2] ocu.accept(2): IQ: Attempting to validate incoming request. package=au.gov.health.covidsafe
06-13 19:29:43.317 29735 29735 I Finsky  : [2] oai.b(3): IQ: Scheduling install request package_name=au.gov.health.covidsafe, version=21, priority=3, reason=auto_update, account_name=[MTMrNV_gmLp9ywlH9-AMoWUvn1vxCW_oKbl2lni
zdgY], type=0, constraints=((REQ_CHARGING, REQ_GEARHEAD_PROJECTION_OFF, NETWORK=UNMETERED, PROVISIONING_STATE=PROVISIONED),(REQ_DEVICE_IDLE, REQ_GEARHEAD_PROJECTION_OFF, MIN_BATTERY=50, SHOULD_APPLY_BUDGET=true, NETWORK=UNM
ETERED, PROVISIONING_STATE=PROVISIONED),)
06-13 19:29:43.687 29735 29855 W Finsky  : [256] nyu.call(16): IQ: Resolved conflict: package=au.gov.health.covidsafe. oldConstraints=[(REQ_CHARGING, REQ_GEARHEAD_PROJECTION_OFF, NETWORK=UNMETERED, PROVISIONING_STATE=PROVIS
IONED), (REQ_DEVICE_IDLE, REQ_GEARHEAD_PROJECTION_OFF, MIN_BATTERY=50, SHOULD_APPLY_BUDGET=true, NETWORK=UNMETERED, PROVISIONING_STATE=PROVISIONED)], newConstraints=[(REQ_CHARGING, REQ_GEARHEAD_PROJECTION_OFF, NETWORK=UNMET
ERED, PROVISIONING_STATE=PROVISIONED), (REQ_DEVICE_IDLE, REQ_GEARHEAD_PROJECTION_OFF, MIN_BATTERY=50, SHOULD_APPLY_BUDGET=true, NETWORK=UNMETERED, PROVISIONING_STATE=PROVISIONED)]
06-13 19:29:44.290 29735 29855 I Finsky  : [256] oai.b(41): IQ: Notifying installation update. package=au.gov.health.covidsafe, status=SCHEDULED
```

We do not have an explanation for why this is happening, or a suggested fix. The best theory we've been able to come up with is that the installation of the update is never being scheduled because COVIDSafe is constantly active in the background, which is atypical behavior for most apps. Normally this feature prevents auto-updates from interrupting playback from apps like your music or podcast player. However, we lack sufficient evidence to conclude that this is definitely what's going on.

Furthermore, we also have some reports that something similar might be happening on iPhone and have seen cases of iPhones running versions that are several weeks old, but have not investigated it further to confirm auto-update settings.

*Summary:* We recommend that users check for updates and install them manually, on both Android and iPhone. This involves going to the Google Play Store, or Apple App Store apps and checking the list of available updates. As at the time of posting, the current versions are v1.0.28 for Android, and v1.6 for iOS.

<a name="PhoneName"></a>
## Unique phone names (e.g. "Name's iPhone")

As a result of operating as a BLE GATT server, the system provides a default service that allows any connected device to obtain the device's name. This value is user-configurable, and often defaults to the device model (e.g. "Samsung Galaxy G8"), but for usability and convenience reasons, some users will customise this value. It's useful to do so because this is what is shown in, for example, car stereo systems.

This value serves as a persistent identifier, which is likely to be unique if the user has customised it *in any way* (although including the owner's name is the worst case). The three categories are:

* Default (phone model)  (likely to be unique in many scenarios)

* Anything involving the owner's name  (unique, gives away name)

* Any other user-chosen value  (extremely likely to be unique)

As described in the first post, as for the phone model, the phone name can be used in conjunction with other identifiers such as the MAC address or the tracing payload to enable long-term tracking and re-identification of a device. This was first raised with the DTA within days of the launch of COVIDSafe.

In v1.0.18, COVIDSafe implemented a change that aims to partially mitigate this, however it is flawed in a number of ways.

During registration, the user is now shown the following screen:

```
The current name of your device is %s.

Other Bluetooth® devices around you will be able to see this name. You may like to consider making the device name anonymous.

New device name: "Android phone"

[Change and continue] [Skip and keep as it is]
```

The textbox is pre-populated with the text "Android phone". The goal here appears to be that now that the model name is hidden, if every user changes their phone name to the same thing, then the phone can no longer be identified.

However:

* This flow only applies to new users, not users upgrading from earlier versions.
* If a user changes this value to anything other than the default "Android phone" then their device will likely be uniquely identifiable.  In particular, user-chosen input is much more likely to be unique than the defaults that come with most Android devices (e.g. "Samsung Galaxy Tab A").
* The instruction to "make the device name anonymous" is not clear.
* This was not implemented on iPhone.
* Changing the device name is bad for usability as this name is shown, for example, in a car stereo or any other location where the device is paired.

It is worth nothing that an app using the Exposure Notification API from Apple & Google would not have this issue with the phone model or name being exposed.

*Summary:* Don't choose a new phone name at this screen unless you already have a unique/identifying one. If you must change it, don't switch from the default "Android phone."

<a name="PhoneTracking"></a>
## Continuous tracking of phones

### Background on MAC address and TempID-based phone tracking

Modern phones rotate their MAC address (about every 10-15 minutes) to evade tracking.  Even an attacker who connects to them continuously (via a GATT service) doesn't see their new MAC.  It has been explained previously that the continuous TempIDs (whether they last 2 hours or 15 mins) undermine this, because an attacker can observe a phone frequently (say every 5 mins), read both its MAC and its TempID, and thus learn the old-MAC->new-MAC transition when it observes an overlapping TempID appearing with the refreshed MAC.  An overview of MAC-related privacy problems can be found [here](https://docs.google.com/document/d/17sVyBIG5CqhF9XtuEfeG2MfYsFNXuV4yxp3BERDTJoI).

### Problem summary
This attack doesn't even need to observe the MAC: since COVIDSafe allows you to see the ephemeral key changing as you are connected, an attacker doesn't need to use the inbuilt MAC to link successive ephemeral keys and hence track the phone.

This problem was present in the earlier version of COVIDSafe.  Version 2 doesn't make it any worse, but the problem undermines the main privacy improvement of Version 2 if the attacker can connect to the phone continuously or frequently.

### Continuous tracking of iPhones
We found that if a central device connects to an iPhone with the COVIDSafe app running in the foreground, the central device can keep the connection alive by issuing repeated read operations. Together with the fact that the payload changes every time a read is performed, this provides an easy way to perform continuous tracking of an iPhone running the app while the device is within Bluetooth range of the phone. An attacker can simply connect to the iPhone and read the payload repeatedly without disconnecting. The attacker will be able to observe the key rotation every 7.5 minutes, since the ephemeral public key is included in the payload. This allows the attacker to link two consecutive ephemeral keys generated by the COVIDSafe app in the iPhone.

A more efficient attack would be to disconnect once a key rotation is detected, and reconnect before the expiry time of the key (which is 7.5 minutes from the time the change is detected).

Multiple central devices can be used to track an iPhone in areas larger than the Bluetooth range. For example, suppose the attacker wants to track the movement of a phone across two locations A and B. The attacker plants two central devices, one in each location; let's call them Central A and Central B. Central A will connect to phones in location A and continuously read their payloads, recording their ephemeral public keys. Central B will do the same for the phones in location B. Both central devices share their database of ephemeral keys with each other (e.g. via Wi-Fi or cellular network). If a phone moves from location A to B (disconnecting from Central A), Central B will detect the presence of a new device and perform a read to extract the ephemeral public key, which can be checked against the database of keys recorded by central A. If the new device is the same as the one leaving location A, its current ephemeral public key will likely be in that database.

There's a chance that as the phone disconnects from Central A, it changes its ephemeral public key before connecting to Central B (e.g. if the key expires before the phone arrives at location B). This can be reduced by having some overlap in the Bluetooth range of Central A and B, so there will be no gap in the transition from one location to another.

Note that it is not possible to observe the Bluetooth random MAC address rotation while being connected to the phone, so being able to connect to a GATT service indefinitely does not automatically grant the ability to perform continuous tracking. The issue here is caused by the app leaking information about the key rotation while a central device is connected.

### Continuous tracking of Android phones
The Android version of COVIDSafe implements a read cache per device: performing repeated reads in a connection will result in the same payload being returned, so the attack doesn't immediately work.  However, this read cache is cleared if a central writes a payload to the peripheral after reading a payload from it. So by performing a read followed by a write, we can force the read payload to be re-computed, and we are back in the same situation as with iPhones, allowing continuous tracking.

*Summary:* The fact that a connected attacker can observe the ephemeral key changing allows the attacker to link successive ephemeral keys, and hence track the phone for as long as it is within range.

*Fixing the problem:* Richard Nelson's alternative that generates a [fresh ECDH key every time](https://patch-diff.githubusercontent.com/raw/wabzqem/mobile-ios/pull/2.patch) solves this problem, at the expense of increased battery use.  While the attacker can still see the ephemeral key change, it can't disconnect and then look for the same key upon reconnection.  Alternatively, the problem could be addressed by ensuring that the app allows only one exchange per connection; this is fairly straightforward to implement.



<a name="Counter"></a>
## Implications of a plaintext counter
The plaintext counter used as part of the payload encryption leaks some information.
This was observed independently by us and by Robin Doherty at ThoughtWorks.

A crucial technical detail is that every time a message is encrypted, some extra data called an *initialisation vector* (IV) must be used for the encryption. For the operation to be secure, this IV must be unique every time the same key is used to encrypt a different message. The app achieves this by counting the number of times the ephemeral key has been used, and using this 2-byte counter to generate the IV. This counter is sent in plaintext with the encrypted message over Bluetooth.  The ephemeral key is refreshed and the counter is reset to zero when 7.5 minutes have elapsed, a new TempID is downloaded, or the counter reaches 65535 (the highest number 2 bytes can store).

This counter leaks some information about the user's recent past. Someone who has had no contacts through the last 7.5 minutes will present a zero counter to the next person they interact with; a person who presents a nonzero counter has interacted with others since their last refresh.  The converse argument, of course, does not work, because someone who presents a zero counter on one hand may have had no interactions recently, or on the other hand may have refreshed their ephemeral key recently.

Different individuals' refresh times are not synchronised, and hence constitute a fact about a user that persists past the 7.5 minute window.  Although this information resets during any period in which the person is not in contact with others, there are circumstances in which it would persist for some time.

This information might be sufficient in some contexts to infer meaningful information about people. Although probably not a show-stopper for most COVIDSafe users, these scenarios indicate that the use of COVIDSafe does leak some information to others, which may be important for some users.

### Using the counter to contradict a story of having been alone

Suppose Alice leaves the office for 10 minutes and tells Bob not to interact with anyone during that time.  When she returns, she reads Bob's COVIDSafeApp counter. If it isn't zero, she knows that Bob has disobeyed her instruction.

### Using counter refresh time as a persistent identifier

Consider the following scenario:

Alice and Bob have lunch together in a cafe. Suppose the cafe has some way to identify which is Bob and which is Alice. A Bluetooth receiver in the cafe reads Alice and Bob's COVIDSafe App messages and notes the times at which they refresh their ephemeral keys. For example, suppose Alice refreshes at 13:05:00 and Bob at 13:04:00. Then Alice and Bob go on a walk together, for half an hour, leaving their COVIDSafe apps running. When they return to the cafe their reset times make them immediately distinguishable, because Alice's reset time will be an integer multiple of 7.5 minutes after 13:05:00 and Bob's after 13:04:00. Half an hour is a multiple of 7.5 minutes, so in our example a refresh at 13:35:00 could be Alice; a refresh at 13:34:00 could be Bob, and a refresh at 15:30:00 could be neither.

This works in the somewhat-contrived scenario described here, but breaks down if the targeted person is out of range of others' Bluetooth beacons for a time, because the 7.5-minutely reset occurs only when others are in range. It also breaks down if the reset time is disturbed for some other reason, such as downloading a new TempID.

Assuming an accuracy of about 200ms in estimating refresh time, this style of re-identification attack could distinguish about 7.5 * 5 * 60, i.e. about 2000 different values. This is still an improvement over COVIDSafe v1, in which the same TempID was used for 2 hours, but still may expose enough consistent identifying information for tracking over the same two hours.

### Using counter values to infer proximity information

I invite Alice, Bob, and Charlie to a job interview. When they arrive, I sit each of them in a separate isolation room for 10 minutes. When the 10 minutes is up, I go and release them, bringing my COVIDSafe App-running phone. This causes them all to zero their counter at a time of my choosing. Next, I let them all go and play outside together, in a large open space with no other devices. I promise I will not watch what they do. After 5 minutes, I call them back in and engage each of them in another COVIDSafe exchange with my phone. I note each of their counters. Even if I haven't watched them, I can infer who was close to whom through that time (at least, who was within Bluetooth range) - I simply look at whose counters have incremented how much, and this tells me exactly how many pings have been exchanged between each pair A-B, B-C, C-A.

For example if Alice has exchanged 7 messages, Bob has exchanged 5 and Charlie has exchanged 2, I know that Alice and Bob were near each other for longer than Alice was near Charlie, and that Bob and Charlie didn't interact at all.  (Alice and Bob exchanged 5 messages, Alice and Charlie exchanged 2, and Bob and Charlie none.  This is the only possible solution. Obviously this assumes some predictability in Bluetooth-based proximity measurement.)

*Summary:* the plaintext counter leaks some information about the recent contact history of COVIDSafe users, which might be important to certain users in certain circumstances.

*Fixing the problem:* use an encryption method, such as AES-GCM, that doesn't need a plaintext counter.



## Summary and recommendations
The new cryptographic protocol solves some pre-existing privacy problems, leaves some others unchanged, and introduces some new ones.

### For government

Building on TraceTogether was a reasonable choice when it was the only available option, but serious consideration should now be given to adopting the Exposure Notification API from Apple and Google. There is a genuine and profound difference between a centralised and a decentralised model.  I (VT) have explained [in a previous
blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md) that I would vote for a decentralised model because I feel the risks of abuse or leakage of the centralised contact database outweigh its potential benefits for epidemiology etc, especially given that some epidemiological information is still available from human recall.  However, there are solid arguments on both sides.

The security of that centralised database is a serious concern. The DTA's COVIDSafe Test Plan recently [provided to Parliament](https://www.aph.gov.au/DocumentStore.ashx?id=1717dded-c0b0-4c6f-b299-cbde7f3daea0) shows only a login and password to authenticate administrators (Attachment C Section 2.2.1.3 - Admin Portal).  If administrator logins truly lack two-factor authentication, this directly contradicts the Australian Cybersecurity Centre's Essential Eight recommendations, and is not appropriate for such an important and sensitive database.

COVIDSafe suffers from several bugs that are unrelated to profound questions about centralisation, but undermine both reliability and privacy. These are being gradually addressed, but new ones are being gradually introduced at about the same rate.  The Google/Apple API would have huge practical advantages for working more reliably and having fewer unnoticed bugs, regardless of the political questions.

The remaining differences between the most recent version of COVIDSafe and the probable behaviour of the Google-Apple API are:

- The existence of a centralised data store of each infected person's contacts.  Depending on your risk model, this is a strong argument for one solution or the other. 

- The opportunity for AWS to collect 2-hourly metadata on users.  (Again, a bug or a feature depending on your perspective.)

- The opportunity for longer-term tracking as a result of either continuing Bluetooth bugs in COVIDSafe, or the (probably unavoidable) difficulty of synchronising with the MAC rotation.  This is a significant practical privacy advantage of the Google-Apple API.

- Backwards compatibility with old phones.  We don't know how much difference this makes in practice, since in this case (as in all other cases), reliability really matters.  If it isn't running reliably then it may not be as much of an advantage as it seems, but of course we don't know.

It is worth considering whether the significantly better (though not perfect) privacy properties of the Google-Apple solution might result in greater trust and higher uptake, and hence better disease control.  Obviously we won't really know unless we try, but we may get information soon from countries that have adopted a decentralised solution.

We strongly recommend against rolling out another new cryptographic protocol without a period of open review to avoid introducing further problems, or
indeed persisting with the current protocol without genuine open review of the server side and urgent correction of the app's existing bugs.

## Acknowledgements

We'd like to thank the large and active community of Australian techies who have examined, discussed, and tried to correct the code.

### Followup and reuse

Comments, edits, suggestions and pull requests are welcome.

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link:
[https://github.com/vteague/contactTracing/blob/master/blog/2020-06-OutstandingPrivacyIssues.md](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md).
