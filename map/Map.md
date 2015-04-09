![](images/war-room.png)

## How I Learned to Stop Worrying and Love the Functor ##

^ Before Swift came out, I started getting really into Haskell.

^ If you don't know about Haskell, it's a strictly typed, purely functional language.

^ It's probably the best way to learn Functional Programming, so if this talk interests you, I'd recommend checking it out.

^ So when Apple announced Swift in June, I got excited at the possibility to take some of the things I had learned and loved in Haskell, and bring them over to a platform that I love as well.

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

^ The bottom line is that operators are just syntactic sugar, and aren't that important. What's _really_ important are the concepts behind the sugar.

^ I think that `map` is a good introduction to these concepts, without getting bogged down with operators or funky types

^ you're probably at least a little familiar with how it works, but it's almost certainly a more powerful concept than you realize

---

## `Array.map` ##

^ So `map` is probably most commonly associated with `Array`s

^ It's used a lot in languages like Ruby, and I think this makes it a pretty good starting point for us.

^ If you aren't familiar, `Array`'s version of `map` lets you create a new array of objects by applying a transformation to the elements in an existing array.

---

## In the Bad Old Days ##

```objc
NSArray *numbers = @[@1, @2, @3, @4];
NSMutableArray *mutable = [NSMutableArray array];

for (NSNumber *number in numbers) {
    NSNumber *newNumber = @([number integerValue] + 1);
    [mutable addObject: newNumber];
}

NSArray *numbersPlusOne = [mutable copy];
```

^ So thinking back to Objective-C: if we wanted to something like that, this was all we had.

^ (Walk through code)

^ This isn't ideal for a number of reasons

^ You lose immutability unless we manually copy the resulting array

^ You have to manage the iteration yourself

^ And this isn't really very reusable

^ `enumerateObjectsUsingBlock` is a little better, but doesn't really go far enough

^ So how does Swift solve this problem?

---

```swift
extension Array<T> {
  func map<U>(transform: T -> U) -> [U] {
    var array: [U] = []

    for x in self {
      let y = transform(x)
      array.append(y)
    }

    return array
  }
}
```

^ So here's a possible implementation for `Array.map`, which is included in the standard lib.

^ You can see that `map` is really just a simple abstraction of that same iteration pattern we used to use in Objective-C

^ (Walk through code)

^ So why is this interesting?

---

## Usage ##

```swift
let numbers = [1, 2, 3, 4]
let numbersPlusOne = numbers.map { $0 + 1 } // [2, 3, 4, 5]
```

^ For one, moving that abstraction up a level leads to much cleaner code

^ You no longer have to manage the details of the iteration at this level, and can instead focus on the transformation, which is really the important part of what you're trying to do.

^ This is much more declarative, and much less procedural. That makes this code easier to reason about; there are fewer moving parts.

^ You also maintain immutability. You can assign the result directly to `let` and now you have your resulting array as a local constant.

---

## Bonus ##

```swift
func addOne(x: Int) -> Int {
  return x + 1
}

let numbers = [1, 2, 3, 4]
let moreNumbers = numbers.map(addOne) // [2, 3, 4, 5]
```

^ As an added bonus, because Swift functions are closures and Swift closures are functions, you can easily wrap that transformation up in a reusable function, or an instance method, and just pass it directly to `map`.

^ This means that your code is even _more_ reusable.

---

## Real World Usage ##

```swift
struct User {
  let name: String
  let email: String
}

let mcNulty = User(name: "Jimmy McNulty", email: "mcnulty@bpd.gov")
let theBunk = User(name: "Bunk Moreland", email: "thebunk@bpd.gov")

let users = [mcNulty, theBunk]
let emails = users.map { $0.email }

sendEmails(emails)
```

^ So how is this useful in the real world?

^ So in this case, we've got some users, Jimmy and Bunk, and we want to send them emails.

^ `map` is able to make quick work of this by letting us map a transformation (in this case, simply accessing a property) over the original array.

^ That leaves us with a simple list of email addresses that we can pass on down the line.

---

## `Optional.map` ##

^ So Swift also introduces `map` for Optional values

^ This is much less common. Concept is taken from more functional languages such as Haskell.

^ `map` for Optional applies a function to a value if it exists, and returns `.None` if it doesn't.

---

## Optional ##

```swift
enum Optional<T> {
  case .Some(T)
  case .None
}
```

^ This is what Optional looks like

^ Optionals encapsulate the concept of a value's existence. It replaces `nil` in Objective-C

^ Can't use the value directly. It forces you to deal with the possibility of there not being a value

^ That can be really painful for people if they aren't used to this idea.

---

## Conditional Unwrapping ##

```swift
let myOptional: String? = "Hello World"

if let string = myOptional {
  println(string)
}
```

^ The most common way of dealing with this is with conditional unwrapping, also known as `if let` statements.

^ This is the most basic way to pull the value out of an optional.

