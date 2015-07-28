---
title:  "Why I am excited about Elixir"
categories: english code
---

For more than five years I've been a proficient Rubyist, working with [Ruby](http://www.ruby-lang.org/) [day](http://www.sublimevideo.net) and [night](https://github.com/guard). For one year and a half now I'm [working with PHP](https://www.dailymotion.com). PHP was actually my first Web programming language, back in 2004. I'm ok to work with PHP but I really miss the excitment of using a more developer-friendly language like Ruby.

### Exploring new horizons

The good side of not working with Ruby anymore is that I'm opening to new languages: [Go](http://golang.org/), [Rust](http://www.rust-lang.org/) and [Elixir](http://elixir-lang.org/).

I've followed their "Getting started" guides but the one that draw my attention the most was Elixir!

### A promising tag line

<div class="left">
  <img class="left" src="https://upload.wikimedia.org/wikipedia/en/a/a4/Elixir_programming_language_logo.png" title="Elixir language logo" alt="Elixir language logo" />
</div>

I must admit I first heard of Elixir from [José Valim](https://github.com/josevalim), which is [a much respected guy](http://plataformatec.com.br/) in the Ruby community. He started Elixir as [an experiment](http://www.infoq.com/interviews/valim-elixir) and it's now one of the best general-purpose programming language, in my opinion.

> Elixir is a dynamic, functional language designed for building scalable and maintainable applications.

With Elixir, I will be able to:

- still enjoy a dynamic language, with a Ruby-like syntax, as a bonus;
- embrace functional programming, which is a new paradigm for me;
- build scalable and maintainable applications.

### A dynamic and elegant language

Elixir leverages the [Erlang VM](http://www.erlang.org/) and its performance but without the [weird syntax](http://damienkatz.net/2008/03/what_sucks_abou.html).
Even better, it has a nice syntax, quite similar to Ruby's one!
I must admit, this is very important to me (and to every Rubyists, really)!

Following are some examples of Elixir code.

**Protocols are a mechanism to achieve polymorphism in Elixir:**

```elixir
# Note the special module attribute "@doc" here
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end

# Integers are never blank
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# Just empty list is blank
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end
```

**Elixir has a nice interactive shell:**

```iex
# this is similar to Ruby's %w() syntax
iex> ~s(this is a string with "double" quotes, not 'single' ones)
"this is a string with \"double\" quotes, not 'single' ones"

# this is similar to Ruby's %i() syntax
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

**An example that uses [the Ecto library](https://github.com/elixir-lang/ecto) which provides an elegant DSL for writing database queries:**

```elixir
query = from w in Weather,
        where: w.prcp > 0,
        where: w.temp < 20,
        select: w
```

### A functional language

I've always thought [object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming) was *the only right way* to code. Seeing that Elixir is a [functional programming language](https://en.wikipedia.org/wiki/Functional_programming) made me change my mind about that.

Functional programming has many benefits, but the ones that interest me the most are the facts that it avoids changing-state and mutable data. This leads to removing side effects, which is a huge relief!

#### Pattern matching

One thing I already love is pattern matching. It allows destructuring and is a common pattern in Elixir:

```elixir
case File.read "hello" do
  {:ok, body}      -> IO.puts "Success: #{body}"
  {:error, reason} -> IO.puts "Error: #{reason}"
end
```

Here, instead of `try/catch` the `File.read` call as we could do in other languages (actually Elixir supports it too), we can simply define two clauses that would match the two possible outcomes of the function call.

Another example is pattern matching used as filter in a [comprehension](http://elixir-lang.org/getting-started/comprehensions.html):

```iex
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

Here, for each tuple of `values` which match the pattern `{:good, n}`, we multiply `n` by itself.

#### Processes

Elixir is built on [processes](http://elixir-lang.org/getting-started/processes.html) which are extremely lightweight in terms of memory and CPU:

> Processes are isolated from each other, run concurrent to one another and communicate via message passing. Processes are not only the basis for concurrency in Elixir, but they also provide the means for building distributed and fault-tolerant programs.

And it's as simple as that:

```elixir
parent = self()

# Spawns an Elixir process (not an operating system one!)
spawn_link(fn ->
  send parent, {:msg, "hello world"}
end)

# Block until the message is received
receive do
  {:msg, contents} -> IO.puts contents
end
```

Note the `{:msg, contents}` pattern matching again!

#### Module attributes

Another interesting feature is [module attributes](http://elixir-lang.org/getting-started/module-attributes.html). Then can be used to document a module or a function, but also to specify [behaviours](http://elixir-lang.org/getting-started/typespecs-and-behaviours.html#behaviours) that a module implements (behaviours are similar to interfaces in object oriented languages), or even [to store data](http://elixir-lang.org/getting-started/module-attributes.html#as-temporary-storage) during compilation!

```elixir
defmodule Math do
  @moduledoc """
  Provides math-related functions.

  ## Examples

      iex> Math.sum(1, 2)
      3

  """

  @doc """
  Calculates the sum of two numbers.
  """
  def sum(a, b), do: a + b
end
```

### Scalability, maintainability

Since Erlang was *conceived* with high availability and scalability in mind, Elixir which sits on top of it inherits (in a functional programming way of course ^^) Erlang's kick-ass performance!

The guys at Oozou [wrote a great summary](http://blog.oozou.com/why-we-are-excited-about-elixir/) of some of the powerful features that makes Erlang (and Elixir) so great.

### A cure for a great future

Elixir has an awesome [Getting Started](http://elixir-lang.org/getting-started/introduction.html) guide and I couldn't recommend you enough to follow it.

I think the next steps in my Elixir journey will be:

- read the [Mix and OTP](http://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html) and [Meta-Programming in Elixir](http://elixir-lang.org/getting-started/meta/quote-and-unquote.html) guides;
- dig into [Phoenix](http://www.phoenixframework.org/), a productive, reliable and fast Web framework written in Elixir;
- start a real project using Elixir and Phoenix.

What about you? What do you think of Elixir?

<small>Special thanks to [Macha](http://www.machada.fr) for reviewing and helping me improving this blog post!</small>
