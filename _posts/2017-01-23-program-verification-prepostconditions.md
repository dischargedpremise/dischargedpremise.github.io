---
layout: post
title: Getting Started with Program Verification
permalink: /verification-prepostconditions-dafny.html/
tags:
- formal specification, precondition, postcondition, program verification, Dafny  
date: 2017-01-17 23:0:0
---

# Formal Specification of Sequential Programs
We understand the word *specification* to informally mean a description of a problem. In this view, a specification describes essentially *what* is to be accomplished. While specifying the problem, we omit many details of *how* the solution should work. It is understood that these "implementation details" may be varied without affecting *what* is actually achieved.

The role of a program, in some sense, is to transform its inputs to outputs. The role of a *specification* is to formally (i) characterize inputs and outputs of the program, and (ii) state the logical relation between inputs and outputs of the program. By this definition, a *correct program* transforms its inputs to outputs per the specification. Therefore, the correctness of a program is not an objective quality; it is not a property of a program in its own right. 

>A program is correct only with respect to its formal specification.

## The Logic of Specification
The specification of a program is given as a set of sentences in first-order predicate calculus (aka [first-order logic][first-order-logic]). The basic forms of specification include **preconditions** and **postconditions**. 

> A precondition specifies properties that must hold before a program construct is executed.

> A postcondition specifies properties that must hold after the program construct has completely executed.

These foundational ideas are captured in an extremely effective and succinct notation as [Hoare triples][hoare-triple]:
  <div style='text-align: center; margin: 1.5rem auto;'>{P} C {Q}</div>
where P represents the precondition, and Q, the postcondition. The Hoare triple is meant to express the following:

>If command C is started in a state satisfying assertion P, then when (and if) C terminates, the final state is guaranteed to satisfy the assertion Q.

The preconditions must be satisfied in the state in which a program construct begins its execution. The postconditions must be satisfied in the state resulting from the terminating execution of command C.


## Examples in Dafny Language
With these basic guiding idea, we will now work with concrete programs written using [Dafny language][dafny-lang]. Dafny permits us to write specifications in the form of preconditions and postconditions on programs. Dafny verifier checks if the program satisfies the specification. 

The programs shown here are maintained in the [GitHub repository][github-dp-program-verification] associated with this blog. Moreover, you will be able to [play with Dafny programs using a Web browser][dafny-rise4fun]. It is possible to simply copy-paste the code, or  modify the code, and let the verifier respond.

### How to Add 5 to a Number?
Let us begin with a trivial problem: compute the result of adding 5 to a given positive number. 

Our desire is to carefully analyze the meaning of the phrase, *a program is correct only with respect to its specification*. We will study the effect of varying the postconditions for the same program. A moment's thought indicates that depending upon the postconditions, specific assertions may or may not be valid in particular states of a program. 

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
    add5.dfy(...): Error: assertion violation