^ `if let`s check for the presence of a value, and assigns that value to a local constant if it exists. You can then use that constant inside the scope of the conditional

^ This is a really nice piece of syntax to have in the standard lib, but it introduces some weird things to have to consider, like potentially shadowing local variables and that nesting tree of doom, which is somewhat alleviated in Swift 1.2.

^ Optional's version of `map` can streamline things.

---

## `Optional.map` ##

```swift
extension Optional<T> {
  func map<U>(transformation: T -> U) -> U? {
    if let x = self {
      return transformation(x)
    } else {
      return .None
    }
  }
}
```

^ This implementation should look familiar

^ `map` can be implemented as a simple wrapper around `if let`

^ (walk through code)

^ This acts as a way to separate the concern of optionality and isolate it to this function, the same way that `map` for Array separates the concern of iteration

---

## Usage ##

```swift
let optional: String? = "foo"
let foobar = optional.map { $0 + "bar" } // .Some("foobar")

let otherOptional: String? = .None
let nope = optional.map { $0 + "bar" } // .None
```

^ So for example, using `map`, we can easily append something to this string if it exists, but if it _doesn't_, it essentially becomes a noop, the same way we'd expect `nil` to act in Objective-C

---

## Usage ##

```swift
func appendBar(x: String) -> String {
  return x + "bar"
}

let optional = "foo"
let moreFoobar = optional.map(appendBar) // .Some("foobar")
```

^ And again, Swift functions are closures, and vice versa, so we can use named functions with `map` directly.

^ Notice that `appendBar` gets to deal with non-optional values.

^ Using Optional's version of `map` means you get to keep your core functions pure.

---

## Real World Usage ##

```swift
let name = json["name"] // String?
let user = name.map { User(name: $0) } // User?
let company = user.map { Company(employees: [$0]) } // Company?

let numberOfEmployees = company?.numberOfEmployees ?? 0 // Int
```

^ So for example, the fact that dictionary subscripting returns an Optional value can easily make things messy

^ (Walk through code)

^ With `map` we can carry the fact that we might not have a value through the implementation of the function, and only worry about dealing with that failure when we actually need to return something

---

