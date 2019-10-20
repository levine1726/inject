# Inject

Inject is a library that lets you write testable Elixir code that can run concurrently in ExUnit.

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `inject` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:inject, "~> 0.1.0"}
  ]
end
```

It is recommended to disable inject in prod.exs to avoid any performance penalty for registry lookups.
```
config :inject, disabled: true
```

## Usage

Inject is just a couple of functions.

- `inject/1` aliased as `i/1`. Use this function in your implementation code to flag modules for potential injection.

```
defmodule MyApplication do
  import Inject, only: [i: 1]

  def process do
    {:ok, file} = i(File).open("your-mind.txt", [])
    ...
  end
end
```

- `register/2`. Use this function in your tests to register a stubbed implementation for a module.

```
defmodule MyApplicationTest do
  use ExUnit.Case, async: true
  import Inject, only: [register: 2]

  defmodule FileStub do
    def open("your-mind.txt", _opts) do
      {:ok, nil}
    end
  end

  test "use my stub for this test" do
    register(File, FileStub)
    {:ok, nil} = MyApplication.process()
  end
end
```
Defining stubbed modules like this is great, but I like to pair Inject w/ [Double](https://hex.pm/packages/double) for on-the-fly setups.

```
defmodule MyApplicationTest do
  use ExUnit.Case, async: true
  import Inject, only: [register: 2]
  import Double

  test "use my stub for this test" do
    register(File, stub(File, :open, fn(_, _) -> {:ok, nil} end))
    {:ok, nil} = MyApplication.process()
  end
end
```
