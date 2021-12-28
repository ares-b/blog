---
layout: post
title:  "Scala Lazy Evaluation"
author: ares
categories: [ scala ]
image: assets/images/posts/scala-lazy-evaluation/featured.png
toc: true
show-post-image: false
featured: false
---

In Scala, we've a number of immutable collections that provide powerful operations, in particular for combinatorial search.

For instance, let's try to find the second odd number between 1000 and 1000000:

```
(1000 to 1000000).filter(x => x % 2 != 0)(1)
```
In terms of performance, this implementation is reaaaaaaaaaaly bad !

It constructs all prime numbers between 1000 and 1000000, but only ever looks at the second elements of that list.

# Lazy Lists

To make this implementation efficient, we can use lazy evaluation. The idea is to avoid computing the elements of a 
sequence until they are needed for the evaluation of the results (which might be never).

This idea is implemented into Scala's `LazyList` which is similar to the classical implementation List, but LazyList's elements
are evaluated only on demand.

Lazy lists are defined from a constant ``LazyList.empty`` (analog of Nil for strict lists) and a constructor `LazyList.cons` 
(analog of ::).

For instance, `val xs = LazyList.cons(1, LazyList.cons(2, LazyList.empty))`.

In this example, the sub-expressions `LazyList.cons(2, LazyList.empty)` will be evaluated only when we need the second element of 
the LazyList.

LazyLists can be defined just as the other collections using the object LazyList as a factory : `LazyList(1, 2, 3)`.
That will translate to `LazyList.apply(1, 2, 3)`.

Another way of getting LazyLists is to convert some other collections to LazyList, this can be done using the `to(LazyList)` method 
on most Scala's collection library :
```
(1 to 1000000).to(LazyList)
```

Lazy Lists supports almost all methods of lists, for instance, to find the second prime number between 1000 and 1000000 :
```
LazyList.range(1000, 1000000).filter(x => x % 2 != 0)(1)
```
The one major exception to this, is List's `cons` operator :  `::`, `x :: xs` always produces a list, never a LazyList.
However, there's an alternative operator `#::` which produces a lazy list.
```
LazyList.cons(x, xs) <=> x #:: xs 
```

The operator `#::` can be used in expressions as well as patterns.

# Tail Lazy Lists

Let's consider for now, that Lazy Lists are only lazy in their `tail`. `head` and `isEmpty` are computed when the lazy list is created.

This is not the actual behaviour of Lazy Lists, but it makes the understanding of the implementation simpler.

So, let's call this simplified Lazy List `TailLazyLists`:

```
trait TailLazyList[+A] extends Seq[A]{
    def isEmpty: Boolean
    def head: A
    def tail: TailLazyList[A]
    ...
}
```

`TailLazyList` has all the fundamental operations as a normal list. Also, as for normal lists, all other 
methods can be defined in terms of these three fundamental operations.

Now, to finish the implementation, we just have to give concrete implementations of `Empty` and `NonEmpty`
LazyLists in the TailLazyList companion object that defines these three methods.

```
object TailLazyList{
    val empty = new TailLazyList[Nothing]{
        def isEmpty = true
        def head = throw NoSuchElementException("empty.head")
        def tail = throw NoSuchElementException("empty.tail")
        override def toString = "LazyList()"
    }
    
    def cons[T](h: T, t: => TailLazyList[T]): TailLazyList[T] = new TailLazyList[T]{
        def isEmpty = false
        def head = h
        def tail = t
        override def toString = "LazyList("+ h +", ?)" 
    }
} 
```

The main difference between TailLazyList's `cons` and the Struct List's `cons` is that the `tail` parameter 
is a `by-name parameter` of type `TailLazyList[T]`.

The `toString` method in `cons` only returns the head of the list because the tail isn't meant to be evaluated 
for now, as it is a by-name parameter, we don't want to force the lazy list to compute all its elements when 
calling toString method.

So the only difference between Struct Lists and Lazy Lists is `=>`, the by-name parameter. That's why the second argument
of `TailtLazyList.cons` is not evaluated at the point of the call. Instead, it will be evaluated each time someone calls 
tail on a `TailLazyList` object.

