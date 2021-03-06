# Write your first service worker in 5 minutes

## What is a service worker

A service worker is a little file that will allow you to cache files and other assets on a user's machine. How is this different from server-side caching? Because the assets are stored on a user's machine, rather than a server, there is no need to go across a network. This offers a lot of gains for your app's reliability and speed. Since the app is being served from a cache, there's no need to go over the network which can take up a lot of time.

## Why this will help your app

Adding a service worker will not only help increase the speed of your app, it'll offer a reliability that your users haven't seen before. By removing the immediate need for internet access, we can help anyone with a spotty internet connection -- and really _everyone_ has a spotty internet connection.

## Lifecycle

The service worker lifecycle looks a bit like this:

  1. Install
  2. Activate
  3. Fetch

Let's look at these events individually.

#### Install

Prior to the install event, your application does not have a service worker. The browser will detect the registration event from your code and install the service worker. Your service worker will contain a function called `oninstall` that will handle which files are cached onto the user's machine.

#### Activate

The activate event is fired after the service worker is installed and ready to go. This is a really good place to do clean up on old files and caches that you don't need anymore. However, for this example, we won't do anything with our activate event.

#### Fetch

This is where our service worker really shines. When a fetch request is made, our service worker will intercept it using a function aptly named `fetch`. Your service worker can look for a similar fetch request from our cache, or send the request onward.

The interesting thing about the service worker lifecycle is that activate and fetch don't necessarily run back-to-back. Fetch will only run when there is a fetch event to intercept, so it could be some time between the activate event and a fetch event. During that time the service worker is idle.

Now that we have a solid understanding of the service worker lifecycle, let's take a look at a sample.

## Sample service worker

For this example, let's use [FayePI](https://github.com/carmalou/fayePI). This is a little API I wrote to help women learn to build dynamic websites, and the documentation page uses a very simple service worker.

Before a service worker can ever be installed, we have to add a registration function to our app's code.

```
// index.js

if(navigator.serviceWorker) {
      navigator.serviceWorker.register('serviceworker.js');
}
```

That will usually go in your `index.js` file to be fired when the page is loaded. That's the only reference to your service worker in your app-specific code.

Now we'll have a separate file for our service worker.

```
// serviceworker.js

self.oninstall = function() {
    caches.open('fayeFrontEndV1').then(function(cache) {
        cache.addAll([ / ... / ])
        .catch( / ... / );
    })
    .catch( / ... /)
}
```

This is the function that runs when our service worker installs. First, we initialize and open a cache. This is a specific place where the files will be stored on the user's machine.

`caches.open` returns a promise with a reference to the cache we opens. Then we use `addAll` to pass in an array of strings. These are file paths, and they are added to the cache we created. Lastly we'll add a few `catch` functions for any error handling we need.

Next step is activate:

```
// serviceworker.js
self.onactivate = function(event) {
    console.log('sw is up and running!');
}
```

This can be a good place for clean up, but we'll save that for another blog post.

We saved the best for last! Let's take a look at fetch.

```
// serviceworker.js
self.onfetch = function(event) {
    event.respondWith(
        caches.match(event.request)
        .then(function(cachedFiles) {
            if(cachedFiles) {
                return cachedFiles;
            } else {
                return fetch(event.request);
            }
        })
    );
}
```

This function will run when the service worker detects a fetch request. This function in _all_ caches attempting to find a matching request. If a matching request is found, the function returns that request. Otherwise the service worker will go ahead and go over the network with the request.

Let's take a closer look at `event.respondWith` and `caches.match`, both of which are pretty service worker specific.

[`event.respondWith`](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent/respondWith) is a function which allows you to intercept a fetch request and give your own response instead. It's important to use this function instead of simply returning a response because this is what allows your intercepted response to be sent to the correct place.

[`caches.match`](https://developer.mozilla.org/en-US/docs/Web/API/Cache/match) is a function that allows us to search through [CacheStorage](link) and find a match for our request. When we add something to our cache, it'll be stored in a stack, with the oldest additions at the bottom and the newest at the top. `caches.match` will find the newest match and return that. If it doesn't find a match at all, it'll return `null`.

And that's it! That's everything you need for a simple starter service worker! If you think service workers are super cool, I'd recommend seeing what else they can do, including background fetch, [in this blog post.](https://davidwalsh.name/background-sync)

_If you want to learn more about service workers, I hope you'll head over to serviceworkerbook.com and sign up for my mailing list, and follow me on Twitter! You'll be the first to know when my book, 'Let's Take This Offline' is out!_
