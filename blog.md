
# Advent of Code with Cats

## Day1 - Part 1

### Functional Solution

Even though the first part of the Day1 problem can be easily solved in one line of code, we purposely break it down into multiple small functions so that we can demonstrate how to apply type classes to the problem later. We first write a `fuel` function that takes in the mass of the module, divides it by three and subtracts two to find the fuel required to launch the module. 

```scala
def fuel(mass: Int): Int = mass / 3 - 2
```

In the problem, your puzzle input is a long list of all the modules on your spacecraft. To find the fuel required for each module, you could apply the `fuel` function to each module of the long list by using the `map` function. This is what we’ve done by writing a `calculateFuels` function. 

```scala
def calculateFuels(masses: List[Int]): List[Int] = masses.map(fuel)
``` 

Next, to sum up the list of fuels, we make use of the `sum` function available to `Iterable`, like `List` in our case, and write a `sumFuels` function. Finally, we put all of these small functions into a `sumOfFuel` function and solve our Part 1’s problem.

```scala
def sumFuels(fuels: List[Int]): Int = fuels.sum
``` 

The solution of Part 1 can be found in the [day-1-step-1-abstraction](https://github.com/lsug/advent-of-code-typelevel/tree/day-1-step-1-abstraction) branch. 

### Type Abstraction

Before jumping into type classes, we thought it’d be good to start by introducing generic functions and types. Have a look at the three functions below:

```scala
def returnInt(x : Int): Int = x
def returnLong(x : Long): Long = x
def returnBoolean(x : Boolean): Boolean = x
```


`returnInt` is a function that accepts and returns a value of type `Int`, whereas `returnLong` accepts and returns a value of type `Long` and finally `returnBoolean`, a value of type `Boolean`.

Although they each return a different **type** of value, they share the same pattern: return the same value that was used as its argument. Since they do the same thing fundamentally, we could write a single function that does that. In other words, we could abstract over this pattern.

```scala
def identity[A](a: A): A = a
```

The `identity[A]` function is such a function, and is called a **generic function**. The `[A]` is the **type parameter**, the syntax for a generic function in Scala. When we are generalizing, we tend to use letters `A`, `B` and `C` by convention.

Now, to return an `Int`, we can pass in an `Int` value to the `identity` function like this:

```scala
identity[Int](10)`
```

When you run this, the output will be a value of 10.  You could also omit the `[Int]` type parameter and get the same result:

```scala
identity(10)
```

By the same logic, to return a `true` value, we simply write `identity[Boolean](true)` or just `identity(true)`.

You could have as many type parameters, also known as generic types, in a function as you like. For example, there are three different types in `tuple3[A, B, C]` below:

```scala
def tuple3[A, B, C](a: A, b: B, c: C): (A, B, C) = (a, b, c)
```

### So, what is a type? 

You can think of a type as a set of values. For concrete types like `Boolean`, we know it is a set with just two values: `true` and `false`.(draw circle) 
Earlier, we mentioned that `A` is a generic type which means we don’t know anything about the possible values that lie in set `A`. (draw circle)

You might be thinking, "Why is having a generic type in a function a good thing?". Let’s go back to the `identity[A]` function.  Remember how it can take in and return an `Int`, `Long` and `Boolean`. Actually, it can do more than that. It can take in and return **any** type of value due to having a generic type. Compared it to `returnInt`, which can only take in and return a concrete `Int` value, `identity[A]` is a lot more flexible and powerful.

In short, by having a generic function, we can reduce code duplication and achieve polymorphism.

In [day-1-step-1-abstraction](https://github.com/lsug/advent-of-code-typelevel/tree/day-1-step-1-abstraction) branch, there is an exercise called *Abstraction*. Please try to make every function generic, and see if you can pass all the tests.

### Type Classes and Type Class Instance

We’ve seen both types and generic functions, let’s dive into type classes. Before bombarding you with lots of definitions of what a type class is, we thought it would be good to give you a break and go back to the spacecraft. Remember the `fuel` function, let’s try to rewrite the function and make it generic.

```scala
def fuel[A](mass: A): A = mass / 3 - 2
```

If you do that, you’ll notice that the function does not compile. The error message is:
 
```text
value / is not a member of type parameter A
```

What the error message is saying is the compiler cannot find the `/`, division operation under type `A`. Previously, when the mass parameter was a concrete type `Int`, the function compiled because `/` is defined under type `Int`. Actually, it’s not only value `/` undefined, `-` (minus), `3` and `2` are also undefined under type `A`.

To make the generic `fuel[A]` function compile, first, we need to define all of these functions. We do that in a trait `Mass[A]`:

```scala
trait Mass[A] {
    def quotient(a: A, b: A): A
    def minus(a: A, b: A): A
    def two: A
    def three: A
  }
```
> *(quotient is the division of two numbers, but discards the fractional part.*) 

Now, we have the missing functions defined in `Mass[A]`. Next, we need to provide a way for `fuel[A]` to access these functions and we do that by adding another parameter to `fuel[A]`:

```scala
def fuel[A](mass: A)(M: Mass[A]): A = ???
```
Please try and implement the function above before reading on. 

The implementation of `fuel[A]` will look something like this, and it will compile:

```scala
def fuel[A](mass: A)(M: Mass[A]): A = {
    M.minus(M.quotient(mass, M.three), M.two)
  }
``` 

Job done? Not quite. Say you were to find the fuel for a mass of 12, how would you do that? How about this `fuel[Int](12)`? If you run that, you’ll get an error message.

```text
error: missing argument list for method fuel...
```

Remember earlier we added another parameter of type `Mass[A]` to `fuel[A]`.  We need to provide this parameter to the function. Because we are replacing type `A` with type `Int`, we need to provide a value of type `Mass[Int]`. However, where do we find that value from? It turns out we need to do a bit of DIY and construct a value of type `Mass[Int]` ourselves! 

We create an instance of the `Mass[A]` trait.  We use the `new` keyword to create a new instance, and replace generic type `A` with the type we care about, in our case `Int`, to construct a concrete value of `Mass[Int]`. Finally, we implement all the functions in `Mass` for type `Int`.

```scala
val intHasMass: Mass[Int] = new Mass[Int] {
      def quotient(a: Int, b: Int): Int = a / b
      def minus(a: Int, b: Int): Int = a - b
      def two: Int = 2
      def three: Int = 3
    }
``` 

Now, if you run `fuel[Int](12)(intHasMass)`, you should get a value of 2. There you have it, you have just learned to implement and use a type class `Mass[A]` and a type class instance `intHasMass` to solve the problem.

#### Implicits

Earlier when we tried to run `fuel[Int](12)`, we received a "missing parameter" error message.  We had to provide `intHasMass` to the function in order to get it to work.

Actually, there is a way to make this call work without needing to provide `intHasMass`.  We can add an `implicit` keyword to the parameter list of `fuel[A]` and before `intHasMass`.

```scala
def fuel[A](mass: A)(implicit M: Mass[A]): A = ...
implicit val intHasMass: Mass[Int] = new Mass[Int] {..}
```

By doing this, the compiler will look for a value with type `Mass[Int]` and pass in the value marked with `implicit val` every time we make a call to `fuel[Int]`. Because we put the implicit type class instance in a `Mass` object, we need to import the object with `import Mass._`.  This brings the implicit value into scope so that the compiler can find it. 

The codes of this section can be found in [day-1-step-2](https://github.com/lsug/advent-of-code-typelevel/blob/day-1-step-2/src/main/scala/advent/solutions/Day1.scala) branch. 

#### What is a type class?

A type class is a group of functions associated with a type.  `Mass[A]` is a type class with the functions `quotient`, `minus`, `two` and `three`.  By using a group of functions instead of a concrete type, we can write more generic code.  The `fuel` function can be used for any type `A` that has a `Mass` type class.  This is known as **polymorphism**:

```scala
def fuel[A](mass: A)(M: Mass[A]): A = {
    M.minus(M.quotient(mass, M.three), M.two)
  }
``` 

An instance of a type class is an implementation of the type class functions for a specific type.  `intHasMass` is an implementation of the `Mass[A]` typeclass for the type `Int`.

### Cats

#### Monoid
