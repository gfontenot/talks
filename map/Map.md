![](images/war-room.png)

## How I Learned to Stop Worrying and Love the Functor ##

^ Before Swift came out, I started getting really into Haskell, which is a strongly typed, purely functional language used a lot in academia by language theorists.

^ Haskell hasn't had a lot of success as a language that you actually use in production (so far)

^ So when Apple announced Swift in June, I got excited at the possibility to take some of the things I had learned and loved in Haskell, and bring them over to a platform that I could actually make things with.

^ This has led me to be involved in a few libraries that have been accused (justifiably) of trying to push the limits of the language and force it into being a much more functional language than it was probably intended to be originally

---

## Runes ##

```swift
// Optional:
public func <^><T, U>(f: T -> U, a: T?) -> U?
public func <*><T, U>(f: (T -> U)?, a: T?) -> U?
public func >>-<T, U>(a: T?, f: T -> U?) -> U?
public func pure<T>(a: T) -> T?

// Array:
public func <^><T, U>(f: T -> U, a: [T]) -> [U]
public func <*><T, U>(fs: [T -> U], a: [T]) -> [U]
public func >>-<T, U>(a: [T], f: T -> [U]) -> [U]
public func pure<T>(a: T) -> [T]
```

^ Probably the most obvious offender of this claim is Runes.

^ Runes is a library that adds infix operators for some functional concepts (such as `map`, `apply`, and `flatMap`) that help bring Swift's syntax a little closer to that of Haskell, and open the doors for some pretty neat tricks in the language.

---

## Argo ##

```swift
extension User: Decodable {
  static func decode(j: JSON) -> Decoded<User> {
    return User.create
      <^> j <| "id"
      <*> j <| "first_name"
      <*> j <| "last_name"
      <*> j <|? "email"
  }
}
```

^ More interesting to me, is Argo, which uses these functional concepts, along with some fancy tricks using Generics and a couple of custom operators of its own to ease the pain of parsing JSON in Swift, which is kind of a famously annoying problem at this point.

^ If you've ever tried doing JSON parsing manually in Swift, you probably know what I'm talking about.

---

