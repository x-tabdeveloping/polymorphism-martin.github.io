# Polymorphism

Almost every modern statically typed programming language is capable of type polymorphism, either achieved by inheritence (OOP) or by typeclasses (Scala) or polymorphic types (Haskell).



## What is polymorphism?
Type polymorphism is essentially when a type has subtypes. The values of these subtypes can either be treated  as a value of the subtype or a value of the supertype, which is an incredibly useful feature, when we want to do *generics*.
Polymorphism can also be useful, when structure types have a field which is not necessarily existant.

Okay, I know this sounds insane so let me clarify what this means with some code examples
Let's imagine a scenario, when you'd want to create a function which either excepts an `integer` as its input or a `string`.
There are several solutions to this in the world of programming, one of them being templates : 
```scala
template<T>
fun foo(a : T) {
    //do something
}
```
Templates are handled by the compiler and rely on static dispatch, which we will discuss later. There are two problems with this approach. First, there's absolutely no guarantee that only a `string` or an `integer` could be passed to a function. So what if I passed `Unit`?
```scala
foo(Unit) //wtf is this supposed to do?
```
Well we've got no implementation for `Unit`, so our function's gonna crash, which is bad (as if you couldn't guess). The other problem is that the function has no idea which type is passed to it, so we have to introduce new syntax to do that. But I don't feel like doing it, also because this is a bad approach.

Let's have a look at how you would approach this with static dispatch.
```scala
fun foo(x : Int) {
    //do stuff
}

fun foo(x : String) {
    //do stuff
}
```
Looks much better doesn't it? We're still not quite there though...
This approach could get pretty confusing when different types like `Double` and `Int` have the same literals, and then you would have to introduce new literals...
Also what would be the type of `foo` here? And then you'd have to ask which `foo` am I talking about. I hope you feel that this isn't quite the right solution.

And then there comes polymorphism!! I'll present this with rather an OOP approach (although I'm a huge fan of GADT-s I think it's out of our reach for now)

```scala
type T 
type Str(s : String) : T
type Num(i : Integer) : T

fun foo(x : T) {
    //do stuff
}
```
Now that's a lot better. But still, how would you approach getting out the value from the wrapper here? There are to possible solutions. The first one is pretty old and not really efficient, but we'll have a look at it just for good measure :
```scala
fun foo(x : T) {
    if x is Str then
        print((x as Str).s)
    else
        print((x as Num).i)
}
```
Even in those modern languages which follow this guideline there exists such thing as smart casting, where you don't have to explicit about your castings (e.g. Kotlin)

Let's look at the other solution : ***Pattern matching***
```scala
fun foo(x : T) {
    x match
        case Str(s) -> print(s)
        case Num(i) -> print(i)
}
```

Huh there's a lot going on here, let's break it down.
First thing we do is match on the expression `x`. The first case we have to consider is when `x` is a `Str` containing a variable of type `String` which we'll call `s`. If the match succeeds the `String` contained gets bound to the name `s` in the expression followed by the `->`. Then we do the same thing with the case when we have a `Num`.
Because `T` has only two *subtypes* the pattern matching is *exhaustive* meaning, that it will give us an answer in every case, for every expression of type `T`.

Let's think of a function that depending on which subtype it's gotten returns us a `String`.

```scala
fun foo(x : T) = 
    x match
        case Str(_) -> "Str"
        case Num(_) -> "Num"
```

What's that underscore???
The underscore here means, that the matching is going to discard the value inside `x`, nothing gets bound. As we don't care about what's wrapped in `x` just the type this, is okay, but we have to be explicit even about the variable we don't care about.
Pattern matching is always type safe, because a pattern matching ***has to be exhaustive***.
Even if I don't care about anything else just the `Str`-s, I have to explicitely handle every other case, cause otherwise our function would we partial (which is again a bad thing). Let's have a look at our next example.
```scala
fun foo(x : T) = 
    x match
        case Str(s) -> "Str " ++ s
        case _ -> "Not an Str"
```

As I mentioned earlier the underscore matches in every possible situation, so our pattern matching remains exhaustive
