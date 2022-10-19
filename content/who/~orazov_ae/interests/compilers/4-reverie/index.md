---
title: 4. Rêverie sur la syntaxe
customdate: 2022 Oct 19 (水) 10:43PM
---

Today I thought a lot about how my dream language syntax could look like.

Here is what I could think of (using completely contrived examples):

```

of Int -> *_ -> Int                  <<<-------- 1) type declaration is
fun cycle i last = {                             separated from arguments
    if i == 0 { return *last; };
                                                 2) ideally you could omit
    *last = *last + 1;                           all or only some parts of
    return cycle(i - 1, last);                   type annotations (like *_)
} -> Int;

```

The keyword **of** means _of type_.

```
fun min_or_max flavor =
    of Int -> Int -> Bool
    fun _ a b = if flavor {         <<<-------- 1) nested function declaration
        a < b                                   2) fun _ means "anonymous fun"
    } else {
        a > b
    } -> Bool;
```

This is how you would write a generic type declaration, recognizing the fact
that `VariantRecord` is a **type constructor** with a parameter _T_;

```
of Type -> Type
type VariantRecord T = union {
   | Exception
   | String T
   | (Int, Bool)
};
```

Perhaps you could then **pattern match** on the structure like so

```
of VariantRecord Char -> _
fun use_pattern_matching arg = match arg {
    s of String Char:
        return s.size;

    of (Int, true):
        return arg.0;

    e of Exception:
        std::abort();
} -> Int;
```

Ideally, writing the of in the beginning could be omitted. Still, wrinting
types is useful for documentative purposes. And Haskell's HLS can fill in the
type holes upon your request.

Conversion operator **~>**

```
of *Unit -> Int
fun convert_from memory = {
    memory ~> *String -> size    <<<------- convert raw memory to pointer to
} -> Int;                                   string and access the field size
                                            using the -> operator just like in C
```

Some more of that

```
fun main _ _ = {
    var v = (new Int) ~> *Unit;
    *v ~> *Int = 123 ~> Int;
    var result = cycle(4005, v) ~> Int;
    assert(*v == 4005);
    assert(*v == result);
} -> Unit;
```

I think is is somewhat like rust and hare and haskell and friends and it's
glorious and beautiful. Really, it's only when you start designing the language
you start the appreciate the work done for you in other compilers.

Right now I am very far from this. But I **want** it.
