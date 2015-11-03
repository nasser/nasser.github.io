---
layout: page
title: Pattern Overloading
--- 

C-like languages have a problem of overloaded syntax that I noticed while teaching high school students. Consider the following snippets in such a language:

    foo(45)
    
    function foo(int x) {
    
    for(int i=0;i < 10; i++) {
    
    if(x > 10) {
    
    case(x) {

An experienced programmer in this family would see

1. Function invocation
2. Function definition
3. Control flow examples

In my experience, new programmers see these constructs as instances of the same idea `identifier left-paren some-stuff right-paren` which is not an unreasonable conclusion to reach. The syntax for each construct is shockingly similar given that their semantics are wildly different. This has resulted in some hopelessly wrong code along the lines of

    foo(x < 20); { ... }
    
On top of that parentheses in these languages have yet *another* use: grouping. Bringing it all together, we can write code that uses the same symbols and syntax to mean wildly different things in a small space.

    int foo(int x) {
      if(x < 10) {
        return x * (width() + getSize(thing));
      }
      return 0;
    }

My argument is not that this is overly difficult to learn, but there is very little in the text of the code to remind the programmer which particular interpretation of a pattern they are encountering. You just have to memorize the different cases and the control flow keywords to get by.

The argument tends to be that there is a small number of symbols on the QWERTY keyboards we inherited, and that we have to make due. I don't quite buy that. I think this is sloppy design that has persisted due to programmer comfort over the years.

I can sum up my feelings in a simple mantra

<aside> Syntactic similarity should mirror semantic similarity </aside>

Or, to take a quote from the [UX world](https://userexperiences.co/clarity-in-the-details-design-ui-with-more-dimensions-ae1fa1473863),

<aside> Similar things should look similar and dissimilar things should look dissimilar </aside>

A language that I feel gets it right is Clojure. As a Lisp, its syntax is *extremely* consistent, and makes few compromises in the name of 'convenience'.

    (+ 1 2)
    
    (map inc [1 2 3])
    
    (vals {:foo "bar" :baz qux})
    
Parentheses mean *exactly one thing*: invocation. Square brackets mean *exactly one thing*: a vector. Curly braces mean *exactly one thing*: a hash map. In the cases where Clojure *does* overload symbols, the overloaded use is always prefixed by a `#` sign, so you can tell the difference between e.g. a map literal and a set literal.

    {:foo "bar"}  ;; map
    
    #{:foo "bar"} ;; set

The context is *explicitly in the text*. This is *good design*.

One place where Clojure is less explicit is in the fact that invoking functions and macros looks identical, but in reality is doing very different things. I will not count this against the language, because macros are specifically meant as tools to modify the syntax itself, so this similarity makes more sense.

I find myself valuing consistent, predictable syntax over anything overly 'clever', 'easy' or 'convenient' these days.