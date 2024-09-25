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

How to personalize recommendations using by SurrealDB's graph capabilities.

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

Before we jump in on how awesome is SurrealDB, let us first review  <span v-mark.circle.red="1">LLMs.</span>

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
            <p class="text-base font-normal text-gray-500 dark:text-gray-400">
            The context you pass to the LLM, memory, user query, etc...</p>
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
            <p class="text-base font-normal text-gray-500 dark:text-gray-400">The place where you store transformed text (and/or images) from documents (or products?!) to embeddings in a database somewhere </p>
        </div>
    </div>
</div>


---
---
## LLMs
<br class="my-2"/>

```rust {all|8-9|18-19|20-23|all}
use super::*;
use tokio::io::AsyncWriteExt;
use tokio_stream::StreamExt;

#[tokio::test]
async fn test_generate() {
    let ollama = Ollama::default().with_model("llama3.1");
    let response = ollama.invoke("Howdy AI,...").await.unwrap();
    println!("{}", response);
}

#[tokio::test]
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
                <h5 class="text-gray-900 dark:text-white">Context Facts</h5>
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

## SurrealDB Superpowers

<ol class="mt-4">
    <div v-click class="my-8">
        <li class="text-lg font-bold uppercase"> Rust </li>
        <div class="w-3/5 my-2"> It is a reliable and high-performance language. If you want software to work for years you are in the right place. </div>
    </div>
    <div v-click class="my-8">
        <li class="text-lg font-bold uppercase">Flavorless (in a good way)</li>
        <div class="w-3/5 my-2"> Blending the strengths of SQL, NoSQL, and graph databases</div>
    </div>
    <div v-click class="my-8">
        <li class="text-lg font-bold uppercase">Simplicity</li>
        <div class="w-3/5 my-2"> 
        Schemaful or Schemaless, defining data objects is super easy to accomplish within minutes.
    </div>
</div>    
</ol>

---
---

## Sample models

<div class="my-4">What is actually defined in production!</div>

```rust
#[derive(Clone, Debug, PartialEq, Serialize, Deserialize, SimpleObject)]
#[graphql(concrete(name = "LTV", params()))]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
pub struct LTV {
    pub gross_margin_online_52w_sum: Option<f64>,
    ...
    pub ltv_ol_gm_group: Option<String>,
}
```

<br class="my-2" />

```rust  {all|1-2|3-7|8|all}
let client = any::connect("memory").await.unwrap();
client.use_ns("ml").use_db("ltv").await.unwrap();
let model = LTV {
    gross_margin_online_52w_sum: Some(0.5),
    ...
    ltv_ol_gm_group: Some("test".to_owned()),
};
let res = Mutation::create_customer(&client, model.clone()).await.unwrap();
assert_eq!(res.gross_margin_online_52w_sum, model.gross_margin_online_52w_sum);
```

---
---

## Sample models, continued

<div class="my-4">Also, production! Painless linking</div>

```rust {all|8-15|all}
async fn link_product_variant(
    db: &Surreal<Any>,
    product_code: &str,
    item_id: &str,
) -> Result<(), Error> {
    let product_id: String = format!("product_{}", &product_code);
    let variant_id: String = format!("variant_{}", &item_id);
    let sql = format!(
        r#"
        relate product:{}->product_variant->variant:{}
        "#,
        product_id, variant_id
    );

    db.query(sql).await?;

    Ok(())
}
```
---
---

## LLM Contexts

All LLM context facts can be easily grabbed in one query

```rust
pub async fn find_product(
    db: &Surreal<Any>,
    product_ids: Vec<&str>,
) -> Result<Vec<Product>, Error> {
    let sql: String = format!(
        r#"
        select *
        , ->product_option_group->option_group.* as options
        , ->product_variant->variant.* as variants
        from [{}]
        parallel;
        "#,
        product_ids.join(", ")
    );
    let mut response = db.query(sql).await?;
    let products: Vec<Product> = response.take(0)?; // or many?!
    Ok(products)
}
```


---
---

## Embeddings

SurrealDB offers native support for embeddings

```sql
DEFINE FUNCTION IF NOT EXISTS fn::embeddings_complete($embedding_model: string, $input: string) {
    RETURN http::post(
        "http://ollama.svc/v1/embeddings",
        {
            "model": $embedding_model,
            "input": $input
        })["data"][0]["embedding"]
};
```

```sql
DEFINE FUNCTION IF NOT EXISTS fn::search_for_documents($input_vector: array<float>, $threshold: float) {
   LET $results = (
     SELECT  url, title, text,
        vector::similarity::cosine(content_vector, $input_vector) AS similarity
    FROM wiki_embedding
    WHERE content_vector <|1|> $input_vector
    ORDER BY similarity DESC
    LIMIT 5
   );
   RETURN { results: $results, count: array::len($results), threshold: $threshold};
};
```
<div class="float-right">
<a class="text-sm" href="https://surrealdb.com/blog/building-a-retrieval-augmented-generation-rag-app-with-openai-and-surrealdb"> Source</a>
</div>

---
layout: image
image: https://joshowens.dev/static/bbbfef552d51fc10bcee51a3e1de614f/beb1d/but_does_meteor_scale_meme.jpg
backgroundSize: contain
---


<div class="absolute bottom-0 left-200 text-sm">
<a href="https://joshowens.dev/but-does-meteor-scale"> Source </a>
</div>


---
---
## What about scale?

For highly-available and highly-scalable setups, SurrealDB offers you some options for distributed backends:

<ol>
    <li> <span class="underline">TiKV</span>: SurrealDB can be run on top of a TiKV cluster.
    </li>
    <li> <span class="underline">FoundationDB</span>: SurrealDB can be run on top of a FoundationDB cluster as well.
    </li>
</ol>

<br class="mt-4"/>
See documentation <a href="https://surrealdb.com/docs/surrealdb/installation/running/tikv"> here</a>

---
---
## TIKV

Deploy a TIKV cluster on kubernetes, sample:

```yaml {all}{maxHeight:'200px'}
apiVersion: pingcap.com/v1alpha1
kind: TidbCluster
metadata:
  name: {{ .Values.cluster.name }}
  annotations:
    tikv.tidb.pingcap.com/delete-slots: '[]'
