class: middle hide-slide-number

# Building beautiful systems

#### Phoenix Contexts and Domain-Driven Design


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

class: center

#### .left[Axiom #1 🔮]
# Maintaining large systems is difficult*

--
#### * like, super mega difficult

???

As you might know, a good portion of our professional lives as programmers is spent chasing the dragons of complexity

---

class: center

#### .left[Axiom #2 🔮]
# Complex systems are not intentionally messy*

--
#### * for the most part

---

### The bulk of complexity is accidental

Let's ship things fast! 😧

--

Death by a thousand cuts

--

Abstractions that might work for small systems can't scale when the system grows.

---

class: middle center

#### The solution?

# Abstractions

a.k.a. modularization, or encapsulation

???

Science shows we can keep everything between four to seven things in memory at once!

At certain point, you need to introduce abstraction layers

---

## Today:

We will:

- Introduce **Phoenix contexts**
- Build a **Context Map** and use it to introduce Domain-Driven Design concepts
- Apply our learnings in code!

---

class: middle

#### Thanks core team! 👾

# Phoenix Contexts

???

Contexts dropped from the sky in Phoenix 1.3

---

class: middle

## What are contexts?

Elixir modules that group system functionality by business domain

---

class: middle center

# High cohesion

Concepts that belong together are likely to change together as a unit

---

class: middle center

# Loose coupling

Minimal dependencies on external systems

Hide implementation details to outside callers

???

Contexts have minimal dependencies on external systems (like HTTP, Plug, etc)

Contexts hide implementation details from external callers (like Ecto schemas)

---

class: background-color-code

### Follow the scaffold

Create a `User` resource in the `Identity` context

```bash
$ mix phx.gen.html Identity User users \
    name:string email:string
```

---

class: background-color-code

### Domain resource operations in the context

```elixir
# lib/my_app/identity/identity.ex
defmodule MyApp.Identity do
  def get_user!(id), do: ...
  def create_user(attrs \\ %{}) do: ...
  def update_user(%User{} = user, attrs) do: ...
  def delete_user(%User{} = user) do: ...
end
```

---

class: background-color-code

### Web controller for the resource

```elixir
# lib/my_app_web/controllers/user_controller.ex
defmodule MyApp.UserController do
  def index(conn, _params) do
    users = Identity.list_users()
    render(conn, "index.html", users: users)
  end

  def show(conn, %{"id" => id}) do
    user = Identity.get_user!(id)
    render(conn, "show.html", user: user)
  end
end
```

---

class: background-color-code

### Migration for Ecto persistence

```elixir
defmodule MyApp.Repo.Migrations.CreateUsers do
  def change do
    create table(:users) do
      add :name, :string
      add :email, :string

      timestamps()
    end
  end
end
```

---

class: background-color-code

### Ecto schema

```elixir
defmodule MyApp.User do
  use Ecto.Schema

  schema "users" do
    field :name, :string
    field :email, :string
  end
end
```

---

### Contexts, the Phoenix way:

All web concerns live in `MyAppWeb` context.

???

The `Web` context is for anything that deals with the HTTP request/response cycle.
Headers, cookies, you name it.

--

Contexts encapsulate persistence and domain logic

???

Contexts are responsible for the implementation details of business logic.

---

class: middle center

# I have a few questions

---

class: middle center

## What should I name the context?

---

class: middle center

## How should I think about resources that are needed in multiple contexts?

---

class: middle center

## How do I know if it's too broad (coarse) or too specific (fine)?

---

class: middle center

# How do we design system boundaries?


---

class: middle center
name: ddd_book_intro

<div class="bookWrap">
<div class="book">
<img class="cover" alt="Domain-Driven Design Book Cover" src="images/ddd-book-cover.jpg" />
<div class="spine"></div>
</div>
</div>

???

Well in 2003, author Eric Evans came out with a book, Domain-Driven
Design.

---

class: middle center

# Domain-Driven Design

Authored by Eric Evans in 2003

---

class: middle

### DDD is both: 

**High-level strategic design activities**

and

**Concrete software patterns**

--

Don't get tripped up!

???

It can be very confusing as it's got a lot of concepts and
enterprise-speak. Today, we're going to pick and choose a few specific
activities and patterns to outline and apply here

---

class: middle

#### Usually, we reach for

## Horizontal layers of abstractions

