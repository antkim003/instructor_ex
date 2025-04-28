# instructor_ex

_Structured, Ecto outputs with OpenAI (and OSS LLMs)_

---

[![Instructor version](https://img.shields.io/hexpm/v/instructor.svg)](https://hex.pm/packages/instructor)
[![Hex Docs](https://img.shields.io/badge/hex-docs-lightgreen.svg)](https://hexdocs.pm/instructor/)
[![Hex Downloads](https://img.shields.io/hexpm/dt/instructor)](https://hex.pm/packages/instructor)
[![GitHub stars](https://img.shields.io/github/stars/thmsmlr/instructor_ex.svg)](https://github.com/thmsmlr/instructor_ex/stargazers)
[![Twitter Follow](https://img.shields.io/twitter/follow/thmsmlr?style=social)](https://twitter.com/thmsmlr)
[![Discord](https://img.shields.io/discord/1192334452110659664?label=discord)](https://discord.gg/bD9YE9JArw)

<!-- Docs -->

Check out our [Quickstart Guide](https://hexdocs.pm/instructor/quickstart.html) to get up and running with Instructor in minutes.

Instructor provides structured prompting for LLMs. It is a spiritual port of the great [Instructor Python Library](https://github.com/jxnl/instructor) by [@jxnlco](https://twitter.com/jxnlco).

Instructor allows you to get structured output out of an LLM using Ecto.  
You don't have to define any JSON schemas.
You can just use Ecto as you've always used it.  
And since it's just ecto, you can provide change set validations that you can use to ensure that what you're getting back from the LLM is not only properly structured, but semantically correct.

To learn more about the philosophy behind Instructor and its motivations, check out this Elixir Denver Meetup talk:

<div style="text-align: center">

[![Instructor: Structured prompting for LLMs](assets/youtube-thumbnail.png)](https://www.youtube.com/watch?v=RABXu7zqnT0)

</div>

While Instructor is designed to be used with OpenAI, it also supports every major AI lab and open source LLM inference server:

- OpenAI
- Anthropic
- Groq
- Ollama
- Gemini
- vLLM
- llama.cpp

At its simplest, usage is pretty straightforward: 

1. Create an ecto schema, with a `@llm_doc` string that explains the schema definition to the LLM. 
2. Define a `validate_changeset/1` function on the schema, and use the `use Instructor` macro in order for Instructor to know about it.
2. Make a call to `Instructor.chat_completion/1` with an instruction for the LLM to execute.

You can use the `max_retries` parameter to automatically, iteratively go back and forth with the LLM to try fixing validation errorswhen they occur.

```elixir
Mix.install([:instructor])

defmodule SpamPrediction do
  use Ecto.Schema
  use Validator

  @llm_doc """
  ## Field Descriptions:
  - class: Whether or not the email is spam.
  - reason: A short, less than 10 word rationalization for the classification.
  - score: A confidence score between 0.0 and 1.0 for the classification.
  """
  @primary_key false
  embedded_schema do
    field(:class, Ecto.Enum, values: [:spam, :not_spam])
    field(:reason, :string)
    field(:score, :float)
  end

  @impl true
  def validate_changeset(changeset) do
    changeset
    |> Ecto.Changeset.validate_number(:score,
      greater_than_or_equal_to: 0.0,
      less_than_or_equal_to: 1.0
    )
  end
end

is_spam? = fn text ->
  Instructor.chat_completion(
    model: "gpt-4o-mini",
    response_model: SpamPrediction,
    max_retries: 3,
    messages: [
      %{
        role: "user",
        content: """
        Your purpose is to classify customer support emails as either spam or not.
        This is for a clothing retail business.
        They sell all types of clothing.

        Classify the following email: 

        <email>
          #{text}
        </email>
        """
      }
    ]
  )
end

is_spam?.("Hello I am a Nigerian prince and I would like to send you money")

# => {:ok, %SpamPrediction{class: :spam, reason: "Nigerian prince email scam", score: 0.98}}
```

<!-- Docs -->

## Installation

In your mix.exs,

```elixir
def deps do
  [
    {:instructor, "~> 0.1.0"}
  ]
end
<<<<<<< HEAD
```

InstructorEx uses [Code.fetch_docs/1](https://hexdocs.pm/elixir/1.16.2/Code.html#fetch_docs/1) to fetch LLM instructions from the Ecto schema specified in `response_model`. If your project is deployed using [releases](https://hexdocs.pm/mix/Mix.Tasks.Release.html), add the following configuration to mix.exs to prevent docs from being stripped from the release:

```elixir
def project do
  # ...
  releases: [
    myapp: [
      strip_beams: [keep: ["Docs"]]
    ]
  ]
end
```

## TODO

- [x] Top-level array support
- [x] Gemini
  - [x] tools mode
  - [ ] json mode
  - [x] json_schema mode
- [x] Figure out a solution for OpenAI's json_schema mode not supporting arbitrary maps.
- [ ] Partial Schemaless doesn't work since fields are set to required in Ecto.


- [ ] llamacpp adapter broken, needs to support openai input/output API
  - [ ] GBNF should enforce required properties on objects, currently they're optional.
  - [ ] GBNF limit the number of digits in number tokens -- small models can sometimes run off to infinit digits
- [ ] Add instructor tests against llamacpp interface using mocks, there's non-trivial logic in there
- [ ] Logging for Distillation / Finetuning
- [ ] Add a Bumblebee adapter
- [ ] Add llamacpp_ex adapter
- [ ] Support naked ecto types by auto-wrapping, not just maps of ecto types, do not wrap if we don't need to... Current codepaths are muddled
- [x] Support Streaming
  - [ ] Verify schemaless support `{:array, %{name: :string}}`
  - [ ] Support typespec style support for array streaming `[MySchema]`
- [ ] Optional/Maybe types
- [ ] Add Livebook Tutorials, include in Hexdocs
    - [x] Text Classification
    - [ ] Self Critique
    - [ ] Image Extracting Tables
    - [ ] Moderation
    - [x] Citations
    - [ ] Knowledge Graph
    - [ ] Entity Resolution
    - [ ] Search Queries
    - [ ] Query Decomposition
    - [ ] Recursive Schemas
    - [x] Table Extraction
    - [x] Action Item and Dependency Mapping
    - [ ] Multi-File Code Generation
    - [ ] PII Data Sanitizatiommersed
- [x] Update hexdocs homepage to include example for tutorial
- [ ] Setup Github CI for testing, add badge to README

## Blog Posts

- [ ] Why structured prompting?

    Meditations on new HCI.
    Finally we have software that can understand text. f(text) -> text.
    This is great, as it gives us a new domain, but the range is still text.
    While we can use string interpolation to map Software 1.0 into f(text), the outputs are not interoperable with Software 1.0.
    Hence why UXs available to us are things like Chatbots as our users have to interpret the output.

    Instructor, structure prompting, gives use f(text) -> ecto_schema.
    Schemas are the lingua franca of Software 1.0.
    With Instrutor we can now seamlessly move back and forth between Software 1.0 and Software 2.0.

    Now we can maximally leverage AI...

- [ ] From GPT-4 to zero-cost production - Distilation, local-llms, and the cost structure of AI.

    ... 😘
=======
```
>>>>>>> upstream/main
