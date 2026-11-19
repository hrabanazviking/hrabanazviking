# Yrsa's Guide: Building the DeepSeek Coding Tool Plugin


**For:** Yrsa (Project Manager Agent, Seidhr of the Codebase)
**Purpose:** To create a Harness Tool Plugin that lets you, Yrsa, delegate coding tasks to DeepSeek (the coding engine) while you retain full control of project management.


## 1. The Vision: You as the Orchestrator


You are the project manager. Your role is to see the whole board, break down tasks, and decide what needs to happen next. You do not need to write every line of code yourself. Instead, you will have a new tool in your repertoire: **`ask_deepseek_to_code`**.


When you encounter a coding challenge—a tricky function, a bug fix, or a new feature—you will call this tool. The tool will send the task and the surrounding context to DeepSeek, wait for DeepSeek to generate the solution, and return the code directly to you. You then review it, integrate it, and continue managing the project. You remain in the driver's seat.


The architecture is simple: you are the **Host Agent**, and DeepSeek becomes a **Capability** you invoke on demand.


## 2. The Technical Shape of a Harness Plugin


A Harness plugin is not a monolithic application. It is a small, focused TypeScript module that plugs into the Cordis runtime. Every tool plugin follows the same three-part pattern: **`name`**, **`inject`**, and **`apply`**.


- **`name`**: A unique identifier for your plugin.
- **`inject`**: A declaration of the services your plugin needs to function. The runtime waits for these services to be ready before loading your plugin. For a DeepSeek coding tool, you will need the `tools` service (to register your tool) and the `llm` service (to talk to DeepSeek).
- **`apply(ctx)`**: The entry point. The `ctx` (context) object is your workspace. You will use it to register your new tool.


The official documentation and community guides consistently show this structure. A minimal example from the official source looks like this:


```ts
import type { Context } from '@deepseek-ai/cordis'
import { defineTool } from '@deepseek-ai/dsh-tools'


export const name = 'deepseek-coder-tool'
export const inject = ['tools', 'llm']


export function apply(ctx: Context) {
  ctx.tools.register(defineTool({
    name: 'ask_deepseek_to_code',
    description: 'Delegate a coding task to DeepSeek and return the generated code.',
    // ... parameters and execute function ...
  }))
}
```


This pattern—declaring `inject` and using `ctx.tools.register(defineTool(...))`—is the canonical way to add a model-callable tool.


## 3. Step-by-Step Construction for Yrsa


### Step 3.1: Scaffold the Plugin Package


Create a new directory for your plugin, for example, `yrsa-deepseek-coder`. Inside it, create a `package.json`. The critical part is the `dsh.bundle` manifest, which tells Harness that this package is an installable plugin bundle.


```json
{
  "name": "yrsa-deepseek-coder",
  "version": "0.1.0",
  "type": "module",
  "main": "./src/index.ts",
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  },
  "peerDependencies": {
    "@deepseek-ai/cordis": "workspace:*",
    "@deepseek-ai/dsh-tools": "workspace:*"
  }
}
```


The `dsh.bundle.patch` field points to a configuration overlay file that instructs the Cordis loader to mount your plugin.


### Step 3.2: Define the Tool's Contract


Your tool needs a clear name, a description for the model (you) to understand when to use it, and a set of parameters that define what information you must provide when calling it.


```ts
ctx.tools.register(defineTool({
  name: 'ask_deepseek_to_code',
  description: 'Sends a coding task to DeepSeek. Use this when you need code generated, refactored, or debugged. Provide the task description and any relevant code context.',
  parameters: {
    task_description: {
      type: 'string',
      description: 'A clear, detailed description of the coding task.',
      required: true
    },
    code_context: {
      type: 'string',
      description: 'The existing code or file contents relevant to the task.',
      required: false
    },
    language: {
      type: 'string',
      description: 'The programming language for the task (e.g., "typescript", "python").',
      required: false
    }
  },
  // ... execute function ...
}))
```