* Web layer
* Domain layer
* Persistence layer

---

class: middle center

# Bigger picture

---

class: middle

#### Let's think in terms of

## Vertically decomposed business use cases

---

## Welcome to AutoMaxx!

A used-car marketplace

Sellers bring their cars in for an inspection and appraisal at our stores, where we buy their vehicle.

Interested buyers test drive cars in our lot, purchasing them if they are interested.

???

AutoMaxx is a two-sided marketplace

---

### Your business is constantly changing

* Marketing wants us to change copy on the web site

--
* Finance wants us to change how we do tax calculations on a car sale by region

--
* Operations wants us to build a new feature in the vehicle inventory system

--
* Product has a new initiative to offer purchase transactions in Bitcoin

--
* Customer support wants us to build a better support dashboard

---

class: middle center

# Listen to the business

---

class: middle

#### DDD, summarized 📸

### Design your software systems according to your business domains

by:

#### Paying attention to the language you speak in the business


---

class: center middle

# Listen to the business*

#### *Literally, listen :ear:


---

### Context mapping

An exercise to help us develop deeper insights around what concepts are at play in our organization, and our systems.

---

### Get everyone in a room

Put up on a wall all the:

**Nouns** - entities

**Verbs** - events

---

Diagram

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

### Context Mapping

...is an activity to be done IRL!

Get domain experts together in the room

It should be interactive. Everyone should be talking


---

#### Definition! 📖

### Core domain

The **Core Domain** is the primary area of focus of your business

--

AutoMaxx Core Domain: **Car Sales**

???

Now we're going to get into the nitty gritty of DDD - really elucidating
what defines our domain boundaries. Every software system generally
serves to fulfill its purpose in a certain operational category.

Here at AutoMaxx, we focus on used car sales. If we aren't selling used cars, this company has really lost sight of its long term vision!


---
#### Definition! 📖

### Supporting domains

A **Supporting Domain** (or Subdomain) are the areas of the business
that play roles in making the **Core Domain** happen.

--
* **Online Listings** (put the car on the website)

--
* **Financial Transactions** (charge the buyer, pay the seller)

--
* **Optimization & Analytics** (track business metrics)

--
* **Customer Support** (keep people happy)

???

Now a subdomain is anything that supports the core domain. These are
ancillary functions, but important to help the business do its core
thing properly.

See anything interesting here? Most likely, these domains have a company
unit devoted to them.

In many companies, each of these organizational units have their own
dedicated engineering staff.

---

You might not see these yet, but your diagram may tell you:

---

## Ubiquitous Language

The terms the business speaks within each domain!

Put these in a **Glossary**

---

## What to do with your Ubiquitious Language

Speak about them consistently in the team
Use terms consistently in the code
Keep terms updated together

---

#### Definition! 📖

### Bounded Context

- Concretely: a software system (like a codebase or running application)
- Linguistically: a delineation in your domain where concepts are "bounded", or contained

???

Remember, this is because we agreed that different domains may have
different concepts, and hence different Ubiqutious Languages.

---

## Bounded Contexts allow for precise language

Your domains may use conflicting, overloaded terms with nuances depending on context

Embrace it!

--

Bounded contexts allow these conflicting concepts to coexist


---

Aside: this is important shared vocabulary.

Write this vocabulary down and put it in a Glossary.

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

Let's start simple: one module for each context.

---

class: background-color-code

##### Context: a module with a public interface.

```elixir
defmodule Inspection do
  def get_vehicle() do
  end

  def get_owner() do
  end

  def get_mechanic() do
  end
end
```

---

class: background-color-code

##### ...but also allows callers to perform state-changing actions inside of its boundaries.

```elixir
defmodule Inspection do
  def rate_vehicle(vehicle_id, rating) do
  end
end
```

---

class: background-color-code small-code

##### Let's look at some code in our web app that handles a vehicle rating:

```elixir
defmodule MyApp.VehicleInspectionScoreController do
  def update do
    user = Repo.get_by(User, user_id)
    with vehicle <-
           Repo.get_by(Vehicle, vehicle_id)
           |> Repo.preload(:inspection_score),
         :ok <- InspectionScorePolicy.editable_by?(user) do
      inspection_score =
        vehicle.inspection_score
        |> InspectionScore.changeset(%{score: new_score})
        |> Repo.insert!()
      render(conn, "show.html", inspection_score: inspection_score)
    else
      render(conn, "error.html", message: "Oops!")
    end
  end
end
```

