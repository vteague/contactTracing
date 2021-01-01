This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [Contact Tracing and Consent (23 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
- [Tracing the Challenges of COVIDSafe (27 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md) by Chris Culnane, Eleanor McMurtry, Robert Merkel and me.
- [The Missing Server Code and why it Matters (14 May)](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)  by Robert Merkel, Eleanor McMurtry and me.
- [Security Analysis of the UK's NHS Contact Tracing App](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-19UKContactTracing.md) by Chris Culnane and me.
- [COVIDSafe's new payload encryption scheme (15 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-15COVIDSafesNewEncryptionScheme.md) by Chris Culnane, Ben Frengley, Eleanor McMurtry, Jim Mussared, Yaakov Smith, Alwen Tiu and me.
- [Issues with COVIDSafe's new encryption scheme (19 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md) by the same authors.
- [The current state of COVIDSafe (mid-June 2020) (22 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md) by the same authors.
- [**COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2021) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys (16 Dec)](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)

---------------------------------------

Jim Mussared: jim.mussared [at] gmail.com / [@jim_mussared](https://twitter.com/jim_mussared)   
Vanessa Teague: [ThinkingCybersecurity Pty Ltd](https://www.thinkingcybersecurity.com) / [@VTeagueAus](https://twitter.com/vteagueaus)

Last updated: Jan 1 2021, for Android v2.0 / iPhone v2.1.1. New fixes are noted below.

# COVIDSafe issues found by the tech community

## Introduction

COVIDSafe was launched on April 26th, 2020. Several volunteers from the Australian tech community have spent considerable amounts of time analysing the code of the Android and iOS apps in order to help improve their contact tracing functionality and privacy.

Many issues have been found as well as recommendations for how to fix them, and some of these fixes have dramatically improved the effectiveness of the COVIDSafe app. In addition, several of these issues have been found and reported in contact tracing apps used in other countries.

Most, but not all, of these issues have been fixed.  However, due to a quirk in the way that COVIDSafe works, it is not clear that users are actually receiving automatic updates to the app.  If you haven't checked you are running the most recent version, you should check manually and update now.

Oct 28th 2020: The most important message for users is for Issue 25: **users need to open the app, check that its location permissions are OK (have a green tick) and, if not, grant location permission to the app (again).** Note that leaving the global location setting off also prevents scanning - see Issue 23.

Jan 1st 2021: COVIDSafe has been update to use the [Herald](https://vmware.github.io/herald/) framework for Bluetooth communications. Several issues were (re-)introduced during this migration.

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

Note: A new workaround was applied in Version 1.13 which does not have a known expiry date.

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

### 8. Undetectable, permanent long-term tracking of Android devices
Status: Fixed in v1.0.49   
Type: Privacy   
Affects: Android   
More info: [CVE-2020-14292. Identity address leakage through bluetooth transport](https://github.com/alwentiu/CVE-2020-14292)

An attacker could trick COVIDSafe into connecting with the phone's identity address, rather than the random address normally used. This identity address then permanently identifies the phone, and like #5, allows for re-identifying the device even after COVIDSafe is uninstalled and the phone is factory reset.

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

### 13. iPhone app prevents new connections after 100 exchanges
Status: Fixed in v1.9   
Type: Functionality   
Affects: iOS   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-ios/issues/9)

Once a remote device is found during a scan, the app will then attempt to connect and record an encounter with it every 15 seconds. This allows the duration of the encounter to be inferred, even though the background-mode scanning will not find the same device multiple times.

These connection attempts do not time out, and so for every device that goes out of range the phone will remain in an "attempting to connect" state to that device. In addition, phones change their address every few minutes, which means that any nearby phone will appear to go out of range every few minutes.

After about 100 pending connections, the phone becomes unable to connect to new devices. This prevents further encounters from being recorded, but also prevents other apps from initiating connections to other BLE devices (e.g. smart watches, diabetes continuous glucose monitoring, etc).

An fix was attempted in v1.8 but the issue persisted. It was fixed completely in v1.9.

### 14. iPhone app can't exchange messages as expected with other iPhones
Status: Fixed in v1.8.  
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

### 19. Plaintext counter leaks information
Status: Fixed   
Type: Privacy   
Affects: Android and iOS   
More info: [Blog post](https://github.com/vteague/contactTracing#implications-of-a-plaintext-counter)

The implementation of the V2 protocol used a counter which inadvertently leaked information about the number of encounters that the device has recorded in the last 7.5 minutes. This was corrected by substituting a small random number instead of a counter.  This is not a standard use of the cryptographic primitives involved.

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
Status: Fixed in v1.0.39 (pending UI changes)   
Type: Functionality, Privacy   
Affects: Android   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-android/issues/6)

Privacy-conscious users may disable the "Location" global setting on Android. As Bluetooth scanning allows an app to detect the presence of positioning beacons, having location disabled prevents an app from being able to scan for BLE devices.

COVIDSafe fails to detect this condition and will silently fail to scan for other devices.  This prevents two Android phones that both have "Location" off from exchanging TempIDs with each other or with any phone affected by Issue 25. However, it should not prevent an Android phone with "Location" off from successfully exchanging TempIDs both ways in the peripheral role, with another phone that is either an iPhone or an Android phone with "Location" on (and not affected by Issue 25). 

The core issue was fixed in v1.0.39 however there are some [UI problems that need to be addressed when "Location" is off](https://github.com/AU-COVIDSafe/mobile-android/issues/13).
The main screen tells users that it is not active, when it is active and working in most circumstances, as described above.

###  24. The app didn't auto-update 
Status: Uncertain   
Type: Functionality, Privacy, Usability, Security   
Affects: Android   
More info: [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md#the-app-does-not-automatically-update)

The Android app was unable to automatically update to new versions published in the play store.
There were also reports that a similar issue is affecting the iPhone app.

v1.0.39 for Android and v1.8 for iOS add support for push notifications (using Firebase Cloud Messaging and Apple Notifications respectively). v1.8 for iOS and v1.0.48 for Android then implemented a "need to update" notification. 

**We are not certain of the current status of this problem. Users should probably still check manually that they have received the most recent update.**

Early reports suggest that since the change to use Herald, that this now possibly the iPhone app. No push notifications have been sent yet.


### 25. Android app loses location permission after 1.0.39 update
Status: Fixed   
Type: Functionality, Usability   
Affects: Android   
More info: [GitHub Issue](https://github.com/AU-COVIDSafe/mobile-android/issues/14)

The v1.0.39 release changed from using Android's "fine" location permission, to "coarse". This is a good change -- the app never needed "fine" location, so "coarse" is what it should be using. However, due to the way the change was made, after the update the app will likely have no location permission at all until the user manually opens the app to grant the new "coarse" permission.

The v1.13 release changed back to "fine" location permission.  This means **users need to repeat the process: Open the app, check that its location permissions are OK (have a green tick) and, if not, grant location permission to the app (again).** (Note that leaving the global location setting off also prevents scanning - see Issue 23).

There is a notification shown, however it's not a new notification, rather it just updates the text of the existing "COVIDSafe is running" notification, so there is no pop up. Additionally, the icon (which is the most obvious part) does not change. This means that users may not notice that the app is now unable to scan for other devices running COVIDSafe.

*Although the app doesn't actually use location, this permission (either "coarse" or "fine") is required to enable BLE scanning.*

It's confusing to users that the app suddenly seems to require location permission, and might make it seem like the new version is now introducing additional location-based functionality (which it isn't).

This change was then reverted a couple of months later, leading to the exact same confusion.


### 26. Can't click "continue" in Android registration screen   
Status: Fixed   
Type: Functionality, Usability   
Affects: Android   
More info: [GitHub Issue](https://github.com/AU-COVIDSafe/mobile-android/issues/17)

A user interface issue prevented the "Continue" and "Get Pin" buttons from being pressed during the registration screen. It's not intuitive how to work around this issue.


### 27. App silently doesn't function on some Android 5.1/6.0/7.0 devices   
Status: Cannot be fixed   
Type: Functionality   
Affects: Android   
More info: [GitHub Issue](https://github.com/AU-COVIDSafe/mobile-android/issues/18)

Although the COVIDSafe app supports phones running versions of Android all the way back to Android 5.1, some phones running 5.1, 6.0, and 7.0 do not have the Bluetooth functionality required to run COVIDSafe. The app does not detect this and appears like everything is working, when app cannot actually be detected by other phones.

The Herald migration is supposed to address this, but the result is no different. The system continues to rely on the older phones finding newer phones, so it is not possible for two old phones to exchange details. Herald has a feature that allows a third phone to act as a "proxy" but this feature is not enabled in COVIDSafe.


### 28. Android app can corrupt its registration token leading to crash on startup
Status: Likely fixed (no new reports)   
Type: Functionality, Usability  
Affects: Android   
More info: [GitHub Issue](https://github.com/AU-COVIDSafe/mobile-android/issues/23)

A Twitter user [noticed in early July](https://twitter.com/_the_culture/status/1281505842026577920) that the COVIDSafe app was crashing on startup. This was eventually noticed by several more people and reported privately to the DTA, but with no response.

If affected by this issue, the user must uninstall and reinstall the app to recover (or manually remove all app data). Unfortunately this results in having to re-register and loses all encounter history. However, the more likely outcome is that people will just stop using the app.

The root cause and suggested fix is complicated, see the GitHub issue linked above for details. In summary this is in large part due to COVIDSafe using an unstable version of an Android library, copied into the COVIDSafe code rather than used as a library. At the very least this dependency should have been kept up to date and moved to the released version, which would have avoided this being an unrecoverable error.

We are not sure of the current status of this problem.  There seems to be an attempted fix, but we are not sure whether it solves the underlying problem or merely prevents it causing a crash.


### 29. Insecure random number generator leads to device tracking   
Status: Fixed   
Type: Privacy   
Affects: Android   
More info: [Blog post](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md#privacy), [GitHub issue](https://github.com/vmware/herald-for-android/issues/110)   

This is almost identical to issue #1 above, and was introduced in the migration to Herald due to Herald's use of an [insecure random number generator](https://owasp.org/www-community/vulnerabilities/Insecure_Randomness).

This was briefly [fixed upstream in Herald](https://github.com/vmware/herald-for-android/pull/91), and then the fix was [replaced](https://github.com/vmware/herald-for-android/issues/81) with a new use of an insecure random number generator, and this code is still broken upstream.

COVIDSafe have applied a temporary fix to use Android's `SecureRandom`, and fortunately this was included in version 2.0. There are [suggestions from the Herald team](https://github.com/vmware/herald-for-android/issues/109#issuecomment-744508712) that maybe this will lead to the app stopping working on some phones.


### 30. iPhone app prevents Bluetooth from making new connections   
Status: Fixed in v2.1.1   
Type: Functionality   
Affects: iOS   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-ios/issues/38)   

This is identical to #13 above, and was re-introduced in the Herald migration (version 2.0), due to the way that Herald manages connection to iPhones.

This prevented the app from functioning at all on iPhones, and also prevented other Bluetooth functionality from working (such as wireless headphones, medical devices, remote controls).

An emergency patch (version 2.1.1) was released on Dec 23.


### 31. 🚨 Android app registers multiple Bluetooth services 🚨
Status: Not fixed   
Type: Functionality   
Affects: Android   
More info: [GitHub issue (COVIDSafe)](https://github.com/AU-COVIDSafe/mobile-android/issues/33), [GitHub issue (Herald)](https://github.com/vmware/herald-for-android/issues/107)

Since the Herald migration, the Android app uses the Bluetooth interface in a complicated way that can result in the phone registering the COVIDSafe service multiple times. This is particularly easy to trigger when enabling/disabling the Bluetooth functionality on the phone.

The effect is that the phone will appear to look like multiple phones to other nearby phones, which leads to more crowding issues (lower detection efficiency) and higher battery drain. Additionally, after registering the service too many times, the app stops functioning.

After this issue was raised, an attempted fix was included in version 2.0, but it did not solve the issue.


### 32. 🚨 COVIDSafe is a significant battery drain 🚨
Status: Not fixed   
Type: Functionality, Usability   
Affects: Android & iOS   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-ios/issues/31)

There have been numerous reports of COVIDSafe being a serious battery drain, and the Herald update has made this worse. This is driving people to uninstall the app.

Inspection of the Herald code shows many reasons why this is the case -- the app connects to all nearby Apple devices, not just ones running COVIDSafe, as well as very aggressive connection keep-alive and re-connections. Additionally, the phone is prevented from properly entering low power mode.

Even the [DTA's own battery testing](https://www.dta.gov.au/news/covidsafe-captures-close-contacts-new-herald-protocol) shows some fairly extraordinary numbers for battery use for a "background" app.


### 33. 🚨 Location access is confusing users 🚨
Status: Not fixed   
Type: Functionality, Usability   
Affects: iOS  
More info:   GitHub issues: [32](https://github.com/AU-COVIDSafe/mobile-ios/issues/32), [38](https://github.com/AU-COVIDSafe/mobile-ios/issues/38), [34](https://github.com/AU-COVIDSafe/mobile-ios/issues/34), [30](https://github.com/AU-COVIDSafe/mobile-ios/issues/30), [29](https://github.com/AU-COVIDSafe/mobile-ios/issues/29#issuecomment-736531034)   

The Herald update (version 2.0) now requires that the iOS app is granted permission for location access, and iOS will now periodically tell the user how many times COVIDSafe "access the location in the background".

The app doesn't actually use location data, rather the location access is used to "wake up" the app. However there is very little reason for a user to understand this distinction, and a user cannot easily verify this for themselves.

This was raised before release, but was deemed unimportant. However, this has also driven more people to uninstall the app.


### 34. 🚨 Herald reduces connection efficiency 🚨
Status: Not fixed   
Type: Functionality
Affects: Android & iOS   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-android/issues/32)

Herald only exchanges tracing data in one direction for a given connection (with some exceptions). COVIDSafe originally was able to share this data in both directions, meaning that a single connection was sufficient for two phones to record an encounter. Since the Herald update, it now requires a second connection to be established in the reverse direction.

This reduces the efficiency of encounter recording, especially in crowded environments.


### 35. 🚨 Remote denial-of-service attack 🚨
Status: Not fixed   
Type: Security, Functionality   
Affects: Android & iOS   
More info: _coming soon_

Similar to #10 above, two different remote crash vulnerabilities have been identified, affecting both the iOS and Android apps. These allow an attacker to disable the app on any phone in Bluetooth range.

These bugs were introduced during the Herald migration (version 2.0), and are in the integration code, not in Herald itself. They were reported to the DTA on 21 Dec 2020 and are still unfixed. A fix was not attempted in the 2.1.1 update for issue #30 above.


### 36. 🚨 Undetectable, permanent long-term tracking of older Samsung Android devices 🚨
Status: Not fixed   
Type: Security, Privacy   
Affects: Samsung phones running Android 7.1.1 and earlier   
More info: [CVE-2020-35693: A silent pairing pairing vulnerability affecting some Samsung devices](https://github.com/alwentiu/contact-tracing-research/blob/main/samsung.pdf)   

Similar to #7 and #8 above, a vulnerability in Samsung's Bluetooth stack allows an attacker to silently pair with a phone running COVIDSafe. Samsung have acknowledged the issue but they do not plan to issue a patch. However, unlike #7 and #8, there is no workaround possible for COVIDSafe. The DTA have not acknowledged the issue after multiple reports.


### 37. 🚨 COVIDSafe cannot measure distance (e.g. 1.5 metre threshold) 🚨
Status: Not fixed   
Type: Functionality, Privacy   
Affects: iOS & Android   
More info: [GitHub issue](https://github.com/AU-COVIDSafe/mobile-android/issues/31)

Since launch, COVIDSafe has claimed to be able to detect exposures of "less than 1.5 metres for 15 minutes". There has never been any evidence to support this claim, either from the DTA or from any other contact tracing teams.

On the contrary, many experts have repeatedly pointed out that Bluetooth cannot be used to accurately measure distance, as there are far too many complicated interactions and real-world considerations to take into account. At best, it can only be used to get a very crude estimate of "proximity".

The migration to Herald complicates this because the Herald project makes similarly strong claims about their ability to measure distance (despite not actually having any distance measurement functionality).


## Recommendations

### Apple/Google Exposure Notification API

The [Google/Apple Exposure Notification API](https://www.apple.com/covid19/contacttracing) uses Bluetooth in a very different way. If COVIDSafe had been designed to use the Exposure Notification API, it would very likely have prevented all of the functionality, privacy and security issues described above.

The Apple/Google Exposure Notification API is not perfect, and does have its own privacy implications.  For example, Android requires Google location services in order to enable the API, an unnecessary invasion of users' location privacy (though one that is irrelevant to users who already turn Google location on).  Compared with a centralised system like COVIDSafe, it also gives users more information about when they interacted with those who have tested positive, potentially increasing the likelihood that the source of infection could be identified and tracked during the time they were assumed infectious.  However, the Google/Apple API has two overwhelming advantages: better function, and better privacy from government.  It is important to understand that this is not because the database of close contacts is handed to the tech companies: the Google/Apple API is designed not to build that database.

Human contact tracers can be involved in an app based on the Google/Apple API.  The [Irish app](https://www.irishtimes.com/business/technology/what-is-covid-tracker-ireland-1.4298128) allows users to opt in to giving their phone number so they can be contacted by a person if they are identified as having been exposed.  The difference between this and COVIDSafe is that the authority does not receive each infected person's list of face-to-face contacts, just a notification that a particular person has been exposed.

Apple have recently added support for iOS 12.x devices, so the weak excuse that not enough phones support it is no longer even remotely valid.

Although there are pros and cons, there does not seem to be any reasonable analysis or understanding in government of why a centralised model has been chosen. Consider for example Minister Robert's [recent attempt to explain](https://minister.servicesaustralia.gov.au/transcripts/2020-07-07-q-and-speech-national-press-club):
> *there's an opportunity I think here for the big tech companies to lock step in with sovereign governments and assist them with their sovereign approach to doing tracing. Remember, digital tracing simply enhances a manual tracing process. The big tech companies with their exposure notification framework are saying that digital tracing unto itself is enough. The global experience shows quite clearly it is not enough.*

The world is headed for a large and not very well-controlled empirical test, so we will soon see evidence that allows us to compare the countries with decentralised apps based on the Apple/Google API (such as Italy, Germany, Switzerland, Ireland and the UK) against those that stick with a centralised model.  The Australian authorities are choosing to emphasise centralised data gathering, knowing that this carries a cost for basic successful functioning (for all the reasons described above).  The global experience will soon show whether this was wise.

### The server code

The source code for the the COVIDSafe server code has not been released for public inspection. For more information, please see [The missing server code, and why it matters](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)

### Engagement numbers

The update of COVIDSafe has been communicated entirely in terms of "number of downloads". This is a very misleading figure, as it coveys no information about how many people are actually using the app.

The industry standard is to report "active users" over some time period. i.e. "1-day active users" are the number of unique users who are actively using the app in a 24-hour period.

As the app downloads a new tracing ID from the AWS server every hour, using the user's unique authentication token, the server is trivially able to generate this information.

### Formal security reporting process and bug bounty

At launch there was no process for the community to report serious issues, nor has there been any management of issue disclosure.

The DTA should set up a formal process for working with security researchers for this and all future projects. A bug bounty program would be an excellent way to implement this.

The legislation created to support the COVIDSafe app, known as the Privacy Amendment (Public Health Contact Information) Act, also offers no protections to security researchers. Furthermore, the source code has been released under terms that are not conducive to community engagement.

### Auto-updates

The DTA should add code to the app to report the current version to the server when downloading the tracing ID (every hour), allowing the server to respond with a message telling the app that it needs to be updated.

In addition, this would provide better metrics to the DTA on whether users are receiving updates.

Update: As of 1.0.49 for Android and v1.9 for iOS (early August):
* Both iOS and Android apps support a push notification to tell the user to update.
* The tracing ID is now downloaded daily instead of hourly.
* The tracing ID request now includes a flag indicating that the app is at least v1.0.49/v1.9 (but does not include the actual version number).

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
