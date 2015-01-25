---
layout: post
title: Data sync and offline support with Dropbox Datastore
author: Claire Reynaud
permalink: /blog/syncing-data-across-multiple-devices-and-offline-support-with-dropbox-datastore/
---
[Dropbox Datastore][dropbox] is a synchronization service provided by Dropbox: 

> "These days, your app needs to store and sync more than just files. With the Datastore API, structured data like contacts, to-do items, and game state can be synced effortlessly. Datastores support multiple platforms, offline access, and automatic conflict resolution."

## TLDR

[Dropbox Datastore][dropbox] synchronization works like a charm, and offline support is great, but I found that querying the local datastore in your app is a huge pain.

## Looking for a data sync service for an iPhone app

Recently, I have been looking for a data synchronization service to integrate in my time tracking iPhone app.

![My BusyBox time tracking app](/images/screenshot1-4inches-en.jpg)

I am making apps as a freelance, so I am using my BusyBox app to track how much time I spend on each of my customer's projects. I also use it to improve behaviors: tracking how much time I spend on social media, how much time I stand at my desk, or how many pomodoros I do each day.

## Why adding data synchronization?

When I published the 1st version of BusyBox in April 2014, the app was minimalist. As I was starting to work as a freelance, my main goal was to have several apps in the App Store to showcase what I could do. One of the things that was missing in BusyBox was the support of data synchronization. I waited 6 month before adding this feature. Surprisingly, no one complained about it. I got a lot of support requests asking for improvements, but not this one. My guess is that everyone assumes they won't loose their data, until they get a new phone and realize that it is not there anymore.

I decided to add data synchronization in BusyBox because it got 35K downloads so far. Also it got mainly 5 stars reviews. It was time to make it possible to backup, sync and restore users data.

## Looking for possible alternatives

So I have been looking at the possible alternatives, and here are the ones I came across:

* iCloud
* [Dropbox Datastore][dropbox]
* Parse
* Home made

### iCloud

There are several reasons why I won't add iCloud synchronization. First, iCloud is not cross platform. I owned several Android devices in the past. I loved my Nexus One, I loved my Galaxy SII. I also love my iPhone 5S now, but, who knows, maybe in the future I will switch back to Android. So for me data synchronization has to work across several plaftorms (iOS, Android, web), period. Second, I have read a lot of posts explaining how difficult iCloud integration can get. Seriously, go read the Clear app reviews. See how many of the bad ones are complaints about iCloud sync not working. Even the best can't make it work.

### Home made

In a previous post, I explained how you can build a back-end and use it to make an app that works offline. This solution has the advantage of letting you choose the datastore you want for your app. If you want to use CoreData on top of sqlite, you can. It has one main disadvantage though: developing and running your own back-end has a cost. I have 4 apps on the App Store, plus my customer's projects. I know I won't have the time to maintain a back-end.

### Parse

Parse could have been a good alternative, and it had almost the same advantages as Dropbox Datastore:

* It is cross platform
* I don't have to write or run the back-end
* They have an iOS SDK, with a great documentation
* It can work offline

### Dropbox Datastore

I ended up choosing [Dropbox Datastore][dropbox], because it had all the advantages of Parse and 2 more:

* Almost everyone has a Dropbox account nowadays. It is a no-brainer for my app users to choose to sync their time tracking data to Dropbox. They know the service, they already have an account, and they trust it. This is not the case for Parse.
* [Dropbox Datastore][dropbox] provides a version of their store on the device that works even if the user is not logged into Dropbox. This is great for user onboarding. I hate it when I download an app, and I cannot give it a try without having to log in. With Dropbox Datastore, you can try the app right away. Later, if you like it, you can turn on synchronization by linking to your Dropbox account. I really like the Dropbox login screen that comes out of the box: "The app BusyBox would like to connect with your Dropbox. This app will read and write its own datastores but none of your files". It explains in 2 sentences what the app will be allowed to do, and especially that it won't be able to sneak in your files.

![My BusyBox time tracking app](/images/screenshot3-4inches-en.jpg)

## Integrating Dropbox Datastore

To get started using Dropbox Datastore in your app, follow the [documentation][dropbox]. It's very straightforward, you'll get a local datastore that syncs to your Dropbox in no time.

Nevertheless, it took me 2 weeks to integrate Dropbox Datastore in my app. The problem was there is only one way to query data from the datastore:

>-(NSArray *)query:(NSDictionary *)filter error:(DBError **)error
>
>Returns records matching the provided filter, or all records if filter is nil.
>
>Parameters
>
>filter: for every key value pair in filter, the query will only return records where the field with the same name has the same value.

You can only query records which match an exact value. This means, for example, that if you want to query records in a given date range, or records which text includes a given string, it's all on you.

I ended up writing a huge amount of code to be able to query the time tracking data. Before that, I was using CoreData on top of sqlite, and querying was coming out of the box. Also I did not have any performance issue. With Dropbox Datastore, I had performance issues right away with just 2 months worth of data. I ended up maintaining indexes in memory. Dealing with caches is always a pain. I didn't want the cache to be stateful, so I rebuild it from scratch in the background each time the user modifies the data. I also needed the user to see quickly the result of its changes. As a full cache rebuild is time consuming (more than 1s), I do first a "quick and dirty" stateful update. Only after comes the complete cache invalidation. This way the indexes get eventually consistent and I am sure that they do not drift from the actual data reference, which is the datastore. At least I know that no data will get lost in the process.

So it works. But it is very frustrating to have to write so much code for something that was really easy to implement before. My conclusion is that I love the synchronization part from Dropbox Datastore, but that I hate the lack of query support.

## Conclusion

So to summarize, [Dropbox Datastore][dropbox] synchronization works like a charm, and offline support is great, but querying the local datastore in your iOS app is a pain. Nevertheless, I am glad I made this choice to add synchronization in my app. I shipped the new version almost 4 weeks ago, and there hasn't been any issue so far. I'll update this post later to say how it went in the long run.

So what you think about it? Would you give it a try? I would also love to hear from you if you integrated Dropbox Datastore in your app (or Parse), or have interesting links or thoughts to share on the subject.

Thanks for reading!

Claire

[dropbox]: https://www.dropbox.com/developers/datastore
