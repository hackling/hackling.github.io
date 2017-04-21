---
layout: post
title:  "Dry Structs and Types are awkward"
date:   2017-04-21 12:57:00 +1000
categories: dry-rb dry-struct dry-types ruby
---
I had to learn Dry::Struct and Dry::Types the other day to work on a project.
Type checking, seems pretty cool. Something about the code felt wrong, so I checked out the docs.
In particular, I checked out this page: http://dry-rb.org/gems/dry-struct/

Here's the example that I looked at.
{% highlight ruby %}
require 'dry-struct'

module Types
  include Dry::Types.module
end

class User < Dry::Struct
  attribute :name, Types::String.optional
  attribute :age, Types::Coercible::Int
end

user = User.new(name: nil, age: '21')

user.name # nil
user.age # 21

user = User.new(name: 'Jane', age: '21')

user.name # => "Jane"
user.age # => 21
{% endhighlight %}

Ok, neat. It seems like you can have optional types, and do some cool coericion stuff. However, I was finding in our usage of the code, that we were violating the types that we were setting. I could put integers in string fields. I could put nils, in things that weren't optional. I pondered through the docs some more, but I was "missing something", so I wrote some code (with some help) to try and debunk my thoughts.
What I didn't understand was the difference was between `Types::String`, `Types::Strict::String`, and the optional flags on each.

Here's the code:
{% highlight ruby %}
require 'dry-struct'

module Types
  include Dry::Types.module
end

class TypeString < Dry::Struct
  attribute :value, Types::String
end

class StrictString < Dry::Struct
  attribute :value, Types::Strict::String
end

class OptionalStrictString < Dry::Struct
  attribute :value, Types::Strict::String.optional
end

[
  TypeString,
  StrictString,
  OptionalStrictString,
].each do |struct|
  [nil, 1, "string"].each do |value|
    begin
      struct.new(value: value)
    rescue Exception => msg
      puts msg.inspect
    end
  end
end
{% endhighlight %}

The idea of `Types:String.optional` didn't really make sense to me. I assumed that the optional part was implied because it wasn't Strict.
I was also expecting all of them to throw an error when trying to take `1` as an attribute.

Boy, was I wrong.

The output:
{% highlight shell %}
#<Dry::Struct::Error: [StrictString.new] nil (NilClass) has invalid type for :value>
#<Dry::Struct::Error: [StrictString.new] 1 (Fixnum) has invalid type for :value>
#<Dry::Struct::Error: [OptionalStrictString.new] 1 (Fixnum) has invalid type for :value>
{% endhighlight %}

Apparently `Types::String` will take any value. In fact, this is true of ANY of the Types when you haven't made it `Strict`
or `Coercible`.

I managed to get in touch with one of the core contributes _(yay! Open Source)_ to the Dry project, and apparently `Types` without `Strict` or `Coercible` is more like metadata and doesn't actually provide any validation at all!
You pretty much always want to be using `Strict` or `Coercible` to ensure you are getting the right values, or that the library is turning the received values into the forms you're expecting.

The biggest takeway for me was, don't always believe the docs and examples. Sometimes you've got to get your hands dirty to figure things out.
