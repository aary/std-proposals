# Structured bindings with polymorphic lambdas
### Aaryaman Sagar
### August 14, 2017

## Introduction

This paper proposes usage of structured with polymorphic lambdas, adding them
to another place where `auto` can be used as a declarator

```c++
auto map = std::map<int, int>{{1, 2}, {3, 4}};
std::for_each((map, [](auto [key, value]) {
    cout << key << " " << value << endl;
});

```

This would make for nice syntactic sugar to situations such as the above
without having to decompose the tuple-like object manually, similar to how
structured bindings are used in range based for loops

```c++
auto map = std::map<int, int>{{1, 2}, {3, 4}};
for (auto [key, value] : map) {
    cout << key << " " << value << endl;
}
```

## Motivation

### Simplicity and uniformity
Structured binding initialization can be used almost anywhere `auto` is
used to initialize a variable (not considering `auto` deduced return
types), and allowing this to happen in polymorphic lambdas would make code
simpler, easier to read and generalize better

```c++
std::find_if(range, [](const auto& [key, value]) {
    return examine(key, value);
});
```

### Programmer demand
There is some programmer demand and uniform agreement on this feature

    * [Stack Overflow - Can the structured bindings syntax be used in polymorphic lambdas]{https://stackoverflow.com/questions/45541334}
    * [ISO C++ : Structured bindings and polymorphic lambdas](https://goo.gl/fRSwNg)

### Prevalence
It is not uncommon to execute algorithms on containers that contain a value
type that is either a tuple or a tuple-like decomposable class.  And in such
cases code usually deteriorates to manually unpacking the instance of the
decomposable class for maximum reaadability, for example

```c++
auto result = std::count_if(map, [](const auto& key_value_pair) {
    const auto& key = key_value_pair.first;
    const auto& value = key_value_pair.second;

    return examine(key, process_key(key), value);
});
```

The first two lines in the lambda are just noise and can nicely be replaced
with structured bindings in the function parameter

```c++
auto result = std::count_if(map, [](const auto& [key, value]) {
    return examine(key, process_key(key), value);
});
```


## Impact on the standard
The proposal describes a pure language extension which is a non-breaking
change - code that was previously ill-formed now would have well-defined
semantics.

##
If a type can be decomposed into a structured binding expression with
`x` bindings, then it is said to be `x-decomposable` (reference
`[dcl.struct.bind]` for the exact requirements).  Any compiler
diagnostics provided are the same as that for regular structured bindings
initializations - a compiler concept that determines if a type is
`x-decomposable`.

Part of such a concept can be made using the tools available to the
programmer.  In particular, it is possible to make a concept or SFINAE trait
that enables us to check if a type is decomposable via a member or ADL defined
free `get<>()` method/function and the existence of a specialized
`std::tuple_size<>` and `std::tuple_size<>` traits

From the current working draft of the C++17 standard `[dcl.struct.bind]`p2 and
`[dcl.struct.bind]`p3 define decomposability that can be checked by the
programmer at compile time via a concept or SFINAE trait.  However
`[dcl.struct.bind]`p4 describes a method of unpacking that cannot be enforced
purely by the language constructs available as of C++17.  As such a concept
provided by the compiler is required.

This concept `std::decomposable<x>` shall be a concept that accepts a non type
template parameter of type `std::size_t` that determines the cardinality of
the structured bindings decomposition.  This concept holds if a type is
`x-decomposable` (and this will take into consideration the requirements set
forth by `[dcl.struct.bind]` paragraphs 2, 3 and 4.


