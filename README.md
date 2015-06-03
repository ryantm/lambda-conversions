# Conversions, reductions, and abstractions in lambda calculus

These are the notes for a presentation I gave at (LVFPUG)[1] on 2015-06-04.

* (Why)[#Why]
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
languages. Understanding it is helpful in simplifying expressions and
understanding function application and program evaluation.

# Reduction

Some operation you apply to an expression that makes it simpler.

# Abstraction

Some operation you apply to an expression that makes it more complex.

# Conversion

Either a reduction, abstraction, or an operation that maintains the
complexity of the expression.

# Brief introduction to lambda calculus terms

We will use Haskell syntax. Lambda is represented by a
backslash. `->` separates the arguments to the lambda from the body of
the lambda. Variables by letters starting with a lowercase
letter. Application is done by a space between variables.

Here is the identity lambda expression:

```haskell
    \ x -> x
```

It returns what is given to it.

```haskell
    \ x -> x y
```

`x` is a "bound" variable in this expression. The x in the body of the
lambda is bound to the x in the argument list of the lambda. y is
called "free" in this expression. It is not referenced by any part of
the expression.

```haskell
   (\ x -> x y) (\ y -> y + y)
```

Depending on what part you look at y is both bound and free. It is
bound insid ehte right lambda, but it is free for the whole
expression. Here is how to see that by application:

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

In this case, the expression is presumed to live inside of a bigger
expression or "environment" that knows about `y`. If you just rename
it at this level, it will no longer be referencing that environment.

# Eta conversion
# Beta reduction
# Lambda abstraction


1: http://www.meetup.com/las-vegas-functional-programming/
