---
title: "Populating database tables from a CSV in Elixir"
categories: english elixir
---

The next step after [scaffolding my first Phoenix project](/2015-10-22-scaffolding-a-phoenix-project) was to populate my database with data representing French [departments](https://en.wikipedia.org/wiki/Departments_of_France) (departments are administrative divisions, we have 101 of them) and [communes](https://en.wikipedia.org/wiki/Communes_of_France) (communes are the smallest administrative divisions, we have 36,681 of them!).

### Forewords

The CSV I import is from [open data shared by the French government](https://www.data.gouv.fr/fr/datasets/recensement-des-equipements-sportifs-espaces-et-sites-de-pratiques/). The goal is to read this big (24MB) CSV file and insert records for departments and communes not already registered in the database.

I'm using Phoenix **v1.0.4** in the rest of the article.

Here are the steps we will follow to import data from the CSV to our database:

1. [Taming the streaming beast](#taming-the-streaming-beast)
2. [Working with Ecto](#working-with-ecto)
2. [Having fun with an agent](#having-fun-with-an-agent)

<a name="taming-the-streaming-beast"></a>
### 1. Taming the streaming beast

The first thing I realized was that it would have been inefficient to read the whole file, store its content in memory and loop over it. That's why I started digging through the [`Stream`](http://elixir-lang.org/docs/stable/elixir/Stream.html) module.

After some trial and errors, I stumbled upon the [CSV module](https://github.com/beatrichartz/csv) which gives the following sample code:

```elixir
File.stream!("data.csv") |>
CSV.decode(separator: ?\t) |>
Enum.map fn row ->
  Enum.each(row, &String.upcase/1)
end
```

Since my CSV file uses `;` as separator and includes the headers in its first row, I've adapted a bit the snippet to my use-case, as follow:

```elixir
defmodule Seeds.DepartmentsAndCommunes do

  @doc "Imports departments and communes from the given CSV to the database"
  def import_from_csv(csv_path) do
    Agent.start_link(fn -> %{departments: %HashDict{}, communes: []} end, name: __MODULE__)

    File.stream!(Path.expand(csv_path))
    |> CSV.decode(separator: ?;, headers: true)
    |> Stream.each(fn row ->
      _process_csv_row(row, agent)
    end)
    |> Stream.run
  end

  defp _process_csv_row(row, agent) do
    # TODO
  end

end
```

Note:

- I put this code into `priv/repo/seeds.exs` so I can run it with `mix run priv/repo/seeds.exs`;
- I create an agent to store the already processed departments/communes to avoid unnecessary DB queries. More details in the ["Having fun with an agent"](#having-fun-with-an-agent) part.

<a name="working-with-ecto"></a>
### 2. Working with Ecto

The second step is the actual work to populate the database.

Following are the functions needed to insert/update a department into the database:

```elixir
defp _process_department(department_params, agent) do
  # Let's avoid a DB query if this department has already been processed
  if _get_processed(agent, :departments)[department_params[:name]] do
    IO.puts "Department '#{department_params[:name]}' already processed"
  else
    # Otherwise, try to fetch the department from the DB
    department = Repo.get_by(Department, insee_code: department_params[:insee_code])

    # And create a changeset
    department_changeset = _changeset(Department, department, department_params)

    # The department exists already in the DB
    if department do

      # And its name is good
      if department.name == department_params[:name] do
        IO.puts "Department '#{department.name}' is ok"

      # It's named has changed
      else
        Repo.update!(department_changeset)
        IO.puts "Department '#{department.name}' updated"
      end

    # The department doesn't exist yet in the DB
    else
      Repo.insert!(department_changeset)
      department = Repo.get_by(Department, insee_code: department_params[:insee_code])
      IO.puts "Department '#{department_params[:name]}' created"
    end

    # Let's add this department in our agent (we actually store a tuple {department_name => department_id})
    _add_processed(agent, :departments, department_params[:name], department.id)
  end
end
```

Similarly, here is the function to insert/update a commune:

```elixir
defp _process_commune(commune_params) do
  if Enum.member?(_get_processed(:communes), commune_params[:name]) do
    IO.puts "Commune '#{commune_params[:name]}' already processed"
  else
    commune = Repo.get_by(Commune, commune_code: commune_params[:commune_code])
    commune_changeset = _changeset(Commune, commune, commune_params)

    if commune do
      if commune.name == commune_params[:name] && commune.department_id == commune_params[:department_id] do
        IO.puts "Commune '#{commune.name}' is ok"
      else
        Repo.update!(commune_changeset)
        IO.puts "Commune '#{commune.name}' updated"
      end
    else
      Repo.insert!(commune_changeset)
      IO.puts "Commune '#{commune_params[:name]}' created"
    end

    _add_processed(:communes, commune_params[:name])
  end
end
```

<a name="having-fun-with-an-agent"></a>
### 3. Having fun with an agent

To avoid doing two queries per CSV row (one for the department, one for the commune), I am using an [agent](http://elixir-lang.org/docs/stable/elixir/Agent.html). An Elixir Agent is a simple abstraction around state.

As we already saw, I have instantiated an agent with:

```elixir
Agent.start_link(fn -> %{departments: %HashDict{}, communes: []} end, name: __MODULE__)
```

Then I have created a bunch of shortcut functions to access and update the state:

```elixir
defp _get_processed(:departments) do
  Agent.get(__MODULE__, fn %{departments: departments} -> departments end)
end

@doc "Returns the already-processed communes names List"
defp _get_processed(:communes) do
  Agent.get(__MODULE__, fn %{communes: communes} -> communes end)
end

@doc "Adds a {name => db_id} tuple in the already-processed departments HashDict"
defp _add_processed(:departments, name, department_id) do
  Agent.update(__MODULE__, fn %{departments: departments} = processed ->
    %{processed | departments: Dict.put(departments, name, department_id)}
  end)
end

@doc "Adds a commune name tuple in the already-processed communes List"
defp _add_processed(:communes, name) do
  Agent.update(__MODULE__, fn %{communes: communes} = processed ->
    %{processed | communes: [name | communes]}
  end)
end
```

### Conclusion

That's it! You can find the whole script in this [GitLab snippet](https://gitlab.com/snippets/11331).

I really struggled with the "memoization" of the already-processed departments & communes at first: I was
passing two arrays all around the place, I didn't understand how to update them etc.

Then I found this nice ["How I Start"](https://howistart.org/posts/elixir/1) article (with José Valim) which reminded me of the Agent module, why it's useful and how to actually use it.

I must admit, that was an awesome "Eurêka" moment!

Regarding the next steps for this script, here is what I have in mind:

- Fetch the latest CSV file from the [data.gouv.fr API](https://www.data.gouv.fr/api/1/datasets/recensement-des-equipements-sportifs-espaces-et-sites-de-pratiques/) and process it;
- Do this regularly in a background worker.

What about you, have you written similar scripts with Elixir before? Also, if you have ideas on how to improve my script, I'm all hears (hint: you can open a pull-request below)!
