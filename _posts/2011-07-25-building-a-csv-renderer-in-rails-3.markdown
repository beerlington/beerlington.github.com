---
layout: post
title: "Building a Custom CSV Renderer in Rails 3"
date: 2011-07-25 22:18
comments: true
categories: rails renderers
---

This post will walk you through creating a simple renderer in Rails
3.

## What are renderers?

A renderer in Rails is a way of customizing how content is rendered for
the browser or any client interacting with your web service. Rails has
a handful of built in rendering formats such as *html*, *xml*, and
*json*, and exposes an effortless method for adding additional custom rendering
functionality that can be shared among your controllers and
applications.

Custom renderers provide a standard, reusable interface for rendering content,
in turn allowing you to DRY up your application logic architecture.
This post will show you how to add a custom renderer that converts
ActiveRecord collections to a downloadable CSV format.

Below is an example of how the final CSV renderer will be used:

*app/locations_controller.rb*

```ruby
class LocationsController < ApplicationController
  def index
    @locations = Location.all

    respond_to do |format|
      format.html
      format.csv  { render :csv => @locations, :except => [:id] }
    end
  end
end
```

## Some background info:

Before getting started with our CSV renderer, let's look at the
[Rails source](https://github.com/rails/rails/blob/master/actionpack/lib/action_controller/metal/renderers.rb)
to see how it defines its own custom renderers:

*actionpack/lib/action_controller/metal/renderers.rb*

```ruby
module ActionController
  def self.add_renderer(key, &block)
    Renderers.add(key, &block)
  end

  module Renderers
    # Lots of ommitted code...

    def self.add(key, &block)
      define_method("_render_option_#{key}", &block)
      RENDERERS[key] = block
    end

    # More ommitted code...

    add :xml do |xml, options|
      self.content_type ||= Mime::XML
      xml.respond_to?(:to_xml) ? xml.to_xml(options) : xml
    end
  end
end
```

At the top you see an *add_renderer* method, which will be our
interface to add a custom renderer. Ignoring the fact that
I've removed some of the implementation details,
further down you see a class method called *add* which does two things:
1) It defines an internal method used by the rendering stack,
and 2) stores the key (:xml, :json, etc) in a hash.

As we move to the bottom of this file, you'll see where *add* is being used
to define an XML renderer. This method takes a block with two arguments.
The first argument is an object that responds to *to_xml*, and the second
is a set of options that are passed as arguments to *to_xml*. If for some reason
the object doesn't respond to *to_xml* then it simply returns itself.

If you've ever used the Rails scaffold generator, you will notice that
it includes code for responding to XML requests:

*default scaffolded controller.rb*

```ruby
def show
  @location = Location.find(params[:id])

  respond_to do |format|
    format.html # show.html.erb
    format.xml  { render :xml => @location }
  end
end
```

This is exacly what adding the XML renderer has provided, a clean syntax
for allowing the server to respond with XML formatted data.

## Why should I use it?

Let's say you have a *Location* model with the
following schema:

*db/schema.rb*

```ruby
ActiveRecord::Schema.define(:version => 20110726022558) do
  create_table "locations", :force => true do |t|
    t.string   "name"
    t.string   "address"
    t.string   "city"
    t.string   "state"
    t.string   "zip"
    t.datetime "created_at"
    t.datetime "updated_at"
  end
end
```

Although your main application may use all of these fields, you might need to
import this data into some sort of enterprise/exchange-powered legacy
system in CSV format. You could easily define a *to_csv* class method on your
Location model and just call that from the controller, but that presents
a few problems. The Location model shouldn't know anything
about converting itself to a CSV format because this violates the rule that
components should have a single, well-defined purpose. Also, what if you want to
download other models in CSV format? These are problems that respond_to and render
aim to solve.

## How does it work?

Looking back to the first code snippet I posted, you can see that the
CSV render syntax in my controller is very similar to the XML render syntax. The only difference is
that our CSV format allows you to specify which columns (attributes) you
want to include or exclude in the downloaded CSV file. This detail has
almost nothing to do with the renderer itself, and is implemented by
the object's Array class, as shown below.

To understand how this works, let's look at our custom CSV renderer:

*csv_renderer.rb*

```ruby
require 'action_controller/metal/renderers'

ActionController.add_renderer :csv do |csv, options|
  self.response_body = csv.respond_to?(:to_csv) ? csv.to_csv(options) : csv
end
```

