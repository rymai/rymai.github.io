---
title: "Scaffolding a Phoenix project"
categories: english elixir
---

Last weekend, [Macha](http://www.machada.fr) and I had a new project idea that will need an API server used by an Android app (to begin with). Since I'm [very excited about Elixir and Phoenix](/2015/07/28/why-i-am-excited-about-elixir) these days, that's the perfect occasion to start a new [Phoenix](http://www.phoenixframework.org) project!

### Forewords

[Phoenix](http://www.phoenixframework.org) is a [Ruby on Rails](http://rubyonrails.org/)-like web framework written in [Elixir](http://elixir-lang.org/). I'm using Phoenix **v1.0.3** in the rest of the article.

Here are the steps we will follow to scaffold our new Phoenix application:

1. [Setup the environment](#setup-the-environment)
2. [Create the application](#create-the-application)
3. [Create the models](#create-the-models)
4. [Migrate the models](#migrate-the-models)
5. [Scaffold admin views and controllers](#scaffold-admin-views-and-controllers)
6. [Scaffold API controllers](#scaffold-api-controllers)

<a name="setup-the-environment"></a>
### 1. Setup the environment

There are some prerequisite before being able to create a new Phoenix project. Following are a summary of the needed steps but I recommend following the official [Installation page](http://www.phoenixframework.org/docs/installation) if you want more details.

1. [Install Elixir](http://elixir-lang.org/install.html).
2. Install the [Hex](https://hex.pm/) package manager:

    ```bash
    › mix local.hex
    ```
3. Install the Phoenix archive:

    ```bash
    › mix archive.install \
    https://github.com/phoenixframework/phoenix/releases/\
    download/v1.0.3/phoenix_new-1.0.3.ez
    ```
4. You should [install Node.js](https://nodejs.org/en) as well (to compile the static assets).

<a name="create-the-application"></a>
### 2. Create the application

Creating the application is a simple matter of running:

```bash
› mix phoenix.new ./geo_sport-web --app geo_sport
```

This will create a new `geo_sport-web` folder. Note that we pass an explicit app name `geo_sport` because `-` are not allowed in app names, but I want it in the folder name.

<a name="create-the-models"></a>
### 3. Create the models

When starting a new project, I like to start by creating my models.

In this new project, we'll need to have a representation of the French [departments](https://en.wikipedia.org/wiki/Departments_of_France) (departments are administrative divisions, we have 101 of them) and [communes](https://en.wikipedia.org/wiki/Communes_of_France) (communes are the smallest administrative divisions, we have 36,681 of them!).

Fortunately, Phoenix gives us a great Mix task called [`phoenix.gen.model`](http://hexdocs.pm/phoenix/Mix.Tasks.Phoenix.Gen.Model.html) to scaffold models (ie. generate the model file, its test file and a migration file).

Let's create our two models:

**Department**

```bash
› mix phoenix.gen.model Department departments \
name:string insee_code:string

* creating priv/repo/migrations/20151019092404_create_department.exs
* creating web/models/department.ex
* creating test/models/department_test.exs

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

**Commune**

```bash
› mix phoenix.gen.model Commune communes \
name:string commune_code:string \
department_id:references:departments

* creating priv/repo/migrations/20151019094807_create_commune.exs
* creating web/models/commune.ex
* creating test/models/commune_test.exs

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

<a name="migrate-the-models"></a>
### 4. Migrate the models

Our models are created, let's follow the instructions and run the migrations to actually create the `departments` and `communes` tables in our PostgreSQL database:

```bash
› mix ecto.migrate

Compiled web/models/commune.ex
Compiled web/models/department.ex
Generated geo_sport app

11:28:43.276 [info]  == Running GeoSport.Repo.Migrations.CreateDepartment.change/0 forward

11:28:43.276 [info]  create table departments

11:28:44.088 [info]  == Running GeoSport.Repo.Migrations.CreateCommune.change/0 forward

11:28:44.088 [info]  create table communes

11:28:44.127 [info]  create index communes_department_id_index

11:28:44.134 [info]  == Migrated in 0.4s
```

We now have two models, their test files, and two migrations:

```bash
› ls -G -l web/models/

-rw-r--r--  1 remy  staff  610 Oct 19 11:48 commune.ex
-rw-r--r--  1 remy  staff  519 Oct 19 11:24 department.ex

› ls -G -l test/models/

-rw-r--r--  1 remy  staff  479 Oct 19 11:48 commune_test.exs
-rw-r--r--  1 remy  staff  478 Oct 19 11:24 department_test.exs

› ls -G -l priv/repo/migrations/

-rw-r--r--  1 remy  staff  220 Oct 19 11:24 20151019092404_create_department.exs
-rw-r--r--  1 remy  staff  346 Oct 19 11:48 20151019094807_create_commune.exs
```

Let's create some views and controllers!

<a name="scaffold-admin-views-and-controllers"></a>
### 5. Scaffold admin views and controllers

For this project, we want to have an admin interface to be able to check what data we have and to be able to quickly edit them if needed.

Again, Phoenix gives us another great Mix task called [`phoenix.gen.html`](http://hexdocs.pm/phoenix/Mix.Tasks.Phoenix.Gen.Html.html) to scaffold controllers, views (i.e. Rails helpers) and templates (i.e. Rails views).

*Note that you can also create models using this task but I want to put my controllers under the `Admin` namespace and this would put the model under that namespace too...*

Let's create our admin views and controllers for our two resources:

**Department**

```bash
› mix phoenix.gen.html Admin.Department departments \
name:string insee_code:string --no-model

* creating web/controllers/admin/department_controller.ex
* creating web/templates/admin/department/edit.html.eex
* creating web/templates/admin/department/form.html.eex
* creating web/templates/admin/department/index.html.eex
* creating web/templates/admin/department/new.html.eex
* creating web/templates/admin/department/show.html.eex
* creating web/views/admin/department_view.ex
* creating test/controllers/admin/department_controller_test.exs

Add the resource to your browser scope in web/router.ex:

    resources "/admin/departments", Admin.DepartmentController
```

**Commune**

```bash
› mix phoenix.gen.html Admin.Commune communes \
name:string commune_code:string \
department_id:references:departments --no-model

* creating web/controllers/admin/commune_controller.ex
* creating web/templates/admin/commune/edit.html.eex
* creating web/templates/admin/commune/form.html.eex
* creating web/templates/admin/commune/index.html.eex
* creating web/templates/admin/commune/new.html.eex
* creating web/templates/admin/commune/show.html.eex
* creating web/views/admin/commune_view.ex
* creating test/controllers/admin/commune_controller_test.exs

Add the resource to your browser scope in web/router.ex:

    resources "/admin/communes", Admin.CommuneController
```
*Notes:*

- The `--no-model` option is used to deactivate the model generation.
- We have to repeat the model's fields definition so that the scaffold can create the appropriate HTML form fields.
- You will have to replace `alias GeoSport.Admin.Department` with `alias GeoSport.Department` in `web/controllers/admin/department_controller.ex` (same thing for the `Commune` resource). I go into more details on why [at the end of this article](#model-namespacing-issue).

#### Discover Phoenix `web/router.ex` file

Our controller, view and templates have been created, let's follow the instructions and add routes for these resources. Open the `web/router.ex` file, it should look like this:

```elixir
defmodule GeoSport.Router do
  use GeoSport.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", GeoSport do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
  end

  # Other scopes may use custom stacks.
  # scope "/api", GeoSport do
  #   pipe_through :api
  # end
end
```
If you know Rails, you should see that this is very similar to the `config/routes.rb` file! However, there are some neat things to note:

- You can define pipelines –which are stacks of [Plug](https://github.com/elixir-lang/plug)s (think of it as a stack of Rack middleware)– in the routing file.
- You explicitely pipes a group of routes through a pipeline using the [`pipe_through/1`](http://hexdocs.pm/phoenix/Phoenix.Router.html#pipe_through/1) function.

#### Add routes for the scaffolded resources

Let's comment `get "/", PageController, :index` line, add our new resource (without the `/admin` prefix) route and modify the scope to `"/admin"` instead of `"/"`:

```elixir
[...]

  scope "/admin", GeoSport do
    pipe_through :browser # Use the default browser stack

    # get "/", PageController, :index
    resources "/departments", Admin.DepartmentController
  end

[...]
```

Now, let's inspect our available routes, using the [`phoenix.routes`](http://hexdocs.pm/phoenix/Mix.Tasks.Phoenix.Routes.html) Mix task:

```bash
› mix phoenix.routes

       department_path  GET     /admin/departments            GeoSport.Admin.DepartmentController :index
       department_path  GET     /admin/departments/:id/edit   GeoSport.Admin.DepartmentController :edit
       department_path  GET     /admin/departments/new        GeoSport.Admin.DepartmentController :new
       department_path  GET     /admin/departments/:id        GeoSport.Admin.DepartmentController :show
       department_path  POST    /admin/departments            GeoSport.Admin.DepartmentController :create
       department_path  PATCH   /admin/departments/:id        GeoSport.Admin.DepartmentController :update
                        PUT     /admin/departments/:id        GeoSport.Admin.DepartmentController :update
       department_path  DELETE  /admin/departments/:id        GeoSport.Admin.DepartmentController :delete
```
Perfect!

#### Start the app for the first time!

Let's see that in action by starting the app using the [`phoenix.server`](http://hexdocs.pm/phoenix/Mix.Tasks.Phoenix.Server.html) Mix task:

```bash
› mix phoenix.server

Compiled web/controllers/admin/department_controller.ex
Generated geo_sport app
[info] Running GeoSport.Endpoint with Cowboy on http://localhost:4000
19 Oct 12:48:13 - info: compiled 5 files into 2 files, copied 3 in 2036ms
```
If you visit [http://localhost:4000/admin/departments](http://localhost:4000/admin/departments), you should see a scaffold page that lists all the departments (none for now). You can create a new department, update it or even delete it from this page. Same thing for the [`Commune` resource](http://localhost:4000/admin/communes))

<a name="scaffold-api-controllers"></a>
### 6. Scaffold API controllers

Now let's create our two resources's API endpoints using the [`phoenix.gen.json`](http://hexdocs.pm/phoenix/Mix.Tasks.Phoenix.Gen.Json.html) Mix task:

**Department**

```bash
› mix phoenix.gen.json Api.V1.Department departments \
name:string insee_code:string --no-model

* creating web/controllers/api/v1/department_controller.ex
* creating web/views/api/v1/department_view.ex
* creating test/controllers/api/v1/department_controller_test.exs
* creating web/views/changeset_view.ex

Add the resource to your api scope in web/router.ex:

    resources "/api/v1/departments", Api.V1.DepartmentController, except: [:new, :edit]

```

**Commune**

```bash
› mix phoenix.gen.json Api.V1.Commune communes \
name:string commune_code:string --no-model       

* creating web/controllers/api/v1/commune_controller.ex
* creating web/views/api/v1/commune_view.ex
* creating test/controllers/api/v1/commune_controller_test.exs

Add the resource to your api scope in web/router.ex:

    resources "/api/v1/communes", Api.V1.CommuneController, except: [:new, :edit]
```
Again, don't forget to replace `alias GeoSport.Api.V1.Department` with `alias GeoSport.Department` in `web/controllers/api/v1/department_controller.ex` (same thing for the `Commune` resource).

#### Add routes for the scaffolded resources

Now let's add the new routes to the –originally commented– `"/api"` scope in `web/router.ex`:

```elixir
[...]

  scope "/api", GeoSport.Api, as: :api do
    pipe_through :api

    scope "/v1", V1, as: :v1 do
      resources "/departments", DepartmentController
      resources "/communes", CommuneController
    end
  end

[...]
```
*Notes:*

- We modified the original `"/api"` scope with a new `GeoSport.Api` module, and a new `as: :api` option.
  - The `GeoSport.Api` module indicates that API controllers will be located under `web/controllers/api/`.
  - The `as: :api` option adds a prefix to the path and URL route helpers (e.g. `api_departments_path` instead of `departments_path`).
- We added a nested scope called `"/v1"` to version our API, with the `V1` module and the `as: :v1` option.
  - The `V1` module indicates that the API V1 controllers will be located under `web/controllers/api/v1/`.
  - The `as: :v1` option adds a prefix to the path and URL route helpers (e.g. `api_v1_departments_path` instead of `api_departments_path`).

Let's inspect our available routes:

```bash
› mix phoenix.routes

       department_path  GET     /admin/departments            GeoSport.Admin.DepartmentController :index
       department_path  GET     /admin/departments/:id/edit   GeoSport.Admin.DepartmentController :edit
       department_path  GET     /admin/departments/new        GeoSport.Admin.DepartmentController :new
       department_path  GET     /admin/departments/:id        GeoSport.Admin.DepartmentController :show
       department_path  POST    /admin/departments            GeoSport.Admin.DepartmentController :create
       department_path  PATCH   /admin/departments/:id        GeoSport.Admin.DepartmentController :update
                        PUT     /admin/departments/:id        GeoSport.Admin.DepartmentController :update
       department_path  DELETE  /admin/departments/:id        GeoSport.Admin.DepartmentController :delete
api_v1_department_path  GET     /api/v1/departments           GeoSport.Api.V1.DepartmentController :index
api_v1_department_path  GET     /api/v1/departments/:id/edit  GeoSport.Api.V1.DepartmentController :edit
api_v1_department_path  GET     /api/v1/departments/new       GeoSport.Api.V1.DepartmentController :new
api_v1_department_path  GET     /api/v1/departments/:id       GeoSport.Api.V1.DepartmentController :show
api_v1_department_path  POST    /api/v1/departments           GeoSport.Api.V1.DepartmentController :create
api_v1_department_path  PATCH   /api/v1/departments/:id       GeoSport.Api.V1.DepartmentController :update
                       PUT     /api/v1/departments/:id       GeoSport.Api.V1.DepartmentController :update
api_v1_department_path  DELETE  /api/v1/departments/:id       GeoSport.Api.V1.DepartmentController :delete
```
Perfect!

#### Admire the new API v1!

Now, if you [created a new department](http://localhost:4000/admin/departments/new), you should see it in the [`/api/v1/departments`](http://localhost:4000/api/v1/departments) endpoint:

```json
{
  "data": [
    {
      "name": "Ain",
      "insee_code": "1",
      "id": 1
    }
  ]
}
```

<a name="model-namespacing-issue"></a>
### Model namespacing issue

In the step [5.](#scaffold-admin-views-and-controllers) and [6.](#scaffold-api-controllers) of this article, we generated namespaced controllers and views but we had to remove the namespace from the model aliases defined in the controllers.

Without this change, we would have gotten the following error when trying to start the app:

```bash
› mix phoenix.server

== Compilation error on file web/controllers/admin/department_controller.ex ==
** (CompileError) web/controllers/admin/department_controller.ex:14: GeoSport.Admin.Department.__struct__/0 is undefined, cannot expand struct GeoSport.Admin.Department
    (elixir) src/elixir_map.erl:58: :elixir_map.translate_struct/4
    (stdlib) lists.erl:1353: :lists.mapfoldl/3
    web/controllers/admin/department_controller.ex:13: (module)
    (stdlib) erl_eval.erl:669: :erl_eval.do_apply/6
```
There is something in `web/controllers/admin/department_controller.ex:14` that prevents the app from starting properly. Let's open the controller:

```elixir
defmodule GeoSport.Admin.DepartmentController do
  use GeoSport.Web, :controller

  alias GeoSport.Admin.Department

  plug :scrub_params, "department" when action in [:create, :update]

  def index(conn, _params) do
    departments = Repo.all(Department)
    render(conn, "index.html", departments: departments)
  end

  def new(conn, _params) do
    changeset = Department.changeset(%Department{})
    render(conn, "new.html", changeset: changeset)
  end

  [...]
end  
```
It looks like the struct `%Department{}` in `changeset = Department.changeset(%Department{})` is the culprit...

Indeed, `Department` is an alias defined 10 lines above for the module `GeoSport.Admin.Department`. Unfortunately, we don't have such module, this line was created by the scaffolding task `mix phoenix.gen.html Admin.Department ...` but our model module is actually `GeoSport.Department`, not `GeoSport.Admin.Department`.

You can fix that by replacing `alias GeoSport.Admin.Department` with `alias GeoSport.Department`.

### Conclusion

Scaffolding a Phoenix app is really easy, as easy as scaffolding a Rails app!

However, there are a few quirks along the way –that are handled in Rails I think– that could be fixed easily:

- If you want to namespace your routes/controllers, you'll have to remove the namespace from the model name passed to [`alias/2`](http://elixir-lang.org/docs/stable/elixir/Kernel.SpecialForms.html#alias/2) in the auto-generated controllers. The generator could accept a new `--namespace` option that could add the namespace to routes/controllers but not to the model name.
- If you want to scaffold controllers/views with the `--no-model` option, you'll have to pass the model fields definition again to ensure that the HTML forms are generated with form fields. The generator could be smart enough to inspect the existing model's schema and use the fields defined in it to know what HTML form fields to generate in the templates.

These two issues might be good candidates for [pull-requests](https://github.com/phoenixframework/phoenix/pulls) to the Phoenix project!

What about you? Did you experience any other quirks while using Phoenix's scaffolding tasks?
