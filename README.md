This is the latest in a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [Contact Tracing and Consent (23 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
- [Tracing the Challenges of COVIDSafe (27 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md) by Chris Culnane, Eleanor McMurtry, Robert Merkel and me.
- [The Missing Server Code and why it Matters (14 May)](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)  by Robert Merkel, Eleanor McMurtry and me.
- [Security Analysis of the UK's NHS Contact Tracing App](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-19UKContactTracing.md) by Chris Culnane and me.
- [COVIDSafe's new payload encryption scheme (15 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-15COVIDSafesNewEncryptionScheme.md)
- [Issues with COVIDSafe's new encryption scheme (19 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md)
- [The current state of COVIDSafe (mid-June 2020) (22 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md)
- [**COVIDSafe issues found by the tech community (7 July) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md)

---------------------------------------

Jim Mussared: jim.mussared [at] gmail.com / [@jim_mussared](https://twitter.com/jim_mussared)   
Vanessa Teague: [ThinkingCybersecurity Pty Ltd](https://www.thinkingcybersecurity.com) / [@VTeagueAus](https://twitter.com/vteagueaus)

Last updated: July 24th 2020

# COVIDSafe issues found by the tech community

## Introduction

COVIDSafe was launched on April 26th, 2020. Several volunteers from the Australian tech community have spent considerable amounts of time analysing the code of the Android and iOS apps in order to help improve their contact tracing functionality and privacy.

Many issues have been found as well as recommendations for how to fix them, and some of these fixes have dramatically improved the effectiveness of the COVIDSafe app. In addition, several of these issues have been found and reported in contact tracing apps used in other countries.

Most, but not all, of these issues have been fixed.  However, due to a quirk in the way that COVIDSafe works, it is not clear that users are actually receiving automatic updates to the app.  **If you haven't checked you are running the most recent version, you should check manually and update now.**

## Types of issues

COVIDSafe has had a variety of different issues affecting both its privacy and its function.

### Privacy

Privacy issues result in the user's privacy being compromised, especially in a way that is not mentioned in the [privacy policy](https://www.health.gov.au/using-our-websites/privacy/privacy-policy-for-covidsafe-app). This may include being able to track a user's movements.

Many of the privacy issues involve the phone exposing any type of long-term identifier. This allows the user's movement to be detected if this same identifier can be recorded at different times and places. It also allows an attacker to detect if a target device is currently present at a given location.

### Security

Security issues compromise the security of the device running COVIDSafe in ways beyond privacy. For example, being able to remote control the device.

### Functionality

These are issues that prevent COVIDSafe from being able to effectively perform contact tracing, such as the app not being able to detect other devices and record the contact.

### Usability

Usability issues lead to user confusion or lack of confidence in the app. For example, user interface issues or minor functionality problems.

## List of issues

### 1. Unique identifier in Bluetooth advertising payload
Status: Fixed   
Type: Privacy   
Affects: Android   
More info: [Issue report](https://docs.google.com/document/d/1u5a5ersKBH6eG362atALrzuXo3zuZ70qrGomWVEC27U/preview#heading=h.dwxxp83i1pl0)

The Bluetooth Low Energy advertising payload is used to discover other devices running COVIDSafe. A three-byte identifier was incorrectly re-used in every payload, which uniquely re-identifies the same device over time.

### 2. Caching of tracing payload by remote MAC address
Status: Fixed   
Type: Privacy   
Affects: Android   
More info: [Issue report](https://docs.google.com/document/d/1u5a5ersKBH6eG362atALrzuXo3zuZ70qrGomWVEC27U/preview#heading=h.21w4dooci0sc)

For no apparent reason, the app caches the tracing payload by the remote device's MAC address. This cache entry was never removed. A device with a fixed address could therefore always re-identify a target device by detecting the same payload.

### 3. Phone model (e.g. "Samsung Galaxy G8") included in tracing payload
Status: Fixed   
Type: Privacy   
Affects: Android & iOS   
More info: [Issue report](https://docs.google.com/document/d/1u5a5ersKBH6eG362atALrzuXo3zuZ70qrGomWVEC27U/preview#heading=h.g4o2mg4wkn1n), [Research](https://www.scss.tcd.ie/Doug.Leith/pubs/bluetooth_rssi_study.pdf)

The V1 protocol includes the phone model, and this is available to any device in Bluetooth range. The phone model is ostensibly used for calibration of the signal strength to distance calculation, however it has been shown that this calculation does not work, and is likely not being used for contact tracing. The V2 protocol now encrypts this field.

### 4. 🚨 Phone name (e.g. "First Name's iPhone") accessible 🚨
Status: **Not fixed**   
Type: Privacy   
Affects: Android & iOS   
More info: [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md#unique-phone-names-eg-names-iphone), [Issue report](https://docs.google.com/document/d/1u5a5ersKBH6eG362atALrzuXo3zuZ70qrGomWVEC27U/preview#heading=h.21w4dooci0sc)

Due to the way COVIDSafe uses Bluetooth, some standard Bluetooth services are enabled. One of these services makes the the device name accessible to remote devices. For example, this name would be shown when pairing with a car stereo.

At best, this name defaults to the model name, but in many cases the user will have customised it, allowing for re-identification of the device, and potentially identifying the owner's name.

An attempt at a fix was made, where the Android app will prompt the user to change their device name. However this UI is confusing, it only applies to new users, and only on Android.

### 5. Undetectable, permanent long-term tracking of Android devices
Status: Fixed   
Type: Privacy   
Affects: Android   
More info: [CVE-2020-12856](https://github.com/alwentiu/COVIDSafe-CVE-2020-12856), [Summary thread](https://twitter.com/jim_mussared/status/1273849252985233408), [Video](https://www.youtube.com/watch?v=tlWqLVfdzG0)

An attacker could trigger a silent pairing process with an Android phone running COVIDSafe. This is completely invisible to the user, no prompt is shown. Once pairing has been completed, the target phone can be re-identified even after COVIDSafe has been un-installed and the phone factory reset.

This was registered as CVE-2020-12858 which was assigned a ["Critical" 9.8 out of 10 severity rating](https://nvd.nist.gov/vuln/detail/CVE-2020-12856).

Note: It is unclear whether the fix for this will still be viable past November 2020 when the Google Play Store requires API level 29.

### 6. Ability to remote control an Android device running COVIDSafe
Status: Fixed   
Type: Security, Privacy   
Affects: Android   
More info: [CVE-2020-12856](https://github.com/alwentiu/COVIDSafe-CVE-2020-12856), [Summary thread](https://twitter.com/jim_mussared/status/1273849252985233408), [Video](https://www.youtube.com/watch?v=YZWUmQR5810)

Related to the previous silent pairing issue, once paired, an attacker can then invisibly "profile switch" and pretend to be a different Bluetooth device, e.g. a keyboard or headphones, allowing remote control of the phone.

### 7. 🚨 Permanent long-term tracking of Android & iOS devices 🚨
Status: **Partial mitigation**   
Type: Privacy   
Affects: Android & iOS   
More info: [CVE-2020-12856](https://github.com/alwentiu/COVIDSafe-CVE-2020-12856), [Summary thread](https://twitter.com/jim_mussared/status/1273849252985233408)

A non-silent version of the above issue, where a prompt is shown that says "COVIDSafe would like to pair with your phone". If the user clicks "Pair", then the above two issues become possible.

Both the Android and iPhone apps now include a prominent message that "COVIDSafe does not send pairing requests".

### 8. 🚨 Undetectable, permanent long-term tracking of Android devices 🚨
Status: **Partially fixed**   
Type: Privacy   
Affects: Android   
More info: Coming soon

An attacker could trick COVIDSafe into connecting with the phone's identity address, rather than the random address normally used. This identity address then permanently identifies the phone, and like #5, allows for re-identifying the device even after COVIDSafe is uninstalled and the phone is factory reset.

This is only fixed when running on Android 6.0 and higher.

### 9. Remote code execution and ability to crash system Bluetooth service on Android
Status: Fixed   
Type: Security, Privacy   
Affects: Android   
More info: [CVE-2020-0022](https://insinuator.net/2020/04/cve-2020-0022-an-android-8-0-9-0-bluetooth-zero-click-rce-bluefrag/)

Once the identity address of the target phone was known (via #5 or #8), any phone running Android 9.0 or lower is now vunerable to BlueFrag ([CVE-2020-0022](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-0022)), which allows remote code execution, remote memory access, and ability to crash the system Bluetooth service.

### 10. Remote crash of iPhone app
Status: Fixed   
Type: Security, Functionality   
Affects: iOS   
More info: [Blog post](https://medium.com/@wabz/covidsafe-ios-vulnerability-cve-2020-12717-30dc003f9708), [Video](https://photos.app.goo.gl/btmoD1DfrMGKWwpR8)

An attacker could create a beacon that would remotely crash the COVIDSafe app on any nearby iPhone. The beacon would transmit a malformed advertising payload that the app would fail to handle.

### 11. iPhone app only functions in the foreground
Status: Fixed   
Type: Functionality   
Affects: iOS   
More info: [Blog post (part 1)](https://medium.com/@wabz/the-broken-covidsafe-ios-application-c652d0a462c4), [Blog post (part 2)](https://medium.com/@wabz/the-unbroken-ios-covidsafe-application-dea520af3694)

Despite claims to the contrary that this had been fixed relative to the Singapore app, the iPhone app could not function in the scanning role while the app was not in the foreground. It would schedule a timer to stop scanning, and then never get an opportunity to start scanning again. This was fixed by no longer stopping the scan.

### 12. iPhone app cannot function while locked
Status: Fixed   
Type: Functionality   
Affects: iOS   
More info: [Issue report](https://docs.google.com/document/d/1dsSxC48cJ91X17PoOybpun1U163YDxxL0CDk3kmAHvY/preview), [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md#locked-iphones-cannot-receive-new-tempids)

The app downloads a new tracing ID every two hours, but in order to do so it needs to access the authentication token in the device keychain which it uses to talk to the Amazon server. When the token was first created, it was not stored in a way that allowed access while the device was locked.

### 13. 🚨 iPhone app prevents new connections after 100 exchanges 🚨
Status: **Not fixed**   
Type: Functionality   
Affects: iOS   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-ios/issues/9)

Once a remote device is found during a scan, the app will then attempt to connect and record an encounter with it every 15 seconds. This allows the duration of the encounter to be inferred, even though the background-mode scanning will not find the same device multiple times.

These connection attempts do not time out, and so for every device that goes out of range the phone will remain in an "attempting to connect" state to that device. In addition, phones change their address every few minutes, which means that any nearby phone will appear to go out of range every few minutes.

After about 100 pending connections, the phone becomes unable to connect to new devices. This prevents further encounters from being recorded, but also prevents other apps from initiating connections to other BLE devices (e.g. smart watches, diabetes continuous glucose monitoring, etc).

### 14. iPhone app can't exchange messages as expected with other iPhones
Status: Fixed   
Type: Functionality   
Affects: iOS   
More info: [Earlier blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md#iPhoneToiPhoneBug), [GitHub issue on errors in the attempted fix](https://github.com/AU-COVIDSafe/mobile-ios/issues/11)

When the V2 payload was introduced (see #3 above), the iPhone implementation incorrectly truncated the new messages, resulting in the encounters being silently ignored and not logged to the database. Fortunately due to redundancy in the design, there is a second mechanism where the payload is exchanged, but this bug overall decreases the reliability of the contact tracing functionality.

A fix was attempted, however it contains a new bug, which means that while the encounters are now logged, they are actually corrupted, which means they will not be able to be decrypted by the server.

This was fixed in v1.8 for iOS.

### 15. Android app can't discover background-mode iPhones
Status: Fixed   
Type: Functionality   
Affects: Android   
More info: [Issue report](https://docs.google.com/document/d/1u5a5ersKBH6eG362atALrzuXo3zuZ70qrGomWVEC27U/preview#heading=h.1ch70d1huf4o), [Technical details](http://www.davidgyoungtech.com/2020/05/07/hacking-the-overflow-area)

Whilst operating in the background, the iOS app will send a differently-formatted BLE advertising payload. The Android app only understood the foreground-mode one, and so would not detect background-mode iPhones. In conjunction with iPhones not being able to operate in the scanning role in the background (#11), this made it almost impossible for any sort of encounter to be detected with or by an iPhone in the background.

This was fixed, however the fix doesn't handle a case where multiple apps are running on the iPhone that each advertise a different BLE service. However this is likely a very rare scenario.

### 16. App logo stops spinning
Status: Fixed   
Type: Usability   
Affects: Android   
More info: [GitHub PR](https://github.com/AU-COVIDSafe/mobile-android/pull/5)

The logo animation did not resume after a user opens and then dismisses the "share" screen.

### 17. Tracing ID expiry is too long
Status: Fixed   
Type: Privacy   
Affects: Android & iOS
More info: [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md)

The COVIDSafe app uses a two-hour expiry for the tracing ID. This is unnecessarily long, and goes against the recommendations from the Singapore team. This allowed the device to be re-identified for a long enough window to, for example, track a user's movement around a shopping centre.

This has been mitigated in the V2 protocol by adding an extra layer of encryption. The tracing ID still lasts for two hours, but the encryption key changes every 7.5 minutes.

### 18. App writes corrupted payloads to encounter log
Status: Fixed   
Type: Functionality   
Affects: Android   
More info: [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md#critical-concurrency-bug-in-encryption-for-covidsafe-on-android)

The implementation of the V2 protocol had a concurrency bug, where simultaneous encounters would result in corrupted entries being written to the log.  It also crashed the app.

### 19. 🚨 Plaintext counter leaks information 🚨
Status: **Not fixed**   
Type: Privacy   
Affects: Android and iOS   
More info: [Blog post](https://github.com/vteague/contactTracing#implications-of-a-plaintext-counter)

The implementation of the V2 protocol uses a counter which inadvertently leaks information about the number of encounters that the device has recorded in the last 7.5 minutes.

### 20. Confusing message tells users they have tested positive to COVID-19
Status: Fixed   
Type: Usability   
Affects: Android   
More info: [COVIDSafe.watch](https://covidsafe.watch/issue-register/you-have-covid-text-caused-public-panic), [9 News](https://www.9news.com.au/national/covidsafe-app-melbourne-woman-feared-coronavirus-after-confusing-message/e9146501-6bbd-4509-b89a-406b2b98ed2a)

This was discovered by the community by inspecting the app's resources was reported as a potential issue very soon after launch. Then within a few days, there were news reports of people being confused by this.

### 21. 🚨 Identifier linking allowing re-identification of devices 🚨
Status: **Not fixed**   
Type: Privacy   
Affects: Android & iOS   
More info: [Issue report](https://docs.google.com/document/d/1u5a5ersKBH6eG362atALrzuXo3zuZ70qrGomWVEC27U/preview#heading=h.4wtj9wjrhx8b), [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md#background-on-mac-address-and-tempid-based-phone-tracking)

Any time where more than one identifier is accessible, even if they change frequently, they must all change at the same time otherwise they can be linked together. There are several known identifiers in COVIDSafe that change at different intervals:

* BLE MAC address
* BLE advertisting payload
* Tracing ID
* Phone model
* Phone name
* Encryption counter

Avoiding this issues is explicitly called out in the design of the Apple/Google Exposure Notification API.

### 22. App fails to download new tracing IDs
Type: Functionality, Privacy   
Status: Fixed   
Affects: Android & iOS   
More info: [Issue report (iOS)](https://docs.google.com/document/d/1iJGShYSOmo1ngKy8V-cW2jmwsvP1hLt7f54HZFcQSVw/preview?usp=sharing)

There were numerous issues discovered by the community in the handling of downloading and expiring tracing IDs. This resulted in scenarios where the app would regularly use the same tracing ID for longer than the two-hour expiry time.

This created a privacy issue, allowing for tracking and re-identification of devices, but also a functionality issue, as expired tracing IDs will not be counted as valid encounters.

### 23. The Android app silently fails to function if "Location" is disabled
Status: Fixed (pending UI changes)   
Type: Functionality, Privacy   
Affects: Android   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-android/issues/6)

Privacy-conscious users may disable the "Location" global setting on Android. As Bluetooth scanning allows an app to detect the presence of positioning beacons, having location disabled prevents an app from being able to scan for BLE devices.

COVIDSafe fails to detect this condition and will silently fail to scan for other devices.

The core issue was fixed in v1.0.39 however there are some [UI problems that need to be addressed](https://github.com/AU-COVIDSafe/mobile-android/issues/13).


###  24. 🚨 The app doesn't auto-update 🚨
Status: **Not fixed**   
Type: Functionality, Privacy, Usability, Security   
Affects: Android   
More info: [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md#the-app-does-not-automatically-update)

The Android app is unable to automatically update to new versions published in the play store.

The cause of this is not currently known, however it's likely because the app is always active while in the background, so the same logic that prevents a podcast player from updating mid-playback possibly applies.

There are also reports that a similar issue is affecting the iPhone app.

**This means that many users are still running the first version of COVIDSafe they installed, and will have not received any of the fixes described above.  So even the issues that are "fixed" in principle in the current version may not actually be fixed in practice on users' phones.**

v1.0.39 for Android and v1.8 for iOS add support for push notifications (using Firebase Cloud Messaging and Apple Notifications respectively). However, the Android app has not yet implemented support for a "need to update" notification.


### 25. 🚨 Android app loses location permission after 1.0.39 update 🚨
Status: **Not fixed**   
Type: Functionality, Usability   
Affects: Android   
More info: [GitHub Issue](https://github.com/AU-COVIDSafe/mobile-android/issues/14)

The v1.0.39 release changed from using Android's "fine" location permission, to "coarse". This is a good change -- the app never needed "fine" location, so "coarse" is what it should be using. However, due to the way the change was made, after the update the app will likely have no location permission at all until the user manually opens the app to grant the new "coarse" permission.

There is a notification shown, however it's not a new notification, rather it just updates the text of the existing "COVIDSafe is running" notification, so there is no pop up. Additionally, the icon (which is the most obvious part) does not change. **This means that many users will not notice that the app is now unable to scan for other devices running COVIDSafe.**

*Although the app doesn't actually use location, this permission (either "coarse" or "fine") is required to enable BLE scanning.*

Fortunately, due to #24 above, it's likely that many users will not yet have received this update.


## Recommendations

### Apple/Google Exposure Notification API

The [Google/Apple Exposure Notification API](https://www.apple.com/covid19/contacttracing) uses Bluetooth in a very different way. If COVIDSafe had been designed to use the Exposure Notification API, it would very likely have prevented all of the functionality, privacy and security issues described above.

The Apple/Google Exposure Notification API is not perfect, and does have its own privacy implications.  For example, Android requires Google location services in order to enable the API, an unnecessary invasion of users' location privacy (though one that is irrelevant to users who already turn Google location on).  Compared with a centralised system like COVIDSafe, it also gives users more information about when they interacted with those who have tested positive, potentially increasing the likelihood that the source of infection could be identified and tracked during the time they were assumed infectious.  However, the Google/Apple API has two overwhelming advantages: better function, and better privacy from government.  It is important to understand that this is not because the database of close contacts is handed to the tech companies: the Google/Apple API is designed not to build that database.

Human contact tracers can be involved in an app based on the Google/Apple API.  The [Irish app](https://www.irishtimes.com/business/technology/what-is-covid-tracker-ireland-1.4298128) allows users to opt in to giving their phone number so they can be contacted by a person if they are identified as having been exposed.  The difference between this and COVIDSafe is that the authority does not receive each infected person's list of face-to-face contacts, just a notification that a particular person has been exposed.

Although there are pros and cons, there does not seem to be any reasonable analysis or understanding in government of why a centralised model has been chosen. Consider for example Minister Robert's [recent attempt to explain](https://minister.servicesaustralia.gov.au/transcripts/2020-07-07-q-and-speech-national-press-club):
> *there's an opportunity I think here for the big tech companies to lock step in with sovereign governments and assist them with their sovereign approach to doing tracing. Remember, digital tracing simply enhances a manual tracing process. The big tech companies with their exposure notification framework are saying that digital tracing unto itself is enough. The global experience shows quite clearly it is not enough.*

The world is headed for a large and not very well-controlled empirical test, so we will soon see evidence that allows us to compare the countries with decentralised apps based on the Apple/Google API (such as Italy, Germany, Switzerland, Ireland and the UK) against those that stick with a centralised model.  The Australian authorities are choosing to emphasise centralised data gathering, knowing that this carries a cost for basic successful functioning (for all the reasons described above).  The global experience will soon show whether this was wise.


### The server code

The source code for the the COVIDSafe server code has not been released for public inspection. For more information, please see [The missing server code, and why it matters](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)

### Engagement numbers

The update of COVIDSafe has been communicated entirely in terms of "number of downloads". This is a very misleading figure, as it coveys no information about how many people are actually using the app.

The industry standard is to report "active users" over some time period. i.e. "1-day active users" are the number of unique users who are actively using the app in a 24-hour period.

As the app downloads a new tracing ID from the AWS server every two hours, using the user's unique authentication token, the server is trivially able to generate this information.

### Formal security reporting process and bug bounty

At launch there was no process for the community to report serious issues, nor has there been any management of issue disclosure.

The DTA should set up a formal process for working with security researchers for this and all future projects. A bug bounty program would be an excellent way to implement this.

The legislation created to support the COVIDSafe app, known as the Privacy Amendment (Public Health Contact Information) Act, also offers no protections to security researchers. Furthermore, the source code has been released under terms that are not conducive to community engagement.

### Auto-updates

The DTA should add code to the app to report the current version to the server when downloading the two-hourly tracing ID, allowing the server to respond with a message telling the app that it needs to be updated.

In addition, this would provide better metrics to the DTA on whether users are receiving updates.

## Acknowledgments

We'd like to thank the large and active community of Australian techies who have examined, discussed, and tried to correct the code. In particular the following people (in no particular order) were involved in discovering the issues described above:

* Alwen Tiu
* Chris Culnane
* Eleanor McMurtry
* Ben Frengley
* Geoffrey Huntley
* Hubert Seiwert
* Jim Mussared
* John Evershed
* Manabu Nakazawa
* Richard Nelson
* Robert Merkel
* Vanessa Teague
* Yaakov Smith

### Followup and reuse

Comments, edits, suggestions and pull requests are welcome.

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link:
[https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md).
