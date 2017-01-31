---
layout: post
title: Getting Started with Program Verification
permalink: /verification-prepostconditions-dafny.html/
tags:
- formal specification, precondition, postcondition, program verification, Dafny  
date: 2017-01-17 23:0:0
---

# Formal Specification of Sequential Programs
We understand the word *specification* to informally mean a description of the problem to be solved. In this view, a specification essentially describes *what* is to be accomplished. While specifying the problem, we mostly omit many details of *how* to achieve the goal. It is understood that these "implementation details" may be varied without affecting *what* is actually achieved.

We may view the role of a program is to transform inputs to outputs, and the role of a *specification* is to formally characterize (i) inputs and outputs of the program, and (ii) the logical relation between inputs and outputs of the program. By this definition, the correctness of a program is not an objective quality; it is not a property of a program in its own right. 

>A program is correct only with respect to its formal specification.

## The Logic of Specification
The specification of a program is given as a set of sentences in first-order predicate calculus (aka [first-order logic][first-order-logic]). The basic forms of specification include **preconditions** and **postconditions**. 

> A precondition specifies properties that must hold before the program-unit under consideration is executed.

> A postcondition specifies properties that must hold after the program unit under consideration has completely executed.

These foundational ideas are captured in extremely effective and succinct notation as [Hoare triples][hoare-triple]:
  <div style='text-align: center; margin: 1.5rem auto;'>{P} C {Q}</div>
where P represents the precondition, and Q, the postcondition. The Hoare triple is meant to express the following:

>If command C is started in a state satisfying assertion P, then when (and if) c terminates, the final state is guaranteed to satisfy the assertion Q.

Preconditions must be satisfied in the state in which a program unit begins its execution. Postconditions must be satisfied in the state resulting from the terminating execution of command C.


## Examples in Dafny Language
With the basic definitions guiding us, we shall now work with concrete programs written using [Dafny language][dafny-lang]. Dafny permits us to write specifications in the form of preconditions and postconditions on programs. Dafny verifier checks if the program satisfies the specification. 

The programs shown here are maintained in the [GitHub repository][github-dp-program-verification] associated with this blog. Moreover, you will be able to [play with Dafny programs using a Web browser][dafny-rise4fun]. You will learn quite a lot by confidently slicing and dicing the code shown in the following sections.

### How to Add 5 to a Number?
Let us begin with a trivial problem: compute the result of adding 5 to a given positive number. 

Our desire is to carefully analyze the meaning of the phrase *a program is correct only with respect to a specification*. We will study the effect of varying the postconditions for the same program. A moment's thought indicates that depending upon postconditions, specific assertions may or may not be valid in particular states of the program. Because of the lack of necessary postconditions, Dafny may fail to deduce the correctness of an assertion. Properly specified postconditions (in the sense of the strength of predicates) are vital for proving essential properties of the program.

Here is the simplest version of our program *sans* explicit preconditions and postconditions:

```
method add5(x: nat) returns (r: nat)
{
    r := x + 5;
}
```

Can we ask the verifier to assert that the program actually computes the correct answer? Let's give it a try with the following "obviously correct" assertion as shown in <a name="check-add5">check_add5</a> below:  

```
method check_add5()
{
    var res;

    res := add5(7);
    assert(res == 12);
}
```

Dafny reports an error:


<span style="color:red;">&#9654;</span>
```
    add5.dfy(11,10): Error: assertion violation
```

Dafny fails to deduce the "obvious" correctness of the assertion because there is no postcondition specifying the properties satisfied by the terminating state of `add5`. 

### Version 2 - Precondition and Postcondition Specification
Let us make up a precondition. Assume that we will only accept positive numbers in the range 0 to 100. We specify a precondition using Dafny's `requires` construct:
<div style='text-align: center; margin: 1.5rem auto;'><span style='color: blue;'> requires</span> 0 <= x <= 100;</div>

The postcondition is straightforward. It should accurately capture the relationship between the output and the input. In this case, the result (represented by `r`) is obtained by adding five to input `x`. This is specified using an `ensures` construct:
<div style='text-align: center; margin: 1.5rem auto;'><span style='color: blue;'> ensures</span> r == (x + 5);</div>


Here is the updated version of the same program:   

```
method add5(x: nat) returns (r: nat)
    requires 0 <= x <= 100;
    ensures r == (x + 5);
{
    r := x + 5;
}
```

We ask Dafny to process this augmented version of `add5` with <a href="./#check-add5">check_add5</a>, and succeed:

<span style="color:lightgreen;">&#9654;</span>
```
Dafny program verifier finished with 4 verified, 0 errors
```

### Version 3 -  Weaker Postconditions



[first-order-logic]: https://en.wikipedia.org/wiki/First-order_logic
[hoare-triple]: https://en.wikipedia.org/wiki/Hoare_logic#Hoare_triple
[for-all-x]: http://www.fecundity.com/logic/
[dafny-lang]: https://www.microsoft.com/en-us/research/project/dafny-a-language-and-program-verifier-for-functional-correctness/
[dafny-rise4fun]: http://rise4fun.com/dafny/
[github-dp-program-verification]: https://github.com/dischargedpremise/program-verification/tree/master/dafny/intro
