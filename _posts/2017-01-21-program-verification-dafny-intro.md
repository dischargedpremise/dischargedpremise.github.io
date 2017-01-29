---
layout: post
title: Getting Started with Program Verification
permalink: /verification-getting-started-dafny.html/
tags:
- Program Verification, postcondition,  Dafny  
date: 2017-01-21 10:0:0
---

# Getting Started with Program Verification
Program verification is the process of applying mathematical proof techniques to establish the correctness of a program with respect to a rigorous specification of program's behavior.

For automatic verification to be effective, we should provide a rigorous specification of program's inputs, outputs, and the *relation between* inputs and outputs. This rigorous specification is part of the *formal specification* of a program. 

We will use Dafny for writing programs and their specifications, and also for verifying their correctness. Dafny is a rich programming language: it has an imperative object-oriented core. A program can be annotated with logic expressions that describe its important behavioral properties. Dafny is also the name of the associated verifier. The verifier checks if (annotated) programs satisfy all requirements, which too are expressed as logic expressions. 

Our aim here is to take a few easy steps for getting acquainted with the central ideas of program verification. In the following sections and in subsequent posts, we shall use Dafny as a vehicle for exploring the rich and interesting landscape of formal verification.


## A Simple Example 
We will use a small number of ideas essential for specifying what a program computes. At the end of working through a bunch of examples, we will be able to recognize a pattern of thinking required to develop verifiable code.

Let's begin with a very simple program to find the greater of two numbers. 
(You can find the complete source code [here][1]).

```
method max2(a: int, b: int) returns (max: int)
{
    max := a;
    if (b > a) {
        max := b;
    }
}	
```
This code is mostly self-explanatory, and anyone with little programming language can informally argue about its correctness. There are a couple of Dafny language nuances to notice, though. First, recall that top-level method definitions in Dafny may be viewed equivalent to static methods of a top-level class in Java or C#. One may safely ignore the environment, and reason about on top-level methods in a modular fashion. Second, recall that assignment to identifier `max` determines the value returned by the procedure; no `return` statement is actually required. 

How do we check that `max2` actually computes the larger of two values passed to it?	

We may provide a method named <a name="check_max2">check_max2</a>:

````
method check_max2()
{
    var m: int;

    m := max2(20, 10);
    assert(m == 20);
}
````

When we run `max2` and `check_max2` through Dafny verifier, it produces an error report that reads more or less like so:

&#9654; ```
max2.dfy(...): Error: assertion violation
```

>This error message is striking. It highlights the fact that **program verification is entirely unlike program testing**. If Dafny were to merely execute the program comprising of the two methods shown above, `check_max2` would have played the role of a unit test, and the test would have obviously passed. However, the verifier reports an assertion violation much like a startling revelation!

### Attesting with postconditions
It is essential to notice that the verifier simply does not have adequate information to statically deduce required properties of `max2`. Dafny does not know what to verify in the first place. It needs to know what we logically expect of `max2`!

For verification to succeed, we must supply more information about the relation between the inputs and the output of `max2`. Dafny could then check that the implementation really satisfies these requirements. Once that is achieved, Dafny will be able to prove that our assertion holds in `check_max2`. 

A *postcondition* is a logic expression that specifies the *relation* between inputs and outputs, which must hold at the end of the execution of the program unit (in the current example, the method `max2`). In Dafny, postconditions are specified using `ensures` expressions, which are textually placed between the method-head (signature) and the method body, like the new definition of <a name="max2-one-post-cond">max2</a>:

```
method max2(a: int, b: int) returns (max: int)
    ensures (a >= b) ==> (max == a);
{
    max := a;
    if (b > a) {
        max := b;
    }
}
```

This single postcondition states that when value `a` is greater or equal to value `b`, `max` will be identical to value `a`. Note this program leaves another case unspecified, where value `a` would be less than value `b`. We shall see the implications of underspecified programs in a moment.

if we verify the new definition of <a href="./#max2-one-post-cond">max2</a> (equipped with a postcondition) along with <a href="./#check_max2">check_max2</a> shown earlier, Dafny breaks the good news thus:

&#9654; ```
Dafny program verifier finished with 4 verified, 0 errors
```

Before we leap in great joy, we could strengthen the requirements for `max2` with one more assertion stating the other behavior like this:

```
method another_check_max2()
{
    var m: int;

    m := max2(10, 20);
    assert(m == 20);
}
```

Notice the difference with <a href="./#check_max2">check_max2</a>. The current assertion essentially requires the postcondition that we explicitly left unspecified: the relation between inputs and the output, particularly when value `a` is less than value `b`. Would the verifier be able to establish this requirement on its own?

When checked, Dafny breaks the bad news:

&#9654; ```
max.dfy(...): Error: assertion violation  
```

>This reveals another crucial aspect of program verification. The verifier does not inspect the program text of `max2` while checking requirements (expressed with asserts, as shown above). It attempts to establish program correctness solely on the basis of preconditions and postconditions specified on `max2`. In this case, <a href="./#max2-one-post-cond">max2</a> is underspecified. Therefore, it is desirable to make postconditions as strong as possible. 

We now supply the required postcondition in the renewed definition of <a href="./#max2-two-post-conds">max2</a>: 

```
method max2(a: int, b: int) returns (max: int)
    ensures (a >= b) ==> (max == a);
    ensures (b > a)  ==> (max == b);
{
    max := a;
    if (b > a) {
        max := b;
    }
}
```

Notice these two postconditions now complete the specification of the input-output relationships. With more educated confidence in our specifications, we could design a method that succinctly expresses all requirements in one method:

```
method check_all_max2()
{
    var m: int;
    
    m := max2(10, 20);
    assert(m == 20);

    m := max2(10, 20);
    assert(m == 20);

    m := max2(10, 10);
    assert(m == 10); 

    m := max2(-20, -10);
    assert(m == -10);
}
```

Dafny reports that our program successfully meets all the requirements:

&#9654; ```
Dafny program verifier finished with 4 verified, 0 errors
```

## Conclusion
It is good to be sure that our program is correct with respect to formally specified requirements. We wrote postconditions to accurately capture the relation between inputs and the output, which enabled Dafny to prove that our program actually satisfies the requirements. In general, it takes more than mere postconditions to prove the correctness of complex programs. We will be required to augment programs with *preconditions* and postcondtions. We will see more of these ideas in other articles of this blog.


[1]: https://github.com/dischargedpremise/program-verification/blob/master/dafny/intro/max2.dfy

