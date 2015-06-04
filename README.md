# Conversions, reductions, and abstractions in lambda calculus

These are the notes for a presentation I gave at [LVFPUG](1) on 2015-06-04.

* [Why](#Why)
* Reduction
* Abstraction
* Conversion
* Brief introduction to lambda calculus terms
* Alpha conversion
* Eta conversion
* Beta reduction
* Lambda abstraction

# Why

Lambda calculus is the foundation of many functional programming
languages. Understanding it is helpful in simplifying terms and
understanding function application and program evaluation.

Sometimes terms like eta-reduction are used by people when refactoring
functional programming languages like Haskell. It would be nice to
know what they mean.

# Reduction

Some operation you apply to an term that makes it simpler.

# Abstraction

Some operation you apply to an term that makes it more complex.

# Conversion

Either a reduction, abstraction, or an operation that maintains the
complexity of the term.

# Brief introduction to lambda calculus terms

We will use Haskell syntax. Lambda is represented by a
backslash. `->` separates the arguments to the lambda from the body of
the lambda. Variables by letters starting with a lowercase
letter. Application is done by a space between variables.

Here is the identity lambda term:

```haskell
    \ x -> x
```

It returns what is given to it.

```haskell
    \ x -> x y
```

`x` is a "bound" variable in this term. The x in the body of the
lambda is bound to the x in the argument list of the lambda. y is
called "free" in this term. It is not referenced by any part of
the term.

```haskell
   (\ x -> x y) (\ y -> y + y)
```

Depending on what part you look at y is both bound and free. It is
bound insid ehte right lambda, but it is free for the whole
term. Here is how to see that by application:

```haskell
    (\ x -> x y) (\ y -> y + y)
    -- Application of outer
    (\ y -> y + y) y
```

The outside `y` is not defined anywhere.

# Alpha conversion

Replacing a bound variable name. Example:

```haskell
    (\ y -> y + y) y
    -- Alpha conversion of inner y
    (\ t -> t + t) y
```

In this case, it helps make it more clear which variables are free and
bound. Also, this is the theoretical foundation for why you can rename
variables.

Alpha conversion cannot be done on a free variable.

```haskell
    (\ y -> y + y) y
    -- Alpha conversion of outer y
    (\ y -> y + y) t
```

In this case, the term is presumed to live inside of a bigger term or
"environment" that has bound `y`. If you just rename it at this
level, it will no longer be referencing that environment.

It also isn't applied inward arbitrarily:

```haskell
    (\ y -> y + (\ y -> y)) y
    -- Alpha conversion of inner y
    (\ t -> t + (\ y -> y)) y
```

The most inner lambda does not get renamed.

You have to be careful to not do it to a variable already being used:

```haskell
    (\ x -> (\ y -> x))
    -- Alpha conversion of x to y changes meaning
    (\ y -> (\ y -> y))
```

It changes the meaning of the function. It was a function that
returned a constant function of x, but it became a function that
returned an identity function.

Fix by renaming with a unique variable or alpha converting the inner
part first.

```haskell
    -- unique u
    (\ x -> (\ y -> x))
    -- Alpha conversion of x to u does not change meaning
    (\ u -> (\ y -> u))
    -- or alpha conversion of inner first
    (\ x -> (\ t -> x))
    (\ y -> (\ t -> y))
```

# Eta conversion

This is probably the most common conversion I have seen when people
talk about Haskell refactoring. Usually people do eta reductions to
simplify a function definition or to turn it into a point free form.

For example:

```haskell
    adder :: Int -> Int -> Int
    adder x y = x + y

    -- infix to prefix
    adder x y = (+) x y

    -- eta reduction
    adder x = (+) x

    -- eta reduction again
    adder = (+)
```

With lambdas eta conversions look like this:

```haskell
    (\y -> (\x -> f x y))
    --eta reduction
    (\x -> f x)
    -- eta reduction
    f
    -- eta abstraction
    (\x -> f x)
    -- eta abstraction again
    (\y -> (\x -> f x y))
```

This conversion cannot be made if `x` is free within `f`. For example:

```haskell
    let f = (\y -> x)

    let before = (\x -> (\y -> x) x)

    -- eta reduction
    let after = (\y -> x)

    -- before z = z
    -- after z = x
```

This does not really happen in the common case where people use eta
reduction in  Haskell refactorings. The simple procedure in Haskell is

1. Get the argument you want to eta reduce to the right side of the
   argument list.
2. Manipulate the right hand side of the function definition so the
   argument only appears at the rightmost position.
3. eta reduce and repeat as desired.

Example:

```Haskell
    pluser :: [Int] -> Int -> [Int]
    pluser l plusBy = map (+ plusBy) l

    -- Option 1: Change interface of function
    pluser :: Int -> [Int] -> [Int]
    pluser plusBy l = map (+ plusBy) l
    -- Then eta reduction without right hand side manipulation
    pluser plusBy = map (+ plusBy)
    -- Continuing we need to extract out plusBy
    pluser plusBy = (map . flip (+)) plusBy
    -- eta reduction
    pluser = map . flip (+)

    -- Option 2: Keep interface of function
    pluser :: [Int] -> Int -> [Int]
    pluser l plusBy = map (+ plusBy) l

    -- Manipulate right hand side
    pluser l plusBy = (map . flip (+)) plusBy l
    -- Manipulate right hand side
    pluser l plusBy = (flip (map . flip (+))) l plusBy
    -- eta reduce
    pluser l = (flip (map . flip (+))) l
    -- eta reduce
    pluser = (flip (map . flip (+)))
```

Refactoring to pointfree solutions sometimes is nice because you do
not have to name variables, but in other cases, it can make it harder
to understand the function. The trivial eta reductions are almost
always done in Haskell. It makes the code shorter and requires one
less name for each trivial reduction.

[Blunt](2) is a helpful tool for discovering pointfree forms.

# Beta reduction

Beta reduction is the more formal name for application. Square
bracket and `:=` operator are introduced to help show how the
reduction proceeds.

Example:

```haskell
    (\ x -> x + x) t
    -- replace variable x with term t
    -- x + x [ x := t ]
    -- finish reduction
    t + t
```

Another example:

```haskell
    (\ x -> x) y
    -- beta reduction: x [ x := y ]
    y
```

Just like normal computation beta reduction is not guaranteed to stop:

```haskell
    (\ x -> x x) (\ x -> x x)
    -- beta reduction x x [ x := (\ x -> x x) ]
    (\ x -> x x) (\ x -> x x)
    -- Same as before!
```

# Lambda abstraction

We've been using lambda abstractions all along. It is just another
name for lambda or anonymous functions. It is used if you want to
introduce new variables to an expression.

Example:

```haskell
    x
    -- lambda abstraction from constant term to constant function
    \ y -> x
    -- lambda abstraction from constant function to constant function
    -- function
    \ z -> (\ y -> x)
```

This abstraction is also useful when you want to inject parameterized
information into a computation. It is what you are directly doing when
you add an argument to the left hand side of a function in Haskell:

```haskell

    addOne :: Int -> Int
    addOne x = x + 1

    --lambda abstraction
    addOneOrTwo :: Bool -> Int -> Int
    addOneOrTwo b x = x + 1

    --use introduced variable
    addOneOrTwo b x =
        if b then
            x + 1
        else
            x + 2
```

[1]: http://www.meetup.com/las-vegas-functional-programming/
[2]: https://blunt.herokuapp.com/
