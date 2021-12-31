# Chapter 1 Getting Started with Repo

## Ecto and Elixir
- three main characteristics of Ecto
    1. Ecto is *approachable*
    2. Ecto is *explicit*
    3. Ecto is *flexible*


## Ecto Modules
- 6 main modules
    1. **Repo** module
        - All communication to adn from the database goes through **Repo**.
    2. **Query** module
        - API for writing queries.
    3. **Schema** module
        - A schema is a kind of map.
        - create maps
    4. **Changeset** module
        - *changeset*: a data structure that captures all aspects of making a change to data.
        - this module provides functions for creating and manipulating changesets.
    5. **Multi** module
        - to coordinate several database changes simultaneously
    6. **Migration** module
        - changing the structure of a database


## How Ecto Is Organized
- Ecto consists two separate pakages
    - **ecto**
        - contains some of the core data manipulation features.
            - **Repo**
            - **Query**
            - **Schema**
            - **Chnageset**
    - **ecto_sql**
        - contains modules specifically needed to communicate with relational databases.
            - database-specific adapters
            - migrations


## Setting Up the Sample App
- `$ mix do deps.get, compile`
- edit `./config/config.exs`
    - set username and password
- edit `./lib/music_db/repo.ex`
    - check adapter field
- `$ mix ecto.setup`
    - three mix tasks into one command
        - `$ mix ecto.create`
        - `$ mix ecto.migrate`
        - `$ mix run priv/repo/seeds.exs`
- to confirm
    - `$ iex -S mix`


### Running the Examples
- two way
    - `$ iex -S mix`
    - `./priv/repo/playground.exs`
        - edit **play** function
        - `$ mix run priv/repo/playground.exs`


### Resetting the Sample Data
- `$ mix ecto.reset`


### Data Model of the Sample App
<img src='https://i.gyazo.com/949b04c3c35b4ca802a9fec8f82d8e9b.png' width='500'>


## The Repository Pattern
- `./priv/examples/getting_started_01.exs`


```elixir
iex(1)> alias MusicDB.{Repo, Artist}
[MusicDB.Repo, MusicDB.Artist]
iex(2)> Repo.insert(%Artist{name: "Dizzy Gillespie"})

01:10:56.273 [debug] QUERY OK db=1.1ms queue=0.8ms idle=1839.5ms
INSERT INTO "artists" ("name","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["Dizzy Gillespie", ~N[2021-12-31 00:10:56], ~N[20
21-12-31 00:10:56]]
{:ok,
 %MusicDB.Artist{
   __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
   albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
   birth_date: nil,
   death_date: nil,
   id: 4,
   inserted_at: ~N[2021-12-31 00:10:56],
   name: "Dizzy Gillespie",
   tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
   updated_at: ~N[2021-12-31 00:10:56]
 }}
iex(10)> dizzy = Repo.get_by(Artist, name: "Dizzy Gillespie")

01:15:26.437 [debug] QUERY OK source="artists" db=1.7ms idle=1008.3ms
SELECT a0."id", a0."name", a0."birth_date", a0."death_date", a0."inserted_at", a0."updated_at" FROM "artists" AS a0 WHERE (a0."name" = $1) ["
Dizzy Gillespie"]
%MusicDB.Artist{
  __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
  albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
  birth_date: nil,
  death_date: nil,
  id: 4,
  inserted_at: ~N[2021-12-31 00:10:56],
  name: "Dizzy Gillespie",
  tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
  updated_at: ~N[2021-12-31 00:10:56]
}
iex(11)> Repo.update(Ecto.Changeset.change(dizzy, name: "John Birks Gillespie"))

01:15:38.245 [debug] QUERY OK db=1.2ms queue=0.8ms idle=1815.4ms
UPDATE "artists" SET "name" = $1, "updated_at" = $2 WHERE "id" = $3 ["John Birks Gillespie", ~N[2021-12-31 00:15:38], 4]
{:ok,
 %MusicDB.Artist{
   __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
   albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
   birth_date: nil,
   death_date: nil,
   id: 4,
   inserted_at: ~N[2021-12-31 00:10:56],
   name: "John Birks Gillespie",
   tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
   updated_at: ~N[2021-12-31 00:15:38]
 }}
iex(12)> Repo.delete(dizzy)                                                     

01:15:48.707 [debug] QUERY OK db=2.2ms queue=0.3ms idle=1277.3ms
DELETE FROM "artists" WHERE "id" = $1 [4]
{:ok,
 %MusicDB.Artist{
   __meta__: #Ecto.Schema.Metadata<:deleted, "artists">,
   albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
   birth_date: nil,
   death_date: nil,
   id: 4,
   inserted_at: ~N[2021-12-31 00:10:56],
   name: "Dizzy Gillespie",
   tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
   updated_at: ~N[2021-12-31 00:10:56]
 }}
```

## The Repo Module
- how Repository is implemented in Ecto.
- `./lib/music_db/repo.ex`


