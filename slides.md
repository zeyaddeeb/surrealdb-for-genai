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

Personalized recommendations powered by SurrealDB's graph capabilities.

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
layout: intro
image: https://repository-images.githubusercontent.com/436658287/48975688-cf92-4b36-9fe1-fbe0a492a74b
---

<div class="flex float-right w-full">

<div class="mx-auto w-5/6"/>

Before we jump in on how awesome <span v-mark.red="0"> SurrealDB </span> is, let us first review LLMs.

</div>
