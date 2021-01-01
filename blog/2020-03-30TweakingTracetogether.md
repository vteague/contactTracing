This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [**Tweaking Tracetogether (30 Mar) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [Contact Tracing and Consent (23 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
- [Tracing the Challenges of COVIDSafe (27 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md) by Chris Culnane, Eleanor McMurtry, Robert Merkel and me.
- [The Missing Server Code and why it Matters (14 May)](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)  by Robert Merkel, Eleanor McMurtry and me.
- [Security Analysis of the UK's NHS Contact Tracing App](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-19UKContactTracing.md) by Chris Culnane and me.
- [COVIDSafe's new payload encryption scheme (15 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-15COVIDSafesNewEncryptionScheme.md) by Chris Culnane, Ben Frengley, Eleanor McMurtry, Jim Mussared, Yaakov
 Smith, Alwen Tiu and me.
- [Issues with COVIDSafe's new encryption scheme (19 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md) by the same authors.
- [The current state of COVIDSafe (mid-June 2020) (22 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md) by the same authors.
- [COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2021)](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys (16 Dec)](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)
--------------------------------------------------------------------------


# Why, and how, Australia needs to change TraceTogether before we use it here

Vanessa Teague

CEO, Thinking Cybersecurity Pty Ltd

March 30th, 2020.

### Note added 14 Apr: [TraceTogether's code](https://github.com/opentrace-community) is now open! Thankyou.
### Note added 19 Apr: This technical description of TraceTogether is slightly incorrect: each person's ID is encrypted, not with the _public key_ of the Singaporean authorities, but with a _secret key_ known only to the Singaporean authorities.  This doesn't change the conclusions of this blog post, but does have a host of important implications described in my third blog post (TBA).

The Australian Financial Review [reported recently](https://www.afr.com/politics/federal/singapore-coronavirus-app-on-approval-fast-track-20200324-p54dhl) that the Australian government is seriously considering adapting Singapore’s TraceTogether contact-tracing app for use in Aus.  It’s a good idea to benefit from Singapore's (probably very good) software engineering and development efforts, especially since they're intending to open the source code soon, but we need to make some changes to ensure that its privacy guarantees match Australia's political setting.

Some apps, like Israel's [Hamagen](https://github.com/MohGovIL/hamagen-react-native), produce a public list of the times and locations where infected people have been. This automates the low-tech idea of publishing a map of possible infection locations (with times) and allowing everyone to take responsibility for checking whether they were there.  Location information for infected people is revealed, perhaps in a not-explicitly-identified form.  
Everyone else's privacy is well protected because they are only reading the map and matching it with their location history on their own phone.
For infected people, there is some effort to remove the most obvious identifiers in their data, but if you’re one of the two people in Benalla who have tested positive, it’s going to be pretty obvious which locations are yours.  Location information is usually easily [re-identifiable from a few points](https://www.nature.com/articles/srep01376).

Singapore's TraceTogether app is completely different.  It works by Bluetooth connections, which detect whether two people are within a few metres of each other, without recording where they are. Whenever you're within Bluetooth range of a person, you send them your ID, encrypted with the public key of the Singaporean authorities.  Everyone’s app keeps a record of all the encrypted IDs they’ve collected, though they can’t tell whose they are.  If a person tests positive for Covid19, they give the authorities the list of encrypted IDs, which the government decrypts and then uses to contact the potentially-exposed people.  So nothing about locations is revealed, even to government (which is good).  However, each infected person’s connections are read by government and the government has a lynchpin role in contacting everyone who has potentially been exposed.  

In the Australian political context, I don’t believe the TraceTogether privacy or trust models are acceptable.  If the system is overwhelmed and goes down, then those crucial notifications to potentially-infected people don’t get sent.  (Can anyone remember when an important Aus govt server went down when we all needed it?) And there’s an obvious potential for abuse of the connection information that the app receives. (Can anyone remember any instances in which an Aus govt authority abused their access to personal data? Journalists had better turn it off if they want to meet a source in person.) There’d also have to be a careful legal analysis of whether all that contact-mapping information that we’d all collect about each other could be subject to a Technical Assistance Notice under TOLA, and hence used for a completely different purpose.  

Fortunately, it doesn’t have to be this way.  A few small tweaks to TraceTogether can shift the information flow to one where ordinary people do the tracing for themselves and share the information with each other, without trusting a central authority to read and convey the information.

Here’s how.

## The goal

There are some open-source proposals for a kind of app that reverses the TraceTogether information flow.  (See [Ari Trachtenberg's write-up](https://www.linkedin.com/pulse/controlling-covid-through-cellphones-ari-trachtenberg) for a good general explanation and [the Covid-Watch/CoEpi collaboration](https://docs.google.com/document/d/1f65V3PI214-uYfZLUZtm55kdVwoazIMqGJrxcYNI4eg/edit#heading=h.6q40wl39kcs8) or [Daniel Reusche's analysis](https://github.com/degregat/ppdt) for some detailed examples---there are probably many more.)  Rather than broadcasting an ID encrypted with the key of the authorities, everyone broadcasts a random-looking number that changes frequently (say every hour or every 15 mins).  Each person's phone remembers the numbers they've seen and sent via Bluetooth. If you get infected, you effectively post on a public list all the random-looking numbers you have broadcast in the past fortnight (or whatever date range is medically justified). Every day, everyone scans the public list for any random numbers they've seen. If there's a match, they know they may have been exposed to the virus.

This achieves exactly the same effective contact-tracing as TraceTogether, without the centralised point that learns everyone's contacts.  Each infected person publishes a list of random-looking numbers that serve as notification for others they were in contact with, but don’t reveal anything about where the person was.  Crucially, no third party learns which people were in contact – each affected person can tell, but nobody else.

## Small steps to achieve the goal

So let’s take a step-by-step edit of TraceTogether to adapt it towards this goal.

First think about removing dependence on government to be a critical part of the reliability of the system. Never mind about privacy for now - just think about shifting the notification responsibility away from a central authority into the hands of ordinary app users. This leads us to:

### TraceTogether Tweak 1 – removing dependence on government for reliability


Suppose that as well as remembering every number you have _received_ you also remember every number you have _sent_. When you are infected, you post the complete list of messages you sent, to some completely public website/repository. Everyone's app now reads everything that has been posted on the public list and compares it to their own list of every number they have received.  If there's a match, they know they have been within Bluetooth range of an infected person.

-----------------------------------------------------------------------------

### Analysis:

This is a small tweak to the app, which would still be compatible with the Singaporean version of its functionality. It removes the possibility for an accidental government failure to prevent people being notified. So this is already a huge improvement.

There is still some reliance on someone to keep a public website of numbers running, but this doesn’t contain sensitive information and could be replicated by a number of untrusted parties, e.g. state authorities, commercial cloud providers (AWS, GoogleCloud, Dropbox), volunteers, etc. 

However, it has two main shortcomings: (1) it would post a lot of numbers per person, which is wasteful. (2) the IDs are still encrypted with the public key of the authorities, even though we're not relying on them to use the decryption key, so the opportunity to use it as a method of surreptitious surveillance remains. This leads us to:

### TraceTogether Tweak 2 – removing dependence on government for privacy


Rather than sending your ID encrypted with government’s key, you send a frequently-changing series of random-looking values not decryptable by the authorities. Nothing else changes.

-----------------------------------------------------------------------------

This is vastly better for privacy, but remains a relatively small tweak from the Singaporean app, so I hope we could still relatively easily merge their security patches and other improvements.

There are a number of nice cryptographic constructions for reducing the total number of different values that need to be posted.  (See technical blog post to follow.)

Tweak 2 does break compatibility with others who use the Singaporean-style app (with government decryption), so the whole country would either have to decide to use tweak 2 or not.

I think that it would be a good strategy to fork TraceTogether, apply both tweaks 1 and 2, and use it in Aus. We'd get Bluetooth-based contact tracing with very good privacy properties, but if we designed it well, I hope we'd continue to benefit from the Singaporean government’s (presumably good quality) app development effort, along with its security patches and other improvements.

## Details, limitations and possible attacks

Of course, all this assumes that the promised open-source version of the TraceTogether appears soon.  If it does, we should thank them for sharing the results of their very hard work with the rest of the world.  

We could also choose to adapt Israel’s Hamagen for those who agree to having their locations made public – a combination of both approaches might work well here.  As long as those who have tested positive for the virus were accurately informed that their locations will be effectively public (though not directly associated with their name), this might help to alert people who weren’t using the Bluetooth app.

I have left out a lot of details here.  Any system needs a way of ensuring that a person has genuinely had a positive diagnosis from a real doctor, before the contact-tracing is initiated.  We would need something like that too – perhaps it should be doctors who upload a person's information, or who sign a digital permission slip that lets the person post their numbers.  Even with manual contact tracing, it is possible for someone to lie about their contacts by either including people they haven't been near, or excluding people they have.  This system cannot prevent that – it is still possible for a person to post misleading information (such as numbers they received rather than numbers they sent) or to refuse to share information.  Similar threats exist for TraceTogether too.  It seems hard to design a system that enforces accuracy without non-consensual surveillance.  

## Summary - where to from here?

This is not perfect, but I believe it represents a good gain in potential-infection tracing at a lesser cost to privacy than the alternatives.
I'll be posting some more details soon, and I hope to link to the relevant parts of TraceTogether's code when it is available.  

We need a commitment from the Australian government to design an app that helps Australians identify potential infection risks without requiring centralised government access to location or contact information.



### Acknowledgements and collaborations

Many thanks to Peter Eckersley, Yehuda Lindell, Ben Rubinstein, and the Data61/Macquarie University team for helpful background or comments on earlier drafts.  Any errors are, of course, mine. 

Comments, edits and suggestions are welcome - the easiest way to contact me is on Twitter @VTeagueAus