```

Dafny fails to deduce the "obvious" correctness of the assertion because `max5` has no postconditions specifying properties satisfied by the terminating state of `add5`. 

### Version 2 - Precondition and Postcondition Specification
Let us make up a precondition. Assume that the domain of `max5` is represented by the set of postive numberss { 0, 1, ..., 99, 100 }. We can specify this domain in a precondition using Dafny's `requires` construct:
<div style='text-align: center; margin: 1.5rem auto;'><span style='color: blue;'> requires</span> 0 <= x <= 100;</div>

The postcondition is straightforward. It should accurately capture the relationship between the output and the input. In this case, the result (represented by `r`) is obtained by adding five to input `x`. This is specified using an `ensures` construct:
<div style='text-align: center; margin: 1.5rem auto;'><span style='color: blue;'> ensures</span> r == (x + 5);</div>


Here is the <a name="strong-postcond-add5">updated version of `add5`</a>:   

```
method add5(x: nat) returns (r: nat)
    requires 0 <= x <= 100;
    ensures r == (x + 5);
{
    r := x + 5;
}
```

We ask Dafny to process this augmented version of `add5` in conjunction with <a href="./#check-add5">check_add5</a>. Our attempt succeeds with the message:

<span style="color:lightgreen;">&#9654;</span>
```
Dafny program verifier finished with 4 verified, 0 errors
```

### Version 3 - Fooling Around with Weaker Postconditions
Let us first get acquainted with a couple of technical terms. Let **V**  stand for the set of variables used in postconditions. An *assignment* refers to a list of distinct identifiers (or variables) bound to values. For example, 
<div style='text-align:center; margin: 1.5rem auto'> let <span style='font-weight: bold'>V</span> = {x, y}</div>  *and*
<div style='text-align:center; margin: 1.5rem auto'> &Sigma;<sub>1</sub> &cong; (x = 10, y = 20)</div>
*and*
<div style='text-align:center; margin: 1.5rem auto'> &Sigma;<sub>2</sub> &cong; (x = 	32, y = 64)</div>
which is taken to mean that assignment &Sigma;<sub>1</sub> binds identifiers x and y to 10 and 20, respectively, and that another assignment &Sigma;<sub>2</sub> binds identifiers x and y to 32 and 64, respectively. Notice that &Sigma;<sub>1</sub> and &Sigma;<sub>2</sub> are two *distinct assignments* or bindings because they vary in at least one variable's bound value. 

>It is also possible to view &Sigma;<sub>1</sub> and <span>&Sigma;<sub>2</sub> </span> as two distinct execution *states* of a program. A *state*, in this view, maps identifiers to values.


Let us now introduce a notion of *relative strength* of a predicate:<a name="predicate-strength"></a>

>Let P and Q be predicates over a set of variables **V**. Let &#8473; be the set of distinct assignments to variables that satisfy P. Let &#8474; be the set of distinct assignments to variables that satisfy Q. Predicate P is ***stronger*** than Q if &#8473; is a ***subset*** of &#8474;. 

<div style='text-align:center; margin: 1.5rem auto'> P is stronger than Q &DoubleRightArrow;  &#8473; &SubsetEqual;  &#8474;.</div>


In the definition of the relative strength of a predicate, <a href="./#predicate-strength">above</a>, we referred to &#8473; (*or* &#8474;) as a set of distinct assignments that satisfy predicate P (*or* Q). For instance, if predicate P is satisfied by three distinct assignments - &Sigma;<sub>1</sub>, &Sigma;<sub>1</sub>, and &Sigma;<sub>3</sub>, we write 
<div style='text-align:center; margin: 1.5rem auto'>&#8473; = { &Sigma;<sub>1</sub>, &Sigma;<sub>2</sub>, &Sigma;<sub>3</sub> }.</div>

When P stronger than Q, it simply implies that assignments in &#8473; are all contained in &#8474;. That is, 
<div style='text-align:center; margin: 1.5rem auto'>
           &#8474; = &#8473; &bigcup; &#8477;&nbsp;&nbsp;&nbsp;&nbsp;<span style='font-style: italic;'>where</span> &#8477; <span style='font-style: italic;'>is not empty.</span> </div>

In such cases, predicate P is a stronger logic statement than Q because the set of states satisfying P are a subset of states in which Q holds.

>If P and Q be predicates over a set of variables **V**, the statement *P is **stronger** than Q* is equivalent to the statement predicate *Q is **weaker** than P*.

Equipped with these definitions, let us vary the postcondition in the previous definition of <a href="./#strong-postcond-add5">add5</a>, reducing the strength of the predicate involved. 

Let us seek to obtain a weaker postcondition for <a href="./#strong-postcond-add5">add5</a>. The current postcondition 
<div style='text-align: center; margin: 1.5rem auto;'><span style='color: blue;'> ensures</span> r == (x + 5);
</div>
is the strongest because it identifies an unique value for the output variable *r*. With the intention of weakening this predicate, we may relax the requirement thus:
<div style='text-align: center; margin: 1.5rem auto;'><span style='color: blue;'> ensures</span> r <span style='font-weight: bold; color: darkblue;'><=</span> (x + 5)</div>

Can we intuitively see why the predicate **r <=(x + 5)** is weaker than **r ==(x + 5)**? Well, the reasoning is straightforward. With the predicate r <=(x + 5),  variable *r* may assume values from a larger domain - *non-negative* values lesser or equal to (x+5), while the with the latter predicate, *r* assumes exactly one value, (x+5). 

We now show the program text of `add5` with a weaker postcondition. Notice `use_add5` too has changed accordingly. Take a close look at the new assertions. 
<a name="weaker-post-cond-add5"></a>This program satisfies Dafny's verifier.

```
method add5(x: nat) returns (r: nat)
    requires 0 <= x <= 100;
    ensures r <= (x + 5);
{
    r := x + 5;
}
   
method use_add5()
{
    var v;

    v := add5(7);
    assert(v >= 0);
    assert(v <= 12);
}
```
In the code above, `use_add5` reveals one detail about the working of program verifiers. With `add5` donning a weaker postcondition, `use_add5` cannot afford stronger assertions despite the fact that `add5`'s implementation guarantees, as a matter of fact, a much stronger postcondition. The postcondition on the program construct is only as strong as it is specified; not more or less. On many occasions, this comes as a surprise to the novice. The following code illustrates the "surprise element" that stems from the weaker postcondition on `add5`: 


```
method check_stronger_assert_add5()
{
    var v;

    v := add5(7);
    assert(v == 12);
}
````

Our attempt to assert a stronger property, now fails. 

<span style="color:red;">&#9654;</span> Dafny flags an error against the assertion in `check_stronger_asserts_add5`.

Recall this property was previously guaranteed by a <a href='./#strong-postcond-add5'>stronger postcondition on `add5`</a>. Under the latest specification of `add5`, we can assert only a weaker property `0 <= v <= 12` in the <a href='./#weaker-post-cond-add5'>updated version of `use_add5`</a>.


## Conclusion
One of the exigencies of specification is to understand exactly which properties are crucial for program's correctness, and to formulate strong postconditions that meet these requirements. We introduced a bunch of related ideas in this article. We demonstrated the effects of stronger and weaker postconditions while keeping our program's implementation details intact.





[first-order-logic]: https://en.wikipedia.org/wiki/First-order_logic
[hoare-triple]: https://en.wikipedia.org/wiki/Hoare_logic#Hoare_triple
[for-all-x]: http://www.fecundity.com/logic/
[dafny-lang]: https://www.microsoft.com/en-us/research/project/dafny-a-language-and-program-verifier-for-functional-correctness/
[dafny-rise4fun]: http://rise4fun.com/dafny/
[github-dp-program-verification]: https://github.com/dischargedpremise/program-verification/tree/master/dafny/intro
