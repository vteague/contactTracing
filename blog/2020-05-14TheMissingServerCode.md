This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [Contact Tracing and Consent (23 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
- [Tracing the Challenges of COVIDSafe (27 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md) by Chris Culnane, Eleanor McMurtry, Robert Merkel and me.
- [**The Missing Server Code and why it Matters (14 May) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)  by Robert Merkel, Eleanor McMurtry and me.
- [Security Analysis of the UK's NHS Contact Tracing App](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-19UKContactTracing.md) by Chris Culnane and me.
- [COVIDSafe's new payload encryption scheme (15 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-15COVIDSafesNewEncryptionScheme.md) by Chris Culnane, Ben Frengley, Eleanor McMurtry, Jim Mussared, Yaakov
 Smith, Alwen Tiu and me.
- [Issues with COVIDSafe's new encryption scheme (19 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md) by the same authors.
- [The current state of COVIDSafe (mid-June 2020) (22 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md) by the same authors.
- [COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2021)](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys (16 Dec)](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)

------------------------------------------------------------------
# The missing server code, and why it matters 

This blog post is joint work by 

[Robert Merkel](https://twitter.com/rgmerk), [Eleanor McMurtry](https://eleanorm.info/) and [Vanessa Teague](https://thinkingcybersecurity.com).

While it is gratifying that the Australian government has -- belatedly -- released the source code to the Android and iOS versions of COVIDSafe, the practical impact has been fairly limited.  

Within minutes of the COVIDSafe app being available from the Play Store, many people, including us, were able to view a pretty good facsimile of the Android version of the app’s source code, through the wonders of freely downloadable pieces of software called [APK Extractor](https://play.google.com/store/apps/details?id=com.ext.ui&hl=en_AU) and [JadX](https://github.com/skylot/jadx).  

But we weren’t restricted to just inspecting decompiled source code.  We were able to observe the app running, and view the data files the app creates in response to proximity to other phones.  We were also able to use [nRF Connect](https://www.nordicsemi.com/Software-and-tools/Development-Tools/nRF-Connect-for-mobile), a Bluetooth debugging app, to connect to phones running COVIDSafe and observe what they did.

Combined with our existing knowledge of OpenTrace, the Singaporean contact tracing app, all of these things were quite sufficient for a small team of experts to find a number of serious privacy-related bugs in the COVIDSafe client.  [Many others did too](https://covidsafe.watch/). We haven’t found any more since the source code release.

However, privacy and security risks of similar magnitude exist on the other side of the COVIDSafe system -- the server code running somewhere in AWS’s data center in Sydney.  In this article, we explain the key role of this server code, and the importance of proper scrutiny of it.

## What is a UniqueID?

In a nutshell, the COVIDSafe app works by exchanging “UniqueIDs” with any other phone running the app that it comes within Bluetooth range with.  The phone will therefore contain a time log of the UniqueIDs that it has seen in the past 21 days.  If you test positive to COVID-19, you can upload your time log.  Somehow, the UniqueIDs in your time log will be converted back to names (or pseudonyms) and phone numbers, and passed on to the contact tracer assigned to your case.  If a contact tracer (in discussion with you) decides that a contact was sufficiently lengthy and close, they will use the contact data linked to the UniqueIDs to get in contact with that person so that they can be tested and self-isolate.

There are two key privacy mechanisms intended to ensure that only government-approved contact tracers can access the contact information linked to each UniqueID.  Firstly, the UniqueIDs appear to be random gibberish -- whatever they do contain, it’s not a plaintext record of people’s names and phone numbers!  Secondly, the UniqueID you broadcast changes every two hours -- so if I walk through a shopping centre in the morning, by the time I return in the afternoon I have a completely new UniqueID.

But here’s the thing: we don’t know what’s in a UniqueID, how the contact tracers will go from a TempID to actual contact details, or whether they really do protect your privacy from everybody else.  That’s because the UniqueIDs aren’t generated on your phone.  They’re generated by the COVIDSafe server code, which remains a secret, and the COVIDSafe app downloads a new one, every couple of hours.

It would not have been any more difficult to write an app in which UniqueIDs were generated directly on a phone -- the UK’s contact tracing app apparently does so, as will apps based on Google and Apple’s API.  But, for whatever reason, the Australian government and their app developers have chosen to follow a different approach.

## What’s in a TempID?

As we’ve said, we don’t know for sure what’s in a COVIDSafe UniqueID, because the government has chosen not to release server code, or even some kind of architecture document explaining the details of how the system works.  However, we can have a look at the system upon which COVIDSafe is based: the open source OpenTrace app and server [publicly released](https://github.com/opentrace-community) by the Singaporean government, and the [white paper](https://bluetrace.io/static/bluetrace_whitepaper-938063656596c104632def383eb33b3c.pdf) explaining how it works.  BlueTrace refers to “TempIDs” rather than “UniqueIDs”, so we’ll use that term to make clear which version of the app we’re talking about.

In OpenTrace, each app user has a User ID, which is directly linked in a database table somewhere to each registered user’s name and phone number.  When a phone requests a new TempID which it will use for a given time period, the server code:

- Takes the User ID with the start and the end of the time period for which the TempID is valid for, and concatenates them together, to form the “plaintext”.
- Generates a random “initialization vector” -- a string of random gibberish precisely 16 bytes long.
- Uses the secret key which only the operator of the servers (the Singaporean government) knows, the initialization vector, and the plaintext, and feeds them to an encryption algorithm called AES-256-GCM.  The algorithm produces two outputs -- the ciphertext, and the auth tag.
- It then concatenates the ciphertext, the initialization vector, and the auth tag together -- but not, obviously, the secret key.  
- Voila, one newly minted TempID!

In the unfortunate event that the “owner” of a TempID comes into contact with somebody who tests positive, the Singaporean government can reverse the process using their secret key, and get back the original plaintext, including the User ID.  They can then go back to the database and get the user’s contact details.

The details of AES-256-GCM are mind-bogglingly complex to non-cryptographers.  But it doesn’t really matter, because cryptographers around the world have been poring over those details for over twenty years, trying to find ways to “break” AES-256-GCM.  Despite many years of trying, no one in the academic cryptography community has found a way to recover the plaintext from the ciphertext without the key.  It’s not completely beyond the realms of possibility that some intelligence agencies can, but even this is very doubtful.

So, while it’s not the way that we would have chosen to implement Temp/UniqueIDs if we were given the contract, the cryptography in the OpenTrace server code meets its design goals.  If they can keep the secret key secret, nobody other than the Singaporean government will be able to get people’s real identities from recorded TempIDs.  And, importantly, the only thing the Singaporean government needs to keep secret is the contents of the key.  Everything else can be disclosed.

COVIDSafe’s client source code is largely a direct copy of OpenTrace; they could have chosen to copy the TempID generation code as well.  However, they may not have done so.

## UniqueIDs are longer than TempIDs

Unlike the Singaporean government, the Australian government has released no information about how UniqueIDs are generated.  We do know, however, that the UniqueIDs that are exchanged between COVIDSafe users are considerably longer than the TempIDs from OpenTrace -- they are 120 bytes long rather than 88.

As we have no information about the contents of UniqueIDs, all we can do is speculate about the difference.  Is the extra data simply “padding”?  Have the COVIDSafe developers chosen to encode extra information into unique IDs?  Have they chosen a different encryption scheme?  

We simply don’t know.

## Who will decrypt the data?

One possible reason for using different cryptography than OpenTrace is because of the split responsibilities in our federal system.  COVIDSafe will be operated by the federal government, while state governments are responsible for running contact tracing programs, and managing quarantines.

We’ve been told that [only state and territory health authorities will be able to decrypt the data](https://www.abc.net.au/news/2020-04-26/coronavirus-tracing-app-covidsafe-australia-covid-19-data/12186068). This may have been important for generating public trust given the commonwealth’s [problematic history](https://www.theguardian.com/australia-news/2020/apr/26/coronavirus-app-will-australians-trust-a-government-with-a-history-of-tech-fails-and-data-breaches) on data security and use.  But if the government has actually attempted to design this guarantee into the code, the OpenTrace TempID generation scheme would not work.  In OpenTrace, one central authority knows all the keys for decrypting all the TempIDs.  In Australia, key management could get complicated when people cross state borders.  

For example, suppose you live in Albury -- you’ll spend most of your time gathering UniqueIDs that are encrypted with a key known to the NSW government.  But if you cross the river and go grocery shopping in Wodonga, you collect some Victorian UniqueIDs (encrypted with a key known to the Victorian authorities).  If you contract COVID-19, you upload all the UniqueIDs you have received.  Who decrypts that data?

The developers of the COVIDSafe App may well have developed a clever protocol for getting the right data decrypted by the right (state or territory) health authority.  They may have chosen a solution where any state health authority can decrypt any UniqueID.  Or there may actually be no technical guarantees at all that COVIDSafe data is unavailable to federal authorities.

There are almost certainly sensible ways in which even the strongest state-based privacy requirements could have been met correctly, effectively, and efficiently, but there are also many, many opportunities for mistakes.  If the design and implementation of the server were public we could have a sensible discussion about those design choices, find any bugs, and ensure that we have a system that lived up to its security and privacy guarantees without compromising its contact tracing effectiveness.

## Where to now?

The best time for the server code to be scrutinized was before the app went live.  But that time has now passed, and millions of Australians are exchanging UniqueIDs with any Bluetooth device that asks politely, with no guarantees that the UniqueIDs are indeed adequately protecting their privacy.

In our view, it would still be better for the server code, particularly the UniqueID generation mechanism, to be released as soon as possible.  If there are bugs, or a design that doesn’t live up to its privacy promises, it would be better for all of us if those flaws are rectified sooner rather than later -- and the easiest way for that to happen is to have many experts analyse the server code and responsibly disclose any issues they find.  

### Followup and reuse

Comments, edits and suggestions are welcome - the easiest way to contact us is on Twitter 
@rgmerk
@noneuclideangrl
@VTeagueAus

Earlier posts are [here](https://github.com/vteague/contactTracing/blob/master/blog/).

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link:
[https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md).

