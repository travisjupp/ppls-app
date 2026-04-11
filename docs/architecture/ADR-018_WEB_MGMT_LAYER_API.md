## ADR-017: Web Management Layer & API (Astro Actions)

## Status
Proposed (Epic: Infra/DX)

## Context
We need a centralized "Command Center" to manage the digital guide, host the staff update pipeline (ADR-016), and provide an API for the React Native "Ask AI" feature. Traditional REST APIs require significant boilerplate (routing, validation, types).

## Decision
We will utilize Astro as the web-based management layer and Astro Actions as the primary API bridge for the native app.

   1. Server-Side Execution: All Redis and LLM interactions will happen within Astro Actions, keeping sensitive API keys hidden from the client.
   2. Type Safety (Zod): Actions will use Zod to validate that questions sent from the mobile app are properly formatted strings, reducing runtime crashes.
   3. Deployment: The Astro site will be deployed as a Node.js SSR server (or Vercel/Netlify functions) to allow for dynamic AI generation.
   4. Admin UI: Staff will use the Astro frontend to preview Markdown changes and manually trigger the "Push to Redis" (ADR-016) pipeline if needed.

## Consequences

- **Pro**: One Language. Both the mobile API and the staff dashboard are managed in one codebase.
- **Pro**: Developer Experience. Astro Actions automatically generate types, making the "Native-to-Cloud" connection nearly seamless.
- **Con**: Requires the Astro site to be in SSR Mode, which requires a small hosting cost (compared to a free static site).

## Implementation: `src/actions/index.ts`
Astro "Middleman" between React Native app and Redis.

```js

import { defineAction } from 'astro:actions';
import { z } from 'astro:schema';
import { RedisVectorStore } from "@langchain/community/vectorstores/redis";
import { OpenAIEmbeddings, ChatOpenAI } from "@langchain/openai";
import { createClient } from "redis";

export const server = {
  askAI: defineAction({
    // 1. Validation (The "Guard")
    input: z.object({
      question: z.string(),
    }),
    handler: async (input) => {
      // 2. Initialize Redis Client
      const client = createClient({ url: process.env.REDIS_URL });
      await client.connect();

      // 3. Connect to the Existing Index (from ADR-016)
      const vectorStore = await RedisVectorStore.fromExistingIndex(
        new OpenAIEmbeddings(),
        { redisClient: client, indexName: "guide_index" }
      );

      // 4. The "Pull" (Finding the matching guide text)
      const results = await vectorStore.similaritySearch(input.question, 3);
      const context = results.map(r => r.pageContent).join("\n\n");

      // 5. The "Response" (LLM Handshake)
      const model = new ChatOpenAI({ modelName: "gpt-4o" });
      const response = await model.invoke([
        ["system", "Use this guide text to answer accurately: " + context],
        ["human", input.question]
      ]);

      await client.disconnect();
      return { answer: response.content };
    }
  })
}
```

## Calling from the Native App

Hit the endpoint from the Astro Action:

```js
// From an RNP component
const handleSearch = async () => {
  const { data, error } = await actions.askAI({ question: "How do I get CalFresh?" });
  if (data) setAnswer(data.answer);
};
```

