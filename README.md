# Optioning

An easy way to retrieve, store, filter, transform and deprecate `options` passed
to a method. Where `options` are the keys our beloved `Hash` as last parameter
for a method call.

## Status
[![Gem Version](https://badge.fury.io/rb/optioning.svg)](http://badge.fury.io/rb/hashing)
[![Build Status](https://travis-ci.org/ricardovaleriano/optioning.svg?branch=master)](http://travis-ci.org/ricardovaleriano/hashing?branch=master)
[![Code Climate](https://codeclimate.com/github/ricardovaleriano/optioning.png)](https://codeclimate.com/github/ricardovaleriano/hashing)

## Installation

Add this line to your application's Gemfile:

    gem 'optioning'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install optioning

## Contact

* API Doc: http://rdoc.info/gems/optioning
* Bugs, issues and feature requests: https://github.com/ricardovaleriano/optioning/issues
* Support: http://stackoverflow.com/questions/tagged/optioning-ruby

## Usage

Given the following `File` class:

```ruby
class File
  def initialize(path, commit = nil, content = nil)
    @path, @commit, @content = path, commit, content
  end
end
```

And a module called `Hashing`, which defines a method `.serialize` that allow
you configure which `ivars` should be used to convert instances of `File` into a
`Hash` like this:

```ruby
{
  path: @path,
  commit: @commit,
  content: @content
}
```

And I can configure a transformation in the values of `path`, `commit` and
`content` when transforming it into a `Hash` like so:

```ruby
require 'hashing'

class File
  extend Hashing

  hasherize :path, :commit, to_hash: ->(value) { value.downcase }
end
```

As the implementor of this module and the `.hasherize` method, I want to be able
to use an instance of `Optioning`, so I can store and retrieve the `ivars` and
the `options` passed to be used along those `ivars`:

```ruby
module Hashing
  def hasherize(*ivars_and_options)
    @options = Optioning.new ivars_and_options

    # ...
  end
end
```

Now in the `Optioning` instance, I can call the following (among others)
methods:

```ruby
@options.raw
# => [:path, :commit, {to_hash: #<Proc:0x007fa4120bd318@(irb):42 (lambda)>}]

@options.values
# => [:path, :commit]

@options.on :to_hash
# => #<Proc:0x007fa4120bd318@(irb):42 (lambda)>
```

### Deprecating options

Now, following our example, if you need to deprecat the `:to_hash` option in
favor of the new `:to` option, you could do:

```ruby
def hasherize(*ivars_and_options)
  @options = Optioning.new ivars_and_options
  @options.deprecate :to_hash, :to, Date.new(2015, 05, 01)

  # ...
end
```

This will replace the deprecated option `:to_hash` for the new one named `:to`
so you can do the following invocation to recover the value passed to the
deprecated `option`:

```ruby
@options.on :to

# => #<Proc:0x007fa4120bd318@(irb):42 (lambda)>
```

#### Deprecation warnings

You can alert your user about those deprecations using the `#deprecated_warn`
method:

```ruby
def hasherize(*ivars_and_options)
  @options = Optioning.new ivars_and_options
  @options.deprecate :to_hash, :to
  @options.deprecated_warn

  # ...
end
```

You can inform the date when the deprecation will not be available anymore.
These date will be parte of the deprecation message:

```ruby
@options.deprecate :to_hash, :to, Date.new(2015, 05, 01)
```

Or if you prefer, you can specify a version of your software that pretend to
remove the deprecated thing:

```ruby
@options.deprecate :to_hash, :to, "v2.0.0"
```

### Ignoring unrecongnized options

If you need, you could fitler the options to mantain just the recognized ones
available. To configure the options that matters to your program, use the method
`#recognize`. And to warn the user in case a unrecognized option is used, call
the `#unrecognized_warn` method:

```ruby
def hasherize(*ivars_and_options)
  @options = Optioning.new ivars_and_options
  @options.recognize :from
  @options.unrecognized_warn

  # ...
end
```

Now, if a user pass an option different than the `:from` one, a warning will
inform that the option is not recognized and will be ignored.

#### Do I Need to register deprecated options as recognized?

Fortunately no. You just need to register your deprecations as usual:

```ruby
def hasherize(*ivars_and_options)
  @options = Optioning.new ivars_and_options
  @options.recognize :from
  @options.deprecate :to_hash, :to
  @options.deprecated_warn
  @options.unrecognized_warn

  # ...
end
```

The `#deprecate` method already knows what to do (that is register the `option`
`:to_hash` as recognized. To sum up, in this last example, the options `:from`
and `:to` are already recongnized by the `Optioning` instance.

### `#process`

The `#process` method will replace all deprecations, warn about them and warn
about unrecognized options all at once, so you can use it like this:

```ruby
def hasherize(*ivars_and_options)
  @options = Optioning.new ivars_and_options
  @options.recognize :from
  @options.deprecate :to_hash, :to
  @options.process

  # ...
end
```

### Fluent interface

And finally, just for a matter of taste, `#deprecate`, `#recognize` and
`#process` returns the `Optioning` instance itself, so you can write the last
example like this (if you want)

```ruby
def hasherize(*ivars_and_options)
  @options = Optioning.new(ivars_and_options).recognize(:from)
  @options.deprecate(:to_hash, :to).process

  # ...
end
```

## Contributing

1. Fork it ( http://github.com/ricardovaleriano/optioning/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
