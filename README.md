# contactTracing
ForkingTraceTogether

Why, and how, Australia needs to change TraceTogether before we use it here

Dr Vanessa Teague
Thinking Cybersecurity Pty Ltd

The Australian Financial Review reported recently that our federal government is seriously considering adapting Singapore’s TraceTogether contact-tracing app for Australia.  It’s a good idea to benefit from Singapores (probably very good) software engineering and development efforts, especially since they’re intending to open-source it soon, but we need to make some changes to ensure that its privacy guarantees match Australia’s political setting.

Some apps, like Israel's Hamagen, involve effectively publishing the times and locations of where infected people have been. For the infected person, the claims their data has been 'de-identified' are pretty nonsensical, but for everyone else privacy is well protected because they're just reading the data.  This automates the low-tech idea of publishing a map of possible infection locations (with times) and allowing everyone to take responsibility for checking whether they were there.  So location information for infected people is revealed, perhaps in a not-explicitly-identified form, but if you’re one of the two people in Benalla who have tested positive, it’s going to be pretty obvious which locations are yours.

Singapore's TraceTogether app is completely different.  It works by BlueTooth connections, which detect whether two people are within a few metres of each other, without recording where they are. Whenever you're within bluetooth range of a person, you send them your ID, encrypted with the public key of the Singaporean authorities.  Everyone’s app keeps a record of all the encrypted IDs they’ve collected, though they can’t tell whose they are.  If a person tests positive for Covid19, they give the authorities the list of encrypted IDs, which the government decrypts and then uses to contact the potentially-exposed people.  So nothing about locations is revealed, even to government (which is good).  However, each infected person’s connections are read by government and the government has a lynchpin role in contacting everyone who has potentially been exposed.  

In the Australian political context, I don’t believe the TraceTogether privacy or trust models are acceptable.  If the system is overwhelmed and goes down, then those crucial notifications to potentially-infected people don’t get sent.  (Can anyone remember when an important Aus govt server went down when we all needed it?) And there’s an obvious potential for abuse of the connection information that the app receives. (Can anyone remember any instances in which an Aus govt authority abused their access to personal data? Journalists had better turn it off if they want to meet a source in person.) There’d also have to be a careful legal analysis of whether all that contact-mapping information that we’d all collect about each other could be subject to a Technical Assistance Notice under TOLA, and hence used for a completely different purpose.  

Fortunately, it doesn’t have to be this way.  A few small tweaks to TraceTogether can shift the information flow to one where ordinary people do the tracing for themselves and share the information with each other, without trusting a central authority to read and convey the information.

Here’s how.

The goal

There are some open-source proposals for a kind of app that reverses the TraceTogether information flow.  (***refs)  Rather than broadcasting an ID encrypted with the key of the authorities, everyone broadcasts a random-looking number that changes frequently (say every hour or every 15 mins).  Each person's phone remembers the numbers they've seen and sent via Bluetooth. When you get infected, you effectively post on a public list all the random-looking numbers you've broadcast in the past fortnight. Every day, everyone scans the public list for any random numbers they've seen. These do exactly the same effective contact-tracing as TraceTogether, without the centralised point that learns everyone's contacts.  You publish a list of random-looking numbers that tell others they were in contact with an infected person, but don’t reveal anything about where you were located.  Crucially, they also don’t reveal to any third party which other people you were close to – each affected person can tell, but nobody else.

Small steps to achieve the goal

So let’s take a step-by-step edit of TraceTogether to adapt it to this style of protocol.

First think about removing dependence on government to be a critical part of the reliability of the system. Never mind about privacy for now - just think about shifting the notification responsibility away from a central authority into the hands of ordinary app users. This leads us to:

TraceTogether Tweak #1 – removing dependence on government for reliability
---------------------------------------------------------------

Suppose that as well as remembering everything you've _heard_ you also remember everything you've _sent_. When you are infected, you post the complete list of messages you sent, to some completely public website/repository. Everyone's app now reads everything that has been posted and compares it to the public list.


Analysis:

This is a small tweak to the app, which would still be compatible with the Singaporean version of its functionality. It removes the possibility for an accidental government failure to prevent people being notified. So this is already a huge improvement.

There is still some reliance on someone to keep a public website of numbers running, but this doesn’t contain sensitive information and could be replicated by a number of untrusted parties, e.g. state authorities, commercial cloud providers (AWS, GoogleCloud, Dropbox), volunteers, etc. 

However, it has two main shortcomings: (1) it would post a lot of numbers per person, which is wasteful. (2) the IDs are still encrypted with the public key of the authorities, even though we're not relying on them to use the decryption key, so the opportunity to use it as a method of surreptitious surveillance remains. This leads us to:

TraceTogether Tweak #2 – removing dependence on government for privacy
----------------------------------------------------------------------

Rather than sending your ID encrypted with government’s key, you send a frequently-changing series of random-looking values not decryptable by the authorities. Nothing else changes.

-----------------------------------------

This is vastly better for privacy, but remains a relatively small tweak from the Singaporean app, so I hope we could still relatively easily merge their security patches and other improvements.

There are a number of nice cryptographic constructions for reducing the total number of different values that need to be posted.  (See technical blog post to follow.)

Tweak #2 does break compatibility with others who use the Singaporean-style app (with government decryption), so the whole country would either have to decide to use tweak #2 or not.

I think that it would be a good strategy to fork TraceTogether, apply both tweaks #1 and #2, and use it in Aus. We'd get bluetooth-based contact tracing with very good privacy properties, but if we designed it well, I hope we'd continue to benefit from the Singaporean government’s (presumably good quality) app development effort, along with its security patches and other improvements.

Of course, all this assumes that the promised open-source version of the app appears soon.  If it does, we should thank them for sharing the results of their very hard work with the rest of the world.  We could also choose to adapt Israel’s Hamagen for those who agree to having their locations made public – a combination of both might work well here.  As long as those who have tested positive for the virus are accurately informed that their locations will be effectively public (though not directly associated with their name), this might help to alert people who weren’t using the bluetooth app.

I’ve left out a lot of details here.  Any system needs a way of ensuring that a person has genuinely had a positive diagnosis from a real doctor, before the contact-tracing is initiated.  We’d need something like that too – perhaps it should be doctors who upload a person’s information, or who sign a digital permission slip that lets the person post their numbers.  Even then, there’s a possibility for a person to post misleading information (such as numbers they received rather than numbers they sent) or to refuse to share information.  So this is not perfect, but overall I believe it represents a good gain in potential-infection tracing at a lesser cost to privacy than the alternatives.



