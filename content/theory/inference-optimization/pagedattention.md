---
title: "PagedAttention"
description: Улучшите использование памяти LLM с помощью (KV) с использованием PagedAttention.
date: 2026-02-28
weight: 50
keywords:
    - vLLM, Hugging Face TGI, TensorRT-LLM
    - PagedAttention
    - KV cache, KV cache optimization, KV caching
    - LLM inference optimization, LLM inference optimization techniques​
    - Speed up LLM inference
---

[PagedAttention](https://blog.vllm.ai/2023/06/20/vllm.html) — это эффективный по памяти способ реализации механизма внимания (attention) в LLM.

Когда LLM генерирует ответ, ей нужно [хранить информацию о предыдущих токенах (KV cache) для каждого токена](../llm-inference-basics/how-does-llm-inference-work#the-two-phases-of-llm-inference). Обычно KV cache занимает большой кусок памяти, потому что хранится как один сплошной блок. Это приводит к фрагментации памяти или её неэффективному использованию: приходится резервировать большой блок даже если он не полностью заполнен.

PagedAttention разбивает этот большой блок на маленькие, как страницы в книге. То есть KV cache хранится в разрозненных (непрерывных) блоках. Для отслеживания используется таблица соответствия (lookup table). LLM загружает только те блоки, которые нужны в данный момент, а не всё сразу.

Это экономит память и делает процесс более эффективным. Более того, одни и те же блоки могут переиспользоваться для разных выходов, если это необходимо.

PagedAttention впервые реализован во vLLM. Позже его внедрили и другие проекты: Hugging Face TGI, TensorRT-LLM и др.


## Дополнительные ресурсы
  * [Efficient Memory Management for Large Language Model Serving with PagedAttention](https://arxiv.org/abs/2309.06180)
