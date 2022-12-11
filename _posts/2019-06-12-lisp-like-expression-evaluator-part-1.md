---
title: A Lisp-like Expression Evaluator - Part 1
date: "2019-6-12"
---

I was asked once to write some code that would parse the following string and evaluate it based on a few simple rules.

```
( + ( + 4 ( * 7 8 ) ) ( * 10 31 ) )
```

The grammar for this string can be described like so:

```
expression: "( operator operand operand )"
operator: "*" | "+"
operand: expression | integer
```

An `expression` is comprised of an open paren `(`, a space, an `operator` then an `operand` followed by another `operand`. Finally a space and a closing paren `)`.

An `operator` can be either an asterix `*` or a plus sign `+`.

An `operand` can be either another `expression` or an integer.

---

Here are a two simple examples:
```
( + 2 5)                    = 7
( * ( + 9 3 ) ( + 4 6 ) )   = 120
 ```

Let's also make a few assumptions:
- All characters in the input are separated by spaces.
- All inputs are guaranteed to be valid. (*Whew!*)

### Breaking it Down

Given the assumptions above we have a pretty convenient way in Swift of breaking a String into an array of Strings:

```swift
let expression = "( + 2 5)"
var components = expression.components(separatedBy: " ")

// ["(", "+", "2", "5)"]
```

### Recurse?

Whenever you see a pattern composed of itself you might think of how it can be broken down recursively. In this case an `expression` can be comprised of `expression`s.

Let's write a function that will take our `[String]` and start parsing it. Eventually we will return an `Int` representing the result of the evaluated `expression`:

```swift
func eval(components: inout [String]) -> Int {
    let first = components.removeFirst()

    if first != "(" {
        return Int(first)!
    }
    
    // ... 
}
```
Here we pop the first String off the Array and if it is _not_ an open paren we cast to an integer and return. What we want to identify is our integer base case. A base case is simply a code-path in our recursive function that does not recurse and produces a result, _trivially_. It can sometimes be thought of as the "terminating case". Here we should think of it as having reached a leaf node in our expression tree. Once it is found, simply return the parsed `Int` value.

If `first` is _not_ an open paren, the only thing it could be is an `operator`. So let's remove that next.

```swift
func eval(components: inout [String]) -> Int {
    let first = components.removeFirst()

    if first != "(" {
        return Int(first)!
    }

    let op = components.removeFirst()

    // ... 
}
```

Now that we know we've popped the `operator` the next two things could only be `operand`s. Since an `operand` can be either an `expression` or an `Int` we can start recursing on the `components` array that is now a bit shorter. 

Remember, for the `Int` `operand` case, we've dealt with that first in our function so recursing here makes sense.

```swift
func eval(components: inout [String]) -> Int {
    let first = components.removeFirst()

    if first != "(" {
        return Int(first)!
    }

    let op = components.removeFirst()

    let left = eval(components: &components)
    let right = eval(components: &components)

    // ... 
}
```

What we're doing here is recursing down the "left" side of the state of the `components` String, which will eventually return an `Int` value. 

In the most trivial example `( + 2 5)` left will return `2`.

Immediately we will do the same thing for the right side `operand` and set that value to a variable called `right`.

In the most trivial example `( + 2 5)` right will return `5`.

So far so good?

As a little clean up, after recursing left and right, we'll need to remember our expression syntax and remove the trailing paren:

```swift
func eval(components: inout [String]) -> Int {
  let first = components.removeFirst()

  if first != "(" {
      return Int(first)!
  }

  let op = components.removeFirst()

  let left = eval(components: &components)
  let right = eval(components: &components)
  components.removeFirst() // Remove that last paren!

  // ... 
}
```

### Pause

Let's set a breakpoint and think for a minute. If you're keeping track of the variables in scope for the trivial example you should have the following:

```swift
op = "+"
left = 2
right = 5
```

We got here by:
- popping the `"("`
- popping the `"+"`
- recursing left, which immediately returned `2`
- recursing right, which immediately returned `5`
- popped the `")"`

Does that make sense? If you stop for a second and look at the steps in this list you can see how it can be applied to any expression tree of this type, of any length.

### Simple Math

Now that have an `operator` and two `Int`s we can evaluate the expression and return its result. For that write a new function to apply the `operator` the `operands`s:

```swift
func apply(_ op: String, _ lhs: Int, _ rhs: Int) -> Int {
  if op == "+" {
      return lhs + rhs
  }
  
  return lhs * rhs
}
```

Now, use it in the recursive function and return the result of our applied `operand` at the tail end of the `eval` function.

```swift
func eval(components: inout [String]) -> Int {
  let first = components.removeFirst()

  if first != "(" {
      return Int(first)!
  }

  let op = components.removeFirst()

  let left = eval(components: &components)
  let right = eval(components: &components)
  components.removeFirst()

  // Do the maths
  return apply(op, left, right)
}
```

### Putting It All Together

To make this a usable function we'll need to call `eval` and pass to it the `[String]` and return the result:

```swift
func evaluate(expression: String) -> Int {
    var components = expression.components(separatedBy: " ")
    return eval(components: &components)
}
```

Note, we're mutating the array of `String`'s and so need to pass it by reference. It is not treated as a value type by our function. 

Here are a few simple tests:

```swift
evaluate(expression: "( + 2 5 )")
// 7

evaluate(expression: "( * ( + 9 3 ) ( + 4 6 ) )")
// 120

evaluate(expression: "( + ( + 4 ( * 7 8 ) ) ( * 10 31 ) )")
// 370
```

### Summary

There are probably more Swift-like ways of parsing strings expessions like this and there's probably some overhead with using `String` instead of `Character` but in-all this is an interesting exploration into string parsing.

In [Part 2]({% post_url 2019-06-16-lisp-like-expression-evaluator-part-2 %}), we'll look at using Swift's Enumeration type to make our expression grammar more robust

