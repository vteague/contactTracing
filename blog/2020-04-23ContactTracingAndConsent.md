This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [**Contact Tracing and Consent (23 Apr) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
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

--------------------------------------------------------------------------------------
# Contact tracing and Consent: why using TraceTogether is different from using your memory

Vanessa Teague

CEO, Thinking Cybersecurity Pty Ltd

Imagine you have just tested positive for Covid19.  You talk with a health professional who asks you to try to remember all the people you have been near over the past two weeks.  

- There are people you were near, but don't know, for example on a crowded train.  Automated electronic contact tracing will help with that.
- There are people you were near, but don't want to, or don't need to, tell the authorities about.  These might be from political meetings or religious services, visits to a lawyer or doctor, or simply time with family and friends.  
You have a moral responsibility to tell the affected people, but you may not need to tell the authorities as well.

How important is the option to spontaneously, undetectably and without prior planning, *not tell* the authorities about some of your contacts?

This is the third in a series of blog posts.  In the [first](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md) 
and [second](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md) I recommended that Australia adopt a decentralized contact tracing protocol such as 
[DP^3T](https://github.com/DP-3T/), 
[MIT-PACT](https://pact.mit.edu/), 
[UW-PACT](https://arxiv.org/abs/2004.03544), 
[covid-watch](https://github.com/DP-3T/) 
or the proposed [Apple-Google API](https://www.apple.com/covid19/contacttracing/).  Worldwide, more than 200 other cryptographers have signed an [open letter explaining the benefits of decentralized protocols](https://www.esat.kuleuven.be/cosic/sites/contact-tracing-joint-statement/). In these protocols, your app can detect that you have been in contact with an infected person, without that person needing to have told the authorities you were nearby.

The Australian government has decided instead to copy Singapore's [TraceTogether](https://www.tracetogether.gov.sg).  Its centralised model means that, when someone tests positive, their list of contacts is given to the authorities. 

The rest of this post examines some important details about TraceTogether's information flow. My firm conclusion is that Australia should not adopt this protocol. 

None of this should be interpreted as any criticism of Singapore's TraceTogether engineers, who have completed a magnificent engineering effort that has probably saved many lives in their country.  Their code is clean, easy to understand, and was built phenomenally fast under tremendous pressure.  Their app shipped weeks ago, and the rest of the world is only just catching up.  I also want to thank them for making its source code openly available, which is helpful both to those who wish to re-use it and also to those of us who wish to argue that there are now better options.

## How it works

When your phone is near another phone, the TraceTogether app sends random-looking beacons over Bluetooth.  These are actually encryptions of your ID.  The phone records all the beacons it has received.  If you test positive, you give the list of encrypted IDs you have received to the authorities.

In my previous blog post about TraceTogether, I said: "Whenever you're within Bluetooth range of a person, you send them your ID, encrypted with the public key of the Singaporean authorities,"
but I was mistaken.  That method wasn't implemented, apparently because it used too much computation for background mode.

According to the [TraceTogether Whitepaper](https://bluetrace.io/static/bluetrace_whitepaper-938063656596c104632def383eb33b3c.pdf)  "In the reference implementation, TempIDs are cryptographically generated by the backend service."  So, rather than encrypting your own ID and sending it to nearby people, you first download your
encrypted IDs from a central server.

TraceTogether downloads encrypted IDs daily, in a process that looks a little like this picture from the similarly-designed European ROBERT project (except ROBERT downloads only at registration):

![Receiving encrypted IDs](https://github.com/vteague/contactTracing/blob/master/blog/RobertPseudonymGen.png "Source: ROBERT project, https://github.com/ROBERT-proximity-tracing/documents/issues/2, last accessed 22 Apr 2020.")

This is different from contact tracing based on human memory.

## Some crucial differences from human memory

### You cannot tell whether you are sending accurate IDs to your contacts.  

When you download your encrypted IDs, you are relying on them to be a truthful reflection of your ID. If a software bug, security problem, or network attack gives you someone else's encrypted IDs instead, you have no way to notice.

If you send IDs that are not yours, then when someone near you tests positive, you will not be notified.  For this reason, if the server is not adequately secured, an automated system has the potential to be a lot less accurate than human memory.

Decentralised protocols do not suffer from this problem unless the attacker compromises the app on your phone.

### You cannot tell whether you are receiving accurate IDs from your contacts. 

You simply record each encrypted ID you receive, without knowing whether the list of encrypted IDs  that you will report to the authorities if you test positive truly matches the people you were near.

Decentralised protocols suffer from this problem partially, but the attacker has to compromise the app on the phone of each person whose identity he wishes to misrepresent.  In Tracetogether, the server or its Internet connection is a single point of failure.

This might also help non-contacts to infer your infection status, for example if they try to get their encrypted ID into the log of your phone, then wait to see whether they are notified of having been exposed.

In both centralised and decentralised protocols, an attacker can replay from one phone the beacons he has heard on a different phone, thus making people seem to have been exposed when they were not.

### The server can tell whether you are running the app.

The TraceTogether whitepaper explains: "server-side TempID generation has a secondary benefit of allowing the health authority to understand adoption and usage levels of the app by logging the issuance of daily batches of TempIDs," which is a polite way of saying that the authorities can check each day whether you are running the app or not.  They cannot tell whether you are using it throughout the day, but it will be obvious if you don't turn it on at all.  So forget any plans you might have had to 'voluntarily' download it but disable or block it.  

### If your list of encrypted contacts is leaked or forced, the IDs can be decrypted by whoever has the key, even if you do not test positive for Covid19

For example, if you are a journalist who has decided to meet a source in person, and you are both running the app, then each phone will store the other person's encrypted ID.  Either phone, if seized by an authority with access to the decryption key or decryption server, reveals the other person's proximity.

If the decryption key leaks, then any attacker who accesses a phone's logs can read off their contact list.

In a decentralised protocol, an attacker who wants to know whether two phones have been near each other must either control both phones or control one of them and hope the other person soon tests positive.  This seems to be an unavoidable consequence of doing contact tracing.

### If the central server uses weak or broken encryption, the encrypted IDs could be easily recovered by third parties.   

That includes both the ones you send while you walk around, and also the list of others' you keep on your phone.

Crucially, even if we get the source code for the Australian app, we cannot test that the encryption is being computed properly, since it is not being computed by the app.  Singapore's server code is openly available (good), but an Australian server could decide to downgrade its encryption at any time, even after deployment, and could do so for some people but not others.

This would make you easily tracked through shopping malls and other public (and private) spaces, even if you never test positive.

Decentralised protocols could also suffer from cryptographic problems that allow easier tracking than we expected, for example if Apple/Google make errors in the cryptography behind their new API or accidentally leak a person's key.  This would allow linking rather than immediate decryption, but might still allow a person to be easily identified.  So I hope that Apple/Google will be absolutely transparent about their implementations so that we can all examine and analyse them. 

### If the central server cooperates with on-site Bluetooth tracking, your location can be easily tracked

(This section added 24 Apr. Thanks to Eleanor McMurtry, [@noneuclideangrl](https://noneuclideangirl.net/), for pointing out that I'd omitted this important case.)

Suppose for example that the central server happens to be a multinational advertising company like Amazon.  And suppose also that it would like to know exactly where you go when you visit a shopping centre.  Then it 
arranges for Bluetooth-based receivers (such as [this infrastructure from Telstra](https://www.telstra.com.au/business-enterprise/products/internet-of-things/solutions/assets-equipment/track-and-monitor?gclid=EAIaIQobChMIxPervd3_6AIVVhaPCh0U9g2cEAAYASAAEgKjYfD_BwE&gclsrc=aw.ds)) to be placed around the shopping centre.
Whenever you broadcast your encrypted ID on the Covid19 tracing app, Amazon can immediately decrypt your ID (it genererated it, remember) and hence, in cooperation with the shopping centre's owners, identify your different interactions throughout the centre.

Obviously similar infrastructure could be established in public places in cooperation with government.  

And if the central server leaks the decryption key (remembering that the key has to be available all the time) then that tracking opportunity is made available to anyone.

Decentralised systems don't have this problem because there is no third-party service with the capacity to decrypt your beacons and recover your ID.  Individuals can be tracked in a similar way if their unique seed is extracted from their phone, but the attack would need to be performed separately on each phone.  Also, when a person tests positive their separate beacons for their infectious period can be linkable, depending on the details of the scheme.  So again there is a huge difference in the ease and scale of the attack in the centralised vs the decentralised solution.


## Who runs the server?

TraceTogether's open source implementation trusts all of this information to the Google Firebase cloud, thus giving Google constant visibility of everyone's IDs, immediate knowledge of the contacts of each infected person, and the job of notifying those exposed.  Firebase's [privacy policy](https://firebase.google.com/support/privacy/) makes it clear that Firebase employees can access personal data.  The obvious alternative, for Australia, would be to run that server as a government IT service.  So I see two options:
1. we assume that Google (or some other corporate partner) won't abuse its detailed personal information about us, our contacts and our illnesses to make an extra buck, or
2. we assume that the commonwealth government won't mess it up, crash the server, leak the information, or post it all on the web in not-really-de-identified form.

Both assumptions are squarely contradicted by past behaviour.

## What does 'voluntary' mean?

Consenting to install it is not the same as consenting to run it constantly, which in turn is not the same as consenting to have your running of it constantly monitored.  Consent to upload also doesn't necessarily mean consent to upload every encounter.  These different consent options need to be clearly separated when presented to users.  

In its TraceTogether form, I would be happy to run it on the train but refuse to run it in my home or office.  I need to see the details of Australia's version before I decide.

Informed consent requires telling us what we're consenting to.  Open source code is a minimal requirement.

## Can we have a debate please?

Singapore's Tracetogether was a great piece of engineering when it was first developed, but it places a great deal of trust in a central authority whose misbehaviour or mistakes could cause both privacy problems and tracing failures.  

Decentralised protocols would be much less susceptible to accidental breaches or deliberate abuse. They do have remaining privacy problems too, though.  See [Vaudenay's analysis of DP3T](https://eprint.iacr.org/2020/399) for a comprehensive discussion.

It is high time Australia's authorities published source code and specifications for their proposed app.  When they do, we can begin a genuine democratic discussion of whether we will tolerate a centralised app, insist on a decentralised one, or refuse to install either.

### Acknowledgements and collaborations

Thanks to Andrew Conway, Richard Chirgwin, Chris Culnane, Peter Eckersley, Rob Merkel, and the MIT-PACT team, for interesting information, discussion and help.  Not all of them agree with the opinions expressed here.
Any errors are, of course, mine. 

Comments, edits and suggestions are welcome - the easiest way to contact me is on Twitter @VTeagueAus
or by email at [my first name] at thinkingcybersecurity.com

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link: [https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md).

