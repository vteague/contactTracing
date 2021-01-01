This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [**Contact Tracing without Surveillance (7 Apr) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
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

# Contact tracing without surveillance 

Vanessa Teague

CEO, Thinking Cybersecurity Pty Ltd

April 7th, 2020.

## Note added 14 Apr: [TraceTogether's code](https://github.com/opentrace-community) is now open! Thankyou.

At the end of [last week's blog](2020-03-30TweakingTracetogether.md) I expressed two hopes
- that Singapore's TraceTogether would be open-sourced soon,
- that the Australian government would commit to ``an app that helps Australians identify potential infection risks without requiring centralised government access to location or contact information.''

So much for that.  Tracetogether is still closed source, with no indication of when it will be open.  And if the Australian government has a plan, they haven't told us about it.


So what should the Australian government be doing?  This blog post surveys the privacy implications of the available options for figuring out how a person became infected.  The main point is the same as last week's: we can choose to do contact tracing in a more (or less) privacy-preserving way. 

There are lots of things you might want to keep private:
- your location history,
- your infection status,
- your close physical contacts,

and there are different entities from whom you might want to hide that information:
- the general public,
- people who know you,
- people who were physically close but don't know you,
- the government,
- private corporations that already know a lot about you (e.g. Google, or your phone company).

I don't know of any system that hides all relevant information from all relevant entities.  It is probably impossible - for example, if you have been near only one other person, and you learn that you might have been infected, you can infer that that person has the virus.

One important fault line is how much you trust a centralized (presumably government) authority to store and convey the information.  This affects both privacy and reliability.

## Location-based apps
There's a huge difference in scale and size, too: a method like Israel's [Hamagen](https://github.com/MohGovIL/hamagen-react-native), which reveals some public information about where infected people have been during the (probably brief) time they were infectious, on a voluntary opt-in basis, is a lot better than gathering everyone's location all the time. 

It's unfortunate that much of the Australian political debate discusses whether we should all agree to share detailed location data constantly with government, as if that is the only option for stopping disease spread.  On the contrary, the Hamagen approach allows _exactly the same location matching_, by ordinary people on their own phone, without their needing to tell anyone else where they have been.  The downside is that more information is released to the public about the movements of a person while infectious - that's the tradeoff.

## Proximity-based apps
The other class of contact-tracing apps, those based on Bluetooth beacons, measure proximity without recording location.  The app sends a series of random-looking numbers, and records the random-looking numbers it hears from others.  Again there are two approaches, the first with a centralised authority trusted for privacy and realiability:
- Type 1. In some apps, including Tracetogether and the current draft of the European [PEPP-PT](https://www.pepp-pt.org) scheme, if you test positive you give the authorities a list of every beacon you have _received_.  The authorities identify and contact the people who sent them.
- Type 2. In the decentralized version, including [covid-watch](https://www.covid-watch.org), the [CTV proposal](https://arxiv.org/pdf/2003.13670.pdf), and [a proposed alternative for PEPP-PT called DP^3T](https://github.com/DP-3T), if you test positive you post publicly a list of every beacon you've _sent_.  Everyone's app reads the list and learns whether they have been exposed.

Type 1 reveals each infected person's physical contacts to the authorities, though it does a reasonable job of keeping your identity and infection status private from other users.  Type 2 tells government nothing (except that you have been infected) but probably makes it a little easier for individuals to infer one another's infection status - this depends on the details of the scheme.   

There are some brilliant ideas for using sophisticated cryptography to get the best of both approaches, but nothing that's had enough review to be confident in yet. 

## Recommendations
In the Australian context, I have severe reservations about the schemes that rely on a centralised (presumably government) entity.  It might crash under the load, thus neglecting to convey time-critical information.  It might leak sensitive information due to an accidental data breach, a deliberate cyber attack, or a naive effort to de-identify the data and post it online for sharing.  (If you think this last option is farfetched, you're not Australian - both the [Australian Federal](https://pursuit.unimelb.edu.au/articles/the-simple-process-of-re-identifying-patients-in-public-health-records) and [Victorian State](https://pursuit.unimelb.edu.au/articles/two-data-points-enough-to-spot-you-in-open-transport-records) governments have posted highly sensitive, easily-identifiable data online because they didn't realise how easily re-identifiable it was.)  Social contact information is [very easily re-identifiable.](https://arxiv.org/abs/1102.4374)

Another concern is that the data acquired for Covid19 tracing might be reused for something completely different.  Australian authorities have a history of inappropriate access to telecommunications metadata to [target journalists](https://www.theguardian.com/australia-news/2019/jul/23/police-made-illegal-metadata-searches-and-obtained-invalid-warrants-targeting-journalists), for example.  Even before this crisis our Home Affairs minister was [openly suggesting that our Defence intelligence services should be allowed to spy on Australians](https://www.abc.net.au/news/2020-02-19/powers-for-asd-spy-dark-web-australians/11980728).  I'm concerned that people's location and contact information could be used for political purposes that have nothing to do with infection tracing. 


We can not have perfect privacy _or_ perfect disease control, but we can make reasonable decisions about the tradeoffs.

I would  vote for a system of the Type 2 style, which maintained the privacy of an individual's location and physical contacts, including from government, possibly at the expense of being more likely to  expose their infection status.  Infection status cannot be very well protected even by the best of systems, is a much less tempting target for reuse in unrelated contexts, and seems compatible with existing behaviour -  a fair number of  people are willing to admit publicly that they've tested positive for COVID19.  However, I acknowledge that this might not be everyone's preference, might not be OK among all groups of people, might not be applicable for other kinds of disease (such as HIV, which still carries significant social stigma), and might not be ideal for countries in which the government is highly trusted (wherever they might be).

This could be achieved by tweaking TraceTogether, as I suggested in my last blog, if their source code ever appears, or by adopting one of the other Type-2 approaches linked above.  It could be combined with the Hamagen approach for those willing to opt in. 

In conclusion, a note of skepticism: although Singapore quickly implemented a highly accurate method of contact tracing, they recently decided to tighten their lockdown and close their schools anyway.  Their [PM said of locally infected people](https://www.straitstimes.com/singapore/health/most-workplaces-to-close-schools-will-move-to-full-home-based-learning-from-next), ``despite our good contact tracing, for nearly half of these cases, we do not know where or from whom the person caught the virus.'' 

When we think about whether to adopt particular system, we should  question
both how effective it will be and whether equally-good disease control could have been achieved with a more privacy-oriented approach.




### Acknowledgements and collaborations

Nothing in this blog post is original, but has been gleaned from discussions and readings from many sources.
Thanks to Peter Eckersley, Kobi Leins, David Watts, the Macquarie/Melbourne/Data61 team (Hassan Asghar, Farhad Farokhi, Dali Kaafar, Ben Rubinstein),  the covid-watch/coepi team (especially James Petrie), the MIT/Boston University team (especially Ran Canetti, Yael Tauman Kalai, Ron Rivest, Emily Shen, Ari Trachtenberg, Mayank Varia and John Wilkinson) and the DP^3T team (especially Kenny Paterson).  Not all of them agree with the opinions expressed here.
Any errors are, of course, mine. 

Comments, edits and suggestions are welcome - the easiest way to contact me is on Twitter @VTeagueAus
or by email at [my first name] at thinkingcybersecurity.com