You could argue that's actually pretty bad if tail gets called multiple times. As it will be evaluated multiple times, 
but don't worry, we'll address this problem in [Lazy Evaluation](#lazy-evaluation) chapter.

Once we have defined the TailLazyList, we can actually implement other list methods such as `filter` :
```
extension[T](xs: TailLazyList[T]){
    def filter(p: T => Boolean): TailLazyList[T] = {
        if isEmpty then xs
        else if p(xs.head) then cons(xs.head, xs.tail.filter(p))
        else xs.tail.filter(p)
    }
}
```

# Lazy Evaluation

Lazy Evaluation addresses performance issues caused by combinatorial search.
It's a language support and run-time construct built to make lazy computations more efficient.

Previously, we've seen the implementation of Tail Lazy Lists, this implementation suffers from a serious 
performance problem, recall, if tail gets called several times, the corresponding lazy list will be re-evaluated 
each time.

This problem can be avoided by storing the result of the first evaluation of tail and re-using the stored result
instead of recomputing tail.

In a purely functional language such as Scala, this optimization is sound since the tail expression will
produce the same result each time it's evaluated.

This scheme is called `lazy evaluation`, opposed to `by-name evaluation` in the case where everything is 
recomputed, and `strict evaluation` or `by-value evaluation` for normal parameters and val definitions.

So, by default, Scala does strict evaluation (or by-value) and it has by-name parameters which gets evaluated 
each time they're called. Now we also want to have `lazy eveluation` where we evaluate zero times or once, but not 
multiple times.

Lazy evaluation of a value definitions can be done using the keyword `lazy` : `lazy val x = <expression>`. 
The value `x` will be evaluated the first time it is used instead of evaluating it immediately. 

For instance, let's consider the following expression : 
```
def expr = {
  val x = {print("x"); 1}
  lazy val y = {print("y"); 2}
  def z = {print("z"); 3}
  z + y + x + z + y + x
}
```
Try to take some time to figure out what is going to be printed when calling `expr`. Then click of the spoiler 
below to see if you're correct.
<span class="spoiler">
    xzyz gets printed as a side effect of evaluation.
</span>

As you can see, `y` get evaluated only once, and it gets evaluated only when needed.

# Overcoming TailLazyList performance issue

So lazy values allow us to overcome our performance problems for lazy list.

TailLazyLists.cons can be implemented more efficiently using `lazy` keyword : 
```
def cons[T](h: T, t: => TailLazyList[T]): TailLazyList[T] = new TailLazyList[T]{
    def isEmpty = false
    def head = h
    lazy val tail = t
    ...
}
```

Instead of a `def tail = t` we now have a `lazy val tail = t`. That's all what we need to overcome the performance
issue we talked about.

The simplified Lazy List implementation we just saw has a lazy tail, but not a lazy head, nor a lazy isEmpty.
Whereas, the real implementation `LazyList` is lazy for all three operations.

To do this, `LazyList` maintains a lazy state variable.
Basically, there's a class `LazyList` in the standard library that has an `init` parameter that computes
the state of the lazy list.
```
class LazyList[+T](init: => State[T]){
    lazy val state: State[T] = init
}
```
Lazy List's state is either empty or cons.
```
enum State[T]{
    case Empty
    case Cons(h: T, t: LazyList[T])
}
```
The state of the Lazy List, like whether it's `empty` or `cons`, what the head is, etc. That is 
altogether computed lazily by the `init` parameter.

Now you may be wondering why Scala doesn't just handle head and isEmpty laziness using `lazy` keyword as it does for tail,
and there's a good reason for that.

A **lazy val** implies using a concurrent lock which will ensure that only one thread at a given time has read access to the value, 
it also implies using a Boolean value to check whether the value has been already computer or not.

Those two constraints make accessing a lazy val quite slow. The general idea is to use `lazy val` only when you're sure that the 
computation you want to be done in a lazy way is very resource consuming, this way, it will cover the extra cost of accessing it.

# Computing Infinite Sequences

We already saw that elements of lazy lists are computed only when they are needed to produce a result.
That opens up the possibility to definite infinite lists.

For instance, here's a lazy list of all natural integers :
```
def from(n: Int): LazyList[Int] = n #:: from(n + 1)
val naturals = from(0)
```

So, basically, in complete confidence, we calculated all natural numbers. In fact, they'll be computed only once somebody asks 
for a particular natural number, and of course you can never ask for all of them at the same time. You'll probably ask for a particular one, 
and Scala will compute all natural number up to that specific one you asked for.

Let's try to get the first five natural numbers from `naturals`.

```
naturals.take(5) // LazyLists(<not computed>)
```

As you can see, `take(x)` didn't make the Lazy List computes the 5th natural number. So, when do we actually compute anything in Lazy Lists ?

Well, one good way to do so, is to convert to a Struct List : 
```
naturals.take(5).toList // List[Int] = List(0, 1, 2, 3, 4)
```

If we tried to compute all natural number by doing `naturals.toList`, the code will never terminate it execution, because numbers are infinite.

# Sieve of Eratosthenes

The best uses cases for Lazy Lists are algorithms that require an upper bound. Indeed, with Lazy Evaluation, those algorithms can be
easily extended to arbitrary size.

The Sieve of Eratosthenes is a good example for this statement, and it can be used to calculate prime numbers.

The idea is as follows :
- Start with all integers from 2, the first prime number
- Eliminate all multiples of 2
- Now, the first element of the resulting list is 3, a prime number
- Eliminate all multiples of 3
- Iterate forever. At each step, the first element of the list will be a prime number

Martin Odersky's Scala Implementation of Sieve of Eratosthenes is as follows :
```
def sieve(s: LazyList[Int]): LazyList[Int] = {
    s.head #:: sieve(s.tail.filter(_ % s.head != 0))
}
val primes = sieves(from(2))
```
Now, let's get the first 10 prime numbers : 
```
primes.take(100).toList // List[Int] = List(2, 3, 5, 7, 11, 13, 17, 19, 23, 29)
```
The result is `List(2, 3, 5, 7, 11, 13, 17, 19, 23, 29)` which is actually the first 10 prime natural numbers.


# References
Programming in Scala Fifth Edition (18 june 2021), Martin Odersky.