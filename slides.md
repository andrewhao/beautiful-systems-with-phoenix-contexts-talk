class: middle

# Building beautiful systems

#### Phoenix Contexts and Domain-Driven Design

<br />

Andrew Hao [@andrewhao](https://www.twitter.com/andrewhao)

<img src="images/c5-logo-white.svg" alt="Carbon Five" style="float: right; margin-top: -10px;" height="75" />

---

class: middle center

## Hi, I'm Andrew

Friendly neighborhood programmer at Carbon Five

---

class: middle center background-color-carbonfive

<img src="images/c5-logo-white.svg" alt="Carbon Five" height="200" />

---

class: middle

#### Today's question

### What are contexts, and how do I use them effectively?

???

What are contexts? After all, they can be confusing. We've got this organization tool, but how do we create and use them effectively?

---

#### Complexity is hard

### Large software systems can get messy.

--
coupling.

--

slow.

--

hidden dependencies.

--

can't keep it in your head.

???

As you may be well aware, large systems are difficult to grow and maintain.

---

### Complex systems are hard to grow

* not without introducing intermediary organization structures.

--

Science shows we can keep everything between four to seven things in memory at once!

At certain point, you need to introduce abstraction layers

---

### Beautiful systems are well-organized

Loosely-coupled, cohesive units

---

class: middle center

### ok so microservices?

whoa there! hang on...

???

So maybe a couple of years ago during the peak of the microservices hype cycle, we would automatically think to ourselves, oh great, let's make a microservice!

But maybe it wasn't such a great idea. Because with microservices, you get extra complexity...

---

background-image: url(/images/niklas-hamann-418782.jpg)

???

And that complexity makes things very sad.

---

class: middle center

Oh, I know! Phoenix gives us contexts!

I'll organize my code in Contexts!

???

Well the great thing is that with Phoenix we get this natural organization structure in the context.

---

class: middle

#### From on high 👾

## Phoenix Contexts

Thanks, Chris!

???

Contexts dropped from the sky in Phoenix 1.3

---

### How do contexts work?

Modules that organize like functionality together

By default, Phoenix apps get a `Web` context and another blank context for everything else.

???

In many ways, contexts are simple concepts. They're just modules that package subsystems together.

---

### But wait, not so fast.

* What should I name the context?
* How do I know if it's too broad (coarse) or too specific (fine)?
* Where are good module boundaries to draw?

???

Lots of questions remain. What are effective ways of organizing my code?

And as it turns out, this same set of questions that plagues us with contexts is the exact same set of questions that plague us when we turned to microservices a few years ago.

Where do we draw the boundaries?

---

class: middle

#### Same basic question

### Where do we draw system boundaries?

---

class: middle center

# 🤔💭

If only we had a set of criteria to understand how to decompose our systems!

---

class: middle

#### A blast from the past 🕰

### Information hiding

[_D.L. Parnas - "On the Criteria to Be Used in Decomposing Systems into Modules"_](https://www.cs.umd.edu/class/spring2003/cmsc838p/Design/criteria.pdf)

???

So I went looking for some inspiration, and learned about Parnas' work
about modularization in the 1970s, words that stand still today.

---

class: middle center background-image-contain background-white

background-image: url(images/parnas-paper.png)

???

In this paper, he took a look at a program that did text processing and
compared two approaches - one that divided up its processing
responsibilities by its procedural components, do A, B, then C, and
another one that was responsible for the individual design decisions to
do various things like scan for words, storing data in internal data
structures, etc.

It might go without saying, but the second one was found much more
flexible and elegant. He said that the second way allowed the user to
change internal decisions like how lines were stored in memory or on
disk in a particular data structure would require minimal changes,
whereas in the first, if you changed the data structure backing the word
list you'd have to change several modules.

---

class: middle

"We propose instead that one begins with a **list of difficult design decisions** or **design decisions which are likely to change**.

"Each module is then designed to **hide such a decision from the others.**" (Emphasis added)

???

His point was that modules need to hide away minute implementation
details from other modules - things that are likely to change! Where you
draw the boundaries in your system matters, because they allow you to
flexibly change your approach without messing with other parts of the
system later.

Sounds obvious, right? This paper eventually got the discussion rolling
into the design principles which we call "High Cohesion, Loose Coupling"
today.

---

class: middle center

Where are the

## difficult design decisions

that are

## likely to change?

???

Even though Parnas' paper looked at a single program and attempted to
rethink its division of labor from flow to responsibility, I want to
zoom out and apply this to our entire business system.

Where do those difficult design decisions come from, within our company?

--

#### Within the business groups that generate them!

---

### Your business is a driver for change

* Marketing wants us to change copy on the web site

--

* Finance wants us to change how we do tax calculations per transaction

--

* Operations wants us to build a new shipment tracking system

--

* Product wants to implement a new integration with Vendor XYZ

--

* Customer support wants us to build a better support dashboard

---

class: middle center

### Huh! The key to building beautiful systems is understanding your business partner(s)

---

class: middle center
name: ddd_book_intro

<div class="bookWrap">
<div class="book">
<img class="cover" alt="Domain-Driven Design Book Cover" src="images/ddd-book-cover.jpg" />
<div class="spine"></div>
</div>
</div>
<script>
console.log('here');
</script>

???

Well in 2003, author Eric Evans came out with a book, Domain-Driven
Design.

---

class: middle

## Introducing Domain-Driven Design

Published by Eric Evans in 2003

DDD is both a set of high-level strategic design activities and concrete software patterns

???

It can be very confusing as it's got a lot of concepts and
enterprise-speak. Today, we're going to pick and choose a few specific
activities and patterns to outline and apply here

---

class: middle

#### Summarized:

### Effective software design requires linguistic clarity

---

## Today:

We will build a **Context Map** and use it to introduce DDD concepts

We will learn what **Phoenix contexts** are and how to best use them

We will learn some **refactoring patterns** we can use to shape our
systems

---

### Context mapping

An exercise to help us develop deeper insights around what concepts are at play in our organization, and our systems.

--

Nouns

Verbs

---

### Context Mapping

...is an activity to be done IRL!

Get domain experts together in the room

It should be interactive. Everyone should be talking

---

Aside: this is important shared vocabulary.

Write this vocabulary down and put it in a Glossary.

---

It might look like this:

PNG physical whiteboard

---

Or it might look like this:

PNG stickies

---

### Context Mapping

Group like concepts and actions together.

There may be overlaps - that's OK if your concepts belong in multiple groups.

More important to just get it down.

---

### Context Mapping

Take a step back.

PNG grouped stickies screenshot

---

This activity is closely related to a few other activities:

Domain modeling
Event Storming
Story Mapping

--

Answering: what happens to what entities in the system?

---

I want you to become Language Addicts

Internalize your domain!

Language Zealots?

If there's one thing that DDD tells us, it's to pay attention to the language, and the organization will follow.

---

Yoda gif

---

## Elixir time!

---

OK so back to that question - how to properly use Phoenix contexts?

---

A context is a self-contained system. A module!

> Quote from Context docs

---

In other words, it should be a part of the system that "hides information" about the specific responsibilities it has been given.

Even more ideally, it should be closely aligned with your organization structure. For example, the Marketing domain may surface the code required to implement the microsite or landing page experiments.

---

class: background-color-code

#### Context: a module with a public interface. it reveals data to callers---

```elixir
defmodule Inspection do
end
Inspection.fetch_vehicle(...)
Identity.fetch_user(...)
```

---

...but also allows callers to perform state-changing actions inside of its boundaries.

```elixir
Scheduling.create_appointment(...)
Identity.create_session(...)
SaleFinalization.process_electronic_payment(...)
```

---

Prefer coarse contexts to fine contexts.

You can optimize, extract later

(Counter to Elixir docs)

---

The beauty of Elixir is communicating between contexts:

In-process synchronous communication?
Inter-process synchronous?
Inter-process async?

You get to choose with the power of OTP!

---

Async:

Task.perform_async

---

Sync:

GenServer.start_link()

---

Concept: Aggregate Root

Minimize the data you pass around. Return a tree data structure of data that logically belongs together.

---

This minimizes the amount of CRUD actions you must implement

This also makes your system easier to reason about.

---

Advanced: Event-driven messaging styles between contexts

In purer forms of DDD, you would emit events over a message bus and listeners would bind to their callbacks if they are interested.

---

Async, fault-tolerant approaches:

Return "job" UUID, then poll for completion.

---

Testing across boundaries:

Don't!

--

Use Mox to implement context mocks

Thanks to coworker Hannah for this idea.

---

...Mox examples

---

class: middle

## Thanks!

👾 Github: [andrewhao](https://www.github.com/andrewhao)

🐦 Twitter: [@andrewhao](https://www.twitter.com/andrewhao)

📬 Email: [andrew@carbonfive.com](mailto:andrew@carbonfive.com)

&nbsp;

<img src="images/c5-logo-white.svg" alt="Carbon Five" style="float: right;" height="75" />

---

#### Credits & Prior Art

* Evans, Eric. [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215).
* Gorodinski, Lev. ["Sub-domains and Bounded Contexts in Domain-Driven Design (DDD)"](http://gorodinski.com/blog/2013/04/29/sub-domains-and-bounded-contexts-in-domain-driven-design-ddd/).
* Hagemann, Stephan. [Component-Based Rails Applications](https://leanpub.com/cbra).
* Parnas, D.L. ["On the Criteria To Be Used in Decomposing Systems into Modules"](http://www.cs.umd.edu/class/spring2003/cmsc838p/Design/criteria.pdf).
* Vernon, Vaughan. [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577).
* W. P. Stevens ; G. J. Myers ; L. L. Constantine. ["Structured Design"](http://ieeexplore.ieee.org/document/5388187/) - IBM Systems Journal, Vol 13 Issue 2, 1974.
* Steinegger, Giessler, Hippchen, Abeck. [Overview of a Domain-Driven Design Approach to Build Microservice-Based Applications](https://cm.tm.kit.edu/download/domain_driven_microservice-architecture_17-03-15.pdf)
