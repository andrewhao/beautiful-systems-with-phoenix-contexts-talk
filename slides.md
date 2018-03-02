class: middle hide-slide-number

# Building beautiful systems

#### Phoenix Contexts and Domain-Driven Design

Andrew Hao [@andrewhao](https://www.twitter.com/andrewhao)

[bit.ly/phoenix-contexts](https://bit.ly/phoenix-contexts)

<img src="images/c5-logo-vertical.png" alt="Carbon Five" style="margin-top: -100px; float: right;" height="75" />

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

# Modularization

???

Science shows we can keep everything between four to seven things in memory at once!

At certain point, you need to introduce abstraction layers

---

class: middle

#### A blast from the past 🕰

### Information hiding

[*D.L. Parnas - "On the Criteria to Be Used in Decomposing Systems into Modules"*](https://www.cs.umd.edu/class/spring2003/cmsc838p/Design/criteria.pdf)

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

## Today:

We will:

- Introduce **Phoenix contexts**
- Build a **Context Map** and use it to introduce Domain-Driven Design concepts
- Apply our learnings with **Elixir**!

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

class: background-color-code

### Follow the scaffold

Create a `User` resource in the `Identity` context

```bash
$ mix phx.gen.html Identity User users \
    name:string email:string
```

---

class: background-color-code

### Follow the scaffold

Create a `User` resource in the `Identity` context

```bash
$ mix phx.gen.html `Identity` User users \
    name:string email:string
```

---

class: background-color-code

### Follow the scaffold

Create a `User` resource in the `Identity` context

```bash
$ mix phx.gen.html Identity `User` users \
    name:string email:string
```

---

class: background-color-code

### Follow the scaffold

Create a `User` resource in the `Identity` context

```bash
$ mix phx.gen.html Identity User `users` \
    `name:string email:string`
```

---

class: background-color-code

### Ecto schema

```elixir
# lib/my_app/identity/user.ex
defmodule MyApp.Identity.User do
  use Ecto.Schema

  schema "users" do
    field :name, :string
    field :email, :string

    timestamps()
  end
end
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

### Contexts, the Phoenix way:

All web concerns live in `MyAppWeb` context.

???

The `Web` context is for anything that deals with the HTTP request/response cycle.
Headers, cookies, you name it.

--

Contexts encapsulate persistence and domain logic

???

Contexts are responsible for the implementation details of business logic.

--

The outer context module is the public interface to the rest of the app

---

class: middle center

# 🙋🏻‍♂️ I have a few questions

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

class: middle

#### 💭 Our fundamental question

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
* Adapters

---

class: middle center

# 🌌 Bigger picture

---

class: middle

#### Let's think in terms of

## Vertically decomposed business use cases

---

## Welcome to AutoMaxx! 🚘🚖🚘 

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

--

#### *Literally, listen 🌽👂


---

class: middle

## Context mapping

An exercise to help us **discover all the concepts** living in our **organization**, and our systems.

---

#### Context Mapping

## Get everyone in a room

Put up on a wall all the:

**Nouns** - entities

**Verbs** - events

---

class: background-image-contain

background-image: url(images/context-map-1.png)

---

#### Context Mapping

## Group like concepts and actions together.

There may be overlaps - that's OK if your concepts belong in multiple groups.

More important to just get it down.

---

class: background-image-contain

background-image: url(images/context-map-2.png)

---

#### Context Mapping

## Take a step back.

Natural clusters and groupings should be emerging

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

class: background-image-contain

background-image: url(images/context-map-2.png)

---

class: background-image-contain

background-image: url(images/context-map-3.png)

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

class: background-image-contain

background-image: url(images/context-map-3.png)

---

class: background-image-contain

background-image: url(images/context-map-4.png)

---

class: middle

#### 👏 Take a step back

## You've discovered your domains!

The **Context Map** points us to our business domains!

These may map to real organization structures (Conway's Law)

---


## Ubiquitous Language

The terms the business speaks within each domain!

* Put these in a **Glossary**
* Speak about them consistently in the team
* Use terms consistently in the code

---

#### Definition! 📖

### Bounded Context

- Concretely: a software system (like a codebase or running application)
- Linguistically: a delineation in your domain where concepts are "bounded", or contained

???

Remember, this is because we agreed that different domains may have
different concepts, and hence different Ubiqutious Languages.

---

class: middle

#### For example:

A `Rating` has a specific meaning in the **Inspection** subdomain (a vehicle health score)

...but it means something else in the **Customer Support** subdomain (a customer support experience score)

---

class: middle

#### For example:

A `User` in the **Identity** context is--

A `Mechanic` in the **Inspection** context, is--

A `Seller` or `Buyer` in the **Financial Transaction** context

---

#### Bounded Contexts

## Precise language, precise models

Your domains may now use their own specific, nuanced terminology.

Powerful modeling tool, and keeps your product in sync with the business stakeholders

---

class: middle

#### 👔 In the I'm Being Serious Department

## Become Language Addicts

Internalize your domain!

???

If there's one thing that DDD tells us, it's to pay attention to the language, and the organization will follow.

---

background-image: url(images/yoda.jpg)

---

class: middle center

# Elixir time! 💁‍🙋‍♂️

---

class: background-color-code

### Contexts can expose domain entities

```elixir
defmodule AutoMaxx.Inspection do
  def get_`vehicle`!() do: ...
  def list_`vehicles`() do: ...
  def update_`vehicle`() do: ...

  def get_`mechanic`!() do: ...

  #...
end
```

---

class: background-color-code

### Contexts can expose methods that perform domain actions

```elixir
defmodule AutoMaxx.Inspection do
  def `add_vehicle_to_garage_queue`(vehicle) do: ...
  def `rate_vehicle`(vehicle, mechanic, inspection_rating) do: ...
end
```

---

class: background-image-contain background-white

background-image: url(images/inspection-ui.png)

---

class: background-color-code small-code

```elixir
defmodule AutoMaxx.VehicleInspectionRatingController do
  def update(conn, %{
        user_id: user_id, vehicle_id: vehicle_id, rating: new_rating
      }) do
    with user <- Repo.get_by(User, user_id),
         vehicle <-
           Repo.get_by(Vehicle, vehicle_id)
           |> Repo.preload(:rating),
         :ok <- InspectionRatingPolicy.editable_by?(vehicle, user) do
      inspection_rating =
        vehicle.rating
        |> rating.changeset(%{value: new_rating})
        |> Repo.insert!()

      render(conn, "show.html", inspection_rating: inspection_rating)
    else
      render(conn, "error.html", message: "Oops!")
    end
  end
end
```

---

class: background-color-code small-code

```elixir
defmodule AutoMaxx.VehicleInspectionRatingController do
  def update(conn, %{
        user_id: user_id, vehicle_id: vehicle_id, rating: new_rating
      }) do
*   with user <- Repo.get_by(User, user_id),
*        vehicle <-
*          Repo.get_by(Vehicle, vehicle_id)
*          |> Repo.preload(:rating),
         :ok <- InspectionRatingPolicy.editable_by?(vehicle, user) do
*     inspection_rating =
*       vehicle.rating
*       |> rating.changeset(%{value: new_rating})
*       |> Repo.insert!()

      render(conn, "show.html", inspection_rating: inspection_rating)
    else
      render(conn, "error.html", message: "Oops!")
    end
  end
end
```

---

class: background-color-code small-code

```elixir
defmodule AutoMaxx.VehicleInspectionRatingController do
  def update(conn, %{
        user_id: user_id, vehicle_id: vehicle_id, rating: new_rating
      }) do
    with user <- Repo.get_by(User, user_id),
         vehicle <-
           Repo.get_by(Vehicle, vehicle_id)
           |> Repo.preload(:rating),
*        :ok <- InspectionRatingPolicy.editable_by?(vehicle, user) do
      inspection_rating =
        vehicle.rating
        |> rating.changeset(%{value: new_rating})
        |> Repo.insert!()

      render(conn, "show.html", inspection_rating: inspection_rating)
    else
      render(conn, "error.html", message: "Oops!")
    end
  end
end
```

---

class: background-color-code small-code

##### Move domain entities into the `Identity` context

```elixir
# lib/automaxx/identity/user.ex
defmodule AutoMaxx.Identity.User do
  schema "users" do: ...
end
```

##### Add domain action to the `Identity` context API

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def get_user(user_id) do
    Repo.get_by(AutoMaxx.Identity.User, id: user_id)
  end
end
```

---

class: background-color-code small-code

##### Move domain entities into the `Inspection` context

```elixir
# lib/automaxx/inspection/vehicle.ex
defmodule AutoMaxx.Inspection.Vehicle do
  schema "vehicles" do
    belongs_to :owner, AutoMaxx.Identity.User
    has_one    :rating, AutoMaxx.Inspection.Rating
  end
end

# lib/automaxx/inspection/rating.ex
defmodule AutoMaxx.Inspection.Rating do
  schema "inspection_ratings" do
    belongs_to :vehicle, AutoMaxx.Inspection.Vehicle
    belongs_to :rated_by, AutoMaxx.Identity.User
  end
end
```

---

class: background-color-code small-code middle

##### Add domain action to the `Inspection` context API

```elixir
defmodule AutoMaxx.Inspection do
  def rate_vehicle(vehicle, mechanic, inspection_rating) do
    with :ok <- Inspection.RatingPolicy.editable_by?(vehicle, mechanic) do
      Repo.get_by(Inspection.Vehicle, vehicle_id)
      |> Repo.preload(:rating)
      |> Inspection.Rating.changeset(%{
        rated_by: mechanic.id,
        value: inspection_rating
      })
      |> Repo.insert!()
    else
      {:auth_error, "Unauthorized"}
    end
  end
end
```

---

class: background-color-code small-code middle

##### Simplify the controller with your new context APIs

```elixir
defmodule AutoMaxx.VehicleInspectionRatingController do
  def update(conn, %{
    user_id: user_id, vehicle_id: vehicle_id, rating: new_rating
  }) do
    with user <- `Identity.get_user(user_id)`,
         vehicle <- `Inspection.get_vehicle(vehicle_id)`,
         {:ok, rating} <- `Inspection.rate_vehicle(vehicle, user, new_rating)` do
      render(conn, "show.html", inspection_rating: rating)
    else
      render(conn, "error.html", message: "Oops!")
    end
  end
end
```

---

class: middle center

# Way better.

---


### What did you notice here?

* Contexts only expose methods at their outer layer.
* Contexts hide internal implementations.
* Use domain language when naming actions, entities and concepts.

???

`rate` versus `update_inspection_rating`

---

class: middle center

# Concept sharing

Or: Double Trouble

---

class: middle

### What happens when we need to use a `User` in a different context?

**`Identity` domain**: `User`

**`Inspection` domain**: `Owner`, `Mechanic`

**`Marketing` domain**: `Visitor`

---

### A few options:

1. **Direct Usage (Do Nothing)**: Just directly use schemas between contexts

---

#### Sharing Concepts

## Direct Usage (Do Nothing)

Mix domain models between contexts

```elixir
defmodule AutoMaxx.Inspection do
  def rate_vehicle(%AutoMaxx.`Identity`.User{} = user,
                   vehicle, rating) do: ...
end
```

Maybe it's OK if you're refactoring, but I discourage this...

---

### A few options:

1. **Direct Usage (Do Nothing)**: Just directly use schemas between contexts

1. **Struct Conversion**: convert to internal concepts at the boundaries with pure structs


---


#### Sharing Concepts

## Struct Conversion

Convert external concepts to internal concepts

Useful for **Read-Only** use cases

---

##### Convert a `Identity.User` to a `Marketing.Visitor`

```elixir
defmodule AutoMaxx.Marketing.Visitor do
  defstruct [:handle, :uuid]
end
```

---

##### Convert a `Identity.User` to a `Marketing.Visitor`

```elixir
defmodule AutoMaxx.`Marketing.Visitor` do
  defstruct [:handle, :uuid]
end

defmodule AutoMaxx.Marketing do
  def visitor_for_user(%AutoMaxx.Identity.User{} = user) do
    new_mapping = user
      |> Map.from_struct()
      |> Map.delete(:email)
      |> Map.put(:handle, user.email)
    struct(AutoMaxx.`Marketing.Visitor`, new_mapping)
  end
end
```

---

##### Convert a `Identity.User` to a `Marketing.Visitor`

```elixir
defmodule AutoMaxx.Marketing.Visitor do
  defstruct [:handle, :uuid]
end

defmodule AutoMaxx.Marketing do
  def visitor_for_user(`%AutoMaxx.Identity.User{}` = user) do
    new_mapping = user
      |> Map.from_struct()
      |> Map.delete(:email)
      |> Map.put(:handle, user.email)
    struct(AutoMaxx.Marketing.Visitor, new_mapping)
  end
end
```

---

class: small-code middle

```elixir
defmodule AutoMaxx.Marketing.EmailSubscriptionController do
  def create(conn, %{user_id: user_id, payload: payload}) do
    Identity.get_user(user_id)

    # Convert the concept at the boundaries
    |> Marketing.visitor_for_user()

    # Then proceed to perform the domain action
    |> Marketing.subscribe_visitor_to_mailchimp(%{payload: payload})
  end
end
```

---

class: small-code middle

```elixir
defmodule AutoMaxx.Marketing.EmailSubscriptionController do
  def create(conn, %{user_id: user_id, payload: payload}) do
*   Identity.get_user(user_id)

    # Convert the concept at the boundaries
    |> Marketing.visitor_for_user()

    # Then proceed to perform the domain action
    |> Marketing.subscribe_visitor_to_mailchimp(%{payload: payload})
  end
end
```

---

class: small-code middle

```elixir
defmodule AutoMaxx.Marketing.EmailSubscriptionController do
  def create(conn, %{user_id: user_id, payload: payload}) do
    Identity.get_user(user_id)

    # Convert the concept at the boundaries
*   |> Marketing.visitor_for_user()

    # Then proceed to perform the domain action
    |> Marketing.subscribe_visitor_to_mailchimp(%{payload: payload})
  end
end
```

---

class: small-code middle

```elixir
defmodule AutoMaxx.Marketing.EmailSubscriptionController do
  def create(conn, %{user_id: user_id, payload: payload}) do
    Identity.get_user(user_id)

    # Convert the concept at the boundaries
    |> Marketing.visitor_for_user()

    # Then proceed to perform the domain action
*   |> Marketing.subscribe_visitor_to_mailchimp(%{payload: payload})
  end
end
```

---

### Pattern matching and types

Strict type guarantees with pattern matching

```elixir
def subscribe_visitor_to_mailchimp(%Visitor{} = visitor) do ...
def visitor_for_user(%User{} = user) do...
```

--

Even better - leverage the powers of typespecs.

```elixir
@spec subscribe_visitor_to_mailchimp(%Visitor{}) :: boolean
@spec visitor_for_user(%User{}) :: %Visitor{}
```

---

### A few options:

1. **Direct Usage (Do Nothing)**: Just directly use schemas between contexts

1. **Struct Conversion**: convert to internal concepts at the boundaries with pure structs

1. **Collaborator Schema**: Create an internal schema persisted in Ecto that uses a reference to the external schema


---

#### Sharing Concepts

## Collaborator Schema

Useful for **Read+Write** use cases

--
```elixir
defmodule AutoMaxx.Inspection.Mechanic do
  schema "mechanics" do
    field :is_contractor, :boolean
    field :certification, :string
    belongs_to :user, AutoMaxx.Identity.User
  end
end
```

---

class: small-code middle

```elixir
defmodule AutoMaxx.Inspection do
  def create_mechanic_from_user(
*       %AutoMaxx.Identity.User{} = user,
        is_contractor
      ) do
*   %AutoMaxx.Inspection.Mechanic{
*     is_contractor: is_contractor,
*     user: user
*   }
*   |> Repo.insert!()
  end

  def mechanic_for_user(user) do
    Repo.get_by(AutoMaxx.Inspection.Mechanic, user_id: user.id)
  end
end
```

---

class: middle

#### Sharing Concepts

## This is known as an **Anti-Corruption Layer**

???

But don't get too hung up on it. Use it when it's important and the nuances are important
to capture.


---

#### Sharing Concepts

### Avoid cross-context joins if you can

```elixir
defmodule AutoMaxx.Marketing.Visitor do:
  schema "marketing_visitors" do:
    belongs_to :user, AutoMaxx.Identity.User
```
--
Instead, store them as external references

```elixir
    field :user_id, :string
    field :user_id, :uuid
```

???

This decouples our two domains.

---

class: middle center

# Aggregates

Data that belongs together

---

class: middle center

## We belong together!

Look for data that naturally falls into groups

???

Minimize the data you pass around. Return a tree data structure of data that logically belongs together.


---

background-image: url('images/without-aggregate-root.png')

---

background-image: url('images/aggregate-root.png')

---

##### DON'T

```elixir
defmodule AutoMaxx.Inspection do
  def get_vehicle() do: # => %Vehicle{}
  def get_vehicle_rating() do: # => %Rating{}
  def get_vehicle_list_price() do: # => %ListPrice{}
end
```
--
##### DO

```elixir
defmodule AutoMaxx.Inspection do
  def get_vehicle() do:
end
# => %Vehicle{rating: %Rating{}, list_price: %ListPrice{}}
```

---

##### DON'T

```elixir
defmodule AutoMaxx.Inspection do
  def update_rating(`rating_id`, new_rating) do...
end
```
--
##### DO

```elixir
defmodule AutoMaxx.Inspection do
  def rate_vehicle(`vehicle_id`, new_rating) do: ...
end
```

---

### Leverage Aggregate Roots

Expose only **Aggregate Roots** in your context.

Passing Aggregates around simplifies your APIs

---

class: middle center

# Event-driven messaging

Decouple your contexts even further

???

In purer forms of DDD, you would emit events over a message bus and listeners would bind to their callbacks if they are interested.

---

## Where is the coupling here?

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def create_user(...) do
    do_create_user()
    |> AutoMaxx.Inspection.maybe_create_mechanic()
    |> AutoMaxx.Marketing.subscribe_user_to_email_list()
    |> AutoMaxx.Analytics.track_event('user_created')
  end
end
```

---

## Where is the coupling here?

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def create_user(...) do
    do_create_user()
*   |> AutoMaxx.Inspection.maybe_create_mechanic()
    |> AutoMaxx.Marketing.subscribe_user_to_email_list()
    |> AutoMaxx.Analytics.track_event('user_created')
  end
end
```

---

## Where is the coupling here?

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def create_user(...) do
    do_create_user()
    |> AutoMaxx.Inspection.maybe_create_mechanic()
*   |> AutoMaxx.Marketing.subscribe_user_to_email_list()
    |> AutoMaxx.Analytics.track_event('user_created')
  end
end
```

---

## Where is the coupling here?

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def create_user(...) do
    do_create_user()
    |> AutoMaxx.Inspection.maybe_create_mechanic()
    |> AutoMaxx.Marketing.subscribe_user_to_email_list()
*   |> AutoMaxx.Analytics.track_event('user_created')
  end
end
```

---
class: background-image-contain

background-image: url('images/event-bus-one.png')

---

## Publish facts (domain events) over a bus

Interested contexts can subscribe to domain events

--

Refer back to your **Context Map** and **Glossary** where your business stakeholders helped you define events!

---

##### Publisher:

Identity context publishes `identity.user.created`

##### Subscribers:

Marketing context puts the user on the email marketing list

Inspection context creates a corresponding `Mechanic` if the user is one

Analytics context publishes metric to Google Analytics

---
class: background-image-contain

background-image: url('images/event-bus-two.png')

---

class: middle

## Using the `event_bus` library

Github: [otobus/event_bus](https://github.com/otobus/event_bus)

Or any other system that gives you pub/sub capabilities

---

class: small-code middle

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def create_user(...) do
    do_create_user()
    |> publish_event(:"identity.user.created")
  end

  use EventBus.EventSource

  defp publish_event(%User{} = user, event_name) do
    EventSource.notify %{id: UUID.uuid4(), topic: event_name} do
      %{user: user}
    end
  end
end
```

---

class: small-code middle

```elixir
# lib/automaxx/identity/identity.ex
defmodule AutoMaxx.Identity do
  def create_user(...) do
    do_create_user()
    |> `publish_event(:"identity.user.created")`
  end

  use EventBus.EventSource

* defp publish_event(%User{} = user, event_name) do
*   EventSource.notify %{id: UUID.uuid4(), topic: event_name} do
*     %{user: user}
*   end
* end
end
```

---

class: small-code middle

##### In each context, create an `EventHandler` module.

```elixir
# lib/automaxx/marketing/event_handler.ex
defmodule AutoMaxx.Marketing.EventHandler do
  def process({:"identity.user.created" = event_name, event_id}) do
    %{data: %{user: user}} = EventBus.fetch_event({event_name, event_id})

    AutoMaxx.Marketing.subscribe_user_to_email_list(user)
  end

  def process({:"some.other.event" = event_name, event_id}) do: ...
end
```

---

class: small-code middle

##### In each context, create an `EventHandler` module.

```elixir
# lib/automaxx/marketing/event_handler.ex
defmodule AutoMaxx.Marketing.EventHandler do
* def process({:"identity.user.created" = event_name, event_id}) do
    %{data: %{user: user}} = EventBus.fetch_event({event_name, event_id})

*   AutoMaxx.Marketing.subscribe_user_to_email_list(user)
  end

  def process({:"some.other.event" = event_name, event_id}) do: ...
end
```

---

class: small-code middle

```elixir
# lib/automaxx/inspection/event_handler.ex
defmodule AutoMaxx.Inspection.EventHandler do
  def process({:"identity.user.created" = event_name, event_id}) do
    %{data: %{user: user}} = EventBus.fetch_event({event_name, event_id})

    AutoMaxx.Inspection.create_mechanic_from_user(user)
  end

  def process({:"yet.another.event" = event_name, event_id}) do: ...
end
```

---

class: middle center

# Caveats!

---

class: middle

#### Caveats

## Start small

Don't overdo it. Refactor things into one context at a time.

---

class: middle

#### Caveats

## DDD concepts first, then patterns

Start with the Context Map. Make sure you understand the core concepts about linguistic clarity.

Keep the team and the code speaking the same language!

---

class: middle

#### Caveats

## For systems of a certain scale

If you have a small-scale system, this might be overkill!

---

class: middle center

# In conclusion

---

class: middle center

## Listen to the business

(Build a **Context Map** and a **Glossary**)

---

class: middle center

## Strict boundaries

**Hide internal details** behind a context

---

class: middle center

## Linguistic precision & domain clarity

**Embrace the nuances** of our domain models

---

class: middle center

## Practical architecture tools

Concept sharing & anti-corruption layers

Ship aggregates around between contexts

Use an event bus

---

class: middle center

## Used the full power of the BEAM

Lean on message passing, OTP, pattern matching and typespecs

---

class: middle center

# That's what I think makes beautiful systems

---

class: middle hide-slide-number

## Thanks!

👾 Github: [andrewhao](https://www.github.com/andrewhao)

🐦 Twitter: [@andrewhao](https://www.twitter.com/andrewhao)

📬 Email: [andrew@carbonfive.com](mailto:andrew@carbonfive.com)

&nbsp;

<img src="images/c5-logo-vertical.png" alt="Carbon Five" style="float: right;" height="75" />

---

#### Credits & Prior Art

* Evans, Eric. [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215).
* Gorodinski, Lev. ["Sub-domains and Bounded Contexts in Domain-Driven Design (DDD)"](http://gorodinski.com/blog/2013/04/29/sub-domains-and-bounded-contexts-in-domain-driven-design-ddd/).
* Hagemann, Stephan. [Component-Based Rails Applications](https://leanpub.com/cbra).
* Parnas, D.L. ["On the Criteria To Be Used in Decomposing Systems into Modules"](http://www.cs.umd.edu/class/spring2003/cmsc838p/Design/criteria.pdf).
* Vernon, Vaughan. [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577).

---

#### Credits & Prior Art

* W. P. Stevens ; G. J. Myers ; L. L. Constantine. ["Structured Design"](http://ieeexplore.ieee.org/document/5388187/) - IBM Systems Journal, Vol 13 Issue 2, 1974.
* Steinegger, Giessler, Hippchen, Abeck. [Overview of a Domain-Driven Design Approach to Build Microservice-Based Applications](https://cm.tm.kit.edu/download/domain_driven_microservice-architecture_17-03-15.pdf)
* Rob Martin - Perhap: Applying DDD and Reactive Architectures: https://www.youtube.com/watch?list=PLqj39LCvnOWZMVugtyKlHMF1o2zPNntFL&time_continue=5&v=kq4qTk18N-c
