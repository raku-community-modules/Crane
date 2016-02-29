# Crane

<!-- intro {{{ -->

Navigate Perl6 [containers](http://doc.perl6.org/language/containers)
and perform tasks.

<!-- end intro }}} -->


<!-- synopsis {{{ -->

## Synopsis

<!-- example code {{{ -->

```perl6
use Crane;

my %inxi = :info({
    :memory([1564.9, 32140.1]),
    :processes(244),
    :uptime<3:16>
});

at(%inxi, 'info')<uptime>:delete;
at(%inxi, qw<info memory>)[0] = 31868.0;

say %inxi.perl; # :info({ :memory(31868.0, 32140.1), :processes(244) })
```

<!-- end example code }}} -->

-------------------------------------------------------------------------------

<!-- end synopsis }}} -->


<!-- description {{{ -->

## Description

Crane aims to be for Perl6 containers what [JSON
Pointer](http://tools.ietf.org/html/rfc6901) and [JSON
Patch](http://tools.ietf.org/html/rfc6902) are for JSON.

### Features

- access nested data structures with JSON Pointer inspired path notation
- get / set values in nested data structures
- diff / patch nested data structures
- list the contents of a nested data structure in accessible format

-------------------------------------------------------------------------------

<!-- end description }}} -->


<!-- exported subroutines {{{ -->

## Exported Subroutines

<!-- exported subroutines toc {{{ -->

- [`at($container,*@path)`](#atcontainerpath)
- [`chisel($container,*@path)`](#chiselcontainerpath)

<!-- end exported subroutines toc }}} -->

<!-- at($container,*@path) {{{ -->

### `at($container,*@path)`

Navigates to and returns container `is rw`.

_arguments:_

* `$container`: _Container, required_ — the target container
* `*@path`: _Path, optional_ — a list of steps for navigating container

_returns:_

* Container at path (`is rw`)

_example:_

```perl6
my %inxi = :info({
    :memory([1564.9, 32140.1]),
    :processes(244),
    :uptime<3:16>
});

at(%inxi, 'info')<uptime>:delete;
at(%inxi, qw<info memory>)[0] = 31868.0;

say %inxi.perl; # :info({ :memory(31868.0, 32140.1), :processes(244) })
```

<!-- end at($container,*@path) }}} -->

<!-- chisel($container,*@path) {{{ -->

### `chisel($container,*@path)`

Navigates to and returns container `is rw`, creating keys and overwriting
values along the way (recursively).

`chisel` makes changes to containers potentially many levels deep,
and will change the structure of containers based on `@path` input.

If the given `@path` calls for overwriting values in `$container`,
those values will be overwritten even when the rest of the operation
fails and even when not assigning values to the returned container.

_arguments:_

* `$container`: _Container, required_ — the target container
* `*@path`: _Path, optional_ — a list of steps for navigating container

_returns:_

* Container at path (`is rw`)

_example:_

```perl6
my %archversion = :bamboo({ :up<0.0.1>, :aur<0.0.2> });
chisel(%archversion, qw<fzf up>) = '0.11.3';
chisel(%archversion, qw<fzf aur>) = '0.11.3';
say %archversion.perl;
{
    :bamboo({ :aur<0.0.2>, :up<0.0.1> }),
    :fzf({ :aur<0.11.3>, :up<0.11.3> })
}
```

<!-- end chisel($container,*@path) }}} -->

-------------------------------------------------------------------------------

<!-- end exported subroutines }}} -->


<!-- methods {{{ -->

## Methods

<!-- methods toc {{{ -->

- [`.exists($container,:@path!,:$k,:$v)`](#existscontainerpathkv)
- [`.get($container,:@path!,:$k,:$v,:$p)`](#getcontainerpathkvp)
- [`.set($container,:@path!,:$value!,:$in-place)`](#setcontainerpathvaluein-place)
- [`.add($container,:@path!,:$value!,:$in-place)`](#addcontainerpathvaluein-place)
- [`.remove($container,:@path!,:$in-place)`](#removecontainerpathin-place)
- [`.replace($container,:@path!,:$value!,:$in-place)`](#replacecontainerpathvaluein-place)
- [`.move($container,:@from!,:@path!,:$in-place)`](#movecontainerfrompathin-place)
- [`.copy($container,:@from!,:@path!,:$in-place)`](#copycontainerfrompathin-place)
- [`.test($container,:@path!,:$value!)`](#testcontainerpathvalue)
- [`.list($container,:@path!)`](#listcontainerpath)
- [`.flatten($container,:@path!)`](#flattencontainerpath)
- [`.transform($container,:@path!,:$block!,:$in-place)`](#transformcontainerpathblockin-place)

<!-- end methods toc }}} -->

<!-- example data structure {{{ -->

All example code assumes `%data` has this structure:

```perl6
my %data =
    :legumes([
        {
            :instock(4),
            :name("pinto beans"),
            :unit("lbs")
        },
        {
            :instock(21),
            :name("lima beans"),
            :unit("lbs")
        },
        {
            :instock(13),
            :name("black eyed peas"),
            :unit("lbs")
        },
        {
            :instock(8),
            :name("split peas"),
            :unit("lbs")
        }
    ]);
```

<!-- end example data structure }}} -->

<!-- .exists($container,:@path!,:$k,:$v) {{{ -->

### `.exists($container,:@path!,:$k,:$v)`

Determines whether a key exists in the container at the
specified path. Works similar to the p6 Hash `:exists` [subscript
adverb](http://doc.perl6.org/type/Hash#%3Aexists). Pass the `:v` flag to
determine whether a defined value is paired to the key at the specified
path.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$k`: _Bool, optional, defaults to True_ — indicates whether to
         check for an existing key at path
* `:$v`: _Bool, optional_ — indicates whether to check for a defined
         value at path

_returns:_

* `True` if exists, otherwise `False`

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.
Passing `:v` flag tests `if $container.defined`. Default behavior is to
raise an error as key operations aren't permitted on root containers.

<!-- end .exists($container,:@path!,:$k,:$v) }}} -->

<!-- .get($container,:@path!,:$k,:$v,:$p) {{{ -->

### `.get($container,:@path!,:$k,:$v,:$p)`

Gets the value from container at the specified path. The default
behavior is to raise an error if path is nonexistent.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$k`: _Bool, optional_ — only return the key at path
* `:$v`: _Bool, optional, defaults to True_ — only return the value
         at path
* `:$p`: _Bool, optional_ — return the key-value pair at path

_returns:_

* the dereferenced key, value or key-value pair

_example:_

```perl6
my $value = Crane.get(%data, :path('legumes', 1));
say $value.perl; # { :instock(21), :name("lima beans"), :unit("lbs") }

my $value-k = Crane.get(%data, :path('legumes', 1), :k);
say $value-k.perl; # 1

my $value-p = Crane.get(%data, :path('legumes', 1), :p);
say $value-p.perl; # 1 => { :instock(21), :name("lima beans"), :unit("lbs") }
```

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.

- if `:v` flag passed (the default): `return $container`
- if `:k` flag passed: raise error "Sorry, not possible to request key
                       operations on the container root"
- if `:p` flag passed: raise error "Sorry, not possible to request key
                       operations on the container root"

<!-- end .get($container,:@path!,:$k,:$v,:$p) }}} -->

<!-- .set($container,:@path!,:$value!,:$in-place) {{{ -->

### `.set($container,:@path!,:$value!,:$in-place)`

Sets the value at the specified path in the container. The default
behavior is to create nonexistent paths (similar to `mkdir -p`) and to
overwrite existing values where necessary.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$value!`: _Any, required_ — the value to be set at the specified path
* `:$in-place`: _Bool, optional, defaults to False_ — whether to modify
                `$container` in-place

_returns:_

* Container (original container is unmodified unless `:in-place` flag
  is passed)

_example:_

```perl6
my %p0;
my %p1 = Crane.set(%p0, :path(qw<peter piper>), :value<man>);
my %p2 = Crane.set(%p1, :path(qw<peter pan>), :value<boy>);
my %p3 = Crane.set(%p2, :path(qw<peter pickle>), :value<dunno>);

say %p0.perl; # {}
say %p1.perl; # { :peter({ :piper("man") }) }
say %p2.perl; # { :peter({ :pan("boy"), :piper("man") }) }
say %p3.perl; # { :peter({ :pan("boy"), :pickle("dunno"), :piper("man") }) }
```

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.

- if value assignment (`=`) to `$container` fails, raise error and
  propogate the original error message
  - value assignment will fail when assigning a List to a `$container`
    of type Hash and vice versa

```perl6
my $a = (1, 2, 3);
my $b = Crane.set($a, :path(), :value<foo>);
say $a; # (1, 2, 3)
say $b; # foo
```

<!-- end .set($container,:@path!,:$value!,:$in-place) }}} -->

<!-- .add($container,:@path!,:$value!,:$in-place) {{{ -->

### `.add($container,:@path!,:$value!,:$in-place)`

Adds a value to the container. If `:@path` points to an existing item
in the container, that item's value is replaced.

In the case of a `Positional` type, the value is inserted before the
given index. Use the relative accessor (`*-0`) instead of an index
(`Int`) for appending to the end of a `Positional`.

Because this operation is designed to add to existing `Associative` types,
its target location will often not exist. However, an `Associative`
type or a `Positional` type containing it does need to exist, and it
remains an error for that not to be the case. For example, a `.add`
operation with a target location of `<a b>` starting with this Hash:

```perl6
{ :a({ :foo(1) }) }
```

is not an error, because "a" exists, and "b" will be added to its
value. It is an error in this Hash:

```perl6
{ :q({ :bar(2) }) }
```

because "a" does not exist.

Think of the `.add` operation as behaving similarly to `mkdir`, not
`mkdir -p`. For example, you cannot do (in shell):

```
$ ls # empty directory
$ mkdir a/b/c
mkdir: cannot create directory ‘a/b/c’: No such file or directory
```

Without the `-p` flag, you'd have to do:

```
$ ls # empty directory
$ mkdir a
$ mkdir a/b
$ mkdir a/b/c
```

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$value!`: _Any, required_ — the value to be added/inserted at the
              specified path
* `:$in-place`: _Bool, optional, defaults to False_ — whether to modify
                `$container` in-place

_returns:_

* Container (original container is unmodified unless `:in-place` flag
  is passed)

_example:_

```perl6
my %legume = :name<carrots>, :unit<lbs>, :instock(3);
my %data-new = Crane.add(%data, :path('legumes', 0), :value(%legume));
```

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.

- if value assignment (`=`) to `$container` fails, raise error and
  propogate the original error message
  - value assignment will fail when assigning a List to a `$container`
    of type Hash and vice versa

```perl6
my @a;
my @b = Crane.add(@a, :path([]), :value<foo>);
say @a.perl; # []
say @b.perl; # ["foo"]
```

<!-- end .add($container,:@path!,:$value!,:$in-place) }}} -->

<!-- .remove($container,:@path!,:$in-place) {{{ -->

### `.remove($container,:@path!,:$in-place)`

Removes the pair at path from `Associative`
types, similar to the p6 Hash `:delete` [subscript
adverb](http://doc.perl6.org/type/Hash#%3Adelete). Splices elements out
from `Positional` types.

The default behavior is to raise an error if the target location is
nonexistent.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$in-place`: _Bool, optional, defaults to False_ — whether to modify
                `$container` in-place

_returns:_

* Container (original container is unmodified unless `:in-place` flag
  is passed)

_example:_

```perl6
my %h = :example<hello>;
my %h2 = Crane.remove(%h, :path(['example']));
say %h.perl; # { :example<hello> }
say %h2.perl; # {}
```

This:

```perl6
%h<a><b>:delete;
```

is equivalent to this:

```perl6
Crane.remove(%h, :path(qw<a b>));
```

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.

```perl6
my $a = [1, 2, 3];
my $b = Crane.remove($a, :path([])); # equivalent to `$a = Empty`
say $a.perl; [1, 2, 3]
say $b; # (Any)
```

<!-- end .remove($container,:@path!,:$in-place) }}} -->

<!-- .replace($container,:@path!,:$value!,:$in-place) {{{ -->

### `.replace($container,:@path!,:$value!,:$in-place)`

Replaces a value. This operation is functionally identical to a `.remove`
operation for a value, followed immediately by a `.add` operation at
the same location with the replacement value.

The default behavior is to raise an error if the target location is
nonexistent.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$value!`: _Any, required_ — the value to be set at the specified path
* `:$in-place`: _Bool, optional, defaults to False_ — whether to modify
                `$container` in-place

_returns:_

* Container (original container is unmodified unless `:in-place` flag
  is passed)

_example:_

```perl6
my %legume = :name("green beans"), :unit<lbs>, :instock(3);
my %data-new = Crane.replace(%data, :path('legumes', 0), :value(%legume));
```

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.

- if value assignment (`=`) to `$container` fails, raise error and
  propogate the original error message
  - value assignment will fail when assigning a List to a `$container`
    of type Hash and vice versa

```perl6
my %a = :a<aaa>, :b<bbb>, :c<ccc>;
my %b = Crane.replace(%a, :path([]), :value({ :vm<moar> }));
say %a.perl; # { :a<aaa>, :b<bbb>, :c<ccc> }
say %b.perl; # { :vm<moar> }
```

<!-- end .replace($container,:@path!,:$value!,:$in-place) }}} -->

<!-- .move($container,:@from!,:@path!,:$in-place) {{{ -->

### `.move($container,:@from!,:@path!,:$in-place)`

Moves the source value identified by `@from` in container to destination
location specified by `@path`. This operation is functionally identical
to a `.remove` operation on the `@from` location, followed immediately
by a `.add` operation at the `@path` location with the value that was
just removed.

The default behavior is to raise an error if the source is nonexistent.

The default behavior is to raise an error if the `@from` location is a
proper prefix of the `@path` location; i.e., a location cannot be moved
into one of its children.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@from!`: _Path, required_ — a list of steps to the source
* `:@path!`: _Path, required_ — a list of steps to the destination
* `:$in-place`: _Bool, optional, defaults to False_ — whether to modify
                `$container` in-place

_returns:_

* Container (original container is unmodified unless `:in-place` flag
  is passed)

_What about operating on the root of the container?_

Pass an empty list as `@from` or `@path` to operate on the root of
the container.

- if value assignment (`=`) to `$container` fails, abort move operation,
  raise error and propogate the original error message
  - value assignment will fail when assigning a List to a `$container`
    of type Hash and vice versa

<!-- end .move($container,:@from!,:@path!,:$in-place) }}} -->

<!-- .copy($container,:@from!,:@path!,:$in-place) {{{ -->

### `.copy($container,:@from!,:@path!,:$in-place)`

Copies the source value identified by `@from` in container to destination
container at location specified by `@path`. This operation is functionally
identical to a `.add` operation at the `@path` location using the value
specified in the `@from`.

The default behavior is to raise an error if the source at `@from`
is nonexistent.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@from!`: _Path, required_ — a list of steps to the source
* `:@path!`: _Path, required_ — a list of steps to the destination
* `:$in-place`: _Bool, optional, defaults to False_ — whether to modify
                `$container` in-place

_returns:_

* Container (original container is unmodified unless `:in-place` flag
  is passed)

_example:_

```perl6
my %h = :example<hello>;
my %h2 = Crane.copy(%h, :from(['example']), :path(['sample']));
say %h.perl; # { :example("hello") }
say %h2.perl; # { :example("hello"), :sample("hello") }
```

_What about operating on the root of the container?_

Pass an empty list as `@from` or `@path` to operate on the root of the
container. Has similar rules / considerations to `.move`.

<!-- end .copy($container,:@from!,:@path!,:$in-place) }}} -->

<!-- .test($container,:@path!,:$value!) {{{ -->

### `.test($container,:@path!,:$value!)`

Tests that the specified value is set at the target location in the
document.

_arguments:_

* `$container`: _Container, required_ — the target container
* `:@path!`: _Path, required_ — a list of steps for navigating container
* `:$value!`: _Any, required_ — the value expected at the specified path

_returns:_

* `True` if expected value exists at `@path`, otherwise `False`

_example:_

```perl6
say so Crane.test(%data, :path('legumes', 0, 'name'), :value("pinto beans")); # True
```

_What about operating on the root of the container?_

Pass an empty list as `@path` to operate on the root of the container.

<!-- end .test($container,:@path!,:$value!) }}} -->

<!-- .list($container,:@path) {{{ -->

### `.list($container,:@path)`

Lists all of the paths available in `$container`.

_arguments:_

* `$container` : _Container, required_ — the target container
* `:@path`: _Path, optional_ — a list of steps for navigating container

_returns:_

* List of path-value pairs

_example:_

Listing a `Hash`:

```perl6
say Crane.list(%data);
(
    {
        :path(["legumes", 0, "instock"]),
        :value(4)
    },
    {
        :path(["legumes", 0, "name"]),
        :value("pinto beans")
    },
    {
        :path(["legumes", 0, "unit"]),
        :value("lbs")
    },
    {
        :path(["legumes", 1, "instock"]),
        :value(21)
    },
    {
        :path(["legumes", 1, "name"]),
        :value("lima beans")
    },
    {
        :path(["legumes", 1, "unit"]),
        :value("lbs")
    },
    {
        :path(["legumes", 2, "instock"]),
        :value(13)
    },
    {
        :path(["legumes", 2, "name"]),
        :value("black eyed peas")
    },
    {
        :path(["legumes", 2, "unit"]),
        :value("lbs")
    },
    {
        :path(["legumes", 3, "instock"]),
        :value(8)
    },
    {
        :path(["legumes", 3, "name"]),
        :value("split peas")
    },
    {
        :path(["legumes", 3, "unit"]),
        :value("lbs")
    }
)
```

Listing a List:

```perl6
my $a = qw<zero one two>;
say Crane.list($a);
(
    {
        :path([0]),
        :value("zero")
    },
    {
        :path([1]),
        :value("one")
    },
    {
        :path([2]),
        :value("two")
    }
)
```

<!-- end .list($container,:@path) }}} -->

<!-- .flatten($container,:@path) {{{ -->

### `.flatten($container,:@path)`

Flattens a container into a single-level `Hash` of path-value pairs.

_arguments:_

* `$container` : _Container, required_ — the target container
* `:@path`: _Path, optional_ — a list of steps for navigating container

_returns:_

* a flattened `Hash` of path-value pairs.

_example:_

```perl6
say Crane.flatten(%data);
{
    # ...
    ['legumes', 1, 'name']    => "lima beans",
    ['legumes', 1, 'unit']    => "lbs",
    ['legumes', 1, 'instock'] => 21,
    ['legumes', 2, 'name']    => "black eyed peas",
    ['legumes', 2, 'unit']    => "lbs",
    ['legumes', 2, 'instock'] => 13,
    ['legumes', 3, 'name']    => "split peas",
    ['legumes', 3, 'unit']    => "lbs",
    ['legumes', 3, 'instock'] => 8
}
```

<!-- end .flatten($container,:@path) }}} -->

<!-- .transform($container,:@path!,:$block!,:$in-place) {{{ -->

### `.transform($container,:@path!,:$block!,:$in-place)`

_example:_

```perl6
my %market =
    :foods({
        :fruits(qw<blueberries marionberries>),
        :veggies(qw<collards onions>)
    });

my @first-fruit = |qw<foods fruits>, 0;
my @second-veggie = |qw<foods veggies>, 1;

my &oh-yeah = -> $s { $s ~ '!' };

Crane.transform(%market, :path(@first-fruit), :block(&oh-yeah));
say so Crane.get(%market, :path(@first-fruit)) eq 'blueberries!'; # True

Crane.transform(%market, :path(@second-veggie), :block(&oh-yeah));
say so Crane.get(%market, :path(@second-veggie)) eq 'onions!'; # True
```

<!-- end .transform($container,:@path!,:$block!,:$in-place) }}} -->

-------------------------------------------------------------------------------

<!-- end methods }}} -->


<!-- classes {{{ -->

## Classes

<!-- classes toc {{{ -->

- [`Crane` Class](#crane-class)

<!-- end classes toc }}} -->

<!-- Crane Class {{{ -->

### `Crane` Class

Encapsulates container related operations for a specified path.

<!-- Crane Class attributes {{{ -->

#### attributes:

* `.path` : _Path_ — contains the `Crane`'s list of steps for navigating
            container

<!-- end Crane Class attributes }}} -->

<!-- Crane Class methods {{{ -->

#### methods:

<!-- .new(@path) {{{ -->

##### `.new(@path)`

Instantiates `Crane` class.

_arguments:_

* `@path` : _Path, required_ — a list of steps for navigating container

_returns:_

* a new `Crane` instance

_example:_

```perl6
my $crane = Crane.new('legumes', 0);
```

<!-- end .new(@path) }}} -->

`Crane` classes get access to all methods documented above, with
the stipulation that the path to the source container is the `Crane`
instance's `.path` attribute.

<!-- end Crane Class methods }}} -->

<!-- end Crane Class }}} -->

-------------------------------------------------------------------------------

<!-- end classes }}} -->


<!-- licensing {{{ -->

## Licensing

This is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE file.

<!-- licensing }}} -->

<!-- vim: ft=markdown fdm=marker fdl=0 -->