You will notice a few differences between our custom renderer
and the XML renderer defined by Rails. First, we are using
*add_renderer* as opposed to just *add*. This is a personal preference
as you can also use: `ActionController::Renderers.add
:csv`. Both methods accomplish exactly the same thing, but I find
*add_renderer* to be slightly more readable.
Second, we are not setting the content_type. The
reason for this is because Rails automatically adds a CSV Mime Type
within the action_pack library. I'm not going to go into the details of how that
works, but feel free to [explore the complete list of Mime Types](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/http/mime_types.rb).
The last detail to note is that we are requiring
*action_controller/metal/renderers* which provides us access to the *add_renderer*
method. You could alternatively require any module that inclues this
file, such as *action_controller/base* if you need access to methods like
[send_data](http://api.rubyonrails.org/classes/ActionController/Streaming.html#method-i-send_data).

Finally, let's look at the *to_csv* Array method. The
reason this method is defined on Array is because ActiveRecord converts
collection queries such as `Location.where(:state => 'vt')` or `Location.all`
to arrays.

*array.rb*

```ruby
class Array

  # Converts an array to CSV formatted string
  # Options include:
  # :only => [:col1, :col2] # Specify which columns to include
  # :except => [:col1, :col2] # Specify which columns to exclude
  # :add_methods => [:method1, :method2] # Include addtional methods that aren't columns
  def to_csv(options={})
    return '' if empty?
    return join(',') unless first.class.respond_to? :column_names

    columns = first.class.column_names
    columns &= options[:only].map(&:to_s) if options[:only]
    columns -= options[:except].map(&:to_s) if options[:except]
    columns += options[:add_methods].map(&:to_s) if options[:add_methods]

    csv = [columns.join(',')]
    csv.concat(map {|v| columns.map {|c| v.send(c) }.join(',') })

    csv.join("\n")
  end
end
```

Above we can see that *to_csv* takes a hash of options which are used to
determine which columns should be included in the CSV file. One
option `:only => [:col1, :col2]` allows you to specify the exact
columns you want included, another option, `:except => [:col1, :col2]`, 
allows you to specify which columns you DO NOT want to include, and the
last option, `:add_methods => [:method1, :method2]` allows you to add
data defined in methods that aren't saved in the database. Calling
*to_csv* without any arguments will include all columns.

## Usage examples:

Tying that all together, we can now use this renderer in our controllers
to let users or clients download CSV data. Here are a few examples:

### Generate a CSV that includes every column:

*app/controllers/location_controller.rb*

```ruby
class LocationsController < ApplicationController
  def index
    @locations = Location.all

    respond_to do |format|
      format.csv  { render :csv => @locations }
    end
  end
end
```

### Generate a CSV that includes every column except the id:

*app/controllers/location_controller.rb*

```ruby
class LocationsController < ApplicationController
  def index
    @locations = Location.all

    respond_to do |format|
      format.csv  { render :csv => @locations, :except => [:id] }
    end
  end
end
```

### Generate a CSV that includes only the state and zipcode:

*app/controllers/location_controller.rb*

```ruby
class LocationsController < ApplicationController
  def index
    @locations = Location.all

    respond_to do |format|
      format.csv  { render :csv => @locations, :only => [:state, :zip] }
    end
  end
end
```

### Generate a CSV that adds a model method:

*app/controllers/location_controller.rb*

```ruby
class LocationsController < ApplicationController
  def index
    @locations = Location.all

    respond_to do |format|
      format.csv  { render :csv => @locations, :add_methods => [:my_method] }
    end
  end
end
```

## Finishing up

This post walked you through a simple yet practical approach to building
a custom Rails renderer in Rails 3. While it may be overkill for one-off
uses in a small code base, it shows it's true potential in large
applications by helping prevent redundant code, and for building APIs that may need to
respond to formats other than JSON and XML.

## Additional Resources
All the code used in this post was developed for a RubyGem called [render_csv](https://github.com/beerlington/render_csv).
For more information about renderers and the Rails rendering stack, I highly recommend
[Crafting Rails Applications](http://pragprog.com/book/jvrails/crafting-rails-applications)
 by [José Valim](https://twitter.com/josevalim). Also, the [Layouts and Rendering Rails Guide](http://guides.rubyonrails.org/layouts_and_rendering.html)
provides a comprehensive overview of rendering in controllers as well as
views.