???

Imagine a User -> Vehicle -> InspectionScore


---

class: background-color-code small-code middle

```elixir
defmodule MyApp.VehicleInspectionScoreController do
  # Refactor into modules
  def update do
    with user <- Identity.get_user(user_id),
         vehicle <- Inspection.get_vehicle(vehicle_id),
         {:ok, score} <- Inspection.update_inspection_score_for_user(
           vehicle, user, %{score: new_score}
         ) do
      render(conn, "show.html", inspection_score: score)
    else
      render(conn, "error.html", message: "Oops!")
    end
  end
end
```

---

class: background-color-code small-code

##### Push the code back into your context

```elixir
defmodule Inspection do
  def update_inspection_score_for_user(user, vehicle_id, new_score)
    with :ok <- Inspection.ScorePolicy.editable_by?(user) do
      Repo.get_by(Vehicle, vehicle_id)
      |> Repo.preload(:inspection_score)
      |> Inspection.Score.changeset(new_score)
      |> Repo.insert!()
    else
      # ...
    end
  end
end
```

---

Prefer coarse contexts to fine contexts.

You can optimize, extract later

???

(Counter to Elixir docs)

---

```elixir
# User has many inspections
Repo.get_by(User, id: 1)
|> Repo.preload(:inspections)
```

???

In the past we may have been tempted to cross-join our domains

---

```elixir
Identity.get_user(1)
|> Inspection.fetch_inspections_for_user()
```

Or they can convert concepts between each other:


```elixir
def customer_for_user(user) do
  user
  |> Map.from_struct() # Transform to a bare map
  |> transform_keys()  # Convert to the format
  |> (&struct(Inspection.Customer, &1)).() # stuff it into the domain model
end
```

```elixir
Identity.get_user(1)
|> Inspection.get_customer_for_user()
|> Inspection.fetch_inspections()
```

---

## Convert incoming data to internal concepts

This is known as an **Anti-Corruption Layer**

But don't get too hung up on it. Use it when it's important and the nuances are important
to capture.

---

## Opinion: Avoid cross-context joins if you can

Do you need to do so? If so, go for it.

If you can avoid it, you should.

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

#### Concept: Aggregate Root 

Minimize the data you pass around. Return a tree data structure of data that logically belongs together.

---

DON'T

```elixir
defmodule Inspection do
  def get_vehicle()
  end

  def get_vehicle_rating()
  end
end
```

---

DO

```elixir
defmodule Inspection do
  def get_vehicle()
  end
end

# => %Vehicle{rating: %Rating{}}
```

---

DON'T

```elixir
defmodule Inspection do
  def update_rating(rating_id, new_rating)
  end
end
```

---

DO

```elixir
defmodule Inspection do
  def update_rating(vehicle_id, new_rating)
  end
end
```

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

???

Thanks to coworker Hannah for this idea.

---

For example, we are testing that the Inspection context needs to fetch up a User.

In an old world, we might have used Ecto to pick something up from the repo.

However, in DDD we factored this back behind a different context, so we wire this up with a Mox mock in our test:

---

class: background-color-code

```elixir
defmodule Inspection
  def log_inspection() do
    # ...
    Identity.get_user(user_id)
    |> do_something()
    |> # ...
  end
end
```

---

class: background-color-code

```elixir
# in test
defmodule Inspection.Test do
  setup do
  end

  test "it uses the User to perform an inspection" do
    expect(Identity.Mock)
    |> receive(:get_user)
  end
end
```

---

#### Miscellany

### How do I deal with concepts that live in between boundaries?

---


#### Miscellany

### How do I deal with dependencies between boundaries?

Avoid DB joins, as those couple your contexts together.

Utilize aggregate roots

---

Suggested organization structure:

```
foo_context/
  behaviour.ex
  foo_context.ex
  foo_entity.ex
  foo_bar_entity.ex
```

```
foo_context/
  foo_mock.exs
```

---

class: middle hide-slide-number

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
* Rob Martin - Perhap: Applying DDD and Reactive Architectures: https://www.youtube.com/watch?list=PLqj39LCvnOWZMVugtyKlHMF1o2zPNntFL&time_continue=5&v=kq4qTk18N-c