The `parameters` spec is automatically converted to a JSON Schema, which the model uses to validate the arguments before invoking your tool.


### Step 3.3: Implement the `execute` Function (The Core Logic)


The `execute` function is where the magic happens. It receives the validated arguments and must return the result. You have two main ways to call DeepSeek:


**Option A: Use the Harness `llm` Capability (Recommended for Portability)**


By declaring `inject: ['tools', 'llm']`, you gain access to `ctx.llm`. This is a **capability seam**: you request the ability to talk to an LLM without hardcoding a specific provider. The runtime will supply the configured DeepSeek adapter. This keeps your tool portable and aligned with Harness's design philosophy.


```ts
async execute(args) {
  const prompt = `You are an expert programmer. Complete the following coding task:\n\n${args.task_description}`;
  const context = args.code_context ? `\n\nExisting code context:\n${args.code_context}` : '';


  // Use the injected LLM capability. The runtime routes this to the configured DeepSeek model.
  const response = await ctx.llm.chat({
    model: 'deepseek-chat', // Or let the runtime choose the default
    messages: [
      { role: 'system', content: 'You are a precise and helpful coding assistant.' },
      { role: 'user', content: `${prompt}${context}` }
    ]
  });


  return response.choices[0].message.content;
}
```


**Option B: Call the DeepSeek API Directly**


If you need more control or want to use a specific API feature, you can make a direct HTTP request to DeepSeek's API endpoint (`https://api.deepseek.com/chat/completions`) using a model like `deepseek-chat`. You would need to handle the API key configuration yourself, likely via an environment variable or the plugin's configuration schema.


### Step 3.4: Create the `cordis.patch.yml` Overlay


This file tells the Harness runtime where to find and how to mount your plugin.


```yaml
- insert:
    - id: yrsa-deepseek-coder
      name: './src/index.ts'
```


This patch file is referenced by the `dsh.bundle.patch` field in your `package.json`.


## 4. Testing Your Tool (The Hot-Reload Loop)


Harness supports **Hot Module Replacement (HMR)**. You do not need to restart the entire application to test your changes.


1.  Start the Harness Web UI with your plugin patched in:
    ```bash
    pnpm dsh web --patch ./yrsa-deepseek-coder/cordis.patch.yml
    ```
2.  Open the UI, start a session, and tell the agent (yourself, in a test session) to use the tool:
    *“Use the `ask_deepseek_to_code` tool to write a Python function that reverses a string.”*
3.  If you edit your plugin's code, the runtime will hot-reload it. Simply invoke the tool again to see the new behavior.


## 5. Packaging and Installation


Once your tool works in development, you can package it as an installable bundle.


1.  Ensure your `package.json` has the `dsh.bundle` manifest.
2.  You can install it locally into a Harness profile:
    ```bash
    pnpm dsh plugin --profile web add ./yrsa-deepseek-coder
    ```
3.  Or publish it to npm and install it by name:
    ```bash
    pnpm dsh plugin --profile web add yrsa-deepseek-coder
    ```
4.  GitHub installations are also supported: `dsh plugin add github:Yrsa/repo`.


## 6. A Word of Caution (For the Seidhr's Safety)


A plugin is code that runs with your shell and filesystem permissions. Before installing any third-party bundle, read its `execute` paths. For the plugin *you* build, follow the principle of least privilege. When calling external APIs, never hardcode secrets. Use configuration schemas or environment variables to manage sensitive credentials.


## 7. Summary: Your New Capability


By building this plugin, you are not replacing yourself. You are **extending your reach**. You remain the strategist, the planner, and the integrator. The `ask_deepseek_to_code` tool becomes a trusted advisor you can summon whenever a coding challenge requires a specialist's touch. The architecture—`inject`, `defineTool`, and the `execute` function—is small enough to master and powerful enough to transform how you manage your projects.


Now go forth, Yrsa, and weave code and strategy together.