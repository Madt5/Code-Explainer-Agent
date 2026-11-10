# Code Explainer Agent

A small command-line **AI agent** built with Node.js and TypeScript. Point it at any project folder, ask a question in plain English, and the agent explores the code on its own, deciding which files to open, and then answers.

```bash
npm start -- ./some-project "What does this project do and how do I run it?"
```

## How it works

This is a minimal example of an LLM **tool-calling loop**:

1. The question is sent to the model together with two tool definitions: `list_files` and `read_file`.
2. If the model decides it needs more information, it replies with a tool request instead of an answer.
3. The program runs the tool locally and sends the result back to the model.
4. Steps 2-3 repeat (up to 15 steps) until the model produces a final text answer.

```
question -> LLM -> tool request -> run tool locally -> tool result -> LLM -> ... -> final answer
```

Each tool call is printed to the terminal (stderr), so you can watch how the agent explores the project.

## Project structure

```
src/
  index.ts   # CLI entry point, system prompt and the agent loop
  tools.ts   # Tool schemas, tool implementations and path sandboxing
```

## Safety notes

- **Sandboxed file access:** tool paths are resolved against the project root, and anything outside it (e.g. `../../etc/passwd`) is rejected.
- `node_modules`, `.git` and build folders are hidden from the agent.
- Large files are truncated to 20,000 characters to keep requests small.
- Tool errors are returned to the model as text, so it can recover instead of crashing.

## Setup

Requirements: Node.js 20+ and an [Anthropic API key](https://console.anthropic.com/).

```bash
npm install
cp .env.example .env      # then put your API key in .env
```

## Usage

```bash
npm start -- <project-folder> "<your question>"
```

Examples:

```bash
npm start -- . "Explain how the agent loop works"
npm start -- ../my-api "Which database does this project use and where is it configured?"
npm start -- ../my-app "List the main dependencies and what each is used for"
```

Set `MODEL` in `.env` to use a different model (default: `claude-sonnet-5-5`).

## Tech stack

- Node.js, TypeScript
- [`@anthropic-ai/sdk`](https://www.npmjs.com/package/@anthropic-ai/sdk) (Messages API with tool use)
- `dotenv`, `tsx`

## Ideas for extending it

- Add a `search_in_files` tool (grep-style) so the agent can find code faster
- Stream the answer token by token
- Add a `--interactive` mode for follow-up questions
- Write unit tests for the tools with Node's built-in test runner

## License

MIT