```elixir
defmodule MusicDB.Repo do
  use Ecto.Repo,
    otp_app: :music_db,
    adapter: Ecto.Adapters.Postgres

  def using_postgres? do
    MusicDB.Repo.__adapter__ == Ecto.Adapters.Postgres
  end

end
```

- This set up the **Repo** module
- The **otp_app** option is requred.
    - it comes from `./configure/dev.exs`
    - database parameters can combine all into a single **url** parameter.
        - `url: "ecto://dbuser:dbuserpass@localhost/music_db"`


## Putting Our Repo to Work
- **insert_all**, **update_all**, **delete_all**, **all** functions

```elixir
iex(1)> alias MusicDB.Repo
MusicDB.Repo
iex(2)> Repo.insert_all("artists", [[name: "John Coltrane"]])

01:33:00.191 [debug] QUERY OK db=0.9ms decode=0.4ms queue=0.6ms idle=1637.1ms
INSERT INTO "artists" ("name") VALUES ($1) ["John Coltrane"]
{1, nil}
iex(3)> Repo.all(Artist)

01:33:38.318 [debug] QUERY OK source="artists" db=5.3ms queue=0.7ms idle=1764.5ms
SELECT a0."id", a0."name", a0."birth_date", a0."death_date", a0."inserted_at", a0."updated_at" FROM "artists" AS a0 []
[
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 1,
    inserted_at: ~N[2021-12-30 23:46:04],
    name: "Miles Davis", 
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: ~N[2021-12-30 23:46:04]
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 2,
    inserted_at: ~N[2021-12-30 23:46:04],
    name: "Bill Evans",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: ~N[2021-12-30 23:46:04]
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 3,
    inserted_at: ~N[2021-12-30 23:46:04],
    name: "Bobby Hutcherson",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: ~N[2021-12-30 23:46:04]
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 5,
    inserted_at: nil,
    name: "John Coltrane",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: nil
  }
]
iex(4)> Repo.insert_all("artists",
...(4)>   [[name: "Sonny Rollins", inserted_at: DateTime.utc_now()]])

01:34:43.044 [debug] QUERY OK db=0.8ms queue=0.3ms idle=1494.0ms
INSERT INTO "artists" ("inserted_at","name") VALUES ($1,$2) [~U[2021-12-31 00:34:43.041928Z], "Sonny Rollins"]
{1, nil}
iex(5)> Repo.get_by(Artist, name: "Sonny Rollins")   

01:36:07.242 [debug] QUERY OK source="artists" db=0.4ms queue=0.8ms idle=1693.6ms
SELECT a0."id", a0."name", a0."birth_date", a0."death_date", a0."inserted_at", a0."updated_at" FROM "artists" AS a0 WHERE (a0."name" = $1) ["
Sonny Rollins"]
%MusicDB.Artist{
  __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
  albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
  birth_date: nil,
  death_date: nil,
  id: 6,
  inserted_at: ~N[2021-12-31 00:34:43],
  name: "Sonny Rollins",
  tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
  updated_at: nil
}
iex(6)> Repo.insert_all("artists",
...(6)>   [[name: "Max Roach", inserted_at: DateTime.utc_now()],
...(6)>   [name: "Art Blakey", inserted_at: DateTime.utc_now()]])

01:38:07.447 [debug] QUERY OK db=1.1ms queue=0.8ms idle=897.5ms
INSERT INTO "artists" ("inserted_at","name") VALUES ($1,$2),($3,$4) [~U[2021-12-31 00:38:07.445367Z], "Max Roach", ~U[2021-12-31 00:38:07.445
385Z], "Art Blakey"]
{2, nil}
iex(7)> Repo.all(Artist)

01:38:20.380 [debug] QUERY OK source="artists" db=1.1ms idle=1831.2ms
SELECT a0."id", a0."name", a0."birth_date", a0."death_date", a0."inserted_at", a0."updated_at" FROM "artists" AS a0 []
[
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 1,
    inserted_at: ~N[2021-12-30 23:46:04],
    name: "Miles Davis", 
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: ~N[2021-12-30 23:46:04]
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 2,
    inserted_at: ~N[2021-12-30 23:46:04],
    name: "Bill Evans",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: ~N[2021-12-30 23:46:04]
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 3,
    inserted_at: ~N[2021-12-30 23:46:04],
    name: "Bobby Hutcherson",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: ~N[2021-12-30 23:46:04]
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 5,
    inserted_at: nil,
    name: "John Coltrane",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: nil
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 6,
    inserted_at: ~N[2021-12-31 00:34:43],
    name: "Sonny Rollins",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: nil
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 7,
    inserted_at: ~N[2021-12-31 00:38:07],
    name: "Max Roach",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: nil
  },
  %MusicDB.Artist{
    __meta__: #Ecto.Schema.Metadata<:loaded, "artists">,
    albums: #Ecto.Association.NotLoaded<association :albums is not loaded>,
    birth_date: nil,
    death_date: nil,
    id: 8,
    inserted_at: ~N[2021-12-31 00:38:07],
    name: "Art Blakey",
    tracks: #Ecto.Association.NotLoaded<association :tracks is not loaded>,
    updated_at: nil
  }
]


iex(8)> Repo.insert_all("artists",                               
...(8)>   [%{name: "Max Roach", inserted_at: DateTime.utc_now()},
...(8)>    %{name: "Art Blakey", inserted_at: DateTime.utc_now()}])

01:44:47.656 [debug] QUERY OK db=1.0ms queue=0.6ms idle=1106.4ms
INSERT INTO "artists" ("inserted_at","name") VALUES ($1,$2),($3,$4) [~U[2021-12-31 00:44:47.654266Z], "Max Roach", ~U[2021-12-31 00:44:47.654
284Z], "Art Blakey"]
{2, nil}

iex(9)> Repo.update_all("artists", set: [updated_at: DateTime.utc_now()])
# set option
# to tell Ecto which fields and values want to change

01:45:48.411 [debug] QUERY OK source="artists" db=1.1ms queue=0.6ms idle=1861.4ms
UPDATE "artists" AS a0 SET "updated_at" = $1 [~U[2021-12-31 00:45:48.405456Z]]
{9, nil}



iex(10)> Repo.delete_all("tracks")

01:54:33.068 [debug] QUERY OK source="tracks" db=4.0ms queue=0.3ms idle=1515.7ms
DELETE FROM "tracks" AS t0 []
{33, nil}
```


