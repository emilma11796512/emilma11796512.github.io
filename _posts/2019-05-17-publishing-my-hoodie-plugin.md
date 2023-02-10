---
layout: post
title: "Experience of publishing a package to npm"
date:   2019-05-17 17:07:00 +0100
tags: [written-in-english, technology, couchdb, pouchdb, hoodie, my-stuff, hoodie-plugin-store-crypto]
lang: "en"
---

Last year I released my first npm-package: __[hoodie-plugin-store-crypto](https://www.npmjs.com/package/hoodie-plugin-store-crypto)__! It is a plugin for the [Hoodie-Project](http://hood.ie), that allows to save encrypted data on a users store.

For reactivating my blog I thought to write about my experience releasing and maintaining it, would be a great topic.

[Goto tl:dr](#tldr)

## Why the plugin

At the time (and now too) I was working on another [project](https://github.com/Terreii/andromeda-viewer/). A chat-app with syncing. But I didn't want to be risk leaking user data, even by mistake!

__Hoodie__ would be responsible for user session and data handling.

But what if I made a big mistake‽ I wasn't interested in what my user would be writing, blocking and banning would be done by a third-party.

So the solution would be __to encrypt everything on the users machine__.

A quick search did brought me to [crypto-pouch](https://www.npmjs.com/package/crypto-pouch). But I couldn't bring it to work with Hoodie. And, if I had read its code correctly, it did un-encrypt all data send to the server! But I could use it as a blue-print for writing my own encryption methods. From it I also learned how to use encryption.

While I was writing the methods, I quickly realized, that probably many hoodie-users would benefit from it being a plugin. And hoodie had finished their [new plugin architecture](http://docs.hood.ie/en/latest/guides/plugins.html).

As a bonus this could also be a great way to truly learn [test-driven-development](https://en.wikipedia.org/wiki/Test-driven_development)!

The idea was simple:
- A plugin that would add an object with the same methods as [hoodies store-client](https://www.npmjs.com/package/@hoodie/store-client) to the hoodie-client!
- but with the encryption of [crypto-pouch](https://www.npmjs.com/package/crypto-pouch).

After asking the maintainers of the two projects if it would be OK to use some of their code, I was of to implementing!

![From Spongebob: 6 month later...](/assets/6-months-later.jpg)

## Pre-release

My time was split between writing my plugin and two versions of the original project (one as a open-source web-app and one as a Swift iOS-App). And most tests had to be changed to handle encryption. All of this combined resulted in the release date be around 7 month after the project started.

While changing big number of existing tests to fit one owns project, is a great way to learn to write tests, it can become tedious after a while. Having other tasks can be handy to recharge and come back later with new excitement.

Test-driven-development turned out to be a wonderful way to write packages! The package being a side-project also meant I hand limited time to work on it. But with test-driven-development I could write some to all tests for a single method on one day. While on the other day implement it, and fix bugs in the tests.

Having npm-packages are a super-power! They allowed me to write a crypto-package without having to deal with different APIs and implementation details. They allow package(-maintainers) to focus on a small set of features. Everything else is handled by other packages and maintainers!

All I hand to focus on was:
- Correctly deriving an encryption-key.
- Locking and unlocking.
- Changing the encryption-key.
- Which encryption-algorithms to use.
- Find a way to handle syncing problems when the encryption-key got changed.
- How CouchDB-Documents are going to be encrypted, while still being correct CouchDB-Docs.
- Implementing methods that use and behave like the hoodie-store methods, while encrypting the data.

## Post-release

Naturally the first release had a lot of bugs and missing features. But most were easy to fix. With one exception: When a user did enter a wrong password, my package didn't fail!

The fix was to add an encrypted string to a special document. And decrypt it when the user unlocks the crypto-store. If the check doesn't exist, then add it once an encrypted document was successfully read. A setting could turn of this auto-fix. This was then released in version 2.0.

__It always helps if you use your own package__. I encountered most shortcomings while using mine.

## tl:dr

Using [test-driven-development](https://en.wikipedia.org/wiki/Test-driven_development) for a side-project is great! It allows you to think about usage and implementation of an API on different days.

Everything becomes easier, if you break it down into smaller problems. Npm-packages are already solved pieces!

Publishing and maintaining a package is a commitment, but a manageable one. Step await if it becomes to much. But it is fun!

If you add features and bug-fixes in small steps, you can overcome all shortcomings. But you should layout an update-path!

Use our own package!

Having your package used is *awesome*!

**Everyone can publish to npm. And that is a good thing!**
