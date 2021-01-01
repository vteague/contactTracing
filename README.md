This is the latest in a series of blog posts on automated contact tracing.

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
- [COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2020)](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [**Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys - this post (16 Dec, updated 19 Dec)**](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)
---------------------------------------

# Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys

Vanessa Teague: [Thinking Cybersecurity Pty Ltd](https://www.thinkingcybersecurity.com)   / [@VTeagueAus](https://twitter.com/vteagueaus)   
and the [Australian National University](https://researchers.anu.edu.au/researchers/teague-v)   

Update 19 Dec: Apple shuffling information.

## Summary
The Google-Apple Exposure Notification (GAEN) system is designed for decentralised notifications of possible exposure to infection.  Each user's device regularly downloads the Diagnosis Keys of people who have tested positive for an infection, then compares this information with its own log of contacts.  This produces a list of exposure events, called ExposureInformation, which gives a date of exposure (without the exact time), a duration (capped at 30 mins) and some other details.  From this, the device estimates its user's risk of infection and makes a decision about whether to notify them.

After adopting this basic decentralised design, some countries have encouraged users to opt in to sharing some of the information that was intended for local use on their phones.  Although I am not aware of any app that encourages users to upload ExposureInformation, I see no prohibition against doing so on an opt-in basis.  The Irish app, for example, uploads ExposureSummary - a much less detailed risk 
estimate - from those users who have opted in.

This blog post describes an attack that could have allowed a malicious authority to infer edges in the social graph through the Google-Apple Exposure Notification API, in very specific circumstances described below.  The main problem was that the client did not shuffle its detailed list of Exposure Information, which caused a problem if the person opted in to uploading it.  

I am confident that Google's implementation is no longer vulnerable - on August 26, three weeks after I notified them, they published [a file that clearly implements the necessary shuffle](https://github.com/google/exposure-notifications-internals/blob/main/exposurenotification/src/main/java/com/google/samples/exposurenotification/matching/ExposureWindowUtils.java#L66) and said that the relevant fix had been applied in June (before I notified them), which is entirely credible.  Apple has also confirmed by email 
that their implementation shuffles in iOS versions 14 and later.

Although no longer an active issue for GAEN, I hope this potential pitfall has some explanatory value and  may help countries such as New Zealand refine their own implementation for residents who don't have GAEN-enabled devices.

Hiding social connections from a central authority is one of the main goals of the decentralised approach.

---

## Introduction


We consider an attack scenario in which an adversary who controls the government server in a GAEN system also reads ExposureInformation from users who opt in to this upload when they are notified of exposure.  We show that ExposureInformation can expose edges in the social graph to the adversary if the order of filtered ExposureInformation records is the same as the order in which Diagnosis Keys were published on the server.

### Background

<!--- Add more detail -->
Soon after releasing the GAEN API,
both Google and Apple released code snippets indicating some details of their implementations.  The code snippets for both Google and Apple's versions contained only deterministic filtering, which selected the Diagnosis Keys that the user had been exposed to, while maintaining the order in which the keys had been downloaded from the server. These were made available via getExposureInformation() (Google) and exposureInfo (Apple).  At the time I examined these snippets (July-August 2020), both companies omitted the complete stack that would have allowed us to check whether some later step performed a rearrangement of the list.  


Google's documentation gave an idea of what information was returned by [getExposureInformation()](https://developers.google.com/android/reference/com/google/android/gms/nearby/exposurenotification/ExposureNotificationClient#getExposureInformation(java.lang.String)),  but did not specify the order in which the records were arranged. (The current version now explicitly mentions shuffling.) 

The README for Apple's [source code snippets](https://developer.apple.com/exposure-notification/) gave a step-by-step explanation of how exposureInfo was derived.  It described a process of filtering the list of Diagnosis Keys that had been retrieved from the server.
It did not mention shuffling.

## An attack against deterministically-filtered Diagnosis Keys

### Assumptions
Suppose the adversary controls the EN Server that publishes Diagnosis Keys.  Although the Diagnosis Keys are published in a randomised order, the adversary knows the order and the correspondence between Diagnosis Keys and infected individuals.  Suppose also that app users are encouraged to opt in to uploads of the complete results of getExposureInformation() if they are notified of exposure.  This information is also readable by the adversary, who knows who uploaded the ExposureInformation, but does not automatically know which people the exposures correspond to.

We assume also that the adversary has considerable auxiliary information through manual contact tracing, with many users voluntarily sharing some edges of their contact graph. 

### Scenario and motivation for the attack

Suppose the adversary wants to know whether Annika was in contact with Wit on a particular day.  Annika has already, voluntarily, shared with a contact tracer (and hence with the adversary) the durations of her contacts with various other people (including Bob, Charlie, and Dan) on the same day that she met with Wit, but she omits to mention her meeting with Wit to the human contact tracers.   Nevertheless she has opted in to uploading her complete ExposureInformation.

Suppose that Wit tests positive for Covid19, and uploads his diagnosis keys for days including the day he met with Annika.  A reasonable-sized subset of Annika's friends (Bob, Charlie and Dan) do so too.  Note that this scenario, which seems like a lot of coincidences, is not actually too farfetched if Annika herself was the source of these infections - she might have tested positive shortly after the day in question, infected Wit and her other friends before she knew she was infectious, and already have spoken to a human contact tracer before she receives notifications that they too have tested positive.

### The attack

The adversary carefully arranges Wit's Diagnosis Key at some identifiable place in the ordering relative to Annika's already-disclosed friends.  For example, Wit's Diagnosis Key could be listed last for that day, with Bob, Charlie and Dan immediately beforehand, in that order.

When the adversary receives Annika's ExposureInformation, he retrieves the exposures relevant to the specific day, and looks for duration and attenuation patterns that match Annika's account of her contacts with Bob, Charlie and Dan.  These should occur in the expected order and be reasonably easy to check for consistency with Annika's statements to the human contact tracer.  Assuming that the adversary accurately re-identifies the ExposureInformation from Bob, Charlie and Dan based on Annika's account,
**an exposure listed afterwards in Annika's ExposureInformation could only be from Wit**.

This gives the adversary a very good indication of a contact between Annika and Wit, even though she specifically omitted to tell the authorities.  The certainty of the inference depends on the confidence of the re-identification of exposures from Bob, Charlie and Dan, but since Annika has no reason to lie, and there could be a fair amount of redundancy in the available information about closeness and duration, this re-identification could be very confident.  If Annika has told the truth about several affected friends, the inference could be made with very high confidence.

### Affected systems

This only affects systems in which users opt in to uploading their ExposureInformation, using a GAEN app that does not shuffle the list relative to the order of published Diagnosis Keys.

### Recommended mitigation

The attack is thwarted by ensuring that the list of keys in ExposureInformation is reshuffled in a way that destroys information about the order of the published Diagnosis Keys.  A cryptographically secure PRNG could be used to generate a shuffle.  Alternatively, the list could be sorted deterministically according to some field that is already present and likely to be unique, such as a hashCode.

Of course, refusing to upload ExposureInformation also defeats the attack.

### Residual information leakage after mitigation

Some information obviously still leaks even if the array order is shuffled.  In the scenario above, the adversary can tell that Annika met at least one other person who uploaded diagnosis keys on the day.  When infection rates are relatively low, this could still be enough to identify Wit - for example, Wit might be the only other person who uploaded keys on that day.  (Again, this is not farfetched if Annika was the source of infection, perhaps bringing the infection into an area where it was not previously prevalent.)  However, the suggested mitigation costs nothing, has no impact on exposure notification effectiveness, and reduces the number of scenarios in which the adversary could be confident of the inferred connection.


## Disclosure (both ways), source code transparency, and current status
This issue was disclosed to both Apple and Google on August 3rd, with a clear statement that I wasn't sure whether the shuffle was present in non-public code.  Google responded promptly, published the correction on August 26, and later said that it had been corrected in June, before I notified them.

Apple replied that it was "working as intended," did not respond to my request to clarify whether it was intended to shuffle, and did not update their public code snippets to include a shuffle.
However, they have now confirmed by email that their implementation does shuffle, from versions iOS 14 and later.   


I appreciate that some code for both Apple and Google's implementations has been made available.  However, there is a huge difference between genuinely openly available code and the "snippets" actually available when I was trying to understand this issue.  [Apple's snippets](https://developer.apple.com/exposure-notification/) do not include a shuffle, but are not comprehensive enough to allow anyone to assess whether a shuffle is present in the non-public fragments.  It also has not changed since before I disclosed this issue on August 3rd, though the implementation has continued developing.  (Zip SHA256sum: fda4d865d34d3ab45272b5d6545ee3288bfc32453e350ad4f91647e934b1d03a).

[Google's repository](https://github.com/google/exposure-notifications-internals/) is far more accessible, and includes an update from September 30 entitled "Add all remaining classes of the EN SDK." Though I have not checked that it is indeed complete, this gives everyone a much better opportunity to examine and improve the code.


## Summary and Conclusion
This attack allows a malicious server to confidently infer some edges of the social graph.  Although it happens only in restricted settings, and only for individuals who have tested positive and opted in to extra data upload, the scenario is quite plausible. 

A simple and effective mitigation is to shuffle ExposureInformation before returning it to the app, which both implementations now do. Apple informs me they shuffle from iOS 14 onwards; Google says that they did so from June (before I notified them).  

I hope this information is valuable to anyone implementing their own Exposure Notification API.

### Acknowledgements

Thanks to Jim Mussared for very valuable help with the Google code and proofreading of this report.

### Followup and reuse

Comments, edits, suggestions and pull requests are welcome.

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link:
[https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md).
