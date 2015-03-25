![](images/war-room.png)

## How I Learned to Stop Worrying and Love the Functor ##

^ I really like Swift

^ I really really like functional programming

^ I honestly think it leads to cleaner, more straight forward, safer code

^ I'm involved in a few libraries that try to push the limits of the language and force it into being a much more functional language than it feels like it was intended to be

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

^ I'm the maintainer of Runes, a library that introduces functional operators for Optional and Array

---

## Argo ##

```swift
extension User: JSONDecodable {
  static func fromJSON(j: JSON) -> ParserResult<User> {
    return User.create
      <^> j <| "id"
      <*> j <| "first_name"
      <*> j <| "last_name"
      <*> j <|? "email"
  }
}
```

^ I'm also heavily involved with Argo, which uses functional concepts (in this case, Optional's ability to be used as an Applicative) to ease the pain of JSON parsing in Swift

---

![](http://gifsec.com/wp-content/uploads/GIF/2014/03/Stare-What-GIF.gif)

^ This is a totally reasonable response to those last couple of slides

^ I'm not advocating jumping directly to these libraries, but once 

^ Using these libraries introduces a large amount of overhead to the education requirements for your team

---

![](images/winter-is-coming.gif)

^ It's clear (to me) that this is the direction programming is headed

^ These concepts _aren't going away_. You can ignore them for now, but at some point it's going to catch up with you.

^ We need to build up a foundation of knowledge inside the community that we can build upon it and advance as a community

---

## We can do better ##

^ We need to take a step back from the brink and re-think the way we communicate these concepts to newcomers.

^ the operators are just sugar. What's _really_ important are the concepts behind the sugar.

---

## (Re-)Introducing: `map` ##

^ I think that `map` is a good introduction to these concepts, without getting bogged down with operators or funky types

^ you're probably at least a little familiar with how it works, but it's almost certainly a way more powerful concept than you realize

---

## A Quick Intro to Generics ##

^ map works extensively with Generics, so it's worth covering quickly

---

## back in the dark ages... ##

```objc
@interface NSArray : NSObject

- (id)firstObject;
- (id)lastObject;

@end
```

^ Objective-C's type system is fairly weak.

^ We create homogeneous arrays through convention, instead of through our types.

^ Return types for accessed types are `id`, and so need to be cast once they are accessed.

---

## faking it ##

```objc
for (NSString *string in strings) {
  NSLog(@"%@", string);
}
```

^ We can easily fake homogeneous arrays while iterating through a collection (in this case, casting the item to NSString)

^ This _usually_ works, but we can't prove that it will always work.

^ No matter how careful you are, there's always that chance that you end up with a type you didn't expect

^ You're essentially telling the compiler "don't worry, I know what I'm doing, just go with it."

^ This probably works for a lot of you, but I'm an idiot. I don't really trust myself to make sure this never breaks.

^ What we really want is to be able to have these types declared at compile time.

---

```swift
struct Array<Int> {
  var first: Int? { get }
  var last: Int? { get }
}

struct Array<String> {
  var first: String? { get }
  var last: String? { get }
}

struct Array<MyBugProofObject> {
  var first: MyBugProofObject? { get }
  var last: MyBugProofObject? { get }
}
```

^ Swift allows us to associate a type with a contained type

^ This is what we want, but this doesn't scale.

^ This example also doesn't compile (redeclaration of Array)

^ We want to be able to define an Array in a way that lets us define the type that the array contains.

---

## generics ##

```swift
struct Array<T> {
  var first: T? { get }
  var last: T? { get }
}

let stringArray = ["foo", "bar", "baz"] // T == String
let stringArray = [1, 2, 3] // T == Int
```

^ Swift's solution is to parameterize the associated type

^ `T` acts as a stand-in for other types and gets assigned at compile time

^ Adding an object of the wrong type is now a compiler error, and any objects that get accessed from the array are typed automatically.

^ And that's it. Generics are just a way of writing types and functions without specifying concrete types.

---

## `Array.map` ##

^ `map` is most commonly associated with `Array`s because of usage in languages like Ruby.

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

^ If we wanted to create an array of objects by applying a transformation to an existing list, this was all we had.

^ This isn't ideal

^ lose immutability unless we copy the resulting array

^ manually iterating through the list

^ not very reusable

^ `enumerateObjectsUsingBlock` is a little better because you can pass a block, but doesn't go far enough

---

```swift
extension Array<T> {
  func map<U>(transform: T -> U) -> [U]
}
```

^ Here's the type signature for `map` as it's defined on `Array` in the standard lib.

^ Given a list of objects of type T (self) and a function that transforms an object from type T to type U, return a list of objects of type U.

^ Note that `T` is the type contained by the array, while `U` is a generic type specific to this one function.

^ `T` and `U` can be the same type, but they don't have to be

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

## `Result` ##

^ We can also implement our own types that work with `map`

^ In this case, we're going to define a new type called `Result`

---

## Result ##

```swift
enum Result<T> {
  case Success(T)
  case Failure(NSError)
}
```

^ Result is basically Optional on steroids. Instead of just having something or nothing, we can add some context to the error case

---

## error pointers suck ##

```swift
let error: NSError?
let data: NSData = someThingThatReturnsJSONData()

let json: AnyObject? = NSJSONSerialization.JSONObjectWithData(data,
                                            options: nil,
                                            error: &error)

if let err = error {
  // crap, this failed
} else {
  if let j: AnyObject = json {
    // hooray! Now we can do something with j
  }
}
```

---

## a fictional API ##

```swift
let data: NSData = someThingThatReturnsJSONData()
let result: Result<[String: AnyObject]> = NSJSONSerialization.JSONObjectWithData(data,
                                                               options: nil)

switch result {
case let .Success(dict): // Do something with the dictionary
case let .Failure(err): // Do something with the error
}
```

^ Since Result wraps up the context of failure in its type, we only need to check a single value to see what happened

^ Result can carry along its failure information, so if the JSON parsing returned an error, we can get that as well

---

## `Result.map` ##

```swift
enum Result<T> {
  static func map<U>(f: T -> U) -> Result<U> {
    switch self {
    case let .Success(x): return f(x)
    case let .Failure(error): return .Failure(error)
    }
  }
}
```

^ `map` for Result lets us apply a transformation to the success case, or carry along the failure case

^ Since we don't have the syntactic sugar that Optional has with `if let`, we use a `switch` statement directly.

^ This is almost identical to the version of `map` for Optional, but with a little more context

---

## usage ##

```swift
// assuming success is of the type T -> Result<T>

let result = success("foo")
let foobar = result.map { $0 + "bar" } // .Success("foobar")

func appendBar(x: String) -> String {
  return x + "bar"
}

let moreFoobar = result.map(appendBar) // .Success("foobar")
```

^ this should be boring by now

^ As with Optional, `map` conditionally applies a function to the success case.

---

## tracking failures ##

```swift
// assuming failure is of the type String -> Result<T>
let fail: Result<Int> = failure("Something went wrong")

let nope = fail.map(appendBar) // .Failure("Something went wrong")
let stillNope = nope.map(appendBar) // .Failure("Something went wrong")
```

^ The difference with Result is that we get to pull along our failure case

^ Failures persist across uses.

---

## Big Finish ##

- `func map<T, U>(a: Array<T>,    f: T -> U) -> Array<U>`
- `func map<T, U>(a: Optional<T>, f: T -> U) -> Optional<U>`
- `func map<T, U>(a: Result<T>,   f: T -> U) -> Result<U>`

Aw shit now you understand Functors
