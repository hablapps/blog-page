# Q For (Mortal) Functional Programmers

## About this series

This series of posts describes our experience while learning
[*q*](https://code.kx.com/q/learn/startingkdb/language/) and *kdb+* by Kx
Systems. Although q is also a functional programming language, it has many
features that make it different from other conventional languages such as
Haskell, OCaml, etc. Given this situation, we'll try to provide an overview of q
basics, connecting the missing pieces to our previous knowledge on functional
programming, using Scala and Spark to guide the explanations. We hope it also
works in the opposite direction, so q programmers can also benefit from these
articles. We enumerate them right now:

1. Q as an (impure) functional language
2. Q as an array processing language
3. Q as a query language for kdb+

The first post introduces q as a functional language, where we'll show the main
q features that are already familiar to the conventional functional programmer
which could be used from the very first day. For its part, the second post put
focus on q as an array processing language, which is probably the main source of
q weirdness, so we'll try to connect it to existing theory on the functional
paradigm. Finally, the last post introduces q and kdb+ as a query language and a
column-oriented database, respectively, to make the data engineer happy.

### Why q?

Each day, we set aside a time to experiment with new technologies, and we are
especially interested on functional languages. A colleague from the Scala
community pointed us towards q. The following items summarise the alleged
language benefits why we decided to give it a go:

- Q is fast, sooo fast
- Q is a functional query language
- Q is a well-founded language that relies on APL
- Q is highly demanded in the financial industry
- Q is quite a challenge

After a few months of reading q material and coding, we can confirm that none of
the previous items is a myth. Although we feel that we still have a long way to
master q/kdb, we are confident that we have now a good perspective on how hard
it is to learn this language. In fact, reflecting this experience is perhaps the
major contribution of this series of posts. Having said so, we're ready to go
now!

## Q as an (impure) functional language

Q was implemented by [Arthur
Whitney](https://queue.acm.org/detail.cfm?id=1531242) (Kx Systems), having its
first appearence in 2003. It relies on the shoulders of Kenneth E. Iverson and
his (Turing awarded) work on *A Programming Language* (APL) that started more
than six decades ago. Iverson emphasises the [importance of
notation](https://dl.acm.org/doi/pdf/10.1145/358896.358899) to concentrate on
more advanced problems, and finds in programming languages the idoneous setting
to make math notation universal, executable and unambiguous. In accordance with
such spirit, most of the APL primitives are still available on q.

We cannot find a better overview for q as the one contained in the first pages
of [Q Tips](https://www.q-tips.net): *"Q is an interpreted, dynamic,
event-driven, functional, array programming language."*. That long definition
let us infer why learning q becomes such a challenge. Letting the eternal war
between interpreted/compiled and dynamic/static aside (compiled and static
always win, right? :), this article put focus on the *functional* feature, where
we functional programmers can benefit from, since we have walked this road
before.

Although there is an emerging interest on [taking q and kdb beyond financial
services](https://www.efinancialcareers.co.uk/news/2017/05/kdbq-banking-alternatives),
I think it is fair to say that nowadays, most of q-related job positions fall
under this umbrella. Thereby, we find it convenient to use a trading indicator
as an example. We'll try to keep it very easy.

> Most of the q/kdb+ tutorials and code snippets revolve around trading
> examples, so it is sometimes helpful to have an specialist nearby.

Our indicator simply consists on calculating the max price of an instrument (for
instance: AAPL, AMZN, etc.) in the last year. We'll assume that the instrument
prices are organized as a kind of table containing all the working days
(Monday-Friday) within the last year. In turn, each of them contains a price
update for every second within the working hours (in particular, within the
range from 09:00 to 17:30). We split the problem in three steps, which
correspond with the following sections:
1. Calculate the max of two numbers
2. Calculate the max price within a day
3. Calculate the max price within the whole year

The first one is just a first contact with the language, where q operators and
types basics are introduced. The second one serves us as an excuse to show the
date api, lists, lambda expressions and *iterators* (which are essentially
higher order functions over collections). Finally, the last section presents
dictionaries and more iterators, where we briefly show an interesting connection
with monads.

We embrace an innovative approach (when compared with other articles in this
blog) where we show q and scala snippets side by side. We want to remark that
our intention isn't to provide a comparison of these languages, but rather
support our explanations by means of snippets from a more conventional
functional language such as scala, for merely didactic purposes.

### Basic operators and types

The q language allows us to calculate the max of two numbers by means of the
operator `|`. We show how to use it in the following snippet, extracted from a
q REPL session, where `q)` acts as the default prompt:
```q
q)3|2
3
```
Scala also has a REPL as part of its ecosystem, where the default prompt is
`scala> `. We could translate the very same logic into Scala using `Math.max`:
```scala
scala> import Math.max
scala> max(3, 2)
val res0: Int = 3
```
As can be seen, we need to import the `Math` module in order to get such
functionality. This can be extrapolated to many other math operators, which are
loaded by default in q, mainly becauses of its bias towards analytics.

It would not be difficult to supply an infix alias for `Math.max` in Scala
(although we adopt `||` instead of `|`, since the latter is already associated
to the *bitwise or* operator):
```scala
scala> implicit class IntExt(x: Int) {
     |   def ||(y: Int): Int = max(x, y)
     | }

scala> 3 || 2
val res1: Int = 3
```
However, it is perhaps more interesting to follow the opposite path and move
`|` to its prefix notation, using a familiar syntax for Haskellers, where we
place the operator in parentheses:
```q
q)(|)[3;2]
3
```
As can be seen, the arguments are separated by semicolons and surrounded by
square brackets. Have you noticed the lack of space characters among them? Q is
really committed to shortness and invites the programmer to limit them. This
subtle difference has a considerable impact while reading q code, although
eventually, you get used to it.

In the previous snippets, we have just produced an output and we have let the
REPL to show it, but we could have assigned a name to the resulting value. The q
notation to introduce a variable `x` is as follows:
```q
q)x:3|2
q)x
3
```
Which we translate into Scala by means of a `var` instead of a`val`, since q
variables can be reassigned:
```scala
scala> var x: Int = max(3, 2)
scala> x
val res0: Int = 3
```
Despite q embraces immutability for a wide range of situations, it is clearly
not a pure functional language. It neither put the same level of pressure on
avoiding mutable state as the one exhibited by Scala.

Another important aspect from the previous snippets are the variable types. We
can see that the Scala version indicates that the type of `x` is `Int`, although
we could have removed it and let the type inferencer work for us. In the case of
q, which is a dynamic language, we could use the `type` primitive to find it:
```q
q)type x
-7h
```
Where you hoping to find something more familiar? Welcome to q! The number 7
indicates that the value of this type is a long. On its part, the negative
symbol sugggests an atomic value, ie. not a list of longs. The final `h` just
indicates that the value that `type` returns has *short* as type. You can find
the complete relation between numbers and types in [this
section](https://code.kx.com/q/basics/datatypes/) from the official
documentation.

### Dealing with times

Once we know how to get the max value of two numbers, we will extend it in order
to calculate the max price of a given list, which corresponds to our first
instrument. To do so, we will generate a list of random numbers to play with. In
particular, we will generate a different price for each intraday second,
starting at 09:00 and closing at 17:30. How many seconds are there in such
interval?
```q
q)n:7h$17:30:00-09:00:00
30600
```
We clumsily adapt the previous expression into Scala as follows:
```scala
scala> import scala.concurrent.duration._
import scala.concurrent.duration._

scala> val n = (17.hours+30.minutes - 9.hours).toSeconds
val n: Long = 30600
```
There are several aspects to discuss here, but we start emphasising that q
primitives for dates and times are just superb. Creating, operating and casting
different units of times is a clean, elegant and intuitive task. In the example
above, we just create the opening and closing seconds and then we subtract them.
See that `7h$` at the beginning? We use the `$` operator to cast an expression
to another type. As you already know, `7` is associated to the *long* type, so
we are casting the subtraction result to its numeric form. Dates and times in
Scala and the JVM are object of discussion, where we can find a vast field of
non-interoperable libraries that make the hell out of a programmer, so we will
not go into further detail around dates. However, we can see that the Scala
expression includes parenthesis to determine associativity, so we will take the
opportunity to discuss it, along with operator precedence.

### Operator precedence and associativity

At first sight, we could infer that `x$y-z` interprets the subtraction before
the casting due to `-` having a higher precedence. But this is not the case. In
fact, no q operator has higher precedende than another, since it will always
interpret expressions from right to left. For instance, `2*3+1` returns `8`. If
you combine this right-biased interpretation with the fact that it is possible
to introduce variable names at any point of an expression, you can find
yourself spending a non-negligible amount of time trying to understand why the
following expression does return `4`:
```q
q)x:0
q)x*3+2-x:1
4
```
Notice that the second line rewrites `x` at the very beginning, remember, at
the rightmost expression. In particular, the subexpression `x:1` assigns `1` to
`x` and returns it as output, just to proceed with the rest of operations. We
show the Scala analogous to remark this aspect:
```scala
scala> var x = 0
var x: Int = 0

scala> x = 1; x * (3 + (2 - x))
val res23: Int = 4
```
Again, this way of interpreting code is yet another hurdle that makes q
difficult to read for newbies, but eventually, you get used to it. Before moving
on, we want to clarify that q programmers can change associativity by using
parenthesis, for instance: `(2*3)+1`, although it is more idiomatic to avoid
them and reorder the code, if possible.

### Lists and random values

At this point, we know the number of random prices that the intraday list will
contain, which we assigned to the variable `n`, so let's generate them. To do
so, we simply use the `?` operator:
```q
q)prices:n?1000f
231.8545 102.0847 974.3216 673.6161 404.1387 626.0377 211.9141 604.1371 52.77..
```
This code generates `n` float numbers in the range that goes from zero to one
thousand. Now, we show what we have considered its Scala counterpart:
```scala
scala> val prices = List.fill(n)(util.Random.nextFloat).map(_ * 1000)
val prices: Seq[Float] = List(618.7332, 216.10922, 481.55737, 257.13562, 95.020..
```
Once again, the generation of random numbers by means of `?` is quite simple
but impure, since this operation is not referentially transparent. We avoid
introducing seeds in the Scala version to keep the comparison clear. Indeed, we
have not seen references to seeds in our brief experience as q programmers.

A nice feature from both q and Scala is that most of times the output reflects
the very same code that we need to build such value. For instance, we can
generate a list of floats using the very same notation:
```q
q)231.8545 102.0847 974.3216
231.8545 102.0847 974.3216
```
whose associated type is `9h`, meaning a list of floats, and that we adapt to
Scala as follows:
```scala
scala> List(231.8545f, 102.0847f, 974.3216f)
val res10: List[Float] = List(231.8545, 102.0847, 974.3216)
```
Having generated a (completely crazy) list of intraday prices, we will finally
proceed to calculate its higher value.

### Iterators

As functional programmers, we would find the greatest value in a list by using
*fold* (which comes from the general notion of
[*catamorphism*](https://bartoszmilewski.com/2013/06/10/understanding-f-algebras/)),
a higher order function that collapses a data structure. In q, the analogous for
this function is the so-called `over` *iterator* (`/`). This operator takes the
reducing function and the list itself as input arguments, so we could get the
highest price by passing the `|` operator as reducer:
```q
q)(|/)prices
999.9987
```
We can get the analogous behaviour in Scala by using the `reduce` method and
`max` as reducer:
```scala
scala> prices.reduce(max)
val res9: Float = 999.99884
```
Note that they don't lead to the very same value since each version produces
its own random numbers.

As a pure and total functional programmer you might be missing the part of the
algebra that corresponds to the `Nil` (or empty list) case. In fact, `over`
would return `()` when we pass an empty list as second argument. This value
represents the empty list and I guess we could map it as a kind of Scala's
`()`, which corresponds to the unique instance for the *Unit* type. So, to a
certain extent, we could consider that `(+/)` returns either the greatest value
or `()` (`Either[Unit, Float]`), [isomorphic to the `Option`
type](https://bartoszmilewski.com/2015/01/13/simple-algebraic-data-types/). The
Scala version simply raises an exception when `reduce` is invoked from an empty
list. To make things safer, `over` can take an additional argument to
contemplate the `Nil` case as well, as we show in the following snippet:
```q
q)0|/prices
999.9987
```
which would produce `0` when prices correspond to an empty list. From the Scala
viewpoint, we can use the pure `fold` method instead of `reduce`:
```scala
scala> prices.fold(0f)(max)
val res11: Float = 999.99884
```

One of the fundamental pilars of APL is the *suggestivity* of notation, where
Iverson emphasises the importance of inferring new behaviours from existing
expressions. In this sense, we could guess that by passing the operator that
calculates the minumum of two values as an argument for `over`, we should be
able to obtain the lowest price:
```q
q)(&/)prices
0.0008079223
```
The same suggestivity applies to Scala, where we can pass the proper operation
as an argument to `reduce`:
```scala
scala> prices.reduce(Math.min)
val res4: Float = 0.02861023
```
Calculating the maximum and minimum values from a given list is so common, that
q supplies `max` as an alias for `+/`, as in `max prices`. Scala does also
supply the analogous alternative, as in `prices.max`.

### Lambda expressions

Calculating the maximum and minimum prices is ok, but we could be interested on
implementing more sophisticated operations over the list of prices. As a simple
example, we could calculate the higher price that don't exceed a given limit. At
this point, one could be wondering if `over` is restricted to native predefined
operators or if we could pass our own operator as argument in order to implement
that logic. Q, being a functional language, provides support for lambda
expressions, as we show next:
```q
q)0{[x;y]$[y<500f;x|y;x]}/prices
499.9798
```
We rely on the Scala adaptation to explain what is going on:
```scala
scala> prices.fold(0f)((x, y) => if (y < 500f) max(x, y) else x)
val res5: Float = 499.98163
```
As you can see, we replace `|` with the lambda expression that implements the
desired logic: getting the max of `x` and `y` as long as `y` is lower than
`500f`. The Scala translation allows inferring that a Q lambda is surrounded by
curly braces, where `[x;y]` correspond to the input parameters and the rest of
the expression (`$[y<500f;x|y;x]`) acts as the body. The `$` operator is just
the analogous for an `if` statement. It's worth mentioning that when the
parameter block is omitted, q will understand names `x`, `y` and `z` as the
first, second and third parameters, respectively. It's quite similar to the
Scala *placeholder* syntax (`_ + _`), without the limitation of having to use
each parameter exactly once. On its part, q has the limitation of lacking
additional names for functions with a number of parameters greater than three.

We could have parameterized the hardcoded limit as an additional argument and
assign the resulting function a name aiming at facilitating its reutilization.
While doing so, we omit the parameter list, as stated in the previous paragraph:
```q
q)lim:{$[y<z;x|y;x]}
```
The Scala counterpart would be:
```scala
scala> val lim: (Float, Float, Float) => Float = (x, y, z) => if (y < z) max(x, y) else x
```
Or perhaps more idiomatically:
```scala
scala> def lim(x: Float, y: Float, z: Float): Float = if (y < z) max(x, y) else x
```
After we've defined them, we could invoke them by supplying the expected
arguments:
```q
q)lim[0f;499.9798f;500f]
499.9798
```
Invocation that can be adapted as follows:
```scala
scala> lim(0f, 499.9798f, 500f)
val res10: Float = 499.9798
```
where both q and Scala use a consistent notation to declare the parameters and
to supply the arguments.

It might worth remarking that the difference between `val` and `def` is that the
first of them evaluates just once, while the second re-evaluates for each usage.
In this sense, we could determine that the q expression `n:{0}`, a lambda
expression which takes no arguments, would be equivalent to the Scala expression
`def n = 0`.

### Projection

Once we have defined the new name to get the maximum value which is in turn
lower than a given limit, we can modularise the previous logic:
```q
q)0 lim[;;500f]/prices
499.9798
```
We can clumsily adapt this code into Scala, but it requires us to rewrite the
order of `lim` parameters and separate them into different parameter blocks:
```
scala> def lim(z: Float)(x: Float, y: Float): Float = if (y < z) max(x, y) else x

scala> prices.fold(0f)(lim(500f))
val res8: Float = 499.88913
```
As you have probably guessed, what q achieves in `lim[;;500f]` is to currificate
it in its third parameter, where the semicolons just determine that we don't
know the first and second arguments yet. Scala can't compete with such
flexibility, since q enables currying on any parameter, so projection has become
one of our favourite q features.

Before moving on to the next section, we'd like to clarify that the *limit*
logic could benefit from a different implementation. In fact, we think that the
following implementation is more idiomatic in q:
```q
q)max prices where prices<500f
499.9798
```
We could say that the previous code is adapted into Scala as follows:
```scala
scala> prices.filter(_ < 500f).max
val res0: Float = 499.88913
```
However, the q approach is radically different, but we should wait for the next
post to fully understand why.

### More iterators

Once we know how to calculate the highest price from a particular day, we could
be slightly more ambitious and calculate the highest price given a whole year.
In this particular situation, it is common practice to introduce an extra layer
of nesting so we end up with a list of days, where each day contains a list of
prices in turn. First of all, we need to generate a whole year of random prices:
```q
q)prices:{n?1000f}each til 365
```
We adapt it into Scala using the following expression:
```scala
scala> val prices = (0 to (365-1)).map(_ => List.fill(n)(util.Random.nextFloat).map(_ * 1000))
```
The previous expressions generate a list of 365 elements and then assign a list
of random prices to each of them. In this sense, the q primitive `til` is able
to produce all the numbers that go from 0 til the number passed as argument. In
fact, this is pretty much the `to` Scala primitive, but in this case you can
specify not only the ending range limit, but also the starting one.  Also, note
that `to` includes the ending limit on the output, so a minor adjustment is
required. We replace each element by the list of random prices by means of the q
primitive `each`, which takes the mapping function as first argument and the
list of elements as the second one. As the Scala snippet shows, `each`
corresponds to the Scala's `map` method, that we functional programmers
associate to the `Functor` typeclass. In fact, taking into account the constant
mapper, we could have used the derived `as` method (from Scalaz) instead:
```scala
scala> val prices = (0 to (365-1)).as(List.fill(n)(util.Random.nextFloat).map(_ * 1000))
```
We don't know if there exists an equivalent for `as` in q, but given the
simplicity to express constant functions in q, we find that the current
expression is perfectly fine just as it is.

Given the year prices, we think that there are two main approaches to calculate
the higher price: 
- Calculate the maximum price for each day and then calculate the maximum one
  among them
- Put all the prices together and just calculate the maximum one
The first approach is carried out in the next code:
```q
q)max max each prices
999.9998
```
that we can easily translate into Scala:
```scala
scala> prices.map(_.max).max
```
Both expressions should be self-content now.

The second approach is implemented as follows:
```q
q)max raze prices
999.9998
```
It is adapted into Scala using `flatten`:
```scala
scala> prices.flatten.max
```
Indeed, `raze` just flattens the list of lists, and corresponds to the `join`
monadic operation. If we take into account that `,` is the list concatenation
operator, it should be straightforward to understand the implementation of
`raze`:
```q
q)raze
,/
```
As can be seen, it just uses the `over` iterator using `,` as reducer.

At this point we must say that the first approach is preferable, since it's more
modular and therefore parallelizable. In fact, there's a variant of `each` which
is referred to as `peach` that we could use to exploit such aspect. However, we
wanted to show the second approach since it's almost mandatory for a functional
programming-related post to make *yet another monad* reference, isn't it?

## Takeaways

- Q is a functional programming language: lambda expressions, currying,
  higher-order functions (iterators), etc.
- Q is impure: side-effects for randomness, mutability, etc.
- Q is dynamic: no typeclasses, no option, either, etc.
- Q code is hard to read: shortness, associativity, etc.
- Really nice financial features, like the date interface
