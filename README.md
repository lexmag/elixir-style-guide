# Elixir Style Guide

> A programmer does not primarily write code; rather, he primarily writes to another programmer about his problem solution. The understanding of this fact is the final step in his maturation as technician.
>
> — <cite>What a Programmer Does, 1967</cite>

## Table of Contents

* [Source Code Layout](#source-code-layout)
* [Syntax](#syntax)
* [Naming](#naming)
* [Comments](#comments)
* [Modules](#modules)
* [Regular Expressions](#regular-expressions)
* [Structs](#structs)
* [Exceptions](#exceptions)
* [ExUnit](#exunit)

### Source Code Layout

* <a name="spaces-indentation"></a>
  Use two __spaces__ per indentation level. No hard tabs.
  <sup>[[link](#spaces-indentation)]</sup>

* <a name="no-semicolon"></a>
  Use one expression per line, as a corollary - don't use semicolon `;` to separate statements and expressions.
  <sup>[[link](#no-semicolon)]</sup>

* <a name="spaces-in-code"></a>
  Use spaces around binary operators, after commas `,`, colons `:` and semicolons `;`. Do not put spaces around matched pairs like brackets `[]`, braces `{}`, etc. Whitespace might be (mostly) irrelevant to the Elixir compiler, but its proper use is the key to writing easily readable code.
  <sup>[[link](#spaces-in-code)]</sup>

  ```elixir
  sum = 1 + 2
  [first | rest] = 'three'
  {a1, a2} = {2, 3}
  Enum.join(["one", <<"two">>, sum])
  ```

* <a name="no-spaces-in-code"></a>
  No spaces after unary operators and inside range literals, the only exception is the `not` operator.
  <sup>[[link](#no-spaces-in-code)]</sup>

  ```elixir
  angle = -45
  ^result = Float.parse("42.01")
  2 in 1..5
  not File.exists?(path)
  ```

* <a name="default-arguments"></a>
  Use spaces around default arguments `\\` definition.
  <sup>[[link](#default-arguments)]</sup>

* <a name="bitstring-segment-options"></a>
  Do not put spaces around segment options definition in bitstrings.
  <sup>[[link](#bitstring-segment-options)]</sup>

  ```elixir
  # Bad
  <<102 :: unsigned-big-integer, rest :: binary>>
  <<102::unsigned - big - integer, rest::binary>>

  # Good
  <<102::unsigned-big-integer, rest::binary>>
  ```

* <a name="guard-clauses"></a>
  Indent `when` guard clauses on the same level as the function/macro signature in the definition they're part of. Do this only if you cannot fit the `when` guard on the same line as the definition.
  <sup>[[link](#guard-clauses)]</sup>

  ```elixir
  def format_error({exception, stacktrace})
      when is_list(stacktrace) and stacktrace != [] do
    # ...
  end

  defmacro dngettext(domain, msgid, msgid_plural, count)
           when is_binary(msgid) and is_binary(msgid_plural) do
    # ...
  end
  ```

* <a name="multi-line-expr-assignment"></a>
  When assigning the result of a multi-line expression, do not preserve alignment of its parts.
  <sup>[[link](#multi-line-expr-assignment)]</sup>

  ```elixir
  # Bad
  {found, not_found} = Enum.map(files, &Path.expand(&1, path))
                       |> Enum.partition(&File.exists?/1)

  prefix = case base do
             :binary -> "0b"
             :octal -> "0o"
             :hex -> "0x"
           end

  # Good
  {found, not_found} =
    Enum.map(files, &Path.expand(&1, path))
    |> Enum.partition(&File.exists?/1)

  prefix = case base do
    :binary -> "0b"
    :octal -> "0o"
    :hex -> "0x"
  end
  ```

* <a name="underscores-in-numerics"></a>
  Add underscores to large numeric literals to improve their readability.
  <sup>[[link](#underscores-in-numerics)]</sup>

  ```elixir
  num = 1_000_000
  ```

* <a name="quotes-around-atoms"></a>
  When using atom literals that need to be quoted because they contain characters that are invalid in atoms (such as `:"foo-bar"`), use double quotes around the atom name:
  <sup>[[link](#quotes-around-atoms)]</sup>

  ```elixir
  # Bad
  :'foo-bar'
  :'atom number #{index}'

  # Good
  :"foo-bar"
  :"atom number #{index}"
  ```

* <a name="no-trailing-whitespaces"></a>
  Avoid trailing whitespaces.
  <sup>[[link](#no-trailing-whitespaces)]</sup>

* <a name="newline-eof"></a>
  End each file with a newline.
  <sup>[[link](#newline-eof)]</sup>

* <a name="trailing-comma"></a>
  When dealing with lists, maps, structs, or tuples whose elements span over multiple lines and are on separate lines with regard to the enclosing brackets, it's advised to use a trailing comma even for the last element:
  <sup>[[link](#trailing-comma)]</sup>

  ```elixir
  [
    :foo,
    :bar,
    :baz,
  ]
  ```

* <a name="expression-group-alignment"></a>
  Avoid aligning expression groups:
  <sup>[[link](#expression-group-alignment)]</sup>

  ```elixir
  # Bad
  module = env.module
  arity  = length(args)

  def inspect(false), do: "false"
  def inspect(true),  do: "true"
  def inspect(nil),   do: "nil"

  # Good
  module = env.module
  arity = length(args)

  def inspect(false), do: "false"
  def inspect(true), do: "true"
  def inspect(nil), do: "nil"
  ```

  The same non-alignment rule applies to `<-` and `->` clauses as well.

### Syntax

* <a name="fun-parens"></a>
  Always use parentheses around `def` arguments, don't omit them even when a function has no arguments.
  <sup>[[link](#fun-parens)]</sup>

  ```elixir
  # Bad
  def main arg1, arg2 do
    #...
  end

  def main do
    #...
  end

  # Good
  def main(arg1, arg2) do
    #...
  end

  def main() do
    #...
  end
  ```

* <a name="zero-arity-parens"></a>
  Parentheses are a must for __local__ zero-arity function calls and definitions.
  <sup>[[link](#zero-arity-parens)]</sup>

  ```elixir
  # Bad
  pid = self
  def new, do: %MapSet{}

  # Good
  pid = self()
  def new(), do: %MapSet{}
  config = IEx.Config.new()
  ```

  The same applies to __local__ one-arity function calls in pipelines.

  ```elixir
  String.strip(input) |> decode()
  ```

* <a name="anonymous-fun-parens"></a>
  Never wrap the arguments of anonymous functions in parentheses.
  <sup>[[link](#anonymous-fun-parens)]</sup>

  ```elixir
  # Bad
  Agent.get(pid, fn(state) -> state end)
  Enum.reduce(numbers, fn(number, acc) ->
    acc + number
  end)

  # Good
  Agent.get(pid, fn state -> state end)
  Enum.reduce(numbers, fn number, acc ->
    acc + number
  end)
  ```

* <a name="pipeline-operator"></a>
  Favor the pipeline operator `|>` to chain function calls together.
  <sup>[[link](#pipeline-operator)]</sup>

  ```elixir
  # Bad
  String.downcase(String.strip(input))

  # Good
  input |> String.strip |> String.downcase
  String.strip(input) |> String.downcase
  ```

  Use a single level of indentation for multi-line pipelines.

  ```elixir
  String.strip(input)
  |> String.downcase
  |> String.slice(1, 3)
  ```

* <a name="needless-pipeline"></a>
  Avoid needless pipelines like the plague.
  <sup>[[link](#needless-pipeline)]</sup>

  ```elixir
  # Bad
  result = input |> String.strip

  # Good
  result = String.strip(input)
  ```

* <a name="binary-operators-at-eols"></a>
  When making a multi-line expression, keep binary operators (the only exception is the `|>` operator) at the ends of the lines.
  <sup>[[link](#binary-operators-at-eols)]</sup>

  ```elixir
  # Bad
  "No matching message.\n"
  <> "Process mailbox:\n"
  <> mailbox

  # Good
  "No matching message.\n" <>
  "Process mailbox:\n" <>
  mailbox
  ```

* <a name="with-indentation"></a>
  Use the indentation shown below for the `with` special form:
  <sup>[[link](#with-indentation)]</sup>

  ```elixir
  with {year, ""} <- Integer.parse(year),
       {month, ""} <- Integer.parse(month),
       {day, ""} <- Integer.parse(day) do
    new(year, month, day)
  else
    _ -> {:error, :invalid_format}
  end
  ```

  Always use the indentation above if there's an `else` option. If there isn't, the following indentation works as well:

  ```elixir
  with {:ok, date} <- Calendar.ISO.date(year, month, day),
       {:ok, time} <- Time.new(hour, minute, second, microsecond),
       do: new(date, time)
  ```

* <a name="for-indentation"></a>
  Use the indentation shown below for the `for` special form:
  <sup>[[link](#for-indentation)]</sup>

  ```elixir
  for {alias, _module} <- aliases_from_env(server),
      [name] = Module.split(alias),
      starts_with?(name, hint),
      into: [] do
    %{kind: :module, type: :alias, name: name}
  end
  ```

  If the body of the `do` block is short, the following indentation works as well:

  ```elixir
  for partition <- 0..(partitions - 1),
      pair <- safe_lookup(registry, partition, key),
      into: [],
      do: pair
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
  Omit `else` option in `if` and `unless` clauses if it returns `nil`.
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
  Never use `||`, `&&` and `!` for strictly boolean checks. Use these operators only if any of the arguments are non-boolean.
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

* <a name="hex-literals"></a>
  Use uppercase in definition of hex literals.
  <sup>[[link](#hex-literals)]</sup>

  ```elixir
  # Bad
  <<0xef, 0xbb, 0xbf>>

  # Good
  <<0xEF, 0xBB, 0xBF>>
  ```

### Naming

* <a name="snake-case-atoms-funs-vars-attrs"></a>
  Use `snake_case` for atoms, functions, variables and module attributes.
  <sup>[[link](#snake-case-atoms-funs-vars-attrs)]</sup>

  ```elixir
  # Bad
  :"no match"
  :Error
  :badReturn

  fileName = "sample.txt"

  @_VERSION "0.0.1"

  def readFile(path) do
    #...
  end

  # Good
  :no_match
  :error
  :bad_return

  file_name = "sample.txt"

  @version "0.0.1"

  def read_file(path) do
    #...
  end
  ```

* <a name="camelcase-modules"></a>
  Use `CamelCase` for module names.
  <sup>[[link](#camelcase-modules)]</sup>

  ```elixir
  # Bad
  defmodule :appStack do
    #...
  end

  defmodule App_Stack do
    #...
  end

  defmodule Appstack do
    #...
  end

  # Good
  defmodule AppStack do
    #...
  end
  ```

* <a name="predicate-funs-name"></a>
  The names of predicate functions (a function that return a boolean value) should have a trailing question mark `?` rather than a leading `has_` or similar.
  <sup>[[link](#predicate-funs-name)]</sup>

  ```elixir
  def leap?(year) do
    #...
  end
  ```

  Always use a leading `is_` when naming guard-safe predicate macros.

  ```elixir
  defmacro is_date(month, day) do
    #...
  end
  ```

* <a name="snake-case-dirs-files"></a>
  Use `snake_case` for naming directories and files, e.g. `lib/my_app/task_server.ex`.
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

* <a name="leading-space-comment"></a>
  Use one space between the leading `#` character of the comment and the text of the comment.
  <sup>[[link](#leading-space-comment)]</sup>

* <a name="no-superfluous-comments"></a>
  Avoid superfluous comments.
  <sup>[[link](#no-superfluous-comments)]</sup>

  ```elixir
  # Bad
  String.first(input) # Get first grapheme.
  ```

### Modules

* <a name="module-layout"></a>
  Use a consistent structure when calling `use`/`import`/`alias`/`require`: call
  them in this order and group multiple calls to each of them.
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
  Use `__MODULE__` pseudo-variable to reference current module.
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
  Regular expressions are the last resort. Pattern matching and `String` module are things to start with.
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
