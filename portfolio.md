---
layout: page
title: Portfolio
permalink: /portfolio
lang: "en"
---

#### Table of content
- [Talents](#talents)
- [usePouchDB](#usepouchdb)
- [hoodie-plugin-store-crypto](#hoodie-plugin-store-crypto)
- [Andromeda-Viewer](#andromeda-viewer)
- [Schichtkalender (shift-calendar)](#schichtkalender-shift-calendar)
- [open source](#open-source)
- [Design](#design)

## Talents

Primarily I program for the web. But it wasn't where I started programming.

While my education (German "Ausbildung") to an electronics technicians for automation technology I did learn to program in __[Instruction list](https://en.wikipedia.org/wiki/Instruction_list)__ to control production pipelines.

After that I (and the web) taught me *web-development*. First without any library. Then with __[React](https://reactjs.org/)__ and later with __[Preact](https://preactjs.com/)__ and __[jQuery](https://jquery.com/)__. Currently I'm learning __[Ember.js](https://emberjs.com/)__.

On the server side I started with __[Express.js](https://expressjs.com/)__ and I'm learning __[Hapi.js](https://hapi.dev/)__. While my database expertise is primarily in __[PouchDB](https://pouchdb.com/)__ and __[CouchDB](https://couchdb.apache.org/)__.

I also did program a small iOS app in __[Swift](https://swift.org/)__. However I did never release it. And I'm a beginner in __[Java](https://www.java.com/)__ and __[Rust](https://www.rust-lang.org/)__.

## usePouchDB

{% include github_repository.html repo="use-pouchdb" %}

usePouchDB is a collection of [React Hooks](https://reactjs.org/docs/hooks-intro.html) to access local PouchDB Databases.

It is intended to be a puzzle piece in replacing the deprecated [CouchApps](https://couchapp.readthedocs.io/en/latest/intro/index.html). And also to make working with PouchDB and CouchDB easier in React Apps.

Every hook corresponds to a PouchDB method. It has the same options and result type. But a different name. A name that is more in the spirit of hooks. For example: PouchDB's method to load a single [document (JSON-Object)](https://pouchdb.com/guides/documents.html "PouchDB's Guide for working with documents") `db.get(id)` becomes `useDoc(id)`.
<br />
Additionally every hook subscribes to changes, and updates it's result to represent the new state in the database.

usePouchDB is entirely written in __[Typescript](https://www.typescriptlang.org/)__.

## hoodie-plugin-store-crypto

{% include github_repository.html repo="hoodie-plugin-store-crypto" %}

*A plugin for [Hoodie](http://hood.ie/) that adds methods to add, read, update and delete encrypted documents and enable end-to-end encryption.*

I used this package to learn test-driven-development and maintaining a package, that is [published on npm](https://www.npmjs.com/package/hoodie-plugin-store-crypto). It was originally intended to be only a part of an other project of mine ([Andromeda-Viewer](#andromeda-viewer)), but I quickly recognized that it would be of good use for others.

[More can be read in my blog-post about it]({% post_url 2019-05-17-publishing-my-hoodie-plugin %}).

## Andromeda-Viewer

{% include github_repository.html repo="andromeda-viewer" %}

*Small web-client for connecting to the virtual worlds of [Second Life](https://secondlife.com/) and [OpenSimulator](http://opensimulator.org/).*

This is my project where I can grow all my talents!
- it must work with a third-party service, over both UDP and HTTP, using not documented protocols.
- it needs a server.
- it must run on mobile and desktop.
- it must be 100% accessible.

## Schichtkalender (shift-calendar)

{% include github_repository.html repo="shift-calendar-rt" name="Schichtkalender (shift-calendar)" %}

*Hosted at [schichtkalender-rt.now.sh](https://schichtkalender-rt.now.sh/).*

*A unofficial shift-calendar for Bosch in Reutlingen.*

The [original version](https://github.com/Terreii/shift-calendar-rt/tree/gh-pages) is one of my oldest projects. But I realized, that it would be easier to re-write it than update it.

With the new version I focused almost exclusively on a mobile experience. With a release size of 459kb (the bundle with 94,51kb no gzip) I'm on a good way.

It is a [PWA](https://en.wikipedia.org/wiki/Progressive_web_applications) and would pass as [JAMstack](https://jamstack.org/), but without a API-backend. At the moment it is *only in German* (because it is a utility app for my work). With it I learned a lot about making a web-app accessible for people with little to almost no computer knowledge.

<div class="display-as-row">
  <img class="with-shadow" src="/assets/schichtkalender-rt.now.sh.png" alt="schichtkalender-rt.now.sh" width="200" height="297" />

  <img class="with-shadow" src="/assets/schichtkalender-rt.now.sh-menu.png" alt="schichtkalender-rt.now.sh with open menu" width="200" height="297" />
</div>

## open source

I also contributed some patches to open source projects:

- [hoodiehq / pouchdb-users](https://github.com/hoodiehq/pouchdb-users/pull/10)
- [hoodiehq / hoodie-store-client](https://github.com/hoodiehq/hoodie-store-client/issues/168)

## Design

Over the years I also helped to design and implement the CSS for web-sites.

My first page-design was of my scout-groups page [dpsg-eningen.de](http://dpsg-eningen.de/).

[![dpsg-eningen.de](/assets/dpsg-eningen.jpg)](http://dpsg-eningen.de/)

Later I lead the team that designed [IronScout2018.de](https://www.ironscout2018.de/).
