--- 
title: Applicatives
customdate: 2022 Aug 29 (月) 08:34AM
---
 
I have recently stumbled on an idea that I think is quite beautiful.
It has to do with functional programming in Haskell.

### Functors

First of all, let's recall what is a functor in haskell.
It's an interface that allows you to abstract traversing a data structure while
applying some kind of function to its elements.

We do that by defining a function `fmap` on our type `f`

``` haskell
fmap :: (a -> b) -> f a -> f b
```

For example, by defining the type constructor `[]` to be an instance of functor
(there is a built-in function `map`, though) you can separate traversing the
list and applying the function to each element of the list.

Other examples can include type constructors such as:
- trees 
- maybies
- results
- IO
  
  The case of IO is interesting because it is not a data structure in a
  conventional sense, but Haskell allows us to work with it via the do notation
  so it's more or less the same thing.

#### Notice that functor takes just one input

If we wanted to, for example, add two results together, we would have to define
a new function 

``` haskell
fmap2 :: (a -> b -> c) -> f a -> f b -> f c
```

And if there are more arguments, we would need to define `fmap3, 4, ..., ω`.  
Not really nice, expecially cosidering the case of generic programming that
functional paradigm encourages.

#### The question is how would the signature of such a type look like?

It would need to have arbitrary number of arguments. This is personally what I
find so fascinating about it.  
This is something I could not figure it out at first.

### Applicatives let us abstract this process using the most natural thing: currying

The interface is as follows:

``` haskell
pure :: g -> f g
<*>  :: f (a -> b) -> f a -> f b
```

`pure` is used to start the process and `<*>` is an extended function application.

It's used like so:

``` haskell
pure fun <*> arg1 <*> arg2
```

So we munch `arg1`, `arg2` and so on part by part, 
each time ending up with a similar construction. 

``` haskell
pure fun :: f (a -> b -> c)
--            ---  -------
--             A      B

pure fun <*> arg1 ::  f (b -> c)
pure fun <*> arg1 <*> arg2 ::  f c
```

And that is it. Quite simple but powerful technique. And something that I
have wondered about before.
