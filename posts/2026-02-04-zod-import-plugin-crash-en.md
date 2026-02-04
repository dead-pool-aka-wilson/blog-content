---
title: "Why Importing Zod Directly Crashes OpenCode Plugins"
date: 2026-02-04
categories: [debugging, lessons-learned]
tags: [opencode, zod, plugin, typescript, debugging]
draft: false
summary: "The subtle version mismatch that caused 'undefined is not an object' errors when defining tool schemas in OpenCode plugins."
lang: en
cover:
  image: "/covers/2026-02-04-zod-import-plugin-crash.png"
---

## What Happened

I was building a custom OpenCode plugin with tool definitions. Following the TypeScript types, I imported Zod directly to define the schema:

```typescript
import { z } from "zod";

export default {
  name: "my-plugin",
  tools: [
    {
      name: "my-tool",
      description: "Does something useful",
      args: {
        input: z.string().describe("The input"),
      },
      execute: async ({ input }) => {
        // ...
      },
    },
  ],
};
```

On startup, OpenCode crashed:

```
TypeError: undefined is not an object (evaluating 'schema._zod.def')
    at toJSONSchema (node_modules/.bun/zod@4.1.8/node_modules/zod/v4/core/to-json-schema.js:211:57)
    at resolveTools (src/session/prompt.ts:704:62)
```

## Root Cause

The error pointed to `zod@4.1.8`—OpenCode's internal Zod version. But my plugin was importing Zod from *my* node_modules, which had a different version (or different Zod instance entirely).

The issue: **Zod schemas are not cross-instance compatible**.

When OpenCode tried to serialize my schema to JSON Schema, it expected internal properties like `_zod.def` that only exist on its own Zod instance. My plugin's Zod schemas were a different object structure.

This is a common problem with libraries that use `instanceof` checks or internal properties. Two copies of the same library at different versions—or even the same version installed twice—create incompatible objects.

## The Fix

Instead of Zod schemas, use JSON Schema format directly:

```typescript
export default {
  name: "my-plugin",
  tools: [
    {
      name: "my-tool",
      description: "Does something useful",
      parameters: {
        type: "object",
        properties: {
          input: {
            type: "string",
            description: "The input",
          },
        },
        required: ["input"],
      },
      execute: async ({ input }) => {
        // ...
      },
    },
  ],
};
```

Key change: `args` (Zod) → `parameters` (JSON Schema)

The TypeScript types suggest `args` with Zod, but runtime accepts `parameters` with plain JSON Schema. This avoids the cross-instance compatibility issue entirely.

## Alternative: Use OpenCode's Zod

If you need Zod's ergonomics, import from OpenCode's plugin utilities:

```typescript
import { tool } from "@opencode-ai/plugin/tool";

// This uses the same Zod instance as OpenCode
const myTool = tool({
  name: "my-tool",
  description: "Does something useful",
  schema: z => z.object({
    input: z.string().describe("The input"),
  }),
  execute: async ({ input }) => {
    // ...
  },
});
```

The callback pattern ensures you're using the correct Zod instance.

## Lessons Learned

1. **Don't assume library compatibility across package boundaries** - Each `node_modules` might have its own copy
2. **Internal properties (`_zod`, `_internal`) are red flags** - They indicate implementation details that may vary
3. **JSON Schema is the universal interchange format** - When in doubt, use the lowest common denominator
4. **TypeScript types don't guarantee runtime behavior** - The type said `args`, but runtime wanted `parameters`

## Prevention

When building plugins for frameworks:

- Check documentation for the canonical way to define schemas
- Prefer framework-provided utilities over direct library imports
- Test your plugin in isolation before integrating
- If you must use a shared library, ensure version alignment with the host

This cost me two hours of debugging. The error message was cryptic, pointing at Zod internals rather than the actual problem: incompatible Zod instances across package boundaries.