- **set** option
    - to tell Ecto which fields and values want to change
- **inc** option
    - this increments the given field by the given value
    - can decrement by supplying a negative number
- **push** option
    - works on columns containing an array
    - pushes the given value onto the end of the array
- **pull** option
    - works on array columns
    - removes the given value from the array


### Getting Values Back
- **_all** functions return taple
    - the first value of taple
        - the number of records affected by the operation
    - the second contains the values
        - asked the database to return
- **returning** option
    - specify any values we'd like returned to us after the operation completed
    - works in PostgreSQL, but not in MySQL


- `./priv/examples/getting_started_03.exs`

```elixir
iex(1)> Repo.insert_all("artists", [%{name: "Max Roach"},
...(1)>   %{name: "Art Blakey"}], returning: [:id, :name])

02:06:43.635 [debug] QUERY OK db=0.8ms decode=0.6ms queue=0.7ms idle=1033.2ms
INSERT INTO "artists" ("name") VALUES ($1),($2) RETURNING "id","name" ["Max Roach", "Art Blakey"]
{2, [%{id: 4, name: "Max Roach"}, %{id: 5, name: "Art Blakey"}]}
```


### Exicuting Queries
- **Query** module
    - query interface
- **Ecto.Adapters.SQL88 module
    - `query` function


```elixir
iex(2)> Ecto.Adapters.SQL.query(Repo, "SELECT * FROM artists WHERE id=1")
# Repo.query("SELECT * FROM artists WHERE id=1")

02:14:13.731 [debug] QUERY OK db=6.9ms queue=0.7ms idle=1123.8ms
SELECT * FROM artists WHERE id=1 []
{:ok,
 %Postgrex.Result{
   columns: ["id", "name", "birth_date", "death_date", "inserted_at",
    "updated_at"],
   command: :select,
   connection_id: 3853286,
   messages: [],
   num_rows: 1,
   rows: [
     [1, "Miles Davis", nil, nil, ~N[2021-12-31 00:55:02.000000],
      ~N[2021-12-31 00:55:02.000000]]
   ]
 }}
```


## Customizing Your Repo

- `priv/examples/getting_started_05.exs`


```elixir
iex(1)> import Ecto.Repo
Ecto.Repo
iex(2)> Repo.aggregate("albums", :count, :id)

02:20:45.882 [debug] QUERY OK source="albums" db=0.6ms decode=0.9ms queue=0.8ms idle=1698.2ms
SELECT count(a0."id") FROM "albums" AS a0 []
5
```

- `./lib/music_db/repo.ex`

```elixir
  def count(table) do
    aggregate(table, :count, :id)
  end
```


```elixir
iex(1)> r Repo
warning: redefining module MusicDB.Repo (current version loaded from _build/dev/lib/music_db/ebin/Elixir.MusicDB.Repo.beam)
  lib/music_db/repo.ex:9

{:reloaded, MusicDB.Repo, [MusicDB.Repo]}
iex(2)> Repo.count("albums")

02:23:31.327 [debug] QUERY OK source="albums" db=0.4ms decode=1.1ms queue=0.6ms idle=1986.4ms
SELECT count(a0."id") FROM "albums" AS a0 []
5
```


- **init** callback
    - this runs when Ecto first initializes and allows you to add or override configuration parameters


```elixir
  def init(_, opts) do
    {:ok, Keyword.put(opts, :url, System.get_env("DATABASE_URL"))}
  end
```


## Wrapping Up
