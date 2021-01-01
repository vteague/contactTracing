This is part of a series of blog posts on automated contact tracing, especially (but not only) Australia's COVIDSafe app.

The complete list is:

- [Tweaking Tracetogether (30 Mar)](https://github.com/vteague/contactTracing/blob/master/blog/2020-03-30TweakingTracetogether.md)
- [Contact Tracing without Surveillance (7 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-07ContactTracingWithoutSurveillance.md)
- [Contact Tracing and Consent (23 Apr)](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-23ContactTracingAndConsent.md)
- [**Tracing the Challenges of COVIDSafe (27 Apr) - this post**](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md) by Chris Culnane, Eleanor McMurtry, Robert Merkel and me.
- [The Missing Server Code and why it Matters (14 May)](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-14TheMissingServerCode.md)  by Robert Merkel, Eleanor McMurtry and me.
- [Security Analysis of the UK's NHS Contact Tracing App](https://github.com/vteague/contactTracing/blob/master/blog/2020-05-19UKContactTracing.md) by Chris Culnane and me.
- [COVIDSafe's new payload encryption scheme (15 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-15COVIDSafesNewEncryptionScheme.md) by Chris Culnane, Ben Frengley, Eleanor McMurtry, Jim Mussared, Yaakov
 Smith, Alwen Tiu and me.
- [Issues with COVIDSafe's new encryption scheme (19 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-19IssueswithCOVIDSafesNewEncryptionScheme.md) by the same authors.
- [The current state of COVIDSafe (mid-June 2020) (22 June)](https://github.com/vteague/contactTracing/blob/master/blog/2020-06-22OutstandingPrivacyIssues.md) by the same authors.
- [COVIDSafe issues found by the tech community (7 July, updated 1 Jan 2021)](https://github.com/vteague/contactTracing/blob/master/blog/2020-07-07IssueSummary.md) by Jim Mussared and me.
- [Fools rush in where angels fear to tread - why Herald won't be ready by Christmas](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-07COVIDSafeHerald.md) by Jim Mussared and me.
- [Why GAEN Exposure Information should be shuffled relative to Diagnosis Keys (16 Dec)](https://github.com/vteague/contactTracing/blob/master/blog/2020-12-16TheImportanceOfShufflingInGAEN.md)
- 
# Tracing the challenges of COVIDSafe

This blog post is joint work by 

[Chris Culnane](https://twitter.com/chrisculnane), [Eleanor McMurtry](https://people.eng.unimelb.edu.au/mcmurtrye/), [Robert Merkel](https://twitter.com/rgmerk) and [Vanessa Teague](https://twitter.com/vteagueaus).

It is made on a best-effort basis using decompiled code from the app, without access to server-side code or technical documentation.

## Overview

The Australian COVIDSafe app's architecture seems approximately similar to the Singaporean TraceTogether architecture, but there are some important differences that users should understand when they are deciding whether to install the app.  Not all of these have been well understood by the Privacy Impact Assessment (PIA) or the Department of Health's response to it.

The basic operation of COVIDSafe is to share encrypted IDs with other users, and to record the encrypted IDs that have been received.  If a person tests positive for COVID19, they upload the list of encrypted IDs they have received, then the central authority decrypts them and notifies those who  may be at risk.

In COVIDSafe, the  encrypted IDs are called UniqueIDs.  Rather than generate them on the phone, COVIDSafe follows TraceTogether in downloading them from the central server.  This brings us to the first main difference.

## The frequency of download and change of UniqueIDs.

[TraceTogether's whitepaper](https://bluetrace.io/static/bluetrace_whitepaper-938063656596c104632def383eb33b3c.pdf) recommends "the issuance of daily batches of TempIDs," which are their equivalent of UniqueIDs.  These are recommended to change every 15 minutes.  So, based on TraceTogether's whitepaper, we believe that the app downloads a day's worth of TempIDs (presumably 96 of them) and uses a new one every 15 minutes. 

The Australian app instead downloads a new UniqueID only every two hours.  It has no batch capacity, so if it cannot reconnect to the Internet within two hours it simply keeps using the same UniqueID.  This has serious privacy implications that are not adequately addressed in the PIA.

The PIA for COVID Safe says:

> 8.17	We understand that the National COVIDSafe Data Store will automatically generate new Unique IDs for each User every two hours and send these new Unique IDs to the User’s App.
>
> 8.18	The App will only accept the new Unique IDs if it is open and running. If the App successfully accepts the new Unique ID, an automatic message will be generated and sent back to the National COVIDSafe Data Store. This message will only effectively indicate a “yes (new Unique ID successfully delivered)” response to the National COVIDSafe Data Store. If the App is not open and running, it will not be able to accept a new Unique ID. It will continue to store the previous Unique ID and use this when the App is opened, until a new Unique ID is generated and accepted.
>
> 8.19	The National COVIDSafe Data Store will regularly compile and store reports (Unique IDReports), based on whether the new Unique ID for a User has been accepted by the App,which will give a measure of what proportion of Users who have downloaded the App have it open and running. These Unique ID Reports will not include any information about which Users have the App open and running, or where any Users are located.

First note that this does not frankly describe the opportunity for the national data store to check, regularly, whether a particular individual has the app up and running.  In Singapore, we believe this information is only polled daily; in Australia it is polled at least two-hourly.  This means that a person who chooses to download the app, but prefers to turn it off at certain times of the day, is informing the Data Store of this choice.

Second, it greatly increases the opportunities for third-party tracking, because a given user advertises the same UniqueID for much longer.  The difference between 15 minutes' and 2 hours' worth of tracking opportunities is substantial.  Suppose for example that the person has a home tracking device such as a Google home mini or Amazon Alexa, or even a cheap Bluetooth-enabled IoT device, which records the person's UniqueID at home before they leave.  Then consider that if the person goes to a shopping mall or other public space, every device that cooperates with their home device can share the information about where they went.

We understand that legislation will attempt to make this illegal, but making it techincally difficult would have been a lot more effective.  How many IoT devices in how many Australians' homes already violate Australian privacy law?  A 15-minute refresh rate for Unique IDs would make this much harder (though perhaps not for Amazon, which hosts the Data Store).

## The sharing, and plaintext logging by other users, of the exact model of the phone

It is not true that all the data shared and stored by COVIDSafe is encrypted.  It shares the phone's exact model in plaintext with other users, who store it alongside the corresponding Unique ID.

The [Singaporean FAQ pages](https://TraceTogether.zendesk.com/hc/en-sg/articles/360043735693-What-data-is-collected-Are-you-able-to-see-my-personal-data-) explicitly mention that "In order to measure distance, information about the phone models and signal strength recorded is also shared, since different phone models transmit at different power," but we were unable to find any mention in COVIDSafe's Privacy policy, FAQ or PIA, though the code clearly does log this information.  This is unfortunate because it does not appear as part of what users consent to.

*Example of recorded logs added 27 Apr*

COVIDSafe records details about the messages it sends and receives, storing these in unencrypted form, though of course the UniqueID is already encrypted.  

An example from our recorded logs is here.  The first column is simply the record number, the next is the time in miliseconds. The random-looking number is the UniqueID.  The 6th and 7th columns list the sender and receiver respectively. These records show our phone exchanging UniqueIDs with an iPhone 6s (also ours). Record 16 shows the message received from the iPhone; record 17 logs the message our phone sent back.

| Num | Timestamp (ms) | ver | UniqueID | org | sender | receiver | signal strength | 
| --- | -------------- | ------- | -------- | --- | ------ | -------- | --------------- |
|16|1587904478384|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-40|
|17|1587904521695|1| [RandomLookingNumEndingIn]8Vix= |AU_DTA| [Our phone's model] |iPhone 6s|-42|

Our receiving phone changed its UniqueID every 2 hours, but the iPhone 6s sent the same UniqueID throughout the 8-hour period - a complete list of records is shown in the Appendix.
We are not quite sure why the UniqueID did not refresh every 2 hours, though this is expected when the Internet connection is unavailable.  Similar long persistence of a UniqueID has also been recorded by [Jim Mussared](https://twitter.com/jim_mussared/status/1254574854826627074).

Note also that the app does not have the capacity to filter contacts based on their physical proximity or duration - it simply logs all the COVIDSafe messages it receives, leaving the determination of risk to the central server.

The relevant code fragment, from the decompiled COVIDSafe App, is also shown in the Appendix.

Although it may seem innocuous, the exact phone model of a person's contacts could be extremely revealing information.  Suppose for example that a person  wishes to understand whether  another person whose phone they have access to has visited some particular mutual acquaintance.  The controlling person could read the (plaintext) logs of COVIDSafe and detect whether the phone models matched their hypothesis.  This becomes even easier if there are multiple people at the same meeting.  This sort of group re-identification could be possible in any situation in which one person had control over another's phone.  Although not very useful for suggesting a particular identity, it would be very valuable in confirming or refuting a theory of having met with a particular person.

In addition, the open broadcasting of the make and model of the phone presents a number of privacy and safety concerns.  The open nature of the broadcast allows anyone to write an app that listens for and records the information. As such, other apps can listen to and retrieve the data. There could be a number of uses of such information. For example, a thief could use the information to determine who has a high value phone worth stealing. Obviously, this is unlikely to happen in the short term, but cannot be discounted as a possibility. 

The greater concern is the impact of broadcasting device parameters that do not change. In modern phones, the Bluetooth MAC address is randomised at intervals, often set as 15 minute intervals, to attempt to prevent devices from being tracked over time, which would reveal the trajectory of that device. If the same information is broadcast across multiple randomisation windows it negates the MAC address randomisation and could permit tracking beyond the window. MAC address randomisation is already [known to have weaknesses](https://arxiv.org/pdf/1904.10600.pdf). Anything that weakens or negates it entirely should be made clear to the end-user when they are consenting to usage of the app. 

By way of an example of how tracking could work, assume an organisation has Bluetooth scanners situated throughout a shopping centre. If there is only one device with a particular make and model in the shopping centre, then that device can be tracked across the entire shopping centre, since the make and model will act as unique ID in that context. Even if there are multiple devices with same make and model, they will still be able to be tracked whilst in non-overlapping sensor zones. They will even be able to be tracked within the same zone, provided that the Encrypted ID is not updated at exactly the same time on both devices. As such, it is likely that even devices with the same make and model will be liable to individual tracking over a much larger area and greater time than was intended in the Bluetooth standard. 


This problem could have been easily avoided if all the information being transmitted had been encrypted.



## The missed opportunity to omit some contacts

When a person tests positive for COVID-19, they upload all the UniqueIDs they have heard over the days they may have been infectious.  COVIDSafe does not give them the option of deleting or omitting some IDs before upload.

This means that users consent to an all-or-nothing communication to the authorities about their contacts.  We do not see why this was necessary.  If they wish to help defeat COVID-19 by notifying strangers in a train or supermarket that they may be at risk, then they also need to share with government a detailed picture of their day's close contacts with family and friends, unless they have remembered to stop the app at those times.  

## Conclusion

Like TraceTogether, there are still serious privacy problems if we consider the central authority to be an adversary.  That authority, whether Amazon, the Australian government or whoever accesses the server, can 
- recognise all your encryptedIDs if they are heard on Bluetooth devices as you go, 
- recognise them on your phone if it acquires it, and 
- learn your contacts if you test positive.

These are probably still the most serious privacy concerns for some COVIDSafe users.  None of this has changed since TraceTogether.  

For other users, the storing of unencrypted phone models in their logs may be the most serious concern, because it allows someone who acquires their phone to analyse their COVIDSafe logs and infer some information about who they have been near.  This doesn’t allow immediate identification, but might help to refute or support a pre-existing idea.  This, too, has not changed since TraceTogether, though the Singaporean FAQ makes it clearer than the Australian privacy policy.

The changes made by the Australian authorities allow easier checking of each person’s regular usage of the app, but do not otherwise significantly increase the authority’s information compared to those existing issues.  The change to two-hourly encrypted IDs does, however, substantially increase the opportunities for third-party tracking based on Bluetooth, though this is still a risk in TraceTogether also.

### Followup and reuse

Comments, edits and suggestions are welcome - the easiest way to contact us is on Twitter 
@chrisculnane
@noneuclideangrl
@rgmerk
@VTeagueAus

or by email at [my first name] at thinkingcybersecurity.com 

Earlier posts are [here](https://github.com/vteague/contactTracing/blob/master/blog/).

You are welcome to quote or reprint this article as long as you acknowledge the original source.  Permanent link:
[https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md](https://github.com/vteague/contactTracing/blob/master/blog/2020-04-27TracingTheChallenges.md).


## Appendix
### Code for producing logs

These code fragments (decompiled from the version as at 26th April) show where COVID Safe records the exact model of the phone from which it has received and recorded the UniqueID.

```
public final void saveDataSaved(BluetoothDevice bluetoothDevice) {
   Intrinsics.checkParameterIsNotNull(bluetoothDevice, "device");
   byte[] bArr = this.writeDataPayload.get(bluetoothDevice.getAddress());
   if (bArr != null) {
      try {
         WriteRequestPayload createReadRequestPayload = WriteRequestPayload.Companion.createReadRequestPayload(bArr);
         Utils.INSTANCE.broadcastStreetPassReceived(this.this$0.getContext(), new ConnectionRecord(createReadRequestPayload.getV(), createReadRequestPayload.getMsg(), createReadRequestPayload.getOrg(), TracerApp.Companion.asPeripheralDevice(), new CentralDevice(createReadRequestPayload.getModelC(), bluetoothDevice.getAddress()), createReadRequestPayload.getRssi(), createReadRequestPayload.getTxPower()));
      } catch (Throwable th) {
         CentralLog.Companion companion = CentralLog.Companion;
         String access$getTAG$p = this.this$0.TAG;
         companion.e(access$getTAG$p, "Failed to save write payload - " + th.getMessage());
      }
      Utils utils = Utils.INSTANCE;
      Context context = this.this$0.getContext();
      String address = bluetoothDevice.getAddress();
      Intrinsics.checkExpressionValueIsNotNull(address, "device.address");
      utils.broadcastDeviceProcessed(context, address);
      this.writeDataPayload.remove(bluetoothDevice.getAddress());
      byte[] remove = this.readPayloadMap.remove(bluetoothDevice.getAddress());
   }
}
```

The asPeripheralDevice call inputs the model of the phone from whom the message was received. (Similar code is present in a method asCentralDevice in the instance the phone is sending rather than receiving a tag.)

```
public final PeripheralDevice asPeripheralDevice() {
   String str = Build.MODEL;
   Intrinsics.checkExpressionValueIsNotNull(str, "Build.MODEL");
   return new PeripheralDevice(str, "SELF");
}
```
### Logs recorded on our phone.

These are the logs made by our CovidSafe app through an 8 hour period, listening to messages received from a CovidSafe app running on our iPhone 6.  As you can see, the iPhone 6's 
UniqueID does not change through the 8 hour period.

| Num | Timestamp (ms) | ver | UniqueID | org | sender | receiver | signal strength | 
| --- | -------------- | ------- | -------- | --- | ------ | -------- | --------------- |
40|1587907232545|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-32|
46|1587907572325|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
62|1587908486634|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-41|
78|1587909389278|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-45|
94|1587910289618|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-41|
110|1587911183945|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
126|1587912088167|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
142|1587912994317|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
158|1587913892653|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
174|1587914755687|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-40|
190|1587915690463|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
206|1587916588241|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
237|1587918357938|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
253|1587919255372|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-41|
269|1587920155379|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-39|
285|1587921092812|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
301|1587921955379|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
317|1587922855652|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-40|
333|1587923755462|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
349|1587924664304|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
365|1587925564366|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-41|
381|1587926455461|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-40|
397|1587927363888|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
413|1587928272806|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-44|
429|1587929188862|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
445|1587930089617|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
461|1587930955535|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-41|
477|1587931855509|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|
496|1587932763228|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-41|
519|1587933669588|1| [RandomLookingNumEndingIn]5K2l= |AU_DTA|iPhone 6s| [Our phone's model] |-43|