![](https://desperateandunrehearsed.files.wordpress.com/2015/02/whats-your-point.gif)

^ So what's going on here?

^ Why are both of these functions named `map` when they seem to do totally different things?

---

## Spot the Difference ##

```swift
// Array
func map<T, U>(x: [T], f: T -> U) -> [U]

// Optional
func map<T, U>(x: T?, f: T -> U) -> U?
```

^ These are the same two functions we've been talking about, these are just the free function versions.

^ They work exactly the same, don't worry too much about it, I just want to be able to look at at the full type signatures.

^ They look similar, right?

^ Both functions, for example, take a function from `T` to `U` as an argument

^ They also both take _something_ to do with a value of the type `T` and return something to do with a value of the type `U`

^ But they aren't quite the same, there's something different, it's just hard to tell _exactly_ what that difference is.

---

## Spot the Difference ##

```swift
// Array
func map<T, U>(x: Array<T>, f: T -> U) -> Array<U>

// Optional
func map<T, U>(x: Optional<T>, f: T -> U) -> Optional<U>
```

^ If we remove the syntactic sugar around the types, the picture starts to clear up a bit

^ Array and Optional start to look like a wrapper type for the generic types `T` and `U`

---

## Contextual Types ##

```swift
func map<T, U>(x:    Array<T>, f: T -> U) ->    Array<U>
func map<T, U>(x: Optional<T>, f: T -> U) -> Optional<U>
```

^ In fact, we can think of these types as contexts for their contained types

^ In the case of Array, we're working with the context of having multiple values. So when we use `map` with array, we apply the function to every possible value.

^ And for Optional, we're working with the context of a value's presence. So we apply the function if it's there, and return `.None` if it isn't.

^ But there are more contexts that seem like they will fit this pattern.

---

## Contextual Types ##

```swift
func map<T, U>(x:    Array<T>, f: T -> U) ->    Array<U>
func map<T, U>(x: Optional<T>, f: T -> U) -> Optional<U>
func map<T, U>(x:   Result<T>, f: T -> U) ->   Result<U>
```

^ Result can represent the context of a failable operation.

^ Result would have `Success` and `Failure` the same way Optional has `Some` and `None`

^ When we use `map` with Result, we apply the function to the value if the result was successful, and pass along the failure if it wasn't.

---

## Contextual Types ##

```swift
func map<T, U>(x:    Array<T>, f: T -> U) ->    Array<U>
func map<T, U>(x: Optional<T>, f: T -> U) -> Optional<U>
func map<T, U>(x:   Result<T>, f: T -> U) ->   Result<U>
func map<T, U>(x:   Future<T>, f: T -> U) ->   Future<U>
```

^ We could also represent the context of a value we don't have yet, using Future

^ So if we `map` over a `Future` value, we apply the function to the value when we get it.

---

## Contextual Types ##

```swift
func map<T, U>(x:    Array<T>, f: T -> U) ->    Array<U>
func map<T, U>(x: Optional<T>, f: T -> U) -> Optional<U>
func map<T, U>(x:   Result<T>, f: T -> U) ->   Result<U>
func map<T, U>(x:   Future<T>, f: T -> U) ->   Future<U>
func map<T, U>(x:   Signal<T>, f: T -> U) ->   Signal<U>
```

^ Or maybe we want to represent the context of a value that changes over time, in which case we might look at Signal

^ Mapping over a Signal of values leaves us with a new signal containing the transformed values.

---

![](http://www.reactiongifs.us/wp-content/uploads/2014/05/everybody_got_that_spaceballs.gif)

^ So what if we could generalize that contextual type into something more reusable.

^ If we could create some kind of protocol that all of these different types could conform to, we'd be able reduce all of those free `map` functions down into a single implementation.

---

## What if... ##

```swift
protocol Context<T> {
  func map<U>(f: T -> U) -> Self<U>
}

func map<C: Context, T, U>(x: C<T>, f: T -> U) -> C<U> {
  return x.map(f)
}
```

^ We might end up with something like this

^ Note that this doesn't actually compile because of limitations in the language

^ In this theoretical implementation, we're declaring a protocol that has a contained type

^ The only thing that this protocol requires is that conforming types implement `map`, which has the same basic type signature we've already seen.

^ This level of abstraction would lead to a whole bunch of new generalizations that would lead to safer, cleaner code.

---

## Hello {{NAME}} ##

```swift
func sayHello(names: Array<String>) -> Array<String> {
  return names.map { "Hello \($0)!" }
}
```

^ For example, we could easily write this function that says hello to a list of people

---

## Hello {{NAME}} ##

```swift
func sayHello(name: Optional<String>) -> Optional<String> {
  return name.map { "Hello \($0)!" }
}
```

^ And then we could reimplement it to say hello to an optional person

---

## Hello {{NAME}} ##

```swift
func sayHello(name: Result<String>) -> Result<String> {
  return name.map { "Hello \($0)!" }
}
```

^ And then we could reimplement it again to say hello to a Result person

---

## Hello {{NAME}} ##

```swift
func sayHello(name: Future<String>) -> Future<String> {
  return name.map { "Hello \($0)!" }
}
```

^ And again for a future person

---

## Hello {{NAME}} ##

```swift
func sayHello(name: Signal<String>) -> Signal<String> {
  return name.map { "Hello \($0)!" }
}
```

^ and yet again for a signal of people

---

## Hello {{NAME}} ##

```swift
func sayHello<C: Context>(name: C<String>) -> C<String> {
  return name.map { "Hello \($0)!" }
}
```

^ _or_ we could just write it once. We could write a function that works with all of those types we already talked about, as well as any type we might come up with in the future

^ I want you to note, as well, that because the only thing we know about this `name` value is that it's a context that holds a string, the _only_ thing we can possibly do with it is map a function over it. That's a level of safety that you don't have in any of these other implementations.

---

## What if... ##

```swift
protocol Context<T> {
  func map<U>(f: T -> U) -> Self<U>
}

func map<C: Context, T, U>(x: C<T>, f: T -> U) -> C<U> {
  return x.map(f)
}
```

^ So that's the kind of thing a type like this would allow us to do

---

## Turns out... ##

```swift
protocol Functor<T> {
  func map<U>(f: T -> U) -> Self<U>
}

func map<F: Functor, T, U>(x: F<T>, f: T -> U) -> F<U> {
  return x.map(f)
}
```

^ As it turns out, this type already has a name: Functor.

^ Functor is one of those scary words from Functional Programming that I'd be willing to bet you've seen fly by on Twitter or StackOverflow

^ It's probably been involved in some obscure metaphor trying to relate it to fish, or a burrito, or something.

^ Functor is nothing to be afraid of, and as I hope you've gotten a glimpse of, is a fairly straightforward concept.

^ A Functor is simply a contextual type that responds to `map`. It's really as simple as that.

---

![](http://27.media.tumblr.com/tumblr_lex3s2CgQN1qe0eclo1_r9_500.gif)

^ I wanted to close out by saying that Swift is a language in flux, and we're shaping its future through our usage.

^ the recent introduction of `flatMap` to the standard lib, which is like a beefed up version of `map` really reinforces my thoughts that we're headed for a _more_ functional future, not less.

^ I'm not saying that using `map` and other functional programming concepts are the end all solution for everything, but they are tools that are available for you, and avoiding them entirely because they come from a different idiom seems short sighted.

---

# Gordon Fontenot #
## @gfontenot ##
## thoughtbot.com ##
## buildphase.fm ##

^ So that's it.

^ You can find me most places as gfontenot

^ Quick plug for thoughtbot. We actually offer coaching for stuff like this.

^ I do a podcast where you can hear me ramble about this and baseball.

