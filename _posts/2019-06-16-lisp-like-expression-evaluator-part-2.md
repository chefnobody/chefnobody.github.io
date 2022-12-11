---
title: A Lisp-like Expression Evaluator - Part 2
date: "2019-6-16"
---

### [Previously]({% post_url 2019-06-12-lisp-like-expression-evaluator-part-1 %}) ...

We built a pretty trivial expression parser for prefix notation mathematic operations.

For example:
```
( + ( + 4 ( * 7 8 ) ) ( * 10 31 ) )

// 370
```

We looked at parsing a `String` into an `[String]` and recursively pulling it apart. There are a few `String` comparisions and `Int` casting which can sometimes be a little error prone, but it works.

### Recursive Enumerations in Swift

The other day I was browsing the Swift Language docs and re-read through the [Enumeration](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html) type. At the bottom there is a section under the Associated Types named Recursive Enumerations. A Recursive Enumeration is defined as an Enumeration that has another instance of the Enumeration as an associated value for one or more of the cases. The example they give stuck out:

```swift
indirect enum ArithmeticExpression {
    case integer(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}
```

In [Part 1]({% post_url 2019-06-12-lisp-like-expression-evaluator-part-1 %}), the expression parsing code handled all three cases as, but we just used the `String` type to encode each expression when we could have leaned on the Swift's Type system a bit more. 

With the `ArithmeticExpression` Type we can reduce some complexity around how we apply expressions together. We can instead use an instance of this Type and switch over its possible cases. This should make the recursion feel much more safe and perhaps easier to reason about.

Here is a new version of the `apply` function we wrote. Let's call it `eval` like the documentation, we'll also change the `integer` case to be `operand` to align with the expression syntax:

```swift
func eval(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .operand(value):
        return value
    case let .addition(left, right):
        return eval(left) + eval(right)
    case let .multiplication(left, right):
        return eval(left) * eval(right)
    }
}
```

### Refactor!

Let's use the `ArithmeticExpression` type from the docs and make a few simple changes to our code to use this type instead.

Before we can evaluate an `ArithmeticExpression` we need to convert the `[String]` to a single `ArithmeticExpression`. Converting information from one format to another is called encoding. Let's write an `encode` function:

```swift
func encode(_ components: inout [String]) -> ArithmeticExpression {
    // ...
}
```

This looks quite a bit like the signature of our original `eval` from Part 1. However instead of an `Int`, we'll return the concrete Enumeration Type. The shape of this function will look nearly the same as our original recursive function however, except for the bit at the end.

After the function determines the `operator` and the two `operand`s, what we want to do is create an instance of the `ArithmeticExpression` corresponding to the `operator`:

```swift
switch op {
case "+":
    return ArithmeticExpression.addition(left, right)
case "*":
    return ArithmeticExpression.multiplication(left, right)
default:
    throw ArithmeticExpressionError
            .encodingError(
                "Operator: \(op) not supported!"
            )
}
```

Here, we can cleanly encode the expression into concrete a Type and throw a sensible error in the event the parsing finds an operator that is not supported. Here is our encoder:

```swift
// Converts an array of Strings to a single ArithmeticExpression
func encode(_ components: inout [String]) throws -> ArithmeticExpression {
    let first = components.removeFirst()
    if first != "(" {
        print(first)
        return ArithmeticExpression.operand(Int(first)!)
    }
    
    let op = components.removeFirst()    
    let left = try encode(&components)
    let right = try encode(&components)
    components.removeFirst()

    switch op {
    case "+":
        return ArithmeticExpression.addition(left, right)
    case "*":
        return ArithmeticExpression.multiplication(left, right)
    default:
        throw ArithmeticExpressionError
                .encodingError(
                    "Operator: \(op) not supported!"
                )
    }
}
```

Finally, our helper function should try to encode the `[String]` and then return the result of the eval'd `ArithmeticExpression`. 

```swift
func evaluate(expression: String) -> Int {
    var components = expression.components(separatedBy: " ")
    let arithmeticExpression = try! encode(&components)
    return eval(arithmeticExpression)
}
```

### Trade Offs

Sure, this looks nicer and maybe feels more Swift-like but we did make some tradeoffs.

### More Code
There is more code this time which usually means more room for bugs. I would argue the additional code, the `ArithmeticExpression` type and its `eval`uation function makes our parsing a bit more robust. Safer, even. We moved logic out of our code and moved it closer to the Type itself. We also have a clean place to throw an error.

### Two Recursive Passes?
I wonder now, after looking at this refactor are we recursing twice? Once through the entire `String` expression and then a second time through the `ArithmeticExpression` object graph? It appears so. The object graph (I think) are just pointers in memory, so I'm going to hazard a guess that's a constant time complexity. 

Maybe that's not awesome for really a large expression, though?

### Safety
Think about the previous implementation and how you would extend it to include additional operations like subtraction `-` or maybe division `%` (🧐). That `apply` function we wrote suddely gets more complicated. Now consider this complexity as an extension to our Type. We would need to add a new case or two to our Type and then update the `switch` statement to create the new cases of our Enumeration and then lastly the `eval` to perform the proper math. 

That code feels safer to me because it is isolated to the Type itself and it not tightly coupled with the program's logic. Extending the Type feels more safe to me.

### Summary

Swift's Enumerations are really robust and can help make code cleaner-looking and easier to reason about.

Cheers 🍺!

### Resources
- [Full source code](https://gist.github.com/chefnobody/e1189b8c1e2788065cc0632afd44b8f3) 