![](http://gifsec.com/wp-content/uploads/GIF/2014/03/Stare-What-GIF.gif)

^ Every time I start talking about this stuff, I get at least one of these.

^ This is a totally reasonable response to those last couple of slides

^ Using these libraries introduces a large amount of educational overhead for your team

^ Not only do developers have to understand Swift, Apple's frameworks, and any added dependencies, but now they also have to understand this fairly opaque set of characters and punctuation littered throughout your codebase.

^ This is daunting, and can be offputting, especially in a community as mature as ours.

---

![](images/winter-is-coming.gif)

^ _but_

^ It's clear (to me) that this really is the direction that we're headed in.

^ The momentum is real.

^ These concepts _aren't going away_. You can ignore them for now, but at some point it's going to catch up with you.

^ We need to create a foundation of knowledge inside the community that we can build upon to avoid stagnation

---

## (Re-)Introducing: `map` ##

^ The bottom line is that operators are just sugar. What's _really_ important are the concepts behind the sugar.

^ I think that `map` is a good introduction to these concepts, without getting bogged down with operators or funky types

^ you're probably at least a little familiar with how it works, but it's almost certainly a more powerful concept than you realize

---

## `Array.map` ##

^ `map` is most commonly associated with `Array`s because of usage in languages like Ruby, which makes it a pretty good starting point for us.

---

## in the bad old days ##

```objc
NSArray *numbers = @[@1, @2, @3, @4];
NSMutableArray *mutable = [NSMutableArray array];

for (NSNumber *number in numbers) {
    mutable.addObject(@(number.integerValue + 1));
}

NSArray *numbersPlusOne = [mutable copy];
```

^ So jumping back, in Objective-C if we wanted to create an array of objects by applying a transformation to an existing list, this was all we had.

^ This isn't ideal

^ lose immutability unless we copy the resulting array

^ manually iterating through the list

^ not very reusable

^ `enumerateObjectsUsingBlock` is a little better because you can pass a block, but doesn't go far enough

---

```swift
extension Array<T> {
  func map<U>(transform: T -> U) -> [U] {
    var array: [U] = []

    for x in self {
      array.append(transform(x))
    }

    return array
  }
}
```

^ So here's a possible implementation for `Array.map`, which is included in the standard lib.

^ You can see that `map` is really just a simple abstraction of that same iteration pattern we used to use in Objective-C

^ So why is this interesting?

---

## usage ##

```swift
let numbers = [1, 2, 3, 4]
let numbersPlusOne = numbers.map { $0 + 1 } // [2, 3, 4, 5]

func addOne(x: Int) -> Int {
  return x + 1
}

let moreNumbers = numbers.map(addOne) // [2, 3, 4, 5]
```

^ No manual iteration

^ Maintain immutability! Assign directly to `let`.

^ Use a named function and pass it directly to `map`

^ I think this is a safer, cleaner, overall nicer option than manual iteration

---

## `Optional.map` ##

^ Swift also introduces `map` for Optional values

^ This is much less common to see. Concept is taken from more functional languages such as Haskell

---

## Optional ##

```swift
enum Optional<T> {
  case .Some(T)
  case .None
}
```

^ Quick recap in case you aren't familiar.

^ Optionals encapsulate the concept of a value's existence.

^ Can't use the value directly. It forces you to deal with the possibility of there not being a value

---

## conditional unwrapping ##

```swift
let myOptional: String? = "Hello World"

if let string = myOptional {
  println(string)
}
```

^ This is the most basic way to pull the value out of an optional.

^ `if let` statements check for the presence of a value, and assigns that value to a local constant if it exists. You can then use that constant inside the scope of the conditional

^ This is a really nice piece of syntax to have in the standard lib, but it introduces some weird things to have to consider, like potentially shadowing local variables and potential nesting.

^ Optional's version of `map` can streamline things.

---

## `Optional.map` ##

```swift
extension Optional<T> {
  func map<U>(f: T -> U) -> U?
}
```

^ Given an Optional value of the type T (self) and a function that transforms an object from type T to type U, return an optional value of the type U

^ Ok, but what does that actually mean? It might help to see an implementation

---

## `Optional.map` ##

```swift
extension Optional<T> {
  func map<U>(f: T -> U) -> U? {
    if let x = self {
      return f(x)
    } else {
      return .None
    }
  }
}
```

^ This implementation should look familiar

^ `map` can be implemented as a simple wrapper around `if let`

^ It can _also_ be implemented using a switch statement and pattern matching, but I'll show an example of that later.

^ In either case, we check to see if `self` has a value. If it does, we return the result of applying the function to the unwrapped value

^ if `self` is `.None`, we return `.None`

^ You can think of `map` as a way to lift functions from the world of non-optional values up into the world where these values might not exist.

^ This acts as a way to separate the concern of optionality and isolate it to this function

^ This is insanely powerful, because it means that you can postpone the fact that there might not be a value for as long as possible.

^ it also means you can keep your core functions pure and use non-optional values, but still easily use them with Optionals

---

## usage ##

```swift
let optional = "foo"
let foobar = optional.map { $0 + "bar" } // .Some("foobar")

func appendBar(x: String) -> String {
  return x + "bar"
}

let moreFoobar = optional.map(appendBar) // .Some("foobar")

```

^ usage is identical to map for arrays

^ Notice that `appendBar` gets to deal with non-optional values

^ Note that the return value is still an optional string

---

## Usage ##

```swift
let nothing: String? = .None
let something: String = "Hello World"

let nope = optional.map(appendBar) // .None
let stillNope = nope.map(appendBar) // .None
let huzzah = something.map(appendBar) // .Some("Hello Worldbar")
```

^ The benefit here is that the syntax doesn't change at all when there is no value.

^ This makes `.None` work very much like `nil` in Objective-C

---

## Real world usage ##

```swift
let name = json["name"] // String?
let user = name.map { User(name: $0) } // User?
let company = user.map { Company(employees: [$0]) } // Company?

let numberOfEmployees = company?.numberOfEmployees ?? 0 // Int
```

^ The fact that dictionary subscripting returns an optional could throw a wrench in our plans

^ We can carry the fact that we might not have a name through the implementation of the function, and only worry about dealing with that failure when we actually need a value

---

## Big Finish ##

- `func map<T, U>(a: Array<T>,    f: T -> U) -> Array<U>`
- `func map<T, U>(a: Optional<T>, f: T -> U) -> Optional<U>`
- `func map<T, U>(a: Result<T>,   f: T -> U) -> Result<U>`

Aw shit now you understand Functors

^ Swift as a language is in flux right now, and we're shaping its future.

^ I'm not saying that using `map` and other functional programming concepts are the end all solution for everything, but they are tools that are available for you, and avoiding them entirely because they come from a different idiom seems silly.

