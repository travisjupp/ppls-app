## ADR-016: AI Semantic Layer (Redis-Powered RAG)

## Status
Proposed (Epic: Infra/AI)

## Context
ADR-015 established a build-time pipeline to convert Markdown into a bundled-assets.json for offline reading. However, a "Static Reader" alone cannot handle natural language queries (e.g., "I have a rent receipt, what do I do now?"). To meet the "Bronze" timeline for a Vertical Slice, we need a high-speed retrieval mechanism that allows an LLM to "read" our guide and answer questions accurately without manual hard-coding.

## Decision
We will implement a Retrieval-Augmented Generation (RAG) pattern using Redis as the Vector Database. This layer will act as the "AI Brain" for the Vertical Slice, indexing the same Markdown source used in ADR-015.

   1. **Dual-Purpose Pipeline**: The bin/generate-guide.js script (from ADR-015) will be extended. While it generates the local JSON bundle for the app, it will also "upsert" (update/insert) text embeddings into a Redis instance.
   2. **Granular Chunking**: To provide precise AI answers, the script will break Markdown files into "chunks" based on headers (H1, H2, H3) rather than passing entire 50-page chapters to the AI.
   3. **Hybrid Search**: We will use Redis Search + Vector Similarity to find content. This allows the AI to filter by "Section" or "Page" (Metadata from ADR-015 front-matter) while performing a semantic search on the text.
   4. **Provisioning Stage**: For the "Bronze" phase, we will only index the "CalFresh" chapter as our Vertical Slice to demonstrate the end-to-end flow.

## Implementation Steps

   1. **Enhance generate-guide.js**: Add a step to generate "Vector Embeddings" (via OpenAI or Local Transformers) for each content chunk.
   2. **Sync to Redis**: Push the JSON chunks and their vectors to a Redis Cloud instance.
   3. **API Proxy**: Create a simple Vercel/Node function that takes a user query, searches Redis, and sends the "Context" to an LLM (like GPT-4o) to generate a response.

## Demonstration: bin/generate-guide.js (AI Extension)
This script builds on the logic in ADR-015 by adding the Redis/Vector push.

### The Redis/Vector Push

```js
const fs = require('fs');
const matter = require('gray-matter');
const { Redis } = require('@langchain/community/vectorstores/redis');
const { OpenAIEmbeddings } = require('@langchain/openai');

async function buildAISemanticLayer() {
    // Read File as standard text
    const file = fs.readFileSync('./raw_content/calfresh_p37.md', 'utf-8');
    // Convert frontmatter and grab content
    const { data, content } = matter(file);

    // 1. Prepare chunks (Linking to ADR-015 Metadata)
    const doc = {
        pageContent: content,
        metadata: {
            id: data.id,
            section: data.section,
            page: data.page,
            source: 'calfresh_p37.md'
        }
    };

    // 2. Initialize Redis Vector Store
    const vectorStore = await Redis.fromTexts(
        [doc.pageContent], /* The Goods */
        [doc.metadata], /* The Labels */
        new OpenAIEmbeddings(), /* The Packager (Goods -> Vector)*/
        { /* Config */
            redisClient: myRedisClient, /* DB ('warehouse') */
            indexName: "guide_index", /* Table */
        }
    );

    console.log(`✅ Indexed ${data.id} into AI Semantic Layer.`);
}
```

### The Redis/Vector Pull

```js
// 1. Re-connect to the existing 'warehouse'
const vectorStore = await Redis.fromExistingIndex(new OpenAIEmbeddings(), {
  redisClient: myRedisClient,
  indexName: "guide_index",
});

// 2. The User's question
const query = "What do I need to bring for CalFresh?";

// 3. Search using the 'vectorStore' variable
const results = await vectorStore.similaritySearch(query, 3); 
// Returns the top 3 most relevant 'Goods (pageContent)' + 'Labels (metadata)'
// from the Markdown, E.g.,:

/*
[
  {
    pageContent: "You can apply for CalFresh in person... have your ID and Rent Receipt ready.",
    metadata: { id: "calfresh-how-to", page: 37, section: "CalFresh" }
  }
]
*/
```

### Pass the Redis "Pull Data" to an LLM

```js
// Backend API (Node.js)
async function getAIResponse(userQuestion, redisResults) {
  // 1. Format the Redis snippets into a single context string
  const context = redisResults.map(r => r.pageContent).join("\n\n");

  // 2. The System Prompt: Sets the rules for the AI
  const prompt = `
    You are a helpful assistant for a community advocacy guide. 
    Use the following guide excerpts to answer the user's question accurately.
    If the information isn't in the context, say you don't know.
    
    Context: ${context}
    User Question: ${userQuestion}
  `;

  // 3. The Call: Send to OpenAI (or whatever model)
  const response = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [{ role: "user", content: prompt }],
  });

  return response.choices[0].message.content;
}
```

### Display to Frontend (RNP)

```js
import React, { useState } from 'react';
import { ScrollView } from 'react-native';
import { Card, Text, ActivityIndicator, Searchbar } from 'react-native-paper';

const AskAIScreen = () => {
  const [query, setQuery] = useState('');
  const [answer, setAnswer] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSearch = async () => {
    setLoading(true);
    // This calls your backend which handles the Redis + LLM flow
    const response = await fetchAIAnswer(query); 
    setAnswer(response);
    setLoading(false);
  };

  return (
    <ScrollView style={{ padding: 16 }}>
      <Searchbar
        placeholder="Ask the guide..."
        value={query}
        onIconPress={handleSearch}
        onSubmitEditing={handleSearch}
        onChangeText={setQuery}
      />

      {loading && <ActivityIndicator style={{ marginTop: 20 }} />}

      {answer && (
        <Card style={{ marginTop: 20, padding: 10 }}>
          <Card.Title title="AI Assistant" subtitle="Sourced from Chapter 37" />
          <Card.Content>
            <Text variant="bodyMedium">{answer}</Text>
          </Card.Content>
        </Card>
      )}
    </ScrollView>
  );
};
```

## Consequences

- **Pro**: Consistency. The AI and the Static Reader (ADR-015) use the exact same source text, ensuring the AI never "hallucinates" information that isn't in the printed guide.
- **Pro**: Speed. Redis retrieval happens in <2ms, keeping the "Ask AI" feature snappy.
- **Con**: Cost. Running the AI Semantic Layer requires a hosted Redis instance and embedding API credits (unlike the 100% free offline JSON reader).
- **Con**: Connectivity. Unlike the Static Reader, the "Ask AI" feature will require an internet connection during the Bronze phase.