spec:
  version: v8.1.1
  timezone: UTC
  pd:
    baseImage: pingcap/pd
    maxFailoverCount: 0
    replicas: {{ .Values.cluster.numReplicas }}
    storageClassName: {{ .Values.cluster.storageClassName }}
    requests:
      cpu: 500m
      memory: 1Gi
      storage: 2Gi
  tikv:
    baseImage: pingcap/tikv:v8.1.1
    replicas:  {{ .Values.cluster.numReplicas }}
    storageClassName: {{ .Values.cluster.storageClassName }}
    requests:
      cpu: {{ .Values.cluster.cpu }}
      storage:  {{ .Values.cluster.storageSize }}
      memory: {{ .Values.cluster.memory }}
```

Connect SurrealDB to the `tikv://` endpoint


```yaml {all|7|all}
image:
  repository: surrealdb/surrealdb
  pullPolicy: IfNotPresent
  tag: v1.5.0

surrealdb:
  path: tikv://cluster.tikv:<port>
  port: 8000
```
---
---
## FoundationDB

Deploy a FoundationDB cluster on kubernetes, sample:

```yaml {all}{maxHeight:'200px'}
apiVersion: apps.foundationdb.org/v1beta2
kind: FoundationDBCluster
metadata:
  name: name
spec:
  version: 7.1.42
  labels:
    filterOnOwnerReference: false
    matchLabels:
      foundationdb.org/fdb-cluster-name: name
    processClassLabels:
      - foundationdb.org/fdb-process-class
    processGroupIDLabels:
      - foundationdb.org/fdb-process-group-id
  routing:
    defineDNSLocalityFields: true
  minimumUptimeSecondsForBounce: 60
  databaseConfiguration:
    redundancy_mode: triple
    logs: 3
    storage: 3
  processCounts:
    cluster_controller: 2
    coordinator: 3
    storage: 3
    log: 3
```
Connect SurrealDB to the `fdb://` config file

```yaml {all|7|all}
image:
  repository: surrealdb/surrealdb
  pullPolicy: IfNotPresent
  tag: v1.5.0

surrealdb:
  path: fdb:///etc/foundationdb/fdb.cluster
  port: 8000
```
---
layout: image
image: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhtW2ITijgWzPdUi1jFjMRuwmFnvdVPz2D8WmXg_O7HLhrVwxK7wqTOU6qhl2S60kg0LwFoNr2M0o2wo0WDE1tQSe_482j4AXQzaz1NuVfcvQqjmEvgXjgel_3DfddOW6l31kf_O7BbgxY/s1600/foun2.png
backgroundSize: contain
---
---
class: text-center
layout: cover
---
# Databases are hard... <br class="my-4"/> SurrealDB makes it <br /> Easy Breezy!

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

# Why did we choose FoundationDB backend?

```rust
let streamer = Kafka::connect(
    std::env::var("KAFKA_BOOTSTRAP_SERVERS")
        .unwrap_or_else(|_| "kafka://localhost:9092".to_owned())
        .parse()
        .unwrap(),
    Default::default(),
)
.await?;

let payload = message.message();
let json = payload.as_str().unwrap();
let data: Product = serde_json::from_str(json)?;

...

let data = Mutation::create_product_from_raw(&db, data.payload).await?;

producer.send(serde_json::to_string(&data))?;

...
```

---
---

# Embedding Models

```rust
match arch.first().map(String::as_str) {
    Some("BertModel") => {
        let config: BertConfig = serde_json::from_str(config_str)?;
        ModelConfig::Bert(config)
    }

    Some("JinaBertForMaskedLM") => {
        let config: JinaBertConfig = serde_json::from_str(config_str)?;
        ModelConfig::JinaBert(config)
    }
    Some("CLIPModel") => {
        let config: ClipConfig = ClipConfig::vit_base_patch32();
        ModelConfig::Clip(config)
    }

...
```

---
---

# No Langchain

```rust
use async_trait::async_trait;

use crate::schemas::Error;

#[async_trait]
pub trait Embedder: Send + Sync {
    async fn embed_documents(&self, documents: &[String]) -> Result<Vec<Vec<f64>>, Error>;
    async fn embed_query(&self, text: &str) -> Result<Vec<f64>, Error>;
    async fn embed_image(&self, image: &[u8]) -> Result<Vec<f64>, Error>;
    async fn embed_voice(&self, voice: &[u8]) -> Result<Vec<f64>, Error>;
    async fn embed_voice_batch(&self, voices: &[Vec<u8>]) -> Result<Vec<Vec<f64>>, Error>;
}

...

let router_layer = RouteLayerBuilder::default()
    .embedder(Ollama::default())
    .add_route(returns_route)
    .add_route(products_route)
    .aggregation_method(AggregationMethod::Sum)
    .build()
    .await
    .unwrap();
...
```

---
---

# Fun Stats

<ul>
    <li>SurrealDB serves ~4M customers monthly</li>
    <li>SurrealDB serves at least ~1.5M recommendations on weekly basis</li>
    <li>Zero incidents in production since launch</li>
</ul>

---
class: text-center
layout: cover
---
# Questions

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
