---
layout: post
title: "A Simplified Query Interface for Relationships in Active Record 4"
date: 2013-03-10 14:42
comments: true
categories: rails active-record
---

```ruby
# Rails 4 now lets you simplify "belongs_to" association queries:

# Before
Post.where(author_id: @author)
Image.where(imageable_id: @cat, imageable_type: 'Cat')

# After
Post.where(author: @author)
Image.where(imageable: @cat)
```

About six months ago I added a [new feature to Active Record](https://github.com/rails/rails/pull/7273)
that allows you to write simpler queries across associated models. Compared to many of
the other [new features in Rails 4](http://edgeguides.rubyonrails.org/4_0_release_notes.html),
this isn't a significant change, however I think it's pretty
handy so I wanted to talk about it and show some examples. Other than a
[few small bug fixes](http://contributors.rubyonrails.org/contributors/pete-brown/commits),
this was my first real contribution to Rails, and the experience was enlightening
(saving that for another post).

## It's All About the Interface

Prior to this change, any queries that used a foreign key column were required
to specify the actual column name as the hash key in the query. The biggest
problem I had with this approach is that it wasn't consistent with other
Active Record APIs. When I'm working with models, I tend to be thinking in
terms of relationships and objects as opposed to database columns.

Take the following example from Rails 3.x where you can build new
objects using the `belongs_to` relationship, but you cannot query with this
relationship. When you are querying, you need to shift into an object + database
hybrid mindset:

```ruby
class Post < ActiveRecord::Base
  belongs_to :author
end

# Creating related objects using the model association:
Post.new(author: Author.first)

# Does NOT work in Rails 3.x
Post.where(author: Author.first) # => NOPE!

# Must specify foreign key to make query work in Rails 3.x
Post.where(author_id: Author.first) # => Yup!
```

In the above example, I am specifying the foreign key on one side, and the object on the other.
I think it can be hard to remember when you can use the associations and when
you can't. Obviously a more practical approach would be to use `Author.first.posts`,
but this gives flexibility in cases where you might not have both sides of the
relationship fully setup.

The above example may seem trivial so here's an example using a
non-conventional relationship:

```ruby
class Post < ActiveRecord::Base
  belongs_to :writer, class_name: 'Author', foreign_key: 'author_id'
end

# Ah crap... I forgot what my foreign key was called!
Post.where(writer_id: Author.first) # => NOPE!

# Must still specify foreign key column here
Post.where(author_id: Author.first) # => Yup!
```

This issue becomes even more apparent when working with polymorphic relationships:

```ruby
class Cat < ActiveRecord::Base
  has_many :images, as: :imageable, dependent: :destroy
end

class Image < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
end

Image.where(imageable_id: Cat.first, imageable_type: 'Cat')
```

## Striving for Consistency

After seeing a few [different](https://github.com/rails/rails/issues/1736)
[issues](https://github.com/rails/rails/issues/5067) get opened, I realized
I wasn't the only one who felt this inconsistency was unintuitive. It was
causing enough confusion that people were reporting it as a bug,
convinced that it "used to work".  This clearly wasn't a bug, and at some
point an unintuitive interface needs to be addressed.

I hadn't spend a ton of time in the Active Record internals so I decided
to dive in and see if I could change the API so it worked with the relationship
name. I knew that each Active Record model tracks its relationships with other
models using "reflections". Each reflection stores various properies such as the
relationship name and macro (ie belongs_to, has_many, etc).

```irb
1.9.3-p327 :007 > Author.reflections
 => {:posts=>#<ActiveRecord::Reflection::AssociationReflection:0x007f8cf391dc28 @macro=:has_many, @name=:posts, @options={:extend=>[]}, @active_record=Author(id: integer, name: string, created_at: datetime, updated_at: datetime), @plural_name="posts", @collection=true>}
1.9.3-p327 :008 > Post.reflections
 => {:author=>#<ActiveRecord::Reflection::AssociationReflection:0x007f8cf4c1afb0 @macro=:belongs_to, @name=:author, @options={}, @active_record=Post(id: integer, body: text, author_id: integer, created_at: datetime, updated_at: datetime), @plural_name="authors", @collection=false>}
```

As I dove further into the Active Record internals, I found the
[ActiveRecord::PredicateBuilder](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/relation/predicate_builder.rb)
which is used for building the "WHERE" clause of every Active Record
query. It was not the most straightforward class I've worked with, but
the integration tests were good so I could do some exploratory testing
and know when I had broken something. After a few days of discussing
with the Rails core team about what behavior should be implemented, I
had some [working code](https://github.com/rails/rails/commit/3da275c4396d7fad250d2b786027ba4f14344bd4).

## Conclusion

In the end, the changes I made to the predicate builder allow you to query across a
belong_to relationship without specifying the foreign key. Does it
enable you to do something you couldn't do before? Not really, however, bringing
more consistency to the API was my main goal.

Now that [Rails 4 beta has been released](http://weblog.rubyonrails.org/2013/2/25/Rails-4-0-beta1/)
I encourage people to download it today and try out some simple examples:

```ruby
Post.where(author: @author)
Image.where(imageable: @cat) # => Polymorphic!
```

I also recommend checking out the [Active Record tests](https://github.com/rails/rails/blob/master/activerecord/test/cases/relation/where_test.rb#L26-L81)
for some more complex examples of how it can be used with polymorphic
relationships and single-table inheritance.
