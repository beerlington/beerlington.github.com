---
layout: post
title: "Direct image uploads from the browser to the cloud with Ember.js and Cloudinary"
date: 2014-07-13 10:19
comments: true
categories: emberjs cloudinary javascript
---

Uploading images to a web application has historically sucked. If you're in the Ruby on Rails world, there are a few gems like [Paperclip](https://github.com/thoughtbot/paperclip) and [Carrierwave](https://github.com/carrierwaveuploader/carrierwave) which are great attempts at solving this problem, but are about as fun to work with as a pile of rocks. I'm picking on these two in particular because they are the [most popular Rails file upload gems](https://www.ruby-toolbox.com/categories/rails_file_uploads), and every time I have to use one of these libraries, I get pissed off.

Some of the issues I have with these gems include:

* They try to cover every possible use case, making it difficult to
  figure out what code you actually need.
* The DSLs are designed with nearly infinite flexibility, making them hard
  to learn and remember.
* They require storing image meta data in the database and it's hard to remember which columns are required.
* You're forced to manually run a server-side process if you decide you want to resize the images after they've already been uploaded.
* They're not opinionated enough and we end up with a slightly different implementation each time.

The last point is the main reason I'm writing this post. I like opinions and hate thinking about something boring after I've done it once. This is why I use Ember.js, Rails, OS X, a car with automatic transmission, and the popcorn setting on my microwave. I also don't set the clock on my microwave, but that has nothing to do with this blog post.

## Re-evaluating Our Needs

When [Agilion](http://agilion.com/) started working on a new customer project last month, I was less than thrilled to find out that we'd need to support image uploads. This was our first Ember.js project that required image uploads, so after a short discussion among our team, we wanted to try something different.

We decided to use [Cloudinary](http://cloudinary.com/) with direct image uploads to see if it made our lives any easier. The idea behind it is that the images go directly to the cloud storage host, and never pass through your server. If you've ever used Stripe for credit card processing, it's basically the same concept but for images. None of us had ever done this before, but the prospect of not dealing with storage, resizing images, and complicated documentation sounded too good to be true.

Our main goal for trying this approach was to reduce the development effort to support image uploading. Since all of our new projects are in Ember.js, we wanted to find a solution that would work well for this framework. As contract developers, we try to reduce our implementation time to keep costs down for our customers, and this seemed like a great opportunity to do that.

## Background Information and a Demo

After doing some research on direct image uploads, we learned that the basic flow works like this:

* User visits page and clicks file upload button
* User selects one or more images
* Images are sent directly to Cloudinary from the browser
* Cloudinary sends back meta data for each image one at a time, including the "public id" (later used to retrieve images)
* Browser makes API request to the server to create a new record for each image, and includes the Cloudinary "public id" in the request

Rather than including a bunch of client-specific code that would not be fully functional, I created a simple [Ember application](https://github.com/beerlington/cats-ui) that is backed by a [Rails API](https://github.com/beerlington/cats-api). They are in separate repos because that is how we develop Ember apps at Agilion.

At the time of writing, the demo is using:

* [ember](http://emberjs.com/) - 1.6.0
* [ember-data](https://github.com/emberjs/data) - 1.0.0-beta.8
* [ember-cli](http://iamstef.net/ember-cli/) - 0.0.37
* [cloudinary_js](https://github.com/cloudinary/cloudinary_js) - 1.0.17
* [jquery-ui](http://jqueryui.com/) - 1.10.4
* [jquery-file-upload](http://blueimp.github.io/jQuery-File-Upload/) - 9.5.7
* [Rails](http://rubyonrails.org/) - 4.1.2
* [Rails::API](https://github.com/rails-api/rails-api) - 0.2.1

## Required Pieces of the Application

### Client-side (Ember)

* Initializer with Cloudinary configuration settings
* Cat Model
* Cats Controller
* Cloudinary File Input View
* Cats Template
* Cloudinary Image Tag Helper

### Server-side (Rails)

* API endpoint to generate authentication signature
* API endpoint to persist a record

I've only included the most important pieces of the Ember code to reduce the length of this post. Things like the router and Cats route are omitted but can be found in the example app. I've also left out the server-side code we use to generate the authentication signature because I want this to be a server-agnostic tutorial.

## Cloudinary Config Initializer

The initializer is simply telling the Cloudinary plugin what the **cloud name** and **public key** are for this application. In an Ember-CLI project, these values are stored in the [environment-specific configuration](https://github.com/beerlington/cats-ui/blob/master/config/environment.js#L30-L31). Note that there *is* a private key, but this is *not* set in the Ember app. The private key is used when creating an authentication signature (see section below) and is only included in the Rails app settings.

**app/initializers/cloudinary.js** - [GitHub](https://github.com/beerlington/cats-ui/blob/master/app/initializers/cloudinary.js)

```js
export default {
  name: 'cloudinary',

  initialize: function(/* container, app */) {
    $.cloudinary.config({
      cloud_name: CatsUiENV.CLOUDINARY_NAME,
      api_key:    CatsUiENV.CLOUDINARY_KEY
    });
  }
};
```

## Cat Model

The Cat model is pretty straightforward. We have a property for the name
of the cat, and a property for the Cloudinary Public ID. Both of these
attributes correspond to database columns on the [Cat model](https://github.com/beerlington/cats-api/blob/master/app/models/cat.rb) in the Rails application. Note that both are required in the demo.

**app/models/cat.js** - [GitHub](https://github.com/beerlington/cats-ui/blob/master/app/models/cat.js)

```js
export default DS.Model.extend({
  name:               DS.attr('string'),
  cloudinaryPublicId: DS.attr('string')
});
```

## The Cats Template

In the demo, the Cats template is used as both an index page to list all of the cats, as well as the location of the new cat form. I have only
included the form portion here.

**app/templates/cats.hbs** - [GitHub](https://github.com/beerlington/cats-ui/blob/master/app/templates/cats.hbs)

{% highlight html %}
{% raw %}
<form role="form" {{action 'createCat' on='submit'}}>
  <div class="form-group">
    <label for="cat-name">Cat Name</label>
    {{input value=newCat.name class="form-control" id="cat-name" placeholder='Cat Name'}}
  </div>
  <div class="form-group">
    <label for="cat-image">File input</label>
    {{view 'cloudinary'}}
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
</form>
{% endraw %}
{% endhighlight %}

In the code above, you can see that we are rendering the file input tag using an Ember view called 'cloudinary'.

## The Cloudinary View

The Cloudinary file input view has a few responsibilities related to setup and configuration of the Cloudinary widget. Here is the entire view in one piece, but it is broken down and explained in detail further down:

**app/views/cloudinary.js** - [GitHub](https://github.com/beerlington/cats-ui/blob/master/app/views/cloudinary.js)

```js
import Ember from 'ember';

export default Ember.View.extend({
  tagName: 'input',
  name: 'file',
  classNames: ['cloudinary-fileupload address-form-upload'],
  attributeBindings: ['name', 'type', 'data-cloudinary-field', 'data-form-data'],
  type: 'file',
  'data-cloudinary-field': 'image_id',

  didInsertElement: function() {
    var _this = this,
      controller = this.get('controller');

    $.get(CatsUiENV.API_NAMESPACE + '/cloudinary_params', {timestamp: Date.now() / 1000}).done(function(response) {
      // Need to ensure that the input attribute is updated before initializing Cloudinary
      Ember.run(function() {
        _this.set('data-form-data', JSON.stringify(response));
      });

      _this.$().cloudinary_fileupload({
        disableImageResize: false,
        imageMaxWidth: 1000,
        imageMaxHeight: 1000,
        acceptFileTypes: /(\.|\/)(gif|jpe?g|png|bmp|ico)$/i,
        maxFileSize: 5000000 // 5MB
      });

      _this.$().bind('fileuploaddone', function (e, data) {
        controller.set('newCat.cloudinaryPublicId', data.result.public_id);
      });
    });
  }
});
```


First, it is responsible for loading the authentication signature from the Rails API (details below):

```js
$.get(CatsUiENV.API_NAMESPACE + '/cloudinary_params', {timestamp: Date.now() / 1000}).done(function(response) {
  Ember.run(function() {
    _this.set('data-form-data', JSON.stringify(response));
  });

  // ...
```

It appends the returned signature object to a data attribute on the file input HTML element called "data-form-data". Note that this is done within an `Ember.run` function to ensure that it is finished before initializing up the Cloudinary file upload. This was something that stumped us at first, and I'm sure there's a better approach.

After the signature data is set, the view initializes the Cloudinary file upload widget on the input.  Here is where we define the settings for our project such as the maximum file size allowed, accepted file types (images) and the maximum dimensions of the image (it automatically scales them down):

```js
_this.$().cloudinary_fileupload({
  disableImageResize: false,
  imageMaxWidth: 1000,
  imageMaxHeight: 1000,
  acceptFileTypes: /(\.|\/)(gif|jpe?g|png|bmp|ico)$/i,
  maxFileSize: 5000000 // 5MB
});
```

The last functionality this view is responsible for is handling the Cloudinary response and setting a cat's Cloudinary public ID. Cloudinary sends various events to the input including progress and completion, to which you can bind functionality. In the demo, we're just binding to the 'fileuploaddone' event and setting a controller property when it's complete.

```js
_this.$().bind('fileuploaddone', function (e, data) {
  controller.set('newCat.cloudinaryPublicId', data.result.public_id);
});
```

## The Cats Controller

In the demo, the cats controller is a simplified version of what you
would use in a production application. Here we are just persisting the cat
with ember-data, and then creating a new Cat record so the form has a
different object.

```js
export default Ember.ArrayController.extend({
  // omitted ...

  actions: {
    createCat: function() {
      var _this = this;

      this.get('newCat').save().then(function() {
        _this.set('newCat', _this.store.createRecord('cat'));
      });
    }
  }
});
```

## Authentication Signature

Another thing that tripped us up was the process for authenticating the Cloudinary image upload request. Cloudinary requires that you generate an authentication signature on the server and pass it to the client *before* initializing the file upload. It wasn't clear from the Cloudinary documentation *why* this needed to be done or what was actually required to do it.

It also seemed counter-intuitive at first that we were depending on the Rails API for something that was supposedly 100% client-side, but from a security standpoint, it makes perfect sense. The authentication signature is a property that is sent to Cloudinary with the images, and is used to authenticate your request.

When the page loads, our application makes a request to the server (via the Cats view) that generates an authentication signature. I've omitted the [controller code to generate the JSON](https://github.com/beerlington/cats-api/blob/master/app/controllers/v1/cloudinary_params_controller.rb), but the request will look something like this:

```
http://localhost:3000/v1/cloudinary_params?timestamp=1405265520.34
```

That request returns a response that looks like this:

```json
{
  "timestamp":1405265520,
  "signature":"4a49f7e9009924b0d811e9bdc8798ca19cdb2da4",
  "api_key":"123456789012345"
}
```

Note that the API key is the same *public* key that is also set in the
Cloudinary initializer above.

As described in the Cats view section, this entire signature object is appended to a data attribute on the file input field, and the Cloudinary upload widget automatically sends it with the image data when a user selects an image.

## Viewing the images

At this point you have an Ember.js application that can upload images directly to Cloudinary, but what good is that if you can't retrieve and view the images?

Another frustrating thing about the Cloudinary documentation is that everything we needed for this project was scattered among at least three different sources. After we had image uploads working, the docs we had referenced assumed we were using our server-side framework (Rails) to render our templates. Since Ember uses Handlebars and the templates are rendered 100% client-side, we needed a different solution.

We ended up creating a helper to handle this functionality, but in retrospect, a component may have been a more semantic option for us.

**app/helpers/cloudinary-tag.js** - [GitHub](https://github.com/beerlington/cats-ui/blob/master/app/helpers/cloudinary-tag.js)

```js
export default Ember.Handlebars.makeBoundHelper(function(publicId, options) {
  if (Em.isBlank(publicId)) { return ''; }

  var height = options.hash.height || 535,
    width = options.hash.width || 535,
    crop = options.hash.crop || 'fill',
    cssClass = options.hash.class || 'cloudinary-image';

  return new Ember.Handlebars.SafeString($.cloudinary.image(publicId, {
    width: width,
    height: height,
    crop: crop,
    class: cssClass
  }).prop('outerHTML'));
});
```

Again, I've omitted the [irrelevant template code](https://github.com/beerlington/cats-ui/blob/master/app/templates/cats.hbs#L5-L14) and only left in what is required to render the HTML image tag:

**app/templates/cats.hbs** - [GitHub](https://github.com/beerlington/cats-ui/blob/master/app/templates/cats.hbs)

{% highlight html %}
{% raw %}
<!-- omitted... -->
{{cloudinary-tag cloudinaryPublicId}}
<!-- omitted... -->
{% endraw %}
{% endhighlight %}

## Conclusion

While not necessarily specific to this approach, the biggest downside I can see is that you're locking yourself into a single vendor. The pricing is reasonable for smallish applications that are just getting off the ground, but if you expect your users to upload thousands of large images, you'll quickly find yourself in Cloudinary's "Enterprise/Custom Plan" (which I suspect is significantly more expensive than straight up Amazon S3). During testing, we started on the free plan which includes 500MB of total storage and 1GB of monthly bandwidth. We ended up blowing through the 1GB of bandwidth in a couple days and had to upgrade to the $35/month "Basic" plan. In Cloudinary's defense, we were uploading large images during testing, which was not really necessary since we could have just been testing with much smaller file sizes.

Despite some initial frustration with the Cloudinary documentation, my overall impression is that this is the future of image and file uploading in JavaScript web applications. By not depending on the server to process image and file uploads, the API code is simpler and easier to maintain. Pushing this responsibility onto a cloud host like Cloudinary allows us to develop faster and focus on functionality that is relevant to our customers' applications.
