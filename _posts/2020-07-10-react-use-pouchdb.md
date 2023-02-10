---
layout: post
title: "usePouchDB - React Hooks for PouchDB"
date: 2020-07-10 16:20:00 +0200
tags: [written-in-english, technology, couchdb, pouchdb, react, react-hooks]
lang: "en"
---

Last fall I (finally) read [CouchDB's Documentation](http://docs.couchdb.org/en/stable/) completely. And since I've read its [security](http://docs.couchdb.org/en/stable/intro/security.html "CouchDB's security documentation") and [design documents](http://docs.couchdb.org/en/stable/ddocs/index.html "design documents documentation index") parts, I can't get the idea out of my head, that all you need for a [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) web app is a front end with [React](https://reactjs.org/) + [React-Hooks](https://reactjs.org/docs/hooks-intro.html) + [PouchDB](https://pouchdb.com/) hosted on a static file host. And [CouchDB](https://couchdb.apache.org/) paired with some [serverless functions (function as a service)](https://en.wikipedia.org/wiki/Function_as_a_service) as the backend end.

So I written those React hooks and published them as [__usePouchDB__](https://christopher-astfalk.de/use-pouchdb/) to [npm](https://www.npmjs.com/package/use-pouchdb). This post will go into some design decisions and how I think they could be used. For a tutorial and API docs please visit [https://christopher-astfalk.de/use-pouchdb/](https://christopher-astfalk.de/use-pouchdb/).

If you use a [service-worker](https://developers.google.com/web/fundamentals/primers/service-workers/ "Service Workers: an Introduction"), then your CRUD web app becomes, with some small changes, even [offline first](http://offlinefirst.org/ "Page of the Offline First movement")! [Create-react-app](https://create-react-app.dev/ "create-react-app's page") for example, comes with a service-worker out of the box.

## Main concepts

The target App I had in mind while developing __usePouchDB__ is a Web-App that solely uses *function components* and *hooks*. With as little as possible server side functionality. Everything on the server should be done by CouchDB or some *serverless functions*. The client should be able to be hosted on a static file host. In short: A modified version of [JAMStack](https://jamstack.org/).

Other Apps are possible, even encouraged, and can be achieved with Redux and [redux-pouchdb](https://github.com/vicentedealencar/redux-pouchdb) combined with usePouchDB. But they are not the focus.

### Provider

A [Context Provider](https://reactjs.org/docs/context.html#contextprovider), that provides access to one or multiple PouchDB databases, is required. With plugins and settings and different database names, there is just to much that can be done using PouchDB to expose everything.

The concept is that the package user would create a *Databases component*, which renders the `<Provider>`. It would be rendered on the top of the component tree and makes all databases available to all components. It could also be used as a bridge to a [Redux store](https://redux.js.org/).

The *Databases component*'s main job would be to *setup* the local and remote databases, *handle sessions* (sign up, login and logout), and *synchronization*. That's a lot to do for a single component, but all this is linked together. On login you what to sync with that users database. In a [couch_peruser](https://docs.couchdb.org/en/stable/config/couch-peruser.html) setup you even have to know the username! Similar, on logout you want to stop syncing and maybe destroy the local databases.

#### Extending Context and multiple databases

If a `<Provider>` embedded in the component tree of another `<Provider>` would extend the context, then you could use special databases that are only needed in a subtree. And combined with [React-Router](https://reactrouter.com/) and [Loadable Components](https://loadable-components.com/) databases could be created for routes. For example a database for sharing between a group of people would only be rendered if its content would be displayed. With a `<Provider>` and a *Databases component* for that *route*, that database would be created on entering that route and closed once the user visits another route.

Every hook has an option to select the used database by a key. That key can be changed and the hook will re-query its content. By default the database's name is used for its key. But a `<Provider>` can also overwrite a database's key. Then that key will be used to select a database, and not it's name. This is useful for remote databases, their name is their full URL.

There is also the concept of a *default database*: it gets used when no database name/key is provided. The `<Provider>` sets which of the databases is the default. A child `<Provider>` overwrites the *default db* for its subtree.

This can be combined to create seamless switching between two databases: For example when your user visits your page the remote database (on CouchDB) is the *default db*. The user can load and interact with all your content. While in the background you create a local database and replicate the remote database to the local one. And once the local database, all [Mango indexes](https://pouchdb.com/guides/mango-queries.html) and [views](https://pouchdb.com/guides/queries.html) are up to date, switch the *default database* to be the local one. Then all hooks will automatically re-query.

Hooks that are re-querying because an option or a database did change, show the last result until the query finishes. The sole exception is [`useDoc`](https://christopher-astfalk.de/use-pouchdb/docs/api/use-doc) (hook version of [`db.get()`](https://pouchdb.com/api.html#fetch_document)). When it has a initial value, then it returns the initial value until the query finishes. To reset hooks, embed them in Components and use those options, which should reset the hook, in its `key` prop.

### Hooks

__usePouchDB__ is a set of *React-Hooks* that re-implement some of [PouchDB's API](https://pouchdb.com/api.html "PouchDB API Reference"). Also every hook subscribes by default to changes. That way every component will become a __function of your database__ (+ some local state) that returns your UI. Similar to [Redux](https://redux.js.org/), where your components became functions of your *Redux store*.

In future updates I will provide a way to opt out of auto subscription and add a method to manual setoff an update. But I don't know how this will look like, yet.

Many of PouchDBs/CouchDBs APIs have objects or arrays as arguments. I wanted to be able to use them without having to use `useMemo` for those arguments, if I didn't want to re-query on every render. That's why I decided to compare their value, and only re-query if they change *by value*.

#### Updates and your own hooks

With [`usePouch`](https://christopher-astfalk.de/use-pouchdb/docs/api/use-pouch) usePouchDB provides a hook to access a database. It is intended to serve as a escape hatch if the build in hooks don't provide what you need.

It can also be used to *create your own hooks*.

One possible hook category are *hooks that return functions to update your data*:

```javascript
import { useCallback } from 'react'
import { usePouch } from 'use-pouchdb'

export function useAddTodo(dbKey = '_default') {
  const db = usePouch(dbKey)

  return useCallback(async text => {
    const result = await db.put({
      _id: `todo_${new Date().toJSON()}`,
      type: 'todo',
      text: text,
      completed: false
    })
    return result
  }, [db])
}
```

This hook returns a callback. When called with a string, that callback will create a new todo-item in your database. Those hooks fill the role of [actions](https://redux.js.org/basics/actions) in a Redux-App: They update your data.

The optional `dbKey` argument is to select the database that should be used. `'_default'` is a special key to select the *default database*.

If you create your own hooks in a similar way, then they can be used everywhere in your app. And they behave like the hooks from usePouchDB: they use the *default database* or optionally another one and they can switch databases.

```jsx
// How the useAddTodo hook can be used
import React, { useState } from 'react'
import { useAddTodo } from './hooks'

export default function AddNewTodo() {
  const [todoText, setTodoText] = useState('')
  const addTodo = useAddTodo()

  return (
    <form
      onSubmit={event => {
        event.preventDefault()
        addTodo(todoText).then(() => {
          setTodoText('')
        })
      }}
    >
      <input
        type='text'
        placeholder='todo text'
        value={todoText}
        onChange={event => {
          setTodoText(event.target.value)
        }}
      />
      <button>Add</button>
    </form>
  )
}
```

#### useFind

All hooks have a similar API to the method they use. But [`useFind`](https://christopher-astfalk.de/use-pouchdb/docs/api/use-find) is a combination of [`db.createIndex()`](https://pouchdb.com/api.html#create_index) and [`db.find()`](https://pouchdb.com/api.html#query_index).

The [docs encourage](https://docs.couchdb.org/en/stable/api/database/find.html#index-selection) to explicitly specify the index to use in a `db.find()` request. And `db.createIndex()` results with the `id` and `name` of the created Index, it also returns them if the index already exists.

It would also be complicated to create an API for two hooks: one to create an index and a second one to query it. And the second one should only query if the first did create its index. And what if the user did forget to use both hooks?

So I combined [`use_index`](https://pouchdb.com/guides/mango-queries.html#use_index) from `db.find()` and the `index`-field of `db.createIndex()` into one `index`-field on the `useFind`-options:
- If the `index`-field is a *string* or an *array of strings*, `useFind` will use them as `use_index` for `db.find()`.
- But if the `index`-field contains an object, that object will be used in the `index`-field of `db.createIndex()`. And the `id` and `name` in its result will be used in the `use_index` field of `db.find()`.
- And if the `index`-field is `null` or `undefined`, then [`db.explain()`](https://pouchdb.com/api.html#explain_index) will be used to find the `id` and `name`.
- There is no `use_index` option on the `useFind` hook.

Once the `id` and `name` is know, `useFind` will not call `db.createIndex()` or `db.explain()` until an option changes.

This has the result, that all what you need to do to *create*, *query* and *subscribe* to a Mango index is this:

```javascript
const { docs, loading, state, error } = useFind({
  index: { // create index if it doesn't exist
    fields: ['type', 'title'],
  },
  selector: {
    type: 'story',
    title: { $exists: true },
  },
  sort: ['title'],
  fields: ['_id', 'title'],
})
```

As a nice bonus: when [partial indexes](https://docs.couchdb.org/en/stable/api/database/find.html#partial-indexes) get added to PouchDB, you can be sure that your query will use them!

### Subscriptions

Internally every database gets a *Subscription-Manager*. It's job is to optimize subscriptions.

When I first tested my hooks idea every hook directly subscribed to its database. But I quickly noticed that with even a small number of hooks PouchDB's event-emitter will log a warning, that to many event-listeners are registered. Everything works just fine, this is just a functionality to help to find memory leaks.

So I created the *Subscription-Manager*. It subscribes to every type only once. Once for document changes and once for every [map/reduce-view](https://pouchdb.com/api.html#filtered-changes) in use of an [`useView`](https://christopher-astfalk.de/use-pouchdb/docs/api/use-view) hook (hook version of [`db.query()`](https://pouchdb.com/api.html#query_database)).

At the moment a *Subscription-Manager* will subscribe to all document changes. I know this is like drinking from a fire hose, but it was easier to implement. Also in most cases every hook makes a full re-query. A potential, but complicated place for optimizations.

But optimizations will follow: if you only subscribe to some documents, then the *Subscription-Manager* should only subscribe to those. And once that list changes, resubscribe.

But there are some hooks that require a subscription to all documents:
- [`useAllDocs`](https://christopher-astfalk.de/use-pouchdb/docs/api/use-all-docs) the hook version of [`db.allDocs()`](https://pouchdb.com/api.html#batch_fetch). But only if you don't use the `key` or `keys` options.
- [`useFind`](https://christopher-astfalk.de/use-pouchdb/docs/api/use-find) the hook version of [`db.createIndex()`](https://pouchdb.com/api.html#create_index) and [`db.find()`](https://pouchdb.com/api.html#query_index) combined.

This all results in following behavior: usePouchDB will subscribe to all changes. You can use as many hooks as you like; they result in only one connection for every remote database. But if you use `useView` you get an additional connection for every view. And often a hook will make a full query.

If you use a local database this all doesn't matter. Because its all local! That is the reason behind my warning that usePouchDB is only optimized for local databases.

## Conclusion

usePouchDB is probably useful. When I moved my experimentation App from Redux to only hooks it became much simpler.

In components I specified what data I needed in that component, and the hooks delivered. I also had some other hooks for other state and some to update my data. When the later ones where call, usePouchDBs hooks automatically updated!

But there is still work left to do.
