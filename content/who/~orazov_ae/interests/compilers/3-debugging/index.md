---
title: 3. Building a sublime debugging experience for SBlang
customdate: 2022 Oct 18 (火) 11:39PM
---

## Building with the aim of repairing 

I strive for the best and most comfortable debugging experience.

In SBlang this takes the follwing form:

- Run `test.sh`

  ```
  [!] Running test 10-test-shouldfail-comparison.sb   ... FAIL!
  Debug artifacts in 'artifacts/10-test-shouldfail-comparison'
  [!] Running test 10-test-shouldfail-timeout.sb      ... FAIL!
  Debug artifacts in 'artifacts/10-test-shouldfail-timeout'
  [!] Running test 10-test-sort-vec.sb                ... OK!
  [!] Running test 11-test-ptr-assignment.sb          ... OK!
  [!] Running test 12-test-comparison.sb              ... OK!
  [!] Running test 13-test-sort-small.sb              ... OK!
  [!] Running test 14-test-struct-layout.sb           ... OK!
  [!] Running test 15-test-library-vector.sb          ... OK!
  [!] Running test 16-test-fibonacci.sb               ... OK!
  [!] Running test 17-test-self-hosting.sb            ... OK!
  [!] Running test 18-test-pointer-arith.sb           ... OK!
  [!] Running test 19-test-tail-calls.sb              ... OK!
  -----------------------------------------------------------
  All tests passed! (ideally)
  ```

  Or in its full glory:
  {{< figure src="black-box-testing.png#center" width=600 >}}

- For each of the failing tests you get a info note telling you that there are
  build artifacts from the test waiting for you in the `artifacts` directory

  ```
  > tree
  artifacts
  ├── 10-test-shouldfail-comparison
  │   ├── dots
  │   │   └── raw
  │   ├── graph.sh
  │   └── log
  └── 10-test-shouldfail-timeout
      ├── dots
      │   └── raw
      ├── graph.sh
      └── log

  4 directories, 6 files
  ```

- These directories contain mostly everything you need to do debug the problem

  - Log contains the execution log of the program: its stack at all points in
    time as well as the indication of the specific place where the program has
    died.

  ```
  ...

  Current instruction [1 13]: push_value           00 00 01 00 00 00       Value [tag:Int, value:1]
  [!] Stack:
  0       1       2       3       4       5       6       7       8       9       10  
  s0      false   1       0       0       0       0       0       0       0       0

  Died here: line = 1, column = 8

  ...
  ```

  - The dots directory works together with graph.sh to provide a much more
    visual approach to debuuging. 

{{< rawhtml >}} 

<video width=100% controls autoplay>
    <source src="./graphviz-debugging.mkv">
</video>

{{< /rawhtml >}}

    - Notice that the stack actually has variable names and types annotations
      
      This is a part of attempt to bring proper debuuging support eventually,
      just like you would have in dwarf. This turns out to be rather tricky.
      But we are getting there.

    - Colors have special meanings too: red means writing to an interpreter
      memory location and green means reading from it. The more time passes
      from the access the move pale the color becomes.

This workflow is a marked improvement over what I had before, namely
compiling the tests right into the binary itself.
