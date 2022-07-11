---
title: Haskell
subtitle: Excercise
---

``` haskell
---------------------------------------------------------------------

delAllUpper :: String -> String

delAllUpper s =
  unwords $
    filter
      (not . all Data.Char.isUpper)
      $ words s

---------------------------------------------------------------------
```
