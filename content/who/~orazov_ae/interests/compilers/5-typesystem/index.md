---
title: 5. Generics, HM, Unification, Constraint resolution prototype
customdate: 2022 Nov 08 (火) 10:06PM
---

Since the last post I have implemented the syntax and the unification based
type system I had described.

While it was difficult, it also was a lot of fun.

The first week I spent rewriting parser. Then in a giant leap of one more week
I implemented HM. There is still much work to do, for example in constraint
solver (very useful implication constraints) but the system is already quite
usable.

By the way, a good implementation of HM brings generics for free.

Here is how a generic tree in SBlang looks like:

```
type Tree a = struct {
    left: *Tree(a),
    right: *Tree(a),
    value: a,
};

fun consTree val = {
    var tree = new Tree(_);      <<------- It's compiler's job to figure
                                           out what kind of tree we want
    tree->left = unit ~> _;
    tree->right = unit ~> _;     <<------- This is just to convert
                                           `unit` to *Tree(_)

    tree->value = val;           <<------ the final type must be `a -> Tree(a)`
    tree
};

fun main = {
    var tr1 = consTree(1);       <<------ consTree instantiated as
                                          Int -> *Tree(Int),
                                          simple
    var tree = new Tree(Int);
    var tr2 = consTree(*tree);   <------- and here it is instantiated as
                                          Tree(Int) -> *Tree(Tree(Int))
};
```

And here is an example of gory inner workings of constraint solver (debug output)

```
...
Solving constraint Unit ~> $10
Solving constraint struct { left: *Tree( $8, ), right: *Tree( $8, ), value: $8, } .left ~ ($28 => $10)
Solving constraint Unit ~> $11
Solving constraint struct { left: *Tree( $8, ), right: *Tree( $8, ), value: $8, } .right ~ ($31 => $11)
Solving constraint struct { left: *Tree( $8, ), right: *Tree( $8, ), value: $8, } .value ~ ($34 => $15)


Unit ~> ($10 => *Tree( $8, ))
Unit ~> ($11 => *Tree( $8, ))


Solving constraint Unit ~> ($10 => *Tree( $8, ))
Solving constraint Unit ~> ($11 => *Tree( $8, ))


($50) ~ $18
Call ($52 -> *Tree( $52, ))
(Int -> $56) ~ ($52 -> *Tree( $52, ))
$19 ~ $56
$21 ~ *Tree( Int, )
$21 ~ *$62
Call ($58 -> *Tree( $58, ))
($62 -> $64) ~ ($58 -> *Tree( $58, ))
$22 ~ $64
Unit ~ $50


Solving constraint ($50) ~ $18
Solving constraint Call ($52 -> *Tree( $52, ))
Solving constraint (Int -> $56) ~ ($52 -> *Tree( $52, ))
Solving constraint $19 ~ $56
Solving constraint $21 ~ *Tree( Int, )
Solving constraint ($21 => *Tree( Int, )) ~ *$62
Solving constraint Call ($58 -> *Tree( $58, ))
Solving constraint ($62 -> $64) ~ ($58 -> *Tree( $58, ))
Solving constraint $22 ~ $64
Solving constraint Unit ~ $50


Int ~ $52
($56 => $19) ~ *Tree( $52, )
Tree( Int, ) ~ $62
$62 ~ $58
($64 => $22) ~ *Tree( $58, )
```


And here is an example of instantiation:
```
...
[!] Symbol
[!] Poly (($23 => G8) -> ($24 => *Tree( G8, )))          <<<----- After typeinference unbound variable
[!] Mono (Int -> *Tree( Int, ))                                       aquired generic qualities
[!] Symbol                                                        Based on constraints at the callsite we
[!] Poly (($23 => G8) -> ($24 => *Tree( G8, )))                       need to instantiate the right version
[!] Mono (Tree( Int, ) -> *Tree( Tree( Int, ), ))
name: consTree type: (Int -> *Tree( Int, ))
name: consTree type: (Tree( Int, ) -> *Tree( Tree( Int, ), ))
name: main type: (Unit)
Measuring Tree( Int, ) for left                          <<<----- Measuring is done in codegen phase for each
Measuring Tree( Int, ) for right                                  possible backend separately (width of the
Measuring Tree( Int, ) for value                                  machine word, etc)
...
```
