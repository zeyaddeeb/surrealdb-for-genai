---
theme: default
title: SurrealDB for Gen AI
class: text-center
highlighter: shiki
fonts:
  sans: Roboto
  serif: Roboto Slab
  mono: Fira Code
  local: Fira Code, Roboto, Roboto Slab
transition: slide-left
favicon: https://raw.githubusercontent.com/zeyaddeeb/zeyaddeeb.github.io/master/favicon.ico
mdc: true
info: |
  ## SurrealDB Developer Conference
  September 26th, 2024

  By [Zeyad Deeb](https://zeyaddeeb.com/)

layout: cover
---
# Scaling <span v-mark.red="0"> SurrealDB </span> for GenAI

How Saks Fifth Avenue personalize recommendations using by SurrealDB's graph capabilities.

<div class="uppercase text-sm tracking-widest">
Zeyad Deeb
</div>

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/zeyaddeeb/surrealdb-for-genai" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<style>
h1 {
  background-color: #ffff;
  background-image: linear-gradient(45deg, #9A018F 10%, #8B0294 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
---

<div class="flex float-right w-full">

<div class="mx-auto w-5/6"/>

Before we jump in on how awesome is SurrealDB, let us first review  <span v-mark.circle.red="0">LLMs.</span>

</div>
---
---
<div class="flex">
    <div class="relative mb-6 mb-0 w-150">
        <div class="flex items-center">
            <div class="z-10 flex items-center justify-center w-6 h-6 bg-blue-100 rounded-full ring-0 ring-white dark:bg-blue-900 ring-8 dark:ring-gray-900 shrink-0">
            </div>
            <div class="hidden flex w-full bg-gray-200 h-0.5 dark:bg-gray-700"/>
        </div>
        <div class="mt-3 pe-8">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white">LLM</h3>
            <p class="text-base font-normal text-gray-500 dark:text-gray-400">Pick your poison (OpenAI, llama, Anthropic, etc...) </p>
        </div>
    </div>
    <div class="relative mb-6 mb-0 w-150">
        <div class="flex items-center">
            <div class="z-10 flex items-center justify-center w-6 h-6 bg-blue-100 rounded-full ring-0 ring-white dark:bg-blue-900 ring-8 dark:ring-gray-900 shrink-0">
            </div>
            <div class="hidden flex w-full bg-gray-200 h-0.5 dark:bg-gray-700"/>
        </div>
        <div class="mt-3 pe-8">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Memory</h3>
            <p class="text-base font-normal text-gray-500 dark:text-gray-400">Store the actual conversation history for the LLM to know about follow-ups, etc.. </p>
        </div>
    </div>
    <div class="relative mb-6 mb-0 w-150">
        <div class="flex items-center">
            <div class="z-10 flex items-center justify-center w-6 h-6 bg-blue-100 rounded-full ring-0 ring-white dark:bg-blue-900 ring-8 dark:ring-gray-900 shrink-0">
            </div>
            <div class="hidden flex w-full bg-gray-200 h-0.5 dark:bg-gray-700"/>
        </div>
        <div class="mt-3 pe-8">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Prompt Templates</h3>
            <p class="text-base font-normal text-gray-500 dark:text-gray-400">Store the actual conversation history for the LLM to know about follow-ups, etc.. </p>
        </div>
    </div>
    <div class="relative mb-6 mb-0 w-150">
        <div class="flex items-center">
            <div class="z-10 flex items-center justify-center w-6 h-6 bg-blue-100 rounded-full ring-0 ring-white dark:bg-blue-900 ring-8 dark:ring-gray-900 shrink-0">
            </div>
            <div class="hidden flex w-full bg-gray-200 h-0.5 dark:bg-gray-700"/>
        </div>
        <div class="mt-3 pe-8">
            <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Vectorstore</h3>
            <p class="text-base font-normal text-gray-500 dark:text-gray-400">Transforms text and images from documents (or products?!) and store those embeddings in a database somewhere </p>
        </div>
    </div>
</div>

<!-- <div v-click class="absolute top-62 left-160">
  <ul>
    <li>SQLite</li>
    <li>MySQL</li>
    <li>PostgreSQL</li>
    <li class="text-sm" style="margin-left: 17px; padding-left: 7px;">
      SQL Server <span class="chip">Preview</span>
    </li>
    <li class="text-sm" style="margin-left: 17px; padding-left: 7px;">
      MongoDB <span class="chip">Early Access</span>
    </li>
  </ul>
</div> -->

---
---
## LLMs
<br class="my-2"/>

```rust {all|8-9|18-19|20-23|all}
use super::*;
use tokio::io::AsyncWriteExt;
use tokio_stream::StreamExt;

#[tokio::test]
#[ignore]
async fn test_generate() {
    let ollama = Ollama::default().with_model("llama3.1");
    let response = ollama.invoke("Howdy AI,...").await.unwrap();
    println!("{}", response);
}

#[tokio::test]
#[ignore]
async fn test_stream() {
    let ollama = Ollama::default().with_model("llama3.1");

    let message = Message::new_user_message("Show me a product like Nancy Sinatra these boots are made for walking?");
    let mut stream = ollama.stream(&vec![message]).await.unwrap();
    let mut stdout = tokio::io::stdout();
    while let Some(res) = stream.next().await {
        let data = res.unwrap();
        stdout.write(data.content.as_bytes()).await.unwrap();
    }
    stdout.write(b"\n").await.unwrap();
    stdout.flush().await.unwrap();
}
```
---
---

## Memory
<br class="my-2"/>

```rust {all|9-10|all}
use std::sync::Arc;

use anyhow::Result;
use tokio::sync::Mutex;

use crate::schemas::{memory::BaseMemory, messages::Message, prompt::Prompt};

pub struct SimpleMemory {
    messages: Vec<Message>,
    memory: String,
}

impl Into<Arc<dyn BaseMemory>> for SimpleMemory {
    fn into(self) -> Arc<dyn BaseMemory> {
        Arc::new(self)
    }
}
...
```
---
---

## Prompt Templates
<br class="my-2"/>

```rust {all|1|4-19|21|22-24|all}
use handlebars::Handlebars;
#[test]
fn test_chat() {
    let prompt_template = template!(
        "my template",
        r#"
            {{#chat}}
            {{#system}}
            You are an expert at {{programming_language}}. 
            {{/system}}
            {{#user}}
            What is your favorite aspect of {{programming_language}}?
            {{/user}}
            {{#assistant}}
            Type safety, of course!
            {{/assistant}}
            {{/chat}}
        "#
    );
    let mut context = HashMap::new();
    context.insert("programming_language", "rust");
    let prompt = prompt_template
        .render_context("my template", &context)
        .unwrap();
...

```

---
---

## Vectorstore
<br class="my-2"/>
```rust
#[async_trait]
impl<C: Connection> VectorStore for Store<C> {
    async fn add_documents(
        &self,
        docs: &[Document],
        opt: &VecStoreOptions,
    ) -> Result<Vec<String>, Box<dyn Error>> {
        let texts: Vec<String> = docs.iter().map(|d| d.page_content.clone()).collect();

        let embedder = opt.embedder.as_ref().unwrap_or(&self.embedder);

        let vectors = embedder.embed_documents(&texts).await?;
        if vectors.len() != docs.len() {
            return Err(Box::new(std::io::Error::new(
                std::io::ErrorKind::Other,
                "Number of vectors and documents do not match",
            )));
        }
...
```

---
layout: image
image: https://miro.medium.com/v2/resize:fit:1000/format:webp/0*TkYucn7jMJvEv9Uu.jpeg
backgroundSize: contain
---


<div class="absolute bottom-0 left-200 text-sm">
<a href="https://machine-learning-made-simple.medium.com/why-step-by-step-prompting-works-in-language-models-4cf5858f8270"> Source </a>
</div>


---
---

## THE PROBLEM
<div class="mt-4 relative border-s border-gray-200 dark:border-gray-700">                  
    <div class="mb-2 ms-4">
        <div class="absolute w-3 h-3 bg-gray-200 rounded-full mt-2.5 -start-1.5 border border-white dark:border-gray-900 dark:bg-gray-700"/>
        <h3 class="text-lg font-semibold text-gray-900 dark:text-white">User Query</h3>
        <p class="text-base font-normal text-gray-500 dark:text-gray-400">Can I get shoes like what Nancy Sinatra wore in these boots are made for walking?</p>
        <div v-click class="relative border-s border-gray-200 dark:border-gray-700">                  
            <div class="mb-2 ms-4">
                <h5 class="text-gray-900 dark:text-white">Session Facts</h5>
                <p class="font-normal text-gray-500 dark:text-gray-400">User searched for wedding 2 days ago</p>        
                <p class="font-normal text-gray-500 dark:text-gray-400">User returned white boots last year</p>        
            </div>
        </div>
    </div>
    <div v-click class="ms-4">
        <div class="absolute w-3 h-3 bg-gray-200 rounded-full mt-1.5 -start-1.5 border border-white dark:border-gray-900 dark:bg-gray-700"></div>
        <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Assistant Response</h3>
        <p class="text-base font-normal text-gray-500 dark:text-gray-400">Yes, check these out</p>
        <div class="mx-4">
            <div class="flex flex-col-3 gap-2">
                <div>
                    <img class="h-auto rounded-lg" src="https://cdn.saksfifthavenue.com/is/image/saks/0400020357558_NAPPAALMOND?wid=150&hei=150&resMode=sharp2&op_usm=0.9,1.0,8,0&fmt=webp" alt="">
                </div>
                <div>
                    <img class="h-auto rounded-lg" src="https://cdn.saksfifthavenue.com/is/image/saks/0400020193682_CREAM?wid=150&hei=150&resMode=sharp2&op_usm=0.9,1.0,8,0&fmt=webp" alt="">
                </div>
                <div>
                    <img class="h-auto rounded-lg" src="https://cdn.saksfifthavenue.com/is/image/saks/0400021534354_CREAM?wid=150&hei=150&resMode=sharp2&op_usm=0.9,1.0,8,0&fmt=webp" alt="">
                </div>
            </div>
        </div>
    </div>
</div>

---
---
## Pipelines not chains

<br class="my-2"/>
```rust

let pipeline = pipeline
    .load_template(
        "question",
        "{{#chat}}{{#user}}What is my name?{{/user}}{{/chat}}",
    )
    .unwrap();

let prompt = "{{#chat}}{{#user}}My name is Zeyad{{/user}}{{/chat}}";
let pipeline = pipeline.load_template("name", prompt1).unwrap();
pipeline.execute("name1").await.unwrap();

let res = pipeline.execute("question").await.unwrap().content();

assert!(res.to_lowercase().contains("zeyad"));

let mut memory = pipeline.memory.as_ref().unwrap().lock().await;
let messages = memory.memory().to_messages().unwrap();
assert_eq!(messages.len(), 4);

...
```
---
layout: image
image: https://repository-images.githubusercontent.com/436658287/48975688-cf92-4b36-9fe1-fbe0a492a74b
backgroundSize: contain
---
---

## How SurrealDB is different?