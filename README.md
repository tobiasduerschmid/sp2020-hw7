# Instructions
Complete the code in the `bubblesort.dfy` and `*.py` files according to the
writeup PDF.

## Dafny
To complete the Dafny part of this assignment, you should finish the code in
`bubblesort.dfy` following the directions in the writeup PDF. There are two
options for running Dafny:

1. Use the online Dafny editor/interpreter at https://rise4fun.com/Dafny/1xSS
   This link is pre-populated with the starter BubbleSort code, but you'll still
   have to modify `bubblesort.dfy` after you get it working to complete the
   assignment.

2. Download [Dafny](https://github.com/dafny-lang/dafny) and run it
   locally. There are precompiled binaries: note that they require .NET (or
   [Mono](https://www.mono-project.com/)) to run.

The Dafny GitHub page has [a collection of links to Dafny
resources](https://github.com/dafny-lang/dafny#read-more).

## PyExZ3
To run the python code you'll need Python 3.2.3+, the Python Z3 bindings, and
PyExZ3.

### Setup

1. After installing Python 3, use `pip` (or `pip3`, depending on your local
   setup) to install the package `z3-solver` (i.e., `pip3 install z3-solver`).
2. Clone the [PyExZ3 repository](https://github.com/thomasjball/PyExZ3).
3. Check your setup by running the tests in the PyExZ3 directory with `python3
   run_tests.py test`. My (Jeremy's) personal setup passes all tests except for
   `bignum.py`.

### Example run

Run `signed.py` with PyExZ3 as follows:

```
python3 /path/to/pyexz3.py signed.py
```

You should see the following output:

```
PyExZ3 (Python Exploration with Z3)
Exploring signed.signed
[('x', 0)]
SAT: PATH IS UNSAFE
[('x', -16)]
SAT: PATH IS UNSAFE
-------------------> signed test failed <---------------------
Expected: {'UNSAT: PATH IS SAFE': 1, 'SAT: PATH IS UNSAFE': 1}, found: {'SAT: PATH IS UNSAFE': 2}
```

This means that PyExZ3 explored 2 paths: one where it chose `x = 0`, and
explored the `if` branch, and another where `x = -16` to explore the else
branch. For the constraints generated by both paths, the check `s.check()` is
SAT, implying both paths are unsafe. The `if` branch is indeed unsafe, since `y`
can be negative. Unfortunately, the `else` branch is also considered UNSAFE (it
SATisfies `y0 < 0`), because we haven't specified any constraints to say
otherwise. Based on the else branch, `y0` cannot be negative (and the
constraints on this path should result in UNSAT, i.e., not negative). Constraint
generation must be added reflect that. Once the constraints are added correctly,
the test will pass, for example:

```
PyExZ3 (Python Exploration with Z3)
Exploring signed.signed
[('x', 0)]
UNSAT: PATH IS SAFE
[('x', -16)]
SAT: PATH IS UNSAFE
signed test passed <---
```

### Guidance: signed_3.py

In `signed_3.py`, the variable `y` is overwritten (line 16): this means we need
a fresh symbolic variable to track updates to `y`. There are two symbolic
variables to track y: `y0` and `y1`. You'll notice only `y1`, the last symbolic
variable, is checked for negativity (line 24).

So, think carefully about which symbolic variables should refer to which `y`'s
in the `y = y * y` statement. You should not need to declare additional fresh
symbolic variables: two are enough.

### Guidance: signed_4.py

`signed_4.py` overwrites the variable `y` like `signed_3.py`, but this time in a
loop. This means we need to create fresh symbolic variables inside the loop. The
code has this set up for you already: on each iteration of the loop, there is a
`previous_y` and `fresh_y`. `previous_y` will refer to the previous symbolic
variable for `y` at the beginning of the loop, and during the loop. You need to
encode the correct VCGen only using `fresh_y` and `previous_y`.

Note the check condition (line 36) checks the value of `y5`, which is declared
`Int('y5')` (and not its own variable `y5`) since `y5` is the final `fresh_y`
produced in the loop. Note that `Int('y5')` can be used anywhere in your VC
instead of a dedicated variable `y5`. This is important for adding the VC Gen in
the `else` branch at line 31: think carefully about which symbolic variables for
`y` the statement `y = y` should refer to.
