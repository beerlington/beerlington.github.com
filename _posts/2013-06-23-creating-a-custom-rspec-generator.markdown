---
layout: post
title: "Creating a Custom Rspec Generator"
date: 2013-06-23 12:43
comments: true
categories: rspec rails
---

I'm a huge proponent of test driven development and have been feeling
sort of guilty about a gem that I wrote which does not encourage
testing whatsoever. The gem, [ClassyEnum](https://github.com/beerlington/classy_enum),
provides class-based enumerator functionality on top of Active Record. This blog post is not so much
about the gem itself, so If you're interested in reading more about it,
the [README](http://beerlington.com/classy_enum/) has some good examples.

Historically, ClassyEnum has had a built-in generator to
quickly create classes that represent enum members. While this has served me
well, it has never generated any spec files along with these classes.
I always end up either creating them manually, or just forgoing tests
altogether. Neither option was great, so I wanted to see what it would
take to create spec files automatically, similar to how Rails can
generate model specs when using the model or scaffold generators.

When ClassyEnum is installed in your Rails project, you can run the generator like so:

```irb
$ rails g classy_enum Priority low medium high
      create  app/enums
      create  app/enums/priority.rb
```

Which produces the following file:

*app/enums/priority.rb*

```ruby
class Priority < ClassyEnum::Base
end

class Priority::Low < Priority
end

class Priority::Medium < Priority
end

class Priority::High < Priority
end
```

Each Priority subclass represents an enum member, and behaves like a
true Ruby class. This boilerplate code starts out innocent enough,
as most code does, but over time, I find myself adding logic
and properties to these classes. Depending on how lazy I am,
sometimes I test it, sometimes I don't.

## Exploring the Un*spec*tacular Generator

The code for the "main" generator in ClassyEnum is fairly straightforward. It
has a description, takes a few arguments, and copies a dynamically generated
file into app/enums, creating the directory if it does not exist.

*lib/generators/classy_enum/classy_enum_generator.rb*

```ruby
class ClassyEnumGenerator < Rails::Generators::NamedBase
  desc "Generate a ClassyEnum definition in app/enums/"

  argument :name, :type => :string, :required => true, :banner => 'EnumName'
  argument :values, :type => :array, :default => [], :banner => 'value1 value2 value3 etc...'

  source_root File.expand_path("../templates", __FILE__)

  def copy_files # :nodoc:
    empty_directory 'app/enums'
    template "enum.rb", "app/enums/#{file_name}.rb"
  end
end
```

I would recommend the [Rails Generator Guide](http://guides.rubyonrails.org/generators.html)
if you aren't familiar with the syntax. You may also want to read the
[Thor documentation](http://rdoc.info/github/wycats/thor/master/Thor/Actions.html)
which is the foundation for the Rails generator DSL.

## Making the Generator *Spec*tacular

I had four main requirements for my generator's new behavior:

1. It must automatically create specs in spec/enums when the generator
   is run.
2. The specs it creates must work out of the box (even if they are
   pending).
3. It cannot break existing behavior.
4. It must be future proof by not relying on or hacking internal Rails or Rspec code

After digging around on Stack Overflow and reading the Rails Guide,
I discovered that Rails exposes a [`hook_for`](http://api.rubyonrails.org/classes/Rails/Generators/Base.html#method-c-hook_for)
method. When passed the `:test_framework` argument,
the main generator can automatically figure out which test framework your application is
using, and based on naming conventions, which spec generator to load.
All I had to do was add the hook to my existing generator, and create
some support files to go along with it.

*lib/generators/rspec/classy_enum_generator.rb*

```ruby
class ClassyEnumGenerator < Rails::Generators::NamedBase
  desc "Generate a ClassyEnum definition in app/enums/"

  argument :name, :type => :string, :required => true, :banner => 'EnumName'
  argument :values, :type => :array, :default => [], :banner => 'value1 value2 value3 etc...'

  source_root File.expand_path("../templates", __FILE__)

  def copy_files # :nodoc:
    empty_directory 'app/enums'
    template "enum.rb", "app/enums/#{file_name}.rb"
  end

  hook_for :test_framework # <======= Add the hook here
end
```

More information about this hook can be found in the
["Customizing Your Workflow"](http://guides.rubyonrails.org/generators.html#customizing-your-workflow)
section of the Rails Guide.

After adding just the hook, I knew it wasn't going to work, but because I like
taking baby steps when coding, I rebuilt and
installed the gem, updated my project's ClassyEnum dependency,
and ran the generator anyway:

```irb
rails g classy_enum Priority low medium high
      create  app/enums
      create  app/enums/priority.rb
       error  rspec [not found]
```

This error is just saying that the test_framework generator could
not be found, which was expected because I had not created it yet.
Since I don't have any tests for the generator itself, I used this message
as a failing test, and my passing test would be when the enum spec was generated.

According to the Rails Guide, the `hook_for` method will search in a
few places for the generator, looking for one of a few different
class names. By default it looks for a class named after the generator that
invoked the hook, namespaced with the test framework's class name and "Generators".
In my case this would be `Rspec::Generators::ClassyEnumGenerator`.
I could have alternatively used the `:as => ` option to specify
a different class name, but I wanted to use the default.

I needed the behavior of the enum spec generator to basically mimic that
of my enum class generator, the only difference being the spec location
and which template was used. My final Rspec generator class is shown
here:

*lib/generators/rspec/classy_enum_generator.rb*

```ruby
module Rspec
  module Generators
    class ClassyEnumGenerator < Rails::Generators::NamedBase
      desc "Generate a ClassyEnum spec in spec/enums/"

      argument :name, :type => :string, :required => true, :banner => 'EnumName'
      argument :values, :type => :array, :default => [], :banner => 'value1 value2 value3 etc...'

      source_root File.expand_path("../templates", __FILE__)

      def copy_files # :nodoc:
        empty_directory 'spec/enums'
        template "enum_spec.rb", "spec/enums/#{file_name}_spec.rb"
      end
    end
  end
end
```

And the spec template file:

*lib/generators/rspec/templates/enum_spec.rb*

```erb
require 'spec_helper'
<% values.each do |arg| %>
describe <%= "#{class_name}::#{arg.camelize}" %> do
  pending "add some examples to (or delete) #{__FILE__}"
end
<%- end -%>
```

After adding these two files to ClassyEnum, I reinstalled and
reran the generator in my project and everything worked!

Here is my "passing" test:

```irb
rails g classy_enum Priority low medium high
      create  app/enums
      create  app/enums/priority.rb
      invoke  rspec
      create    spec/enums
      create    spec/enums/priority_spec.rb
```

Which generates the following spec file:

*spec/enums/priority_spec.rb*

```ruby
require 'spec_helper'

describe Priority::Low do
  pending "add some examples to (or delete) #{__FILE__}"
end

describe Priority::Medium do
  pending "add some examples to (or delete) #{__FILE__}"
end

describe Priority::High do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

Since I am using the hook_for method, it includes some other behavior,
such as supporting the `--skip-test-framework` flag out of the box. This
would allow someone to generate the enum classes without any specs, but
I don't recommend doing that. :)

## Wrapping Up

I'm happy with ClassyEnum generator once again. It creates spec files by
default which makes me feel like I'm able to practice what I preach. I
was able to achieve all four of my goals without any crazy hacks or making
any sacrifices. I was also able to easily add support for TestUnit by
adding an additional generator and template file.

