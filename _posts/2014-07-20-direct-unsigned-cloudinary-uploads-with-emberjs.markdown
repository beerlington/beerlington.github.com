---
layout: post
title: "Direct unsigned image uploads with Cloudinary and Ember.js"
date: 2014-07-20 13:50
comments: true
categories: emberjs cloudinary javascript
---

This is an update to my previous post on [Direct image uploads from the browser to the cloud with Ember.js and Cloudinary]({% post_url 2014-07-13-direct-image-uploads-with-emberjs-and-cloudinary %}). A few days after writing that, Cloudinary announced [support for unsigned image uploads](http://cloudinary.com/blog/direct_upload_made_easy_from_browser_or_mobile_app_to_the_cloud). This post is intended to supplement my original tutorial and only includes the pieces that need to change from those examples. For comparison, [here is everything that can be removed from the example Rails API](https://github.com/beerlington/cats-api/compare/unsigned-uploads) and [everything that needs to change in the Ember application](https://github.com/beerlington/cats-ui/compare/unsigned-uploads).

**Note:** You will need [cloudinary_js](https://github.com/cloudinary/cloudinary_js) >= 1.0.17 for this feature. If you used my [example ember-cli project](https://github.com/beerlington/cats-ui), you will already have this version.

## Signed vs Unsigned Uploads

A *signed* upload, as described in my original post, is an upload request to Cloudinary that sends an extra "authentication signature" parameter with the image data. This signature is created using your Cloudinary secret key, and for security reasons, requires a server-side API request to generate it. Signed uploads are tricky to work with because they require you to have total control over the server-side application, which may not be the case if you're just building a client-side application. They are also hard to troubleshoot because of the order that things must be done in the JavaScript.

On the other hand, *unsigned* uploads do not require an authenticated signature and are easier to get started with because you *completely* bypass your API server. The way unsigned uploads work is that they use an [upload_preset](http://cloudinary.com/blog/centralized_control_for_image_upload_image_size_format_thumbnail_generation_tagging_and_more) generated in the Cloudinary management UI, rather than the API key and secret that we used previously. According to the Cloudinary docs, "not all upload parameters can be specified directly when performing unsigned upload calls", but it seems like most of the functionality you would want is still provided. The only feature I found disabled with unsigned uploads was the ability to overwrite existing images with the same public ID.

When you add a new preset in the Cloudinary UI, it will automatically generate a "Preset name". While you're in there, you also specify settings like tags, a folder, or even eager transformations so they don't have to be created on-the-fly. All you need to get up and running is the preset name - everything else can be left with the default settings. Note that preset settings will take precedence over any options sent with the image upload request.

## Environment Settings

In an ember-cli project, environment specific settings are defined in config/environment.js. We'll add a setting for the preset in addition to the existing cloud name, and remove `CLOUDINARY_KEY` because it is no longer required.

```js
// omitted ...

if (environment === 'development') {
  ENV.CLOUDINARY_NAME = 'dev_cloud';
  ENV.CLOUDINARY_UPLOAD_PRESET = 'abc123';
}

if (environment === 'production') {
  ENV.CLOUDINARY_NAME = 'production_cloud';
  ENV.CLOUDINARY_UPLOAD_PRESET = '123abc';
}

// omitted
```

## Cloudinary Config Initializer

Since you no longer need to specify the **api_key**, that option can be removed from the initializer, leaving just the cloud name.

```js
export default {
  name: 'cloudinary',

  initialize: function(/* container, app */) {
    $.cloudinary.config({
      cloud_name: CatsUiENV.CLOUDINARY_NAME
    });
  }
};
```

## Changes to the View

The Ember view we introduced previously can be simplified because the app no longer needs to load the authentication signature from the server. We don't need to wrap the Cloudinary setup in a callback that uses `Ember.run` either, which felt a little hacky to me.

```js
export default Ember.View.extend({
  tagName: 'input',
  name: 'file',
  classNames: ['cloudinary-fileupload'],
  attributeBindings: ['name', 'type'],
  type: 'file',

  didInsertElement: function() {
    var controller = this.get('controller');

    this.$().unsigned_cloudinary_upload(
      CatsUiENV.CLOUDINARY_UPLOAD_PRESET, {
        cloud_name: CatsUiENV.CLOUDINARY_NAME
      }, {
        disableImageResize: false,
        imageMaxWidth: 1000,
        imageMaxHeight: 1000,
        acceptFileTypes: /(\.|\/)(gif|jpe?g|png|bmp|ico)$/i,
        maxFileSize: 5000000 // 5MB
      }
    );

    this.$().bind('fileuploaddone', function (e, data) {
      controller.set('newCat.cloudinaryPublicId', data.result.public_id);
    });
  }
});
```

The entire view is included above, but I wanted to point out a few changes in this particular block of code:

```js
this.$().unsigned_cloudinary_upload(
  CatsUiENV.CLOUDINARY_UPLOAD_PRESET, {
    cloud_name: CatsUiENV.CLOUDINARY_NAME
  }, {
    disableImageResize: false,
    imageMaxWidth: 1000,
    imageMaxHeight: 1000,
    acceptFileTypes: /(\.|\/)(gif|jpe?g|png|bmp|ico)$/i,
    maxFileSize: 5000000 // 5MB
  }
);
```

You'll notice above that we're now using `unsigned_cloudinary_upload()` which takes three arguments, as opposed to `cloudinary_fileupload()` from the previous post which only required one. The first argument is the name of the upload preset that you create in the Cloudinary settings UI. The second argument is the list of Cloudinary-specific upload parameters that are *not* defined by the preset. The last argument is the list of [jQuery File Upload options](https://github.com/blueimp/jQuery-File-Upload/wiki/Options#image-preview--resize-options) that we specified in the original implementation.

One of the things that confused me was the fact that [Cloudinary's tutorial](http://cloudinary.com/blog/direct_upload_made_easy_from_browser_or_mobile_app_to_the_cloud) passes the `cloud_name` option to the `unsigned_cloudinary_upload()` function. It seems redundant to specify the value in both places, but I tried removing the cloud name setting from the config initializer and got an `"Uncaught Unknown cloud_name"` error.

For the sake of demonstrating where Cloudinary recommends putting it, I am specifying `cloud_name` in both the initializer *and* the view, but this is not required. In otherwords, `cloud_name` does not need to be passed to `unsigned_cloudinary_upload()`, but would still require an empty object literal to be passed as the second argument like so:


```js
this.$().unsigned_cloudinary_upload(
  CatsUiENV.CLOUDINARY_UPLOAD_PRESET, {}, {
    disableImageResize: false,
    imageMaxWidth: 1000,
    imageMaxHeight: 1000,
    acceptFileTypes: /(\.|\/)(gif|jpe?g|png|bmp|ico)$/i,
    maxFileSize: 5000000 // 5MB
  }
);
```

## Conclusion ##

Maybe I'm missing something obvious, but what prevents someone from copying your cloud name and preset name and using them to upload files from a different application? In an ember-cli application, the settings from `config/environment.js` are not obfuscated and can be easily viewed in the source of the generated index.html page. The only info I could find that sort of addresses the issue was a single sentence on the [Cloudinary blog](http://cloudinary.com/blog/direct_upload_made_easy_from_browser_or_mobile_app_to_the_cloud) stating "Unsigned uploading makes this much simpler to use in modern apps, while security measures are taken to detect and prevent abuse attempts.". They don't state exactly what security measures they're taking, so I guess you'll have to take their word for it.

Assuming they have the security thing figured out, this is a much easier way to get started with direct image uploading to Cloudinary from an Ember.js app. There is no longer a dependency on the server to generate an authentication signature, which was arguably the most confusing aspect of setting this up.
