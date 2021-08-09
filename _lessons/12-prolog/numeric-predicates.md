---
layout: page
title: Numeric predicates
---

# Numeric predicates
{: .no_toc .mb-2 }

- TOC
{:toc}

## Readings

- [Pre-recorded lecture](https://www.youtube.com/playlist?list=PLeIbBi3CwMZynn3lPPWeWhRe5N0Glv2nT)
- [Writing on the board]({{ site.baseurl }}{% link _lessons/12-prolog/num-pred.png %}) during class
- [99 Problems in Prolog](https://www.ic.unicamp.br/~meidanis/courses/mc336/2009s2/prolog/problemas/).
- Math in Prolog chapter from [Introduction to Programming
  Languages](https://en.wikibooks.org/wiki/Introduction_to_Programming_Languages).

## Using numerical predicates

How would we define a predicate in Prolog to determine the length of a list?

``` prolog
myLength([], 0).
myLength([_|Tail], Len) :- myLength(Tail, TailLen), Len = TailLen + 1.
```

The predicate `myLength` successively decomposes a list and, at each step,
"increments" its length. So we would expect the query `myLength([1], 1)` to
hold. However if we run the Prolog interpreter on it it will yield false:

``` prolog
?- length([1], 1).
true.
?- myLength([1], 1).
false.
```

Why is that? Since we know that all computation in Prolog is done via
unification in order to build a refutation, let's look closely at what is going
on.

For the query `? - myLength([1], 1).` to hold, its negation must resolve with
the clausal form of the `myLength` rule:

``` prolog
myLength([_|Tail], Len) v ~myLength(Tail, TailLen) v ~(Len = TailLen + 1)
```

which has an unifier `{Tail = [], Len = 1}`. Thus we obtain the clause

``` prolog
~myLength([], TailLen) v ~(1 = TailLen + 1)
```

We can now resolve this clause with the fact `myLength([], 0)` via the unifier `{TailLen = 0}`. Thus we obtain the clause

``` prolog
~(1 = 0 + 1)
```

There are no other resolutions we apply. Since Prolog cannot unify `1` and `0 +
1` (this is like trying to unify `a` and `f(a,b)`), the refutation fails. How
can we change this?

### The `is` predicate

Prolog is capable of performing *unification modulo arithmetic*, i.e., it can
apply arithmetic reasoning during unification and unify terms that are not
syntactically equal if they can be evaluated to equal terms. This is how the
`length` predicate is implemented and can say the query `length([1],1).` holds.

This predicate allows us to do explicity do unification modulo arithmetic.

``` prolog
?- X = 1, Y is X + 2.
X = 1,
Y = 3.
```

The `is` predicate works by evaluating the arithmetic expression into a
number. Above when the interpreter reason on `Y is X + 2` the variable `X` has
been instantiated to `1`, so `X+2` became `1+2`, which is evaluated to `3` and
`Y` is instantiated to it.

When the arithmetic expression cannot be evaluated into a number the interpreter
will rase an exception, for example

``` prolog
?- Y is X + 2, X = 1.
```

will lead to such an error. Another limitation of `is` is that it is not
commutative. For example, `2 is 1+1` holds but `1+1 is 2` does not. Only the
second argument is evaluated when doing the unification.


#### Fixing `myLength`

With the above in mind, we can rewrite the predicate `myLength` so that Prolog
can apply unification modulo arithmetic:

``` prolog
myLength([], 0).
myLength([_|Tail], Len) :- myLength(Tail, TailLen), Len is TailLen + 1.
```

With this definition, when building the refutation as we were before we derive
`~(1 is 0 + 1)`. With the `is` predicate, the predicate `0 + 1` is evaluated to
`1` before unification is performed, so the actual unification problem is to
unify `1` (left-hand side of `is`) and `1` (right-hard side of `is` after
evaluation), which is trivially unifiable. Thus from `~(1 is 0 + 1)` we derive
`~(true)`, then `false`, thus fininshing the refutation.

#### Building evaluable arithmetic

In building arithmetic expressions to be evaluated, one can use the following
binary predicates `+, -, *, /, <, >, =<, >=, =:=, =\=` and unary predicates:
`abs(Z), sqrt(Z), -`.

So we can for example do the following queries

``` prolog
?- X is 1/2.
?- X is 1.0/2.0.
?- X is 2/1.
?- X is 2.0/1.0.
?- 2 < 3.
?- 1+1 = 2.
?- 1+1 =:= 2.
```

Note that the fact that `is` only evaluates its second argument is very
important for writing Prolog rules. For example, if we write

``` prolog
myLength([], 0).
myLength([_|T], N) :- myLength(T, N1), N =:= N1 + 1.
```

we force both `N` and `N1 + 1` in the last term of the rule to be evaluated. We could still use this definition in a query

``` prolog
?- myLength([1], 1).
```

but what about

``` prolog
?- myLength([1], N).
```

## Examples

Unification modulo arithmetic allows us easily write Prolog programs for doing
some mathematics.

### Summing the elements of a list

``` prolog
sum([],0).
sum([Head|Tail],X) :- sum(Tail,TailSum), X is Head + TailSum.
```

### Factorial

``` prolog
fact(0, 1).
fact(1, 1).
fact(N, F) :- N > 1, N1 is N - 1, fact(N1, F1), F is F1 * N.
```

Prolog also has predicates to check whether a term has a predefined meaning. For
example, `integer` checks whether a term is an integer number, while `number`
whether a term is an integer or real number. For example we could rewrite the above definition to automatically fail on computations requested on non-integers:

``` prolog
fact(0, 1).
fact(1, 1).
fact(N, F) :- integer(N), N > 1, N1 is N - 1, fact(N1, F1), F is F1 * N.
```

### Greatest common divisor

``` prolog
gcd(X, Y, Z) :- X =:= Y, Z is X.
gcd(X, Y, Denom) :- X > Y, NewY is X - Y, gcd(Y, NewY, Denom).
gcd(X, Y, Denom) :- X < Y, gcd(Y, X, Denom).
```

## The `findall` predicate

How would we compute all the subsets of a set (represented as a list)?

``` prolog
subSet([], []).
subSet([H|T], [H|R]) :- subSet(T, R).
subSet([_|T], R) :- subSet(T, R).
```

The fact establishes that the empty set (i.e., the empty list) is a subset. The
first rule that a subset contains the first element of the set and possibly some
of the other elements. The third rule that ignoring one element of a set (the
head of the list), a subset will be contained in the set of the remaining
elements.

Thus for example

``` prolog
?- subSet([1,2],S).
S = [1, 2] ;
S = [1] ;
S = [2] ;
S = [].
```

in which the first unifier is built in a refutation based on two resolutions
with the rule `subSet([H|T], [H|R]) :- subSet(T, R).` and one with the fact
`subSet([], []).`; the second unifier based on one resolution with the first
rule, one with the second rule (`subSet([H|T], [H|R]) :- subSet(T, R).`), and
one with the fact; the third unifier based on one resolution with second rule,
one with the first rule, and one with the fact; and, finally, the last unifier
is based on two resolutions with the second rule and one with the fact.

How would we compute the power set of a set? The power set of a set is the set
of all of its subsets. So for this we would need to collect all the unifiers for
the second argument of `subSet` that make it hold. Luckily, Prolog has a useful
pre-defined predicate providing this, `findall`.

The predicate takes three arguments:
- a term `T`
- a predicate `P`
- a term `R`

The interpretation of `findall` is such that all the instantiations of `T`,
which would be generated in making `P` hold, are added to a list and unified
with `R`.

We can thus write a power set predicate as

``` prolog
powerSet(S, P) :- findall(SS, subSet(S, SS), P).
```

since for each possible way in which `subSet(S, SS)` can hold the respective
instantiation for `SS` will be accumulated in a list, which will be unifier with
`P` when all possibilities are exhausted.

### All the permutations of a list

``` prolog
perm([], []).
perm(List, [H|Perm]) :- select(H, List, Rest), perm(Rest, Perm).
```

Similarly to the power set, we can compute all the permutations of a list by
combining `findall` with `perm`:

``` prolog
allPerm(L, R) :- findall(P, perm(L, P), R).
```

## The eight-queens problem

The [eight-queens problem](https://en.wikipedia.org/wiki/Eight_queens_puzzle) is
a classic puzzle: how to place eight queens in an 8x8 chees board such that no
two queens threaten each other? Using Prolog we can solve this problem not
bothering how to actually solve them: it suffices to encode in Prolog the
restrictions for a possible solution and let its unification + resolution
machinery compute the solution for us.

From the chess rules, a solution for the eight-queens problem must be such no
two queens share the same row, column or diagonal.

### Representing a configuration of the board (and the column test)

A configuration of the board is where the eight queens are positioned in it. We
thus need eight positions to represent a configuration. Since queens cannot
share columns, there must be one queen in each column. We can start from that
when representing possible legal configurations.

Using a list of integers, from 1 to 8, with each position in a list representing
in which row the queen of the respective column is, we can represent a
configuration.

For example, `[4,2,7,3,6,8,5,1]` means that the queen in the first column is in
row 4, the queen in the second column in the row 2 and so on.

### The row test

Queens cannot share rows. Since we represent the row a queen is on in a
configuration with a number from 1 to 8, it must be the case that there are no
repetitions in a configuration and that all from 1 to 8 occur. This is to say
that a configuration must be a permutation of of the numbers from 1 to 8.

### The diagonal test

A queen placed at column `X` and row `Y` occupies two diagonals: one from
bottom-left to top-right and one from top-left to bottom-right. We name these
diagonals: the former is the diagonal `C = X - Y` and the latter is the diagonal
`D = X + Y`.

So we can define the diagonal test predicate as follows:

``` prolog
test([], _, _, _).
test([Y|Ys], X, Cs, Ds) :-
    C is X-Y, \+ member(C, Cs),
    D is X+Y, \+ member(D, Ds),
    X1 is X + 1,
    test(Ys, X1, [C|Cs], [D|Ds]).
```

which is such that given a configuration `Q` the predicate `test(Q, 1, [], []).`
holds, if, for each queen, its diagonals have not been previously occupied. The
predicate `\+` is true if and only if its argument cannot be shown to hold.

### Generating all possible configurations

All possible configurations of the board amount to all possible permutations of
a list. Thus we can build a solution to the eight queens problem by:

``` prolog
queen8(Q) :- perm([1,2,3,4,5,6,7,8], Q), test(Q, 1, [], []).
```

### How many solutions exist for the eight-queens problem?

How do we find all solutions? Or how de we count all solutions? We can rely
again on `findall`:

``` prolog
allQueen8(A) :- findall(Q, queen8(Q), A).
countAllQueen8(C) :- allQueen8(A), length(A, C).
```

- How would you write a Prolog solution for the N-queen problem?

## Sudoku

How would you solve [sudoku](https://en.wikipedia.org/wiki/Sudoku) in Prolog?
Check problem 97
[here](https://www.ic.unicamp.br/~meidanis/courses/mc336/2009s2/prolog/problemas/).

A solution is available
[here](https://www.swi-prolog.org/pldoc/man?section=clpfd-sudoku) using a
library in Prolog good for encoding combinatorial problems.
