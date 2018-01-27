---
title: Manual
version: 0.4
nav: 3
sidebar: manual
---

# Quickstart with sbt

Create a `build.sbt` file in an empty folder with the following contents:

```scala
scalaVersion := "2.12.4"

resolvers += Resolver.bintrayRepo("stg-tud", "maven")

libraryDependencies += "de.tuda.stg" %% "rescala" % "0.21.0"
```

Install [sbt](http://www.scala-sbt.org/) and run `sbt console` inside the folder,
this should allow you to follow along the following examples.

# API Documentation

* [Signal documentation](../scaladoc/#rescala.reactives.Signal)
* [Event documentation](../scaladoc/#rescala.reactives.Event)


# Introduction

This manual covers the main features of the *Rescala* programming language.
[Signals and Vars] presents time-changing values
in *Rescala*, [Events](#events) describes events,
[Conversion Functions](#conversion-functions) covers the conversion functions between
events and time-changing values, [Technicalities](#technicalities)
presents technical details that are necessary to correctly run
*Rescala*, [Related](#related) outlines the related work.

While a major aspect of *Rescala*'s design is the integration of events
and signals, they can be used separately. For example a programmer can
use only *Rescala* events to design application that do not need
time-changing values.

**Scope** The manual serves as an introduction of the concepts in *Rescala*.
The full API is covered in the [scaladoc](../scaladoc/) especially for [Signals](../scaladoc/#rescala.reactives.Signal) and [Events](../scaladoc/#rescala.reactives.Signal).
More details can be found in [[7, 3]](#ref).

The manual introduces the concepts related to functional reactive
programming and event-based programming from a practical
perspective. The readers interested in a more general presentation of
these topics can find thee essential
references in the [related work](#related).

The code examples in the manual serve as a self contained Scala REPL session,
all code is executed and results are annotated as comments using [tut](https://github.com/tpolecat/tut).
To use all features of *Rescala* the only required import is:

```scala
import rescala._
// import rescala._
```

Most code blocks can be executed on their own when adding this import,
but some require definitions from the prior blocks.


# Signals and Vars
[Signals and Vars]: #signals-and-vars

A signal expresses functional dependencies among values.
Intuitively, the value of a signal is computed from one or multiple input values.
Whenever any inputs changes, the value of the signal is also updated.

For example:

```scala
val a = Var(2)
// a: rescala.Var[Int] = a:15

val b = Var(3)
// b: rescala.Var[Int] = b:15

val c = Signal { a() + b() }
// c: rescala.Signal[Int] = c:17

println((a.now, b.now, c.now))
// (2,3,5)

a set 4

println((a.now, b.now, c.now))
// (4,3,7)

b set 5

println((a.now, b.now, c.now))
// (4,5,9)
```

In the code above, the signal `c` is defined to be `a + b` (details on syntax follows in the next section).
When `a` or `b` are updated, the value of `c` is updated as well.


## Vars

A `Var[T]` holds a simple value of type `T` and does not have any inputs.
`Var[T]` is a subtype of `Signal[T]` and can be used as an input for any signal.
Examples for var declarations are:

```scala
val a = Var(0)
// a: rescala.Var[Int] = a:15

val b = Var("Hello World")
// b: rescala.Var[String] = b:15

val c = Var(List(1,2,3))
// c: rescala.Var[List[Int]] = c:15

val d = Var((x: Int) => x * 2)
// d: rescala.Var[Int => Int] = d:15
```

Vars enable the framework to track changes of input values.
Vars can be changed directly by the programmer:

```scala
a set 10

b.set("The `set` method does the same as the update syntax above")

c.transform( list => 0 :: list )
```

Vars are used by the framework to track changes to inputs,
the value of a var must not be mutated indirectly,
as such changes are hidden to the framework.


## Signals

### Defining Signals
 Signals are defined by the syntax
```Signal{```*sigexpr*```}```, where *sigexpr* is a side
effect-free expression. Signals are parametric types. A signal that
carries integer values has the type ```Signal[Int]```.

### Signal expressions
 When, inside a signal expression
defining a signal ```s```, a var or a signal is called with the
```()``` operator, the var or the signal are added to the values
```s``` depends on. In that case, ```s``` *is a dependency* of
the vars and the signals in the signal expression. For example in the
code snippet:

```scala
  val a = Var(0)
// a: rescala.Var[Int] = a:15

  val b = Var(0)
// b: rescala.Var[Int] = b:15

  val s = Signal{ a() + b() } // Multiple vars in a signal expression
// s: rescala.Signal[Int] = s:17
```

The signal ```s``` is a dependency of the vars ```a``` and ```b```,
meaning that the values of ```s``` depends on both ```a``` and
```b```. The following code snippets define valid signal
declarations.

```scala
val a = Var(0)
// a: rescala.Var[Int] = a:15

val b = Var(0)
// b: rescala.Var[Int] = b:15

val c = Var(0)
// c: rescala.Var[Int] = c:15

val r: Signal[Int] = Signal{ a() + 1 } // Explicit type in var decl
// r: rescala.Signal[Int] = r:16

val s = Signal{ a() + b() } // Multiple vars is a signal expression
// s: rescala.Signal[Int] = s:17

val t = Signal{ s() * c() + 10 } // Mix signals and vars in signal expressions
// t: rescala.Signal[Int] = t:17

val u = Signal{ s() * t() } // A signal that depends on other signals
// u: rescala.Signal[Int] = u:17
```

```scala
val a = Var(0)
// a: rescala.Var[Int] = a:15

val b = Var(2)
// b: rescala.Var[Int] = b:15

val c = Var(true)
// c: rescala.Var[Boolean] = c:15

val s = Signal{ if (c()) a() else b() }
// s: rescala.Signal[Int] = s:18
```

```scala
def factorial(n: Int) = Range.inclusive(1,n).fold(1)(_ * _)
// factorial: (n: Int)Int

val a = Var(0)
// a: rescala.Var[Int] = a:15

val s: Signal[Int] = Signal{ // A signal expression can be any code block
  val tmp = a() * 2
  val k = factorial(tmp)
  k + 2  // Returns an Int
}
// s: rescala.Signal[Int] = s:17
```



### Accessing reactive values
 The current value of a
signal or a var can be accessed using the ```now``` method. For
example:

```scala
val a = Var(0)
// a: rescala.Var[Int] = a:15

val b = Var(2)
// b: rescala.Var[Int] = b:15

val c = Var(true)
// c: rescala.Var[Boolean] = c:15

val s: Signal[Int] = Signal{ a() + b() }
// s: rescala.Signal[Int] = s:17

val t: Signal[Boolean] = Signal{ !c() }
// t: rescala.Signal[Boolean] = t:16

val x: Int = a.now
// x: Int = 0

val y: Int = s.now
// y: Int = 2

val z: Boolean = t.now
// z: Boolean = false

println(z)
// false
```

## Example: speed
The following example computes the displacement `space` of a
particle that is moving at constant speed `SPEED`. The
application prints all the values associated to the displacement over
time.

```scala
val SPEED = 10
// SPEED: Int = 10

val time = Var(0)
// time: rescala.Var[Int] = time:15

val space = Signal{ SPEED * time() }
// space: rescala.Signal[Int] = space:17

space.changed += ((x: Int) => println(x))
// res9: rescala.reactives.Observe[rescala.parrp.ParRP] = res9:18

while (time.now < 5) {
  Thread sleep 20
  time set time.now + 1
}
// 10
// 20
// 30
// 40
// 50
```

The application behaves as follows. Every 20 milliseconds, the value
of the `time` var is increased by 1 (Line 9).
When the value of the `time` var changes, the signal expression
at Line 3 is reevaluated and the value of `space` is
updated. Finally, the current value of the `space` signal is
printed every time the value of the signal changes.

Printing the value of a signal deserves some more considerations.
Technically, this is achieved by converting the ```space``` signal to
an event that is fired every time the signal changes its value
(Line 5). The conversion is performed by the
`changed` operator. The `+=` operator attaches an handler to
the event returned by the `changed` operator. When the event
fires, the handler is executed. Line 5 is equivalent to
the following code:

```scala
val e: Event[Int] = space.changed
// e: rescala.Event[Int] = (changed space:17)

val handler:  (Int => Unit) =  ((x: Int) => println(x))
// handler: Int => Unit = $$Lambda$16327/1938350102@6aa76e6c

e observe handler
// res11: rescala.reactives.Observe[rescala.parrp.ParRP] = res11:18
```

Note that using `println(space.now)` would also print the
value of the signal, but only at the point in time in which the print
statement is executed. Instead, the approach described so far prints
*all* values of the signal. More details about converting signals
into events and back are provided in [Conversion Functions](#conversion-functions).

---

# Events

*Rescala* supports different kind of events. Imperative events are
directly triggered from the user. Declarative events trigger when the
events they depend on trigger. In reactive applications, events are
typically used to model changes that happen at discrete points in
time. For example a mouse click from the user or the arrival of a new
network packet. Some features of *Rescala* events are valid for all
event types.


* Events carry a value. The value is associated to the event when
  the event is fired and received by all the registered handlers when
  each handler is executed.

* Events are generic types parametrized with the type of value
  they carry, like `Event[T]` and `Evt[T]` where
  `T` is the value carried by the event.

* Both imperative events and declarative events are subtypes of
  `Event[T]` and can referred to generically.


# Imperative events

*Rescala* imperative events are triggered imperatively by the
programmer. One can think to imperative events as a generalization of
a method call which supports (multiple) bodies that are registered and
unregistered dynamically.

## Defining  Events

Imperative events are defined by the `Evt[T]`
type. The value of the parameter `T` defines the value that is
attached to the event. An event with no parameter attached has
signature `Evt[Unit]`. The following code snippet show
valid events definitions:

```scala
val e1 = Evt[Unit]()
// e1: rescala.Evt[Unit] = rescala.RescalaInterface#Evt:69

val e2 = Evt[Int]()
// e2: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e3 = Evt[String]()
// e3: rescala.Evt[String] = rescala.RescalaInterface#Evt:69

val e4 = Evt[Boolean]()
// e4: rescala.Evt[Boolean] = rescala.RescalaInterface#Evt:69

val e5: Evt[Int] = Evt[Int]()
// e5: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

class Foo
// defined class Foo

val e6 = Evt[Foo]()
// e6: rescala.Evt[Foo] = rescala.RescalaInterface#Evt:69
```

It is possible to attach more than one value to the same event. This
is easily accomplished by using a tuple as a generic parameter
type. For example:

```scala
val e1 = Evt[(Int,Int)]()
// e1: rescala.Evt[(Int, Int)] = rescala.RescalaInterface#Evt:69

val e2 = Evt[(String,String)]()
// e2: rescala.Evt[(String, String)] = rescala.RescalaInterface#Evt:69

val e3 = Evt[(String,Int)]()
// e3: rescala.Evt[(String, Int)] = rescala.RescalaInterface#Evt:69

val e4 = Evt[(Boolean,String,Int)]()
// e4: rescala.Evt[(Boolean, String, Int)] = rescala.RescalaInterface#Evt:69

val e5: Evt[(Int,Int)] = Evt[(Int,Int)]()
// e5: rescala.Evt[(Int, Int)] = rescala.RescalaInterface#Evt:69
```

Note that an imperative event is also an event. Therefore the
following declaration is also valid:

```scala
val e1: Event[Int] = Evt[Int]()
// e1: rescala.Event[Int] = rescala.RescalaInterface#Evt:69
```

## Registering Handlers

Handlers are code blocks that are executed when the event fires. The
`+=` operator attaches the handler to the event. The handler is a
first class function that receives the attached value as a parameter.
The following are valid handler definitions.

```scala
var state = 0
// state: Int = 0

val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

e += { println(_) }
// res12: rescala.reactives.Observe[rescala.parrp.ParRP] = res12:17

e += (x => println(x))
// res13: rescala.reactives.Observe[rescala.parrp.ParRP] = res13:18

e += ((x: Int) => println(x))
// res14: rescala.reactives.Observe[rescala.parrp.ParRP] = res14:18

e += (x => {  // Multiple statements in the handler
  state = x
  println(x)
})
// res15: rescala.reactives.Observe[rescala.parrp.ParRP] = res15:19
```

The signature of the handler must conform the signature of the event,
since the handler is supposed to process the attached value and
perform side effects. For example is the event is of type
`Event[(Int,Int)]` the handler must be of type `(Int,Int) => Unit`.

```scala
val e = Evt[(Int,String)]()
// e: rescala.Evt[(Int, String)] = rescala.RescalaInterface#Evt:69

e += (x => {
  println(x._1)
  println(x._2)
})
// res16: rescala.reactives.Observe[rescala.parrp.ParRP] = res16:18

e += ((x: (Int,String)) => {
  println(x)
})
// res17: rescala.reactives.Observe[rescala.parrp.ParRP] = res17:18
```

Note that events without arguments still need an argument
in the handler.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

e += { x => println() }
// res18: rescala.reactives.Observe[rescala.parrp.ParRP] = res18:17

e += { (x: Int) => println() }
// res19: rescala.reactives.Observe[rescala.parrp.ParRP] = res19:17
```

Scala allows one to refer to a method using the partially applied
function syntax. This approach can be used to directly register a
method as an event handler. For example:

```scala
def m1(x: Int) = {
  val y = x + 1
  println(y)
}
// m1: (x: Int)Unit

val e = Evt[Int]
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

e += m1 _
// res20: rescala.reactives.Observe[rescala.parrp.ParRP] = res20:18

e.fire(10)
// 11
```

## Firing Events

Events can be fired with the same syntax of a method call. When an
event is fired, a proper value must be associated to the event
call. Clearly, the value must conform the signature of the event. For
example:

```scala
val e1 = Evt[Int]()
// e1: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e2 = Evt[Boolean]()
// e2: rescala.Evt[Boolean] = rescala.RescalaInterface#Evt:69

val e3 = Evt[(Int,String)]()
// e3: rescala.Evt[(Int, String)] = rescala.RescalaInterface#Evt:69

e1.fire(10)

e2.fire(false)

e3.fire((10,"Hallo"))
```

When a handler is registered to an event, the handler is executed
every time the event is fired. The actual parameter is provided to the
handler.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

e += { x => println(x) }
// res25: rescala.reactives.Observe[rescala.parrp.ParRP] = res25:18

e.fire(10)
// 10

e.fire(10)
// 10
```

If multiple handlers are registered, all of them are executed when the
event is fired. Applications should not rely on any specific execution
order for handler execution.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

e += { x => println(x) }
// res28: rescala.reactives.Observe[rescala.parrp.ParRP] = res28:18

e += { x => println(f"n: $x")}
// res29: rescala.reactives.Observe[rescala.parrp.ParRP] = res29:18

e.fire(10)
// 10
// n: 10

e.fire(10)
// 10
// n: 10
```

## Unregistering Handlers

Handlers can be unregistered from events with the `remove`
operator. When a handler is unregistered, it is not executed when the
event is fired.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val handler1 = e += println
// handler1: rescala.reactives.Observe[rescala.parrp.ParRP] = handler1:16

val handler2 = e += { x => println(s"n: $x") }
// handler2: rescala.reactives.Observe[rescala.parrp.ParRP] = handler2:17

e.fire(10)
// 10
// n: 10

handler2.remove()

e.fire(10)
// 10

handler1.remove()

e.fire(10)
```

# Declarative Events

*Rescala* supports declarative events, which are defined as a
combination of other events. For this purpose it offers operators like
`e_1 || e_2` , `e_1 && p` , `e_1.map(f)`. Event composition allows to
express the application logic in a clear and declarative way. Also,
the update logic is better localized because a single expression
models all the sources and the transformations that define an event
occurrence.

## Defining Declarative Events

Declarative events are defined by composing other events. The
following code snippet shows some examples of valid definitions for
declarative events.

```scala
val e1 = Evt[Int]()
// e1: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e2 = Evt[Int]()
// e2: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e3 = e1 || e2
// e3: rescala.reactives.Event[Int,rescala.parrp.ParRP] = (or rescala.RescalaInterface#Evt:69 rescala.RescalaInterface#Evt:69)

val e4 = e1 && ((x: Int)=> x>10)
// e4: rescala.reactives.Event[Int,rescala.parrp.ParRP] = (filter rescala.RescalaInterface#Evt:69)

val e5 = e1 map ((x: Int)=> x.toString)
// e5: rescala.reactives.Event[String,rescala.parrp.ParRP] = e5:17
```

# Event Operators

This section presents in details the operators that allow one to
compose events into declarative events.

## OR Events

The event `e_1 || e_2` is fired upon the occurrence of one among `e_1`
or `e_2`. Note that the events that appear in the event expression
must have the same parameter type (`Int` in the next example).

```scala
val e1 = Evt[Int]()
// e1: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e2 = Evt[Int]()
// e2: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e1_OR_e2 = e1 || e2
// e1_OR_e2: rescala.reactives.Event[Int,rescala.parrp.ParRP] = (or rescala.RescalaInterface#Evt:69 rescala.RescalaInterface#Evt:69)

e1_OR_e2 += ((x: Int) => println(x))
// res37: rescala.reactives.Observe[rescala.parrp.ParRP] = res37:18

e1.fire(1)
// 1

e2.fire(2)
// 2
```

## Predicate Events

The event `e && p` is fired if `e` occurs and the predicate `p` is
satisfied. The predicate is a function that accepts the event
parameter as a formal parameter and returns `Boolean`. In other
words the `&&` operator filters the events according to their
parameter and a predicate.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e_AND: Event[Int] = e && ((x: Int) => x>10)
// e_AND: rescala.Event[Int] = (filter rescala.RescalaInterface#Evt:69)

e_AND += ((x: Int) => println(x))
// res40: rescala.reactives.Observe[rescala.parrp.ParRP] = res40:18

e.fire(5)

e.fire(15)
// 15
```

## Map Events

The event `e.map f` is obtained by applying `f` to the value carried
by `e`. The map function must take the event parameter as a formal
parameter. The return type of the map function is the type parameter
value of the resulting event.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e_MAP: Event[String] = e map ((x: Int) => x.toString)
// e_MAP: rescala.Event[String] = e_MAP:17

e_MAP += ((x: String) => println(s"Here: $x"))
// res43: rescala.reactives.Observe[rescala.parrp.ParRP] = res43:18

e.fire(5)
// Here: 5

e.fire(15)
// Here: 15
```

{::comment}
## dropParam

The `dropParam` operator transforms an event into an event with
`Unit` parameter. In the following example the `dropParam`
operator transforms an `Event[Int]` into an `Event[Unit]`.

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e_drop: Event[Unit] = e.dropParam
// e_drop: rescala.Event[Unit] = e_drop:16

e_drop += (_ => println("*"))
// res46: rescala.reactives.Observe[rescala.parrp.ParRP] = res46:17

e.fire(10)
// *

e.fire(10)
// *
```

The typical use case for the `dropParam` operator is to make events
with different types compatible. For example the following snippet is
rejected by the compiler since it attempts to combine two events of
different types with the `||` operator.

```scala
scala> /* WRONG - DON'T DO THIS */
     | val e1 = Evt[Int]()
e1: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

scala> val e2 = Evt[Unit]()
e2: rescala.Evt[Unit] = rescala.RescalaInterface#Evt:69

scala> val e1_OR_e2 = e1 || e2  // Compiler error
<console>:17: warning: a type was inferred to be `AnyVal`; this may indicate a programming error.
       val e1_OR_e2 = e1 || e2  // Compiler error
                         ^
e1_OR_e2: rescala.reactives.Event[AnyVal,rescala.parrp.ParRP] = (or rescala.RescalaInterface#Evt:69 rescala.RescalaInterface#Evt:69)
```

The following example is correct. The `dropParam` operator allows
one to make the events compatible with each other.

```scala
val e1 = Evt[Int]()
// e1: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val e2 = Evt[Unit]()
// e2: rescala.Evt[Unit] = rescala.RescalaInterface#Evt:69

val e1_OR_e2: Event[Unit] = e1.dropParam || e2
// e1_OR_e2: rescala.Event[Unit] = (or e1_OR_e2:17 rescala.RescalaInterface#Evt:69)
```
{:/comment}

---

# Conversion Functions

*Rescala* provides functions that interface signals and
events. Conversion functions are fundamental to introduce
time-changing values into OO applications -- which are usually
event-based.

# Basic Conversion Functions

This section covers the basic conversions between signals and events.
Figure 1 shows how basic conversion functions can
bridge signals and events. Events (Figure 1,
left) occur at discrete point in time (x axis) and have an associate
value (y axis). Signals, instead, hold a value for a continuous
interval of time (Figure 1, right). The
`latest` conversion functions creates a signal from an event. The
signal holds the value associated to an event. The value is hold until
the event is fired again and a new value is available. The
`changed` conversion function creates an event from a signal. The
function fires a new event every time a signal changes its value.

<figure markdown="1">
![Event-Signal](./images/event-signal.png)
<figcaption>Figure 1: Basic conversion functions.</figcaption>
</figure>

## Event to Signal: Latest

Returns a signal holding the latest value of the event `e`. The
initial value of the signal is set to `init`.

`latest[T](e: Event[T], init: T): Signal[T]`

Example:

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val s: Signal[Int] = e.latest(10)
// s: rescala.Signal[Int] = s:16

assert(s.now == 10)

e.fire(1)

assert(s.now == 1)

e.fire(2)

assert(s.now == 2)

e.fire(1)

assert(s.now == 1)
```

## Signal to Event: Changed

The `changed` function applies to a signal and returns an event
that is fired every time the signal changes its value.

`changed[U >: T]: Event[U]`

Example:

```scala
var test = 0
// test: Int = 0

val v =  Var(1)
// v: rescala.Var[Int] = v:15

val s = Signal{ v() + 1 }
// s: rescala.Signal[Int] = s:16

val e: Event[Int] = s.changed
// e: rescala.Event[Int] = (changed s:16)

e += ((x:Int)=>{test+=1})
// res57: rescala.reactives.Observe[rescala.parrp.ParRP] = res57:18

v.set(2)

assert(test == 1)

v.set(3)

assert(test == 2)
```

## Fold

The `fold` function creates a signal by folding events with a
given function. Initially the signal holds the `init`
value. Every time a new event arrives, the function `f` is
applied to the previous value of the signal and to the value
associated to the event. The result is the new value of the signal.

`fold[T,A](e: Event[T], init: A)(f :(A,T)=>A): Signal[A]`

Example:

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val f = (x:Int,y:Int)=>(x+y)
// f: (Int, Int) => Int = $$Lambda$16465/1421044470@61128770

val s: Signal[Int] = e.fold(10)(f)
// s: rescala.Signal[Int] = s:17

e.fire(1)

e.fire(2)

assert(s.now == 13)
```

## Iterate

Returns a signal holding the value computed by `f` on the
occurrence of an event. Differently from `fold`, there is no
carried value, i.e. the value of the signal does not depend on the
current value but only on the accumulated value.

`iterate[A](e: Event[_], init: A)(f: A=>A): Signal[A]`

Example:

```scala
var test: Int = 0
// test: Int = 0

val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val f = (x:Int)=>{test=x; x+1}
// f: Int => Int = $$Lambda$16468/1217894656@494159fe

val s: Signal[Int] = e.iterate(10)(f)
// s: rescala.Signal[Int] = s:17

e.fire(1)

assert(test == 10)

assert(s.now == 11)

e.fire(2)

assert(test == 11)

assert(s.now == 12)

e.fire(1)

assert(test == 12)

assert(s.now == 13)
```

## LatestOption

The `latestOption` function is a variant of the `latest`
function which uses the `Option` type to distinguish the case in
which the event did not fire yet. Holds the latest value of an event
as `Some(val)` or `None`.

`latestOption[T](e: Event[T]): Signal[Option[T]]`

Example:

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val s: Signal[Option[Int]] = e.latestOption()
// s: rescala.Signal[Option[Int]] = s:16

assert(s.now == None)

e.fire(1)

assert(s.now == Option(1))

e.fire(2)

assert(s.now == Option(2))

e.fire(1)

assert(s.now == Option(1))
```

## Last

The `last` function generalizes the `latest` function and
returns a signal which holds the last `n` events.

`last[T](e: Event[T], n: Int): Signal[List[T]]`

Initially, an empty list is returned. Then the values are
progressively filled up to the size specified by the
programmer. Example:

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val s: Signal[scala.collection.LinearSeq[Int]] = e.last(5)
// s: rescala.Signal[scala.collection.LinearSeq[Int]] = s:16

s observe println
// Queue()
// res81: rescala.reactives.Observe[rescala.parrp.ParRP] = res81:17

e.fire(1)
// Queue(1)

e.fire(2)
// Queue(1, 2)

e.fire(3);e.fire(4);e.fire(5)
// Queue(1, 2, 3)
// Queue(1, 2, 3, 4)
// Queue(1, 2, 3, 4, 5)

e.fire(6)
// Queue(2, 3, 4, 5, 6)
```

## List

Collects the event values in a (growing) list. This function should be
used carefully. Since the entire history of events is maintained, the
function can potentially introduce a memory overflow.

`list[T](e: Event[T]): Signal[List[T]]`

## Count

Returns a signal that counts the occurrences of the event. Initially,
when the event has never been fired yet, the signal holds the value
0. The argument of the event is simply discarded.

`count(e: Event[_]): Signal[Int]`

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val s: Signal[Int] = e.count
// s: rescala.Signal[Int] = s:16

assert(s.now == 0)

e.fire(1)

e.fire(3)

assert(s.now == 2)
```

## Change

The ```change``` function is similar to ```changed```, but it
provides both the old and the new value of the signal in a tuple.

```change[U >: T]: Event[(U, U)]```

Example:

```scala
val s = Var(5)
// s: rescala.Var[Int] = s:15

val e = s.change
// e: rescala.reactives.Event[rescala.reactives.Signals.Diff[Int],rescala.parrp.ParRP] = e:16

e += println
// res90: rescala.reactives.Observe[rescala.parrp.ParRP] = res90:17

s.set(10)
// Diff(Value(5), Value(10))

s.set(20)
// Diff(Value(10), Value(20))
```

## ChangedTo

The ```changedTo``` function is similar to ```changed```, but it
fires an event only when the signal changes its value to a given
value.

```changedTo[V](value: V): Event[Unit]```

```scala
var test = 0
// test: Int = 0

val v =  Var(1)
// v: rescala.Var[Int] = v:15

val s = Signal{ v() + 1 }
// s: rescala.Signal[Int] = s:16

val e: Event[Unit] = s.changedTo(3)
// e: rescala.Event[Unit] = e:16

e += ((x:Unit)=>{test+=1})
// res93: rescala.reactives.Observe[rescala.parrp.ParRP] = res93:18

assert(test == 0)

v set(2)

assert(test == 1)

v set(3)

assert(test == 1)
```

## Reset

When the ```reset``` function is called for the first time, the
```init``` value is used by the factory to determine the signal
returned by the ```reset``` function. When the event occurs the
 factory is applied to the event value to determine the new signal.

`reset[T,A](e: Event[T], init: T)(factory: (T)=>Signal[A]): Signal[A]`

Example:

```scala
val e = Evt[Int]()
// e: rescala.Evt[Int] = rescala.RescalaInterface#Evt:69

val v1 =  Var(0)
// v1: rescala.Var[Int] = v1:15

val v2 =  Var(10)
// v2: rescala.Var[Int] = v2:15

val s1 = Signal{ v1() + 1 }
// s1: rescala.Signal[Int] = s1:16

val s2 = Signal{ v2() + 1 }
// s2: rescala.Signal[Int] = s2:16

def factory(x: Int) = x%2 match {
  case 0 => s1
  case 1 => s2
}
// factory: (x: Int)rescala.Signal[Int]

val s3 = e.reset(100)(factory)
// s3: rescala.reactives.Signal[Int,rescala.parrp.ParRP] = s3:17

assert(s3.now == 1)

v1.set(1)

assert(s3.now == 2)

e.fire(101)

assert(s3.now == 11)

v2.set(11)

assert(s3.now == 12)
```

## Switch/toggle

The ```toggle``` function switches alternatively between the given
signals on the occurrence of an event ```e```. The value attached to
the event is simply discarded.

```toggle[T](e : Event[_], a: Signal[T], b: Signal[T]): Signal[T]```

The `switchTo` function switches the value of the signal on the
occurrence of the event ```e```. The resulting signal is a constant
signal whose value is the value carried by the event ```e```.

```switchTo[T](e : Event[T], original: Signal[T]): Signal[T]```

The ```switchOnce``` function switches to a new signal provided as a
parameter, once, on the occurrence of the event ```e```.

`switchOnce[T](e: Event[_], original: Signal[T], newSignal: Signal[T]): Signal[T]`

## Flatten

The ```unwrap``` function is used to ``unwrap'' an event inside a signal.

```def unwrap[T](wrappedEvent: Signal[Event[T]]): Event[T]```

It can, for instance, be used to detect if any signal within a collection of signals
fired a changed event:

```scala
val v1 = Var(1)
// v1: rescala.Var[Int] = v1:15

val v2 = Var("Test")
// v2: rescala.Var[String] = v2:15

val v3 = Var(true)
// v3: rescala.Var[Boolean] = v3:15

val collection: List[Signal[_]] = List(v1, v2, v3)
// collection: List[rescala.Signal[_]] = List(v1:15, v2:15, v3:15)

val innerChanges = Signal {collection.map(_.changed).reduce((a, b) => a || b)}
// innerChanges: rescala.Signal[rescala.reactives.Event[Any,rescala.parrp.ParRP]] = innerChanges:18

val anyChanged = innerChanges.flatten
// anyChanged: rescala.reactives.Event[Any,rescala.parrp.ParRP] = anyChanged:16

anyChanged += println
// res106: rescala.reactives.Observe[rescala.parrp.ParRP] = res106:17

v1.set(10)
// 10

v2.set("Changed")
// Changed

v3.set(false)
// false
```

---

# Common Pitfalls

In this section we
collect the most common pitfalls for users that are new to reactive
programming and *Rescala*.

## Accessing values in signal expressions

The ```()```
operator used on a signal or a var, inside a signal expression,
returns the signal/var value *and* creates a dependency. The
```now``` operator returns the current value but does *not*
create a dependency. For example the following signal declaration
creates a dependency between ```a``` and ```s```, and a dependency
between ```b``` and ```s```.

```scala
val s = Signal{ a() + b() }
// s: rescala.Signal[Int] = s:17
```

The following code instead establishes only a dependency between
```b``` and ```s```.

```scala
val s = Signal{ a.now + b() }
// <console>:17: warning: Using `now` inside a reactive expression does not create a dependency, and can result in glitches. Use `apply` instead.
//        val s = Signal{ a.now + b() }
//                          ^
// s: rescala.Signal[Int] = s:17
```

In other words, in the last example, if ```a``` is updated, ```s```
is not automatically updated. With the exception of the rare cases in
which this behavior is desirable, using ```now``` inside a signal
expression is almost certainly a mistake. As a rule of dumb, signals
and vars appear in signal expressions with the ```()``` operator.


## Attempting to assign a signal
Signals are not
assignable. Signal depends on other signals and vars, the dependency
is expressed by the signal expression. The value of the signal is
automatically updated when one of the values it depends on
changes. Any attempt to set the value of a signal manually is a
mistake.


## Side effects in signal expressions
Signal expressions
should be pure. i.e. they should not modify external variables. For
example the following code is conceptually wrong because the variable
```c``` is imperatively assigned form inside the signal expression
(Line 4).

```scala
var c = 0                 /* WRONG - DON'T DO IT */
// c: Int = 0

val s = Signal{
  val sum = a() + b();
  c = sum * 2
}
// s: rescala.Signal[Unit] = s:18

// …
println(c)
// 4
```

A possible solution is to refactor the code above to a more functional
style. For example, by removing the variable ```c``` and replacing it
directly with the signal.

```scala
val c = Signal{
  val sum = a() + b();
  sum * 2
}
// c: rescala.Signal[Int] = c:17

// …
println(c.now)
// 4
```

## Cyclic dependencies
When a signal ```s``` is defined, a
dependency is establishes with each of the signals or vars that appear
in the signal expression of ```s```. Cyclic dependencies produce a
runtime error and must be avoided. For example the following code:

```scala
val a = Var(0)             /* WRONG - DON'T DO IT */
// a: rescala.Var[Int] = a:15

val s = Signal{ a() + t() }
// <console>:17: error: overloaded method value + with alternatives:
//   (x: Double)Double <and>
//   (x: Float)Float <and>
//   (x: Long)Long <and>
//   (x: Int)Int <and>
//   (x: Char)Int <and>
//   (x: Short)Int <and>
//   (x: Byte)Int <and>
//   (x: String)String
//  cannot be applied to (Boolean)
//        val s = Signal{ a() + t() }
//                            ^

val t = Signal{ a() + s() + 1 }
// <console>:17: error: overloaded method value + with alternatives:
//   (x: Double)Double <and>
//   (x: Float)Float <and>
//   (x: Long)Long <and>
//   (x: Int)Int <and>
//   (x: Char)Int <and>
//   (x: Short)Int <and>
//   (x: Byte)Int <and>
//   (x: String)String
//  cannot be applied to (Unit)
//        val t = Signal{ a() + s() + 1 }
//                            ^
```

creates a mutual dependency between ```s``` and
```t```. Similarly, indirect cyclic dependencies must be avoided.

## Objects and mutability
Vars and signals may behave
unexpectedly with mutable objects. Consider the following example.

```scala
class Foo(init: Int){            /* WRONG - DON'T DO IT */
  var x = init
}
// defined class Foo

val foo = new Foo(1)
// foo: Foo = Foo@2c35de6

val varFoo = Var(foo)
// varFoo: rescala.Var[Foo] = varFoo:16

val s = Signal{ varFoo().x + 10 }
// s: rescala.Signal[Int] = s:16

// s.now == 11
foo.x = 2
// foo.x: Int = 2

// s.now == 11
```

One may expect that after increasing the value of ```foo.x``` in
Line 9, the signal expression is evaluated again and updated
to 12. The reason why the application behaves differently is that
signals and vars hold *references* to objects, not the objects
themselves. When the statement in Line 9 is executed, the
value of the ```x``` field changes, but the reference hold by the
```varFoo``` var is the same. For this reason, no change is detected
by the var, the var does not propagate the change to the signal, and
the signal is not reevaluated.

A solution to this problem is to use immutable objects. Since the
objects cannot be modified, the only way to change a filed is to
create an entirely new object and assign it to the var. As a result,
the var is reevaluated.

```scala
class Foo(val x: Int){}
// defined class Foo

val foo = new Foo(1)
// foo: Foo = Foo@10d91c48

val varFoo = Var(foo)
// varFoo: rescala.Var[Foo] = varFoo:16

val s = Signal{ varFoo().x + 10 }
// s: rescala.Signal[Int] = s:16

// s.now == 11
varFoo set (new Foo(2))

// s.now == 12
```

Alternatively, one can still use mutable objects but assign again the
var to force the reevaluation. However this style of programming is
confusing for the reader and should be avoided when possible.

```scala
class Foo(init: Int){   /* WRONG - DON'T DO IT */
  var x = init
}
// defined class Foo

val foo = new Foo(1)
// foo: Foo = Foo@300b95db

val varFoo = Var(foo)
// varFoo: rescala.Var[Foo] = varFoo:16

val s = Signal{ varFoo().x + 10 }
// s: rescala.Signal[Int] = s:16

// s.now == 11
foo.x = 2
// foo.x: Int = 2

varFoo set foo

// s.now == 11
```

## Functions of reactive values
Functions that operate on
traditional values are not automatically transformed to operate on
signals. For example consider the following functions:

```scala
def increment(x: Int): Int = x + 1
// increment: (x: Int)Int
```

The following code does not compile because the compiler expects an
integer, not a var as a parameter of the ```increment``` function. In
addition, since the ```increment``` function returns an integer,
```b``` has type ```Int```, and the call ```b()``` in the signal
expression is also rejected by the compiler.

```scala
val a = Var(1)           /* WRONG - DON'T DO IT */
// a: rescala.Var[Int] = a:15

val b = increment(a)
// <console>:17: error: type mismatch;
//  found   : rescala.Var[Int]
//     (which expands to)  rescala.reactives.Var[Int,rescala.parrp.ParRP]
//  required: Int
//        val b = increment(a)
//                          ^

val s = Signal{ b() + 1 }
// s: rescala.Signal[Int] = s:16
```

The following code snippet is syntactically correct, but the signal
has a constant value 2 and is not updated when the var changes.

```scala
val a = Var(1)
// a: rescala.Var[Int] = a:15

val b = increment(a.now)
// b: Int = 2

val s = Signal{ b + 1 }
// s: rescala.Signal[Int] = s:16
```

The following solution is syntactically correct and the signal
```s``` is updated every time the var ```a``` is updated.

```scala
val a = Var(1)
// a: rescala.Var[Int] = a:15

val s = Signal{ increment(a()) + 1 }
// s: rescala.Signal[Int] = s:17
```

---

# Essential Related Work
{: #related }

A more academic presentation of *Rescala* is in [[7]](#ref). A
complete bibliography on reactive programming is beyond the scope of
this work. The interested reader can refer
to[[1]](#ref) for an overview of reactive programming
and to[[8]](#ref) for the issues
concerning the integration of RP with object-oriented programming.


*Rescala* builds on ideas originally developed in
EScala [[3]](#ref) -- which supports
event combination and implicit events. Other reactive languages
directly represent time-changing values and remove inversion of
control. Among the others, we mention
FrTime [[2]](#ref) (Scheme),
FlapJax [[6]](#ref) (Javascript),
AmbientTalk/R [[4]](#ref) and
Scala.React [[5]](#ref) (Scala).

---

# Acknowledgments

Several people contributed to this manual with their ideas and
comments. Among the others Gerold Hintz and Pascal Weisenburger.

---

# References
{: #ref}
[1] E. Bainomugisha, A. Lombide Carreton, T. Van Cutsem, S. Mostinckx, and
W. De Meuter. A survey on reactive programming. ACM Comput. Surv. 2013.

[2] G. H. Cooper and S. Krishnamurthi. Embedding dynamic dataflow in a call-byvalue
language. In ESOP, pages 294–308, 2006.

[3] V. Gasiunas, L. Satabin, M. Mezini, A. Ńũnez, and J. Noýe. EScala: modular
event-driven object interactions in Scala. AOSD ’11, pages 227–240. ACM, 2011.

[4] A. Lombide Carreton, S. Mostinckx, T. Cutsem, and W. Meuter. Loosely-coupled
distributed reactive programming in mobile ad hoc networks. In J. Vitek, editor,
Objects, Models, Components, Patterns, volume 6141 of Lecture Notes in Computer
Science, pages 41–60. Springer Berlin Heidelberg, 2010.

[5] I. Maier and M. Odersky. Deprecating the Observer Pattern with Scala.react. Technical
report, 2012.

[6] L. A. Meyerovich, A. Guha, J. Baskin, G. H. Cooper, M. Greenberg, A. Bromfield,
and S. Krishnamurthi. Flapjax: a programming language for ajax applications.
OOPSLA ’09, pages 1–20. ACM, 2009.

[7] G. Salvaneschi, G. Hintz, and M. Mezini. Rescala: Bridging between objectoriented
and functional style in reactive applications. In Proceedings of the 13th
International Conference on Aspect-Oriented Software Development, AOSD ’14,
New York, NY, USA, Accepted for publication, 2014. ACM.

[8] G. Salvaneschi and M. Mezini. Reactive behavior in object-oriented applications:
an analysis and a research roadmap. In Proceedings of the 12th annual international
conference on Aspect-oriented software development, AOSD ’13, pages
37–48, New York, NY, USA, 2013. ACM.
