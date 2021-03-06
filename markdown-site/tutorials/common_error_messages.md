
Common error messages
=====================

This page gives a list of quick explanations for some error or warning
messages that are emitted by the OCaml compilers. Longer explanations
are usually given in dedicated sections of this tutorial.

Type errors
-----------

### This expression has type ... but is here used with type ...

When the type of an object is not compatible with the context in which
it is used, it is frequent to obtain this kind of message:

    # 1 + 2.5;;File "", line 1, characters 4-7:
    Error: This expression has type float but an expression was expected of type
             int

"This expression has type *X* but is here used with type *Y*" means that
if the contents of the expression is isolated (2.5), its type is
inferred as *X* (float). But the context, i.e. everything which is
around (1 + ...) tells that the gap expects an expression of type *Y*
(int) which is not compatible with *X*.

More disturbing is the following message:

     This expression has type my_type but is here used with type my_type

This error happens often while testing some type definitions using the
toplevel. In OCaml, it is perfectly legal to define a type with a name
that is already taken by another type. Consider the following session:

    # type my_type = A | B;;type my_type = A | B
    # let a = A;;val a : my_type = A
    # type my_type = A | B;;type my_type = A | B
    # let b = B;;val b : my_type = B
    # a = b;;File "", line 1, characters 4-5:
    Error: This expression has type my_type/2627
           but an expression was expected of type my_type/2623

For the compiler, the second definition of my\_type is totally
independent from the first definition. So we have defined two types
which have the same name. Since "a" was defined earlier, it belongs to
the first type while "b" belongs to the second type. In this example,
redefining "a" after the last definition of my\_type solves the problem.
This kind of problem should not happen in real programs unless you use
the same name for the same type in the same module, which is highly
discouraged.

### Warning: This optional argument cannot be erased

Functions with optional arguments must have at least one non-labelled
argument. For instance, this is not OK:

    # let f ?(x = 0) ?(y = 0) = print_int (x + y);;File "", line 1, characters 15-43:
    Warning 16: this optional argument cannot be erased.
    val f : ?x:int -> ?y:int -> unit = <fun>

The solution is simply to add one argument of type unit, like this:

    # let f ?(x = 0) ?(y = 0) () = print_int (x + y);;val f : ?x:int -> ?y:int -> unit -> unit = <fun>

See the [Labels](labels.html "Labels") section for more details on
functions with labelled arguments.

### The type of this expression... contains type variables that cannot be generalized

This happens in some cases when the full type of an object is not known
by the compiler when it reaches the end of the compilation unit (file)
but for some reason it cannot remain polymorphic. Example:

    let x = ref None

triggers the following message during the compilation:

    The type of this expression, '_a option ref,
    contains type variables that cannot be generalized

Solution: help the compiler with a type annotation, like for instance:

    let x : string option ref = ref None

or:

    let x = ref (None : string option)

Data of type `'_a` may be allowed temporarily, for instance during a
toplevel session. It means that the given object has an unknown type,
but it cannot be any type: it is not polymorphic data. In the toplevel,
our example gives these results:

    # let x = ref None;;val x : '_a option ref = {contents = None}

The compiler tells us that the type of x is not fully known yet. But by
using `x` later, the compiler can infer the type of `x`:

    # x := Some 0;;- : unit = ()

Now `x` has a known type:

    # x;;- : int option ref = {contents = Some 0}

More details are given in the [OCaml
FAQ](http://caml.inria.fr/pub/old_caml_site/FAQ/FAQ_EXPERT-eng.html#variables_de_types_faibles "http://caml.inria.fr/pub/old_caml_site/FAQ/FAQ_EXPERT-eng.html#variables_de_types_faibles").

Pattern matching warnings and errors
------------------------------------

### This pattern is unused

This warning should be considered as an error, since there is no reason
to intentionally keep such code. It may happen when the programmer
introduced a catch-all pattern unintentionally such as in the following
situation:

    # let test_member x tup =
        match tup with
        | (y, _) | (_, y) when y = x -> true
        | _ -> false;;File "", line 3, characters 16-20:
    Warning 12: this sub-pattern is unused.
    val test_member : 'a -> 'a * 'a -> bool = <fun>

Obviously, the programmer had a misconception of what OCaml's pattern
matching is about. Remember the following:

-   the tree of cases is traversed linearly, from left to right. There
    is *no backtracking* as in regexp matching.
-   a guard ("when" clause) is not part of a pattern. It is simply a
    condition which is evaluated at most once and is used as a last
    resort to jump to the next match case.
-   lowercase identifiers (bindings such as "y" above) are just names,
    so they will always match.

In our example, it is now clear that only the first item of the pair
will ever be tested. This leads to the following results:

    # test_member 1 (1, 0);;- : bool = true
    # test_member 1 (0, 1);;- : bool = false

### This pattern-matching is not exhaustive

OCaml's pattern matching can check whether a set of patterns is
exhaustive or not, based on the *type* only. So in the following
example, the compiler doesn't know what range of ints the "mod" operator
would return:

    # let is_even x =
        match x mod 2 with
        | 0 -> true
        | 1 | -1 -> false;;File "", line 2, characters 4-60:
    Warning 8: this pattern-matching is not exhaustive.
    Here is an example of a value that is not matched:
    2
    val is_even : int -> bool = <fun>

A short solution without pattern matching would be:

    # let is_even x = x mod 2 = 0;;val is_even : int -> bool = <fun>

In general, that kind of simplification is not possible and the best
solution is to add a catch-all case which should never be reached:

    # let is_even x =
        match x mod 2 with
        | 0 -> true
        | 1 | -1 -> false
        | _ -> assert false;;val is_even : int -> bool = <fun>

Problems recompiling valid programs
-----------------------------------

### x.cmi is not a compiled interface

When recompiling some old program or compiling a program from an
external source that was not cleaned properly, it is possible to get
this error message:

    some_module.cmi is not a compiled interface

It means that some\_module.cmi is not valid according to the *current
version* of the OCaml compiler. Most of the time, removing the old
compiled files (\*.cmi, \*.cmo, \*.cmx, ...) and recompiling is
sufficient to solve this problem.

### Warning: Illegal backslash escape in string

Recent versions of OCaml warn you against unprotected backslashes in
strings since they should be doubled. Such a message may be displayed
when compiling an older program, and can be turned off with the "-w x"
option.

    # "\e\n" (* bad practice *);;Warning: File "", line 1, characters 1-3: Illegal backslash escape in string or character (e)
    - : string = "\\e\n"
    # "\\e\n" (* good practice *);;- : string = "\\e\n"
