---
layout: post
title: "The next feature in hoodie-plugin-store-crypto"
date: 2019-12-31 13:52:00 +0100
tags: [written-in-english, technology, couchdb, pouchdb, hoodie, my-stuff, hoodie-plugin-store-crypto]
lang: "en"
---

With my package [hoodie-plugin-store-crypto](https://www.npmjs.com/package/hoodie-plugin-store-crypto) reaching version 3.1 and the year 2019 coming to a close, I wanted to write a post about the next feature I'm going to implement:

A method to encrypt any JSON Object, without saving it.

Wait, *thats it‽*

Well, yes. But the thing is, __hoodie-plugin-store-crypto__ is a [plugin](http://docs.hood.ie/en/latest/guides/plugins.html "Hoodie's plugin documentation") for the [Hoodie project](http://hood.ie/) that enables developers to store encrypted documents in their users data.

This is realized by it being a warper around [Hoodie-client-store](http://docs.hood.ie/en/latest/api/client/hoodie.store.html "Documentation for Hoodie's client store") with the same API. But with the difference, that every document written with it, will be encrypted. Here lies the problem: You had to use your users database! You could have used [`_local/`-docs](https://docs.couchdb.org/en/latest/api/local.html "Local (non-replicating) Documents - CouchDB documentation"), but this would still write to disc.

## API (preview)

This new capability will be implemented using 2 methods:
- `cryptoStore.encrypt(object[, aad])`
- `cryptoStore.decrypt(object[, aad])`

Both will return a Promise resulting in the encrypted object. And they will use the same *crypto-key* as the [other methods](https://github.com/Terreii/hoodie-plugin-store-crypto/blob/latest/docs/api.md).

```javascript
// example API:
hoodie.cryptoStore.encrypt(object[, aad])
  .then(encryptedObject => {
    // do something with it
  })

hoodie.cryptoStore.decrypt(encryptedObject[, aad])
  .then(decryptedObject => {
    // do something
  })

// This function will be similar to hoodie.cryptoStore.add
async function add (object) {
  if (!object._id) {
    object._id = uuid()
  }
  var id = object._id
  delete object._id
  var hoodie = object.hoodie
  delete object.hoodie

  // Encrypt object and use its _id as the aad
  const encrypted = await hoodie.cryptoStore.encrypt(object, object._id)

  object._id = id
  object.hoodie = hoodie

  // Store the encrypted object
  const storedObject = await hoodie.store.add(encrypted)

  // Decrypt the stored object
  const decrypted = await hoodie.cryptoStore.decrypt(storedObject, storedObject._id)
  return decrypted
}
```

## Why

For one: there a many [uses cases](#some-use-cases) for this API.

But, second, for now you had to implement your own encryption stack:
- Check if `crypto.subtle` is available and use `browserify-aes` and `pbkdf2` if not.
- Generate a `salt`.
- Generate a `key`.
- `encrypt` and/or `decrypt`.
- And test all of it.

All of this is already done by my plugin. So why re-implement it?

## Some use cases

In combination with [`cryptoStore.withPassword`](https://github.com/Terreii/hoodie-plugin-store-crypto/blob/latest/docs/api.md#cryptostorewithpassword "API documentation of withPassword") it will be really be flexible.

You will be able to use it with any password. Be it from your App's bundle, an API, or some stored in a document. Remember to change them frequently!

__Please remember that those are only ideas. And not tested by me! And I have *no background in cryptography*! All I'm doing is package it in a nice to use API!__

__Use at your own risk!__

Now follows some uses cases I could think of:

### Unlocking

With a password included in your bundle (or from an API), you could store the users password in the [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) (which will never be saved on disc) and unlock your users `cryptoStore` on reload.

This useful for example if the user is a mobile user. When they switch the browser tab or app, your app might be removed from memory. But things in __sessionStorage__ will remain. [Watch a video about the __Page Lifecycle API__](https://www.youtube.com/watch?v=UlLQPguE7UQ).

And with the [Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API) you could also unlock all other open tabs.

```javascript
// combine it with usePassword
hoodie.cryptoStore.usePassword(yourSecretKey, yourSecretSalt)
  .then(result => {
    const { store, salt } = result

    return store.encrypt(object)
  })

  .then(encrypted => {
    window.sessionStorage.setItem('secret', JSON.stringify(encrypted))
  })
```

Both could be done using encrypted `_local/` documents. But they are *stored to disc* (although only local). And this is for some industries and Apps an absolute no-go!

### Only one password

If you don't what to know your users encryption password, they have to enter two passwords. One to sign in and one to unlock the encryption.

But with the new methods, you could generate a secure password for your users. And use it to sign in.

Then encrypt the generated password with password the user has entered. Store it in your [_users database](https://docs.couchdb.org/en/latest/intro/security.html#authentication-database "Documentation of CouchDBs _users database").

API end-point could then return the encrypted password. Which will then be decrypted by the users password. And with the decrypted password you sign your user in.

```javascript
async function signIn (username, password) {
  // fetch the encrypted password from the api
  const response = await fetch(`/api/sign_in/${username}`)
  const encryptedSignInPw = await response.json()

  // generate an instance of the cryptoStore API with the password
  // the user entered and the salt that was returned by the API.
  const { store } = await hoodie.cryptoStore.withPassword(
    password,
    encryptedSignInPw.salt
  )

  // decrypt the sign in password
  const signInPassword = await store.decrypt(encryptedSignInPw)

  // sign in with the generated sign in password
  await hoodie.account.signIn({
    username,
    password: signInPassword.password
  })
  // and unlock the cryptoStore
  await hoodie.cryptoStore.unlock(password)
}
```

## Closing

Those two small methods open up many possibilities.

__But remember: *I'm not an expert!* You have to do your own risk analysis and testing, if you want to implement any of the examples!__

I hope your 2019 was great and that your 2020 will be even better! Have a happy new year 🎆!
