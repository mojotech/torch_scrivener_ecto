# Torch Scrivener.Ecto

This fork of the [original Scrivener.Ecto project](https://github.com/drewolson/scrivener_ecto) is being minimally
maintained to keep a working version of the [Torch](https://github.com/mojotech/torch) project functional for current
and future versions of Ecto.  Feel free to submit any Pull Requests to this repository that you might have previously
submitted directly to the original `scrivener_ecto` repository as that repository is not in active maintenance mode.

This forked version can be found on hex.pm under the package name
[torch_scrivener_ecto](https://hex.pm/packages/torch_scrivener_ecto).

[![Build
Status](https://github.com/drewolson/scrivener_ecto/actions/workflows/test.yml/badge.svg?branch=master)](https://github.com/drewolson/scrivener_ecto/actions/workflows/test.yml)

## Low Maintenance Warning

This library is in low maintenance mode, which means the author is currently only responding to pull requests and breaking issues.

## Usage

Scrivener.Ecto allows you to paginate your Ecto queries with Scrivener. It gives you useful information such as the total number of pages, the current page, and the current page's entries. It works nicely with Phoenix as well.

First, you'll want to `use` Scrivener in your application's Ecto Repo. This will add a `paginate` function to your Repo. This `paginate` function expects to be called with, at a minimum, an Ecto query. It will then paginate the query and execute it, returning a `Scrivener.Page`. Defaults for `page_size` can be configured when you `use` Scrivener. If no `page_size` is provided, Scrivener will use `10` by default.

You may also want to call `paginate` with a params map along with your query. If provided with a params map, Scrivener will use the values in the keys `"page"` and `"page_size"` before using any configured defaults.

Note: Scrivener.Ecto only supports Ecto backends that allow subqueries (e.g. PostgreSQL).

## Example

```elixir
defmodule MyApp.Repo do
  use Ecto.Repo, otp_app: :my_app, adapter: Ecto.Adapters.Postgres
  use Scrivener, page_size: 10
end
```

```elixir
defmodule MyApp.Person do
  use Ecto.Schema

  schema "people" do
    field :name, :string
    field :age, :integer

    has_many :friends, MyApp.Person
  end
end
```

```elixir
def index(conn, params) do
  page =
    MyApp.Person
    |> where([p], p.age > 30)
    |> order_by(desc: :age)
    |> preload(:friends)
    |> MyApp.Repo.paginate(params)

  render conn, :index,
    people: page.entries,
    page_number: page.page_number,
    page_size: page.page_size,
    total_pages: page.total_pages,
    total_entries: page.total_entries
end
```

```elixir
page =
  MyApp.Person
  |> where([p], p.age > 30)
  |> order_by(desc: :age)
  |> preload(:friends)
  |> MyApp.Repo.paginate(page: 2, page_size: 5)
```

## Installation

Add `scrivener_ecto` to your `mix.exs` `deps`.

```elixir
defp deps do
  [
    {:torch_scrivener_ecto, "~> 3.0"}
  ]
end
```

## Contributing

First, you'll need to build the test database.

```elixir
MIX_ENV=test mix db.reset
```

This task assumes you have postgres installed and that the `postgres` user can create / drop databases. If you'd prefer to use a different user, you can specify it with the environment variable `SCRIVENER_ECTO_DB_USER`.

With the database built, you can now run the tests.

```elixir
mix test
```
