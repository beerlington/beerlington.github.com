---
layout: post
title: "Ember Simple Auth Recipe - Stay on the current page after authenticating"
date: 2015-02-04 19:27
comments: true
categories: emberjs javascript ember-simple-auth authentication
---

This is the first of a short series of Ember.js authentication recipes using [Ember Simple Auth](http://ember-simple-auth.com/).

I've been using Ember Simple Auth for authentication in my Ember apps for close to a year now. I love it because it is easy to setup, and is extensible beyond its out-of-the-box capabilities.

As with other frameworks or libraries that follow the "convention over configuration" philosophy, doing things the expected way is easy, but doing things otherwise can sometimes be a pain. They usually *allow* you to do what you're trying to do, but it's not always obvious what the best or easiest way *is*.

## The Problem

This post describes how you can support the following situation in your app:

* You have a page in your application that is different depending on whether a user is authenticated or not.
* The unauthenticated version is publicly accessible.
* The authenticated version provides additional functionality specific to the current user.
* You want to allow the user to stay on the current page after logging in.

### Login Route Conventions

With the [routeAfterAuthentication](http://ember-simple-auth.com/ember-simple-auth-api-docs.html#SimpleAuth-Configuration-routeAfterAuthentication) option, Ember Simple Auth lets you define a single route that the user transitions to after logging in:

**app/initializers/my-custom-auth.js**

```js
export default {
  name: 'authentication',
  before: 'simple-auth',

  initialize: function(container) {
    config['simple-auth'] = {
      routeAfterAuthentication: 'my-default-route'
    };
  }
};
```

Any time a user logs into the app, they will be sent to whatever route is specified by `routeAfterAuthentication` - in this case it's `'my-default-route'`.

One exception to this default behavior is when a logged out user tries to access a page that requires authentication. The user is transitioned to the login page, and when they log in, they're automatically brought back to the page they were trying to access.

**What about pages that don't *require* authentication, but instead have a logged out version that differs slightly from the logged in version?**

In this situation we may have a link that takes the user to the login page, or maybe even embed the login form directly in the page. If we use Ember Simple Auth's default behavior, when the user logs in, they will be brought to whatever route is defined by `routeAfterAuthentication`. This is not the behavior what we want.

**We want the user to log in and see the authenticated version of the page they were previously on.**

### Defying Conventions

Ember Simple Auth does provide a way around this, but it's not very obvious from the documentation. When logging in, the method [sessionAuthenticationSucceeded](http://ember-simple-auth.com/ember-simple-auth-api-docs.html#SimpleAuth-ApplicationRouteMixin-sessionAuthenticationSucceeded) is invoked. This method is included with the [ApplicationRouteMixin](http://ember-simple-auth.com/ember-simple-auth-api-docs.html#SimpleAuth-ApplicationRouteMixin-sessionAuthenticationSucceeded) which should be mixed into your app's Application route.

If you look at the [source](https://github.com/simplabs/ember-simple-auth/blob/ddb6b8cec5f7bee3d9b1dd416bf681bc1465f4f8/packages/ember-simple-auth/lib/simple-auth/mixins/application-route-mixin.js#L137-L145) for that method, you can see that before it goes to the default route, it first tries to take the user to the last attempted transition:

```js
sessionAuthenticationSucceeded: function() {
  var attemptedTransition = this.get(Configuration.sessionPropertyName).get('attemptedTransition');
  if (attemptedTransition) {
    attemptedTransition.retry(); // <== would you look at that!
    this.get(Configuration.sessionPropertyName).set('attemptedTransition', null);
  } else {
    this.transitionTo(Configuration.routeAfterAuthentication);
  }
},
 ```

This is how it takes the user back to a page that required authentication once the user logs in, and we can use this behavior to our advantage.

When a route is entered and the beforeModel hook is called, a transition object for the current route is passed in. We can capture this transition and set it on the session and trick Ember Simple Auth into using it the next time the user logs in.

In our route:

```js
export default Ember.Route.extend({
  beforeModel: function(transition) {
    if (!this.get('session.isAuthenticated')) {
      this.set('session.attemptedTransition', transition);
    }
  }
});
```

If we use this logic on a page where we want the user to come back after logging in, it will work exactly like we'd expect. The transition will be retried when `sessionAuthenticationSucceeded` is called, and send the user back to the page they were previously on.

## Conclusion

Conventions are awesome because they make the easy things simple and let developers focus on creating value. One sign of a great library is when you can bend the rules a bit without writing lots of unmaintainable hacks that break every time the library is upgraded. Ember Simple Auth is one of these libraries.

We've looked at a simple example of bending the rules here to create a better experience for the user. To compliment this post, next time we'll look at how to stay on the current page after logging *out*.
