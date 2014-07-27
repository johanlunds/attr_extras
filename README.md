[![Build status](https://secure.travis-ci.org/barsoom/attr_extras.png)](https://travis-ci.org/#!/barsoom/attr_extras/builds)

# attr\_extras

Takes some boilerplate out of Ruby, lowering the barrier to extracting small focused classes, without [the downsides of using `Struct`](http://thepugautomatic.com/2013/08/struct-inheritance-is-overused/).

Instead of

```
class InvoiceBuilder
  def initialize(invoice, employee)
    @invoice, @employee = invoice, employee
  end

  private

  attr_reader :invoice, :employee
end
```

you can just do

```
class InvoiceBuilder
  pattr_initialize :invoice, :employee
end
```

This nicely complements Ruby's built-in `attr_accessor`, `attr_reader` and `attr_writer`.

Supports positional arguments as well as optional and required hash arguments.

Also provides conveniences for creating value objects, method objects, query methods and abstract methods.


## Usage


### `attr_initializer :foo, :bar`. Alternative: `attr_initialize_method`. Rationale for change: I'm not very opposed to the current name, but I think this is more clear. The previous one sounds like it will take some action when you call the metod ("what does it initialize the value to?" I have myself asking). This one is more obvious that it does something regarding the initializer of the class (or constructor if you will, but that's a name Ruby doesn't use).

Defines an initializer that takes two arguments and assigns `@foo` and `@bar`.

`attr_initialize :foo, [:bar, :baz!]` defines an initializer that takes one regular argument, assigning `@foo`, and one hash argument, assigning `@bar` (optional) and `@baz` (required).

`attr_initialize [:bar, :baz!]` defines an initializer that takes one hash argument, assigning `@bar` (optional) and `@baz` (required).


### `attr_private_reader :foo, :bar`. other suggestions: `pattr_reader :foo, :bar` (and more methods can be added if of any use: `pattr_accessors`, `pattr_writer`). Rationale for change: It's very unclear from the current method name `attr_private` that it only declares a writer. I wouldn't have guessed that...

Defines private readers for `@foo` and `@bar`.


### `attr_value_object :foo, :bar` (other suggestions: `attr_as_value_object`, `attr_for_value_object`). Rationale for change: Good to have full name of design pattern in name.

Defines public readers. Does not define writers, as [value objects](http://en.wikipedia.org/wiki/Value_object) are typically immutable.

Defines object equality: two value objects of the same class with the same values are equal.


### `pattr_initialize :foo, :bar` (I'm unsure about the current naming. It's very unclear what it does the first time you see  it. However I don't have a single alternative I have decided on - instead I have several ideas. The downside is that they're longer. More verbose: `attr_initialize_method_and_private_reader` which is not that good. Other ideas that are shorter: `pattr_initializer_and_reader`, `attr_initializer_and_private_reader`. I like those names because they're more like Lego - they build upon the previous method names in the gem and you tuck on an `and_foo` to the declaration in your class as your needs change)

Defines both initializer and private readers: shortcut for

```
attr_initialize :foo, :bar
attr_private :foo, :bar
```

The `attr_initialize` notation for hash arguments is also supported: `pattr_initialize :foo, [:bar, :baz!]`


### `vattr_initialize :foo, :bar`. (alternative namings: `attr_for_value_object_and_initializer`, `attr_value_object_and_initializer`, `attr_initializer_and_value_object`. Also see the previous method's comments and ideas. A "p" is good for "private", a "v" for value object is a bit more streching it.)

Defines initializer, public readers and value object identity: shortcut for

```
attr_initialize :foo, :bar
attr_value :foo, :bar
```

The `attr_initialize` notation for hash arguments is also supported: `vattr_initialize :foo, [:bar, :baz!]`


### `method_object :fooable?, :foo`. Good. (Mnemonic rule: argument order is same as order of "method" and "object" in method-name)

Defines a `.fooable?` class method that takes arguments (`foo`) and delegates to an instance method that can access those arguments as private readers.

This is useful for [method objects](http://refactoring.com/catalog/replaceMethodWithMethodObject.html):

``` ruby
class PriceCalculator
  method_object :calculate,
    :order

  def calculate
    total * factor
  end

  private

  def total
    order.items.map(&:price).inject(:+)
  end

  def factor
    1 + rand
  end
end

class Order
  def price
    PriceCalculator.calculate(self)
  end

  # …
end
```

Shortcut for

``` ruby
attr_initialize :foo
attr_private :foo

def self.fooable?(foo)
  new(foo).fooable?
end
```

The `attr_initialize` notation for hash arguments is also supported: `method_object :fooable?, :foo, [:bar, :baz!]`

You don't have to specify readers if you don't want them: `method_object :fooable?` is also valid.


### `attr_id_query :foo?, :bar?`<br>. Good!

Defines query methods like `foo?`, which is true if (and only if) `foo_id` is truthy. Goes well with Active Record.


### `attr_query :foo?, :bar?`<br>. Good!

Defines query methods like `foo?`, which is true if (and only if) `foo` is truthy.

### `attr_implement :foo, :bar`. Should be named `abstract_method` and `abstract_methods` - has nothing to do with attrs (?).

Defines nullary (0-argument) methods `foo` and `bar` that raise e.g. `"Implement a 'foo()' method"`.

`attr_implement :foo, [:name, :age]` will define a binary (2-argument) method `foo` that raises `"Implement a 'foo(name, age)' method"`.

This is suitable for [abstract methods](http://en.wikipedia.org/wiki/Abstract_method#Abstract_methods) in base classes, e.g. when using the [template method pattern](http://en.wikipedia.org/wiki/Template_method_pattern).


## Philosophy

Findability is a core value.
Hence the long name `attr_initialize`, so you see it when scanning for the initializer;
and the enforced questionmarks with `attr_id_query :foo?`, so you can search for that method.


## Why not use `Struct`?

See: ["Struct inheritance is overused"](http://thepugautomatic.com/2013/08/struct-inheritance-is-overused/)


## Why not use `private; attr_reader :foo`?

Instead of `attr_private :foo`, you could do `private; attr_reader :foo`.

Other than being more to type, declaring `attr_reader` after `private` will actually give you a warning (deserved or not) if you run Ruby with warnings turned on.

If you don't want the dependency on `attr_extras`, you can get rid of the warnings with `attr_reader :foo; private :foo`. Or just define a regular private method.


## Installation

Add this line to your application's `Gemfile`:

    gem "attr_extras"

And then execute:

    bundle

Or install it yourself as:

    gem install attr_extras


## Running the tests

Run then with:

`rake`

Or to see warnings (try not to have any):

`RUBYOPT=-w rake`


## Contributors

* [Henrik Nyh](https://github.com/henrik)
* [Joakim Kolsjö](https://github.com/joakimk)
* [Victor Arias](https://github.com/victorarias)
* [Teo Ljungberg](https://github.com/teoljungberg)
* [Kim Persson](https://github.com/lavinia)


## License

Copyright (c) 2012-2014 [Barsoom AB](http://barsoom.se)

MIT License

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
