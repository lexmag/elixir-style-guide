# Elixir Style Guide

> A programmer does not primarily write code; rather, he primarily writes to another programmer about his problem solution. The understanding of this fact is the final step in his maturation as technician.
>
> — <cite>What a Programmer Does, 1967</cite>

#### Why follow this guide?

> The only tool available right now is this guide written using the blood of the maintainers as ink.
>
> — <cite>[José Valim](https://github.com/elixir-lang/elixir/pull/6022#discussion_r112669015), creator of Elixir</cite>

## Table of Contents

* [Formatting](#formatting)
  * [Whitespace](#whitespace)
  * [Indentation](#indentation)
  * [Term representation](#term-representation)
  * [Parentheses](#parentheses)
  * [Layout](#layout)
* [Linting](#linting)
  * [Naming](#naming)
  * [Comments](#comments)
  * [Modules](#modules)
  * [Regular Expressions](#regular-expressions)
  * [Structs](#structs)
  * [Exceptions](#exceptions)
  * [ExUnit](#exunit)

## Formatting

### Whitespace

* <a name="leading-space-comment"></a>
  Use one space between the leading `#` character of the comment and the text of the comment.
  <sup>[[link](#leading-space-comment)]</sup>

  ```elixir
  # Bad
  #Amount to take is greater than the number of elements

  # Good
  # Amount to take is greater than the number of elements
  ```

### Parentheses

* <a name="fun-parens"></a>
  Always use parentheses around arguments to definitions (such as `def`, `defp`, `defmacro`, `defmacrop`, `defdelegate`). Don't omit them even when a function has no arguments.
  <sup>[[link](#fun-parens)]</sup>

  ```elixir
  # Bad
  def main arg1, arg2 do
    # ...
  end

  defmacro env do
    # ...
  end

  # Good
  def main(arg1, arg2) do
    # ...
  end

  defmacro env() do
    # ...
  end
  ```

## Linting

* <a name="pipeline-operator"></a>
  Favor the pipeline operator `|>` to chain function calls together.
  <sup>[[link](#pipeline-operator)]</sup>

  ```elixir
  # Bad
  String.downcase(String.strip(input))

  # Good
  input |> String.strip() |> String.downcase()
  ```

  For a multi-line pipeline, place each function call on a new line, and retain the level of indentation.

  ```elixir
  input
  |> String.strip()
  |> String.downcase()
  |> String.slice(1, 3)

* <a name="needless-pipeline"></a>
  Avoid needless pipelines like the plague.
  <sup>[[link](#needless-pipeline)]</sup>

  ```elixir
  # Bad
  result = input |> String.strip()

  # Good
  result = String.strip(input)
  ```

* <a name="no-else-with-unless"></a>
  Never use `unless` with `else`. Rewrite these with the positive case first.
  <sup>[[link](#no-else-with-unless)]</sup>

  ```elixir
  # Bad
  unless Enum.empty?(coll) do
    :ok
  else
    :error
  end

  # Good
  if Enum.empty?(coll) do
    :error
  else
    :ok
  end
  ```

* <a name="no-nil-else"></a>
  Omit the `else` option in `if` and `unless` constructs if `else` returns `nil`.
  <sup>[[link](#no-nil-else)]</sup>

  ```elixir
  # Bad
  if byte_size(data) > 0, do: data, else: nil

  # Good
  if byte_size(data) > 0, do: data
  ```

* <a name="true-in-cond"></a>
  If you have an always-matching clause in the `cond` special form, use `true` as its condition.
  <sup>[[link](#true-in-cond)]</sup>

  ```elixir
  # Bad
  cond do
    char in ?0..?9 ->
      char - ?0
    char in ?A..?Z ->
      char - ?A + 10
    :other ->
      char - ?a + 10
  end

  # Good
  cond do
    char in ?0..?9 ->
      char - ?0
    char in ?A..?Z ->
      char - ?A + 10
    true ->
      char - ?a + 10
  end
  ```

* <a name="boolean-operators"></a>
  Never use `||`, `&&`, and `!` for strictly boolean checks. Use these operators only if any of the arguments are non-boolean.
  <sup>[[link](#boolean-operators)]</sup>

  ```elixir
  # Bad
  is_atom(name) && name != nil
  is_binary(task) || is_atom(task)

  # Good
  is_atom(name) and name != nil
  is_binary(task) or is_atom(task)
  line && line != 0
  file || "sample.exs"
  ```

* <a name="patterns-matching-binaries"></a>
  Favor the binary concatenation operator `<>` over bitstring syntax for patterns matching binaries.
  <sup>[[link](#patterns-matching-binaries)]</sup>

  ```elixir
  # Bad
  <<"http://", _rest::bytes>> = input
  <<first::utf8, rest::bytes>> = input

  # Good
  "http://" <> _rest = input
  <<first::utf8>> <> rest = input
  ```

### Naming

* <a name="snake-case-atoms-funs-vars-attrs"></a>
  Use `snake_case` for functions, variables, module attributes, and atoms.
  <sup>[[link](#snake-case-atoms-funs-vars-attrs)]</sup>

  ```elixir
  # Bad
  :"no match"
  :Error
  :badReturn

  fileName = "sample.txt"

  @_VERSION "0.0.1"

  def readFile(path) do
    # ...
  end

  # Good
  :no_match
  :error
  :bad_return

  file_name = "sample.txt"

  @version "0.0.1"

  def read_file(path) do
    # ...
  end
  ```

* <a name="camelcase-modules"></a>
  Use `CamelCase` for module names. Keep uppercase acronyms as uppercase.
  <sup>[[link](#camelcase-modules)]</sup>

  ```elixir
  # Bad
  defmodule :appStack do
    # ...
  end

  defmodule App_Stack do
    # ...
  end

  defmodule Appstack do
    # ...
  end

  defmodule Html do
    # ...
  end

  # Good
  defmodule AppStack do
    # ...
  end

  defmodule HTML do
    # ...
  end
  ```

* <a name="predicate-funs-name"></a>
  The names of predicate functions (functions that return a boolean value) should have a trailing question mark `?` rather than a leading `has_` or similar.
  <sup>[[link](#predicate-funs-name)]</sup>

  ```elixir
  # Bad
  def is_leap(year) do
    # ...
  end

  # Good
  def leap?(year) do
    # ...
  end
  ```

  Always use a leading `is_` when naming guard-safe predicate macros.

  ```elixir
  defmacro is_date(month, day) do
    # ...
  end
  ```

* <a name="snake-case-dirs-files"></a>
  Use `snake_case` for naming directories and files, for example `lib/my_app/task_server.ex`.
  <sup>[[link](#snake-case-dirs-files)]</sup>

* <a name="one-letter-var"></a>
  Avoid using one-letter variable names.
  <sup>[[link](#one-letter-var)]</sup>

### Comments

> Remember, good code is like a good joke: It needs no explanation.
>
> — <cite>Russ Olsen</cite>

* <a name="no-need-for-comments"></a>
  Write self-documenting code and ignore the rest of this section. Seriously!
  <sup>[[link](#no-need-for-comments)]</sup>

* <a name="no-superfluous-comments"></a>
  Avoid superfluous comments.
  <sup>[[link](#no-superfluous-comments)]</sup>

  ```elixir
  # Bad
  String.first(input) # Get first grapheme.
  ```

### Modules

* <a name="module-layout"></a>
  Use a consistent structure when calling `use`/`import`/`alias`/`require`: call them in this order and group multiple calls to each of them.
  <sup>[[link](#module-layout)]</sup>

  ```elixir
  use GenServer

  import Bitwise
  import Kernel, except: [length: 1]

  alias Mix.Utils
  alias MapSet, as: Set

  require Logger
  ```

* <a name="current-module-reference"></a>
  Use the `__MODULE__` pseudo-variable to reference the current module.
  <sup>[[link](#current-module-reference)]</sup>

  ```elixir
  # Bad
  :ets.new(Kernel.LexicalTracker, [:named_table])
  GenServer.start_link(Module.LocalsTracker, nil, [])

  # Good
  :ets.new(__MODULE__, [:named_table])
  GenServer.start_link(__MODULE__, nil, [])
  ```

### Regular Expressions

* <a name="pattern-matching-over-regexp"></a>
  Regular expressions are the last resort. Pattern matching and the `String` module are things to start with.
  <sup>[[link](#pattern-matching-over-regexp)]</sup>

  ```elixir
  # Bad
  Regex.run(~r/#(\d{2})(\d{2})(\d{2})/, color)
  Regex.match?(~r/(email|password)/, input)

  # Good
  <<?#, p1::2-bytes, p2::2-bytes, p3::2-bytes>> = color
  String.contains?(input, ["email", "password"])
  ```

* <a name="non-capturing-regexp"></a>
  Use non-capturing groups when you don't use the captured result.
  <sup>[[link](#non-capturing-regexp)]</sup>

  ```elixir
  ~r/(?:post|zip )code: (\d+)/
  ```

* <a name="caret-and-dollar-regexp"></a>
  Be careful with `^` and `$` as they match start and end of the __line__ respectively. If you want to match the __whole__ string use: `\A` and `\z` (not to be confused with `\Z` which is the equivalent of `\n?\z`).
  <sup>[[link](#caret-and-dollar-regexp)]</sup>

### Structs

* <a name="defstruct-fields-default"></a>
  When calling `defstruct/1`, don't explicitly specify `nil` for fields that default to `nil`.
  <sup>[[link](#defstruct-fields-default)]</sup>

  ```elixir
  # Bad
  defstruct first_name: nil, last_name: nil, admin?: false

  # Good
  defstruct [:first_name, :last_name, admin?: false]
  ```

### Exceptions

* <a name="exception-naming"></a>
  Make exception names end with a trailing `Error`.
  <sup>[[link](#exception-naming)]</sup>

  ```elixir
  # Bad
  BadResponse
  ResponseException

  # Good
  ResponseError
  ```

* <a name="exception-message"></a>
  Use non-capitalized error messages when raising exceptions, with no trailing punctuation.
  <sup>[[link](#exception-message)]</sup>

  ```elixir
  # Bad
  raise ArgumentError, "Malformed payload."

  # Good
  raise ArgumentError, "malformed payload"
  ```

  There is one exception to the rule - always capitalize Mix error messages.

  ```elixir
  Mix.raise "Could not find dependency"
  ```

### ExUnit

* <a name="exunit-assertion-side"></a>
  When asserting (or refuting) something with comparison operators (such as `==`, `<`, `>=`, and similar), put the expression being tested on the left-hand side of the operator and the value you're testing against on the right-hand side.
  <sup>[[link](#exunit-assertion-side)]</sup>

  ```elixir
  # Bad
  assert "héllo" == Atom.to_string(:"héllo")

  # Good
  assert Atom.to_string(:"héllo") == "héllo"
  ```

  When using the match operator `=`, put the pattern on the left-hand side (as it won't work otherwise).

  ```elixir
  assert {:error, _reason} = File.stat("./non_existent_file")
  ```

## License

This work was created by Aleksei Magusev and is licensed under [the CC BY 4.0 license](https://creativecommons.org/licenses/by/4.0).

![Creative Commons License](http://i.creativecommons.org/l/by/4.0/88x31.png)

## Credits

The structure of the guide and some points that are applicable to Elixir were taken from [the community-driven Ruby coding style guide](https://github.com/bbatsov/ruby-style-guide).
