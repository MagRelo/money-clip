*** Forked to add 'staleAfter' option. use this option when 'set'ing a value. When retrieved, the key will be deleted if the current time is after the 'staleAfter' time.

# money-clip

![](https://img.shields.io/npm/dm/money-clip.svg)![](https://img.shields.io/npm/v/money-clip.svg)![](https://img.shields.io/npm/l/money-clip.svg)

For managing your client side cache. Tiny wrapper over Jake Archibald's tiny IndexedDB wrapper, [idb-keyval](https://github.com/jakearchibald/idb-keyval) adding support for versioning and max age.

Designed for use in PWAs that need to cache data fetched from APIs and such.

It exposes mostly the same cache API with a few extras:

1.  `getAll` _added by this lib_
1.  `getConfiguredCache` _lets you pre-apply options_
1.  `get`
1.  `set`
1.  `keys`
1.  `clear`
1.  `del` _called delete in original_

These will all swallow errors because it's designed to work as an enhancement to a PWA. If something fails you simply don't start with a primed cache of data for your app. Nothing lost.

Clientside caching of user data is great, but can also be treacherous. We need some way to version our data, make sure we don't populate our app with really stale data, or populate the app with data from a previous session.

This library provides a way to deal with these issues. On every write or read, you can provide a version number (usually a version number stored in a config in your app). In addition to storing the data itself, this library will also store a timestamp and the version number (if provided).

When trying to read the cache when your application starts, if the data is too stale, or there's a version number mismatch, the previous entry is just deleted, and `null` is returned. In this way, you can throw away potentially problematic cached data by bumping a version number in your config.

If you want to ensure you don't inflate cached data from another user, you can combine a version number in your config with some identifier of the user's session to build a version string. This means that if a different session is used when opening the app, it will lead to a version mismatch and the existing cached will be deleted.

## install

```
npm install money-clip
```

## API

**note: All methods take the same optional options:**

```js
const options = {
  version: 1, // this can be any string or number
  maxAge: Infinity // in number of milliseconds
}
```

### getAll(options)

Returns promise that will resolve to an object of all values or `{}`. Any single key that fails the version/age check will be deleted and not included. Never throws errors.

### get(key, options)

Returns Promise that will resolve to value at key, or `null`. If it fails version checks or age checks, it will be deleted. Never throws errors.

### set(key, options)

Returns promise sets value at key. Never throws.

### keys()

Returns promise that resolve to object keys

### del(key)

Returns promise but deletes content of key.

### clear(key)

Returns promise and clears everything in cache.

### getConfiguredCache(options)

Returns an object with all the methods listed above, but with pre-populated options. So if you prefer you can configure version and max age once, then just use `get`/`set` without passing options.

## example

If you're using Redux, you could, for example combine this with [redux-persist-middleware](https://github.com/HenrikJoreteg/redux-persist-middleware) to lazily cache content of certain reducers when certain action types occur.

```js
import { h, render } from 'preact'
import { Provider } from 'preact-redux'
import { getConfiguredCache } from 'money-clip'
import { createStore, applyMiddleware } from 'redux'
import ms from 'milliseconds'
import rootReducer from './somewhere'
import RootComponent from './components/root'
import getPersistMiddleware from 'redux-persist-middleware'
import config from './my-config'

// get a version of the cache lib with options pre-applied
import cache = getConfiguredCache({
  version: config.cacheVersion,
  maxAge: ms.days(30)
})

cache.getAll().then(data => {
  const store = createStore(
    rootReducer,
    data,
    applyMiddleware(getPersistMiddleware(cache.set))
  )

  render(
    <Provider store={store}>
      <RootComponent />
    </Provider>,
    document.getElementById('app')
  )
})
```

## Change log

* `2.0.1`: Fixed bug where `clear` was not being exported after build.
* `2.0.0`: added `getConfiguredCache` renamed methods to more closely align with `idb-keyval`. Export `keys`, `del`, and `clear` directly. Tests, example, readme.
* `1.0.0`: initial release

## credits

If you like this follow [@HenrikJoreteg](http://twitter.com/henrikjoreteg) on twitter.

## license

[MIT](http://mit.joreteg.com/)
