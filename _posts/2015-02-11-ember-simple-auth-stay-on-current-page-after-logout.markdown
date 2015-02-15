---
layout: post
title: "Ember Simple Auth Recipe 2 - Stay on the current page after logging out"
date: 2015-02-12 07:33
comments: true
categories: emberjs javascript ember-simple-auth authentication
---

This is a follow up to my [previous post]({% post_url 2015-02-11-ember-simple-auth-stay-on-current-page-after-authentication %}) on Ember Simple Auth where we looked at how to stay on the current page after authenticating. In this post we will look at a related scenario - How to stay on the current page after logging out.

This post describes how you can support the following situation in your app:

* You have a page in your application that is different depending on whether a user is authenticated or not.
* The unauthenticated version is publicly accessible.
* The authenticated version provides additional functionality specific to the current user.
* You want to keep the user on the current page after logging out.

### invalidateSession Conventions

By default, when a user logs out (triggering the invalidateSession action), Ember Simple Auth reloads the application by redirecting the user to application's root URL. If the root URL requires authentication, they will then be sent to the login page. This behavior is defined in the `sessionInvalidationSucceeded` action mixed into your application route.

If you look at the [source](https://github.com/simplabs/ember-simple-auth/blob/ddb4bea58bf6301bc738ceabbe6c2859fa00cd01/packages/ember-simple-auth/lib/simple-auth/mixins/application-route-mixin.js#L198-L202) for that method, you can see that it sends you to the application's root URL:

```js
sessionInvalidationSucceeded: function() {
  if (!Ember.testing) {
    window.location.replace(Configuration.applicationRootUrl);
  }
},
 ```

In this situation we may have a log out button on the page that invalidates the session. If we use Ember Simple Auth's default behavior, when the user logs out they will be redirected to the login page, but not brought back to this page after logging back in.

**We want the user to log out and stay on the page they were previously on.**

It turns out Ember Simple Auth provides a way around the default behavior by overriding the `sessionInvalidationSucceeded` action in our application route:

**app/routes/application.js**

```js
import Ember from 'ember';
import ApplicationRouteMixin from 'simple-auth/mixins/application-route-mixin';

export default Ember.Route.extend(ApplicationRouteMixin, {
  actions: {
    sessionInvalidationSucceeded: function(){
      var currentRoute = this.controllerFor('application').get('currentRouteName');

      if (currentRoute === 'stay-on-this-route') {
        window.location.reload();
      } else {
        this._super();
      }
    }
  }
}
```

In our action, we are getting the name of the current route from the application controller and using that to determine what should happen next. If the user is on the route `'stay-on-this-route'`, they will not be redirected to the root URL, and instead stay on the current page*. If they are not on that route, we call `this._super()` and defer to the default behavior.

*Thanks to Gabor Babicz for pointing out that we can use `window.location.reload()` to clear out the in-memory data after logging out. Without this, the Ember Data records would stay in memory until closing the window or logging into another account.

# Conclusion

You're probably asking yourself why anyone would want to do this. It's certainly an edge case, but in the app I've been working on, we have a use case for it - Users who share a computer, but have separate accounts. We want to provide them with a seamless UX if they are logged into the wrong account.

This same app also takes advantage of overriding `sessionInvalidationSucceeded` when a user deletes their account. Rather than redirecting them to the login form, we direct them to an "account deletion" route where they can provide feedback about why they closed their account.

That's all for now, but be on the lookout for more posts on hacking with Ember Simple Auth. Thanks for reading!

Shameless plug... If you're interested in working with awesome developers using Ember.js, come join us at [Agilion](http://agilion.com)!
