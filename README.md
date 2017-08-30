# Scala Notes

## Extractors

* Extractors are implemented by a `unapply` method in a companion Object and used for pattern matching
* Each `case class` has an extractor by default
* Signature of `unapply` method: `def unapply(object: S): Option[T]` for single parameter and `def unapply(object: S): Option[(T1, ..., Tn)]` for multiple.
* The same works for matching on sequences: `def unapplySeq(object: S): Option[Seq[T]]`
* Or with TupleN as a return type to combine fixed and variable parameter extraction: `def unapplySeq(object: S): Option[(T1, .., Tn-1, Seq[T])]`
* When `unapply` returns `None` - it will not be matched.
* Implementing custom extractors is only necessary if there is a need to extract something from a type one has no control over

### Examples
```scala
//Single parameter
...

object User {
  def unapply(user: User): Option[String] = Some(user.name)
}
```

```scala
//Multiple parameters
...

object User {
  def unapply(user: User): Option[(String, Int, Double)] =
    Some((user.name, user.score, user.upgradeProbability))
}
```

```scala
//Sequence
...
object GivenNames {
  def unapplySeq(name: String): Option[Seq[String]] = {
    val names = name.trim.split(" ")
    if (names.forall(_.isEmpty)) None else Some(names)
  }
}

//Matching
def greetWithFirstName(name: String) = name match {
  case GivenNames(firstName, _*) => "Good morning, " + firstName + "!"
  case _ => "Welcome! Please make sure to fill in your name!"
}
```

```scala
//Combined
object Names {
  def unapplySeq(name: String): Option[(String, String, Seq[String])] = {
    val names = name.trim.split(" ")
    if (names.size < 2) None
    else Some((names.last, names.head, names.drop(1).dropRight(1)))
  }
}

//Matching:
def greet(fullName: String) = fullName match {
  case Names(lastName, firstName, _*) => "Good morning, " + firstName + " " + lastName + "!"
  case _ => "Welcome! Please make sure to fill in your name!"
}
```


## Pattern matching

* Pattern matching is a dispatch mechanism: choosing which variant of a function is the correct one to call.
* Patterns (cases) in pattern matching are ordered

### Pattern matching expressions
* The return value of such pattern matching is what is returned by the block belonging to the first matched pattern.
* A pattern matching should be a pure function, and results of it's expressions should be used by other functions, making it easier to test.

```scala
def message(player: Player) = player match {
  case Player(_, score) if score > 100000 => "Get a job, dude!"
  case Player(name, _) => "Hey " + name + ", nice to see you again!"
}
def printMessage(player: Player) = println(message(player))
```

### Patterns in value definitions

* It's a kind of destructuring or "unpacking" a data type into values
* Useful to destructure case classes whose type is known at compile time

```scala
def currentPlayer(): Player = Player("Daniel", 3500)

val Player(name, _) = currentPlayer()
doSomethingWithTheName(name)
```

### Patterns in for comprehensions
* The same destructuring works in for comprehensions

```scala
def gameResults(): Seq[(String, Int)] =
  ("Daniel", 3500) :: ("Melissa", 13000) :: ("John", 7000) :: Nil

def hallOfFame = for {
  (name, score) <- gameResults()
  if (score > 5000)
} yield name
```

## Options
* Scala tries to avoid null value which can lead to problems (like `NullPointerException`) by providing an `Option` type for values that may be present or not
* Option[A] is a container for an optional value of type A. If the value of type A is present, the Option[A] is an instance of Some[A], containing the present value of type A. If the value is absent, the Option[A] is the object None.
* Values of specific types `Some` or `None` should be used if one sets values herself, otherwise the companion object `Option` should be used for values that come from "outside" and can contain `null`s, which will create an instance of a corresponding subtype.

```scala
val greeting: Option[String] = Some("Hello world") // It's fine to use specific type for self-defined values

val absentGreeting: Option[String] = Option(ThisJavaLibMightReturnNil.value) // absentGreeting will be None if ThisJavaLibMightReturnNil.value is null

val presentGreeting: Option[String] = Option(ThisJavaLibMightReturnNil.value) // presentGreeting will be Some("Hello!") if ThisJavaLibMightReturnNil.value is a "Hello!" string
```
### Getting values
* In order to extract a value from `Option` with a fallback to _default_ value if `Option` contains `None` is to use `getOrElse` method:

```scala
val user = User(2, "Johanna", "Doe", 30, None)
println("Gender: " + user.gender.getOrElse("not specified")) // will print "not specified"
```
* It's also possible to extract a value with pattern matching on `Some` subtype, although a little bit too verbose:

```scala
val user = User(2, "Johanna", "Doe", 30, None)
user.gender match {
  case Some(gender) => println("Gender: " + gender)
  case None => println("Gender: not specified")
}
```
* Due to the fact that `Option` has monadic characteristics in Scala, it's possible to use all the typical methods one would use on a collection:

```scala
// foreach
UserRepository.findById(2).foreach(user => println(user.firstName)) // The function passed to foreach will be called exactly once, if the Option is a Some, or never, if it is None
```

```scala
// map
val age = UserRepository.findById(1).map(_.age) // age is Some(32) for an existing user
val age = UserRepository.findById(666).map(_.age) // None for unexisting user
```

* If a value returned from an `Option` is also an `Option` it's convenient to use `flatMap` to flatten any level of nesting:

```scala
val gender = UserRepository.findById(1).map(_.gender) // gender is Option[Option[String]]
val gender1 = UserRepository.findById(1).flatMap(_.gender) // gender is Option[String] (Some("male")) for an existing user
val gender2 = UserRepository.findById(666).flatMap(_.gender) // None for unexisting user

```

```scala
//filter 
UserRepository.findById(1).filter(_.age > 30) // Some(user), because age is > 30
UserRepository.findById(2).filter(_.age > 30) // None, because age is <= 30
UserRepository.findById(666).filter(_.age > 30) // None, because user is already None
```

* For-comprehentions
_TODO_

* Chaining options
_TODO_
