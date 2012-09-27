<<<<<<< HEAD
% An introduction to static analysis
=======
% Static analysis: an overview
>>>>>>> 77e9b1adff6c950e324bf56ad5edc6840370e88f
% Austin Seipp
% AHA 0x0048 - August 27th, 2012

# Quick Prep
<<<<<<< HEAD

## What this talk is:

 * Intro to static analysis as an idea
 * Demos of what they can and cannot find
 * A bit of theory

## What it is not:

 * A thorough review of the tools
 * Boring (I hope!)

# What is static analysis?

Static analysis (in the context of this talk) is the process of **finding bugs
in code, given only a static model of its representation, without executing
it** (i.e. analysing the source code directly.) You can think of it simply as,
automated source code auditing.

The richer the static information about the program, the more accurate and
powerful an analysis can be. Certain languages are more amenable to analysis
than others.

There are lots of ways to do it. We will talk about the idea of 'industrial
strength' analyzers that are built to handle all kinds of real world code.

# How it works

  1) Parse all source code. This basically amounts to the 'frontend' of a
  compiler. This essentially gives you a *Control-flow graph* (CFG) of the
  program.

  2) Begin evaluating the program CFG symbolically at the main entry point, and
  follow every possible path. Every node in the graph represents a point in the
  program, and is associated with a model of the program state. As you do this,
  you build lists of constraints (both abstract, and domain-specific)
  associated with points in the CFG.

  3) During this process, the analyzer may determine certain paths are
  impossible (due to conflicting constraints) and remove them. This is the process
  of 'pruning' the CFG for dead paths.

  4) At the same time, the analyzer will note that certain paths *satisfy* a
  particular constraint.  Typically, this constraint is something like 'a bug
  is found if X Y and Z are satisified.' This is a bug.

# Simple example: code

Find the bug!

~~~~~~ {.c}
double
f(unsigned int x)
{
  if (x >= 0 && x < 43)
    return 42/x;
  else
    return 0.0;
}
~~~~~~

# Simple example: code

Find the bug!

~~~~~~ {.c}
double
f(unsigned int x)
{
  if (x >= 0 && x < 43)
    return 42/x; // BZZT: divide by zero
  else
    return 0.0;
}
~~~~~~

# Simple example: control flow graph

TODO

# Constraints, facts and symbolism

In order to find bugs, the analyzer must both operate in the *abstract* and in the
*concrete*. In the *abstract* it must symbolize and follow control flow paths, and
do things such as evaluate arithmetic or function calls.

But in the *concrete*, it must look for specific errors: division where the
denominator may be zero, for example. Or, a call `alloca(x);` where `x` might
be zero, etc.

As a result, you typically have a decoupling between things that check for *specific*
errors, and the engine that actually *analyzes the constraints*.

Real example to follow.

Definitions:

**False positives**: The reporting of a bug that does not actually exist, i.e.
the analyzer is wrong.

**False negatives**: The failure to report a bug that actually exists.

# Analysis: the Java side


# Analysis: the C/C++/Objective-C side


# An example of a simple analysis with Clang (pt. 1)

But what if we wanted to write an analysis for C/C++? It is surprisingly easy!

Real world code from Clang:

~~~~~~ {.c}
void
DivZeroChecker::checkPreStmt(const BinaryOperator *B,
                             CheckerContext &C) const {
  BinaryOperator::Opcode Op = B->getOpcode();
  // Must be division or modulus
  if (Op != BO_Div &&
      Op != BO_Rem &&
      Op != BO_DivAssign &&
      Op != BO_RemAssign)
    return;
  /* No aggregate types */
  if (!B->getRHS()->getType()->isScalarType())
    return;
  ...
~~~~~~

# An example of a simple analysis with Clang (pt. 2)

~~~~~~ {.c}
  // Get the 'symbolic value' AKA SVal
  // for the denominator. It may be a 'defined' value
  // or 'undefined', AKA undefined behavior.
  SVal Denom = C.getState()->getSVal(B->getRHS(), ...);
  const DefinedSVal *DV = dyn_cast<DefinedSVal>(&Denom);

  // Divide-by-undefined handled in
  // the generic checking for uses of undefined values.
  if (!DV)
    return;
 ...
~~~~~~

# An example of a simple analysis with Clang (pt. 3)

~~~~~~ {.c}
  // Check for divide by zero.
  ConstraintManager &CM = C.getConstraintManager();
  ProgramStateRef stateNotZero, stateZero;
  /* Bifurcate path in CFG, where we represent both
  possiblities: zero & non-zero. If we assume this, and
  then can show it's not possible to be in a state
  where the denominator is not zero, report a bug. */
  llvm::tie(stateNotZero, stateZero) 
    = CM.assumeDual(C.getState(), *DV);

  if (!stateNotZero) {
    assert(stateZero);
    reportBug("Division by zero", stateZero, C);
    return;
  }
~~~~~~

# Politics in my analyzer?

Working on, using, and designing these tools is often a game of politics and
nuance, because there is lots of ugly code in the Real World, and lots of
results you never expected.

In general, most will do things like *minimize false positives at the expense
of false negatives*. The observation is that, it is better to ignore a problem
and give a definitive result, as opposed to report things that may be wrong.

But even when you're right, you're wrong: developers do not like you
criticizing their code...

# Static analysis in the Real World

To continue that line of thought, Coverity wrote a great article about their
experience selling static analysis tools. One of the most surprising points was
that **they had to stop reporting real bugs to survive**, because developers
did not believe them!

Indeed, they had many instances of lost customers that would go like this: they
run tool, and it reports error. They complain it is impossible and the tool is
bad. Coverity audits the results and confirm a real bug exists, but their mind
is already made, and they lost clients.

They had to make their tool stupider, because developers got huffy when it said
they were wrong. Static analysis often has to deal with hostile humans as much
as it does hostile code.

(This very often occurred with their data-race detector from what I understand,
which can obviously expose extremely subtle bugs that can even confuse experts.)

# Closing philosophical note

> "There are things that people just do every day they may have done every day
> their entire careers, been working for 20 years, that just are error prone and
> are things that wind up causing you problems as you go forward. When you look
> at these larger teams... millions of lines of code, and so many developers...
> figuring out better practices is really important. I've really gotten behind
> the static code analysis. The Microsoft analysis tools are always on and
> warnings are errors. Can't build the game if it complains about anything.
>
> And even when it's wrong - even when you have something you can prove "it's
> guaranteed OK" - that's actually a good time to step back and say "if you had
> to explain this to me, maybe you should have to change the way it's working."
> If the analyzer can't figure it out, other programmers might not be able to
> figure it out, and that will cause problems too..." - John Carmack
=======
## What this talk is:

## What it isn't:

# Lorem Ipsum

# More Lorem Ipsum
>>>>>>> 77e9b1adff6c950e324bf56ad5edc6840370e88f

# el fin

Any questions? No? Didn't think so.

Slides online: <http://hacks.yi.org/rants.html>

XOXOXOXOXOs, death threats, et cetera: <mad.one@gmail.com> or tweet me `@stdlib`

# el fin

Thanks for listening
