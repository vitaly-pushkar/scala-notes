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

