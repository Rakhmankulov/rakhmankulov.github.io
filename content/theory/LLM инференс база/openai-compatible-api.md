---
title: OpenAI-совместимый API
description: An OpenAI-compatible API implements the same request and response formats as OpenAI's official API, allowing developers to switch between different models without changing existing code.
date: 2026-02-22
weight: 93

keywords:
    - OpenAI-compatible API, OpenAI-compatible endpoint, OpenAI-compatible server
    - OpenAI API, OpenAI compatibility, ChatGPT
    - LLM inference API
---

Когда LLM запущена, нужен стандартный способ взаимодействия с ней. Для этого и предназначен OpenAI-совместимый API.

## Что такое OpenAI-совместимый API?

OpenAI-совместимый API — это любой API, который повторяет интерфейс, схему запросов/ответов и модель аутентификации оригинального API OpenAI. Хотя OpenAI официально не объявляла этот формат стандартом, их API стал де-факто интерфейсом для LLM.

Взлёт ChatGPT в конце 2022 года показал, насколько этот подход мощный и удобный:

- Чистый, хорошо документированный API позволяет разработчикам легко создавать приложения с LLM.
- Модели вроде `gpt-4o` доступны через простые и единообразные эндпоинты.

В результате API быстро распространился и стал стандартом во многих сферах.

## Почему важна совместимость?

API OpenAI помогли запустить волну AI-разработки, но их широкое распространение создало эффект «запертой экосистемы». Многие инструменты, фреймворки и SDK теперь заточены под схему OpenAI. Это становится проблемой, если вы хотите:

- Перейти на другую модель
- Перейти на self-hosted
- Попробовать нового провайдера инференса

В таких случаях переписывать логику приложения под новый API — долго и рискованно.

OpenAI-совместимые API решают эти задачи:

- **Drop-in замена**: Можно заменить hosted API OpenAI на свой self-hosted или open-source, не меняя код приложения.
- **Бесшовная миграция**: Переход между провайдерами или self-hosted с минимальными изменениями.
- **Единая интеграция**: Совместимость с инструментами и фреймворками, которые используют схему OpenAI (например, эндпоинты `chat/completions`, `embeddings`).

Многие [бэкенды инференса](../getting-started/choosing-the-right-inference-framework) (например, vLLM и SGLang) и фреймворки сервинга моделей (например, BentoML) сразу предоставляют OpenAI-совместимые эндпоинты. Это позволяет легко переключаться между моделями без изменения клиентского кода.

## Как вызвать OpenAI-совместимый API

Вот пример, как просто перенаправить существующий OpenAI-клиент на self-hosted или альтернативный эндпоинт:

```python
from openai import OpenAI

# Используйте свой URL и API-ключ
client = OpenAI(
    base_url="https://your-custom-endpoint.com/v1",
    api_key="your-api-key"
)

response = client.chat.completions.create(
    model="your-model-name",
    messages=[
        {"role": "system", "content": "Вы — полезный ассистент."},
        {"role": "user", "content": "Как интегрировать OpenAI-совместимые API?"}
    ]
)

print(response.choices[0].message)
```

Обратите внимание: OpenAI API требует поле `api_key`. Большинство фреймворков инференса не проверяют это значение, так что можно использовать любое, например `api_key="EMPTY"`.

Можно также вызвать API напрямую через HTTP-запрос. Пример с `curl`:

```bash
curl https://your-custom-endpoint.com/v1/chat/completions \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "your-model-name",
    "messages": [
      {"role": "system", "content": "Вы — полезный ассистент."},
      {"role": "user", "content": "Как интегрировать OpenAI-совместимые API?"}
    ]
  }'
```

Если вы уже используете SDK или REST-интерфейс OpenAI, просто перенаправьте их на свой эндпоинт. Это позволит контролировать развёртывание LLM, снизить зависимость от провайдера и сделать приложение более устойчивым к изменениям.

## Частые вопросы

### OpenAI-совместимый API — это тот же официальный API OpenAI?

Нет. Он только повторяет интерфейс, но не модель или инфраструктуру. Это как говорить на одном «языке», но с разными системами. В зависимости от провайдера бэкенд может быть:

- self-hosted LLM (например, Llama или DeepSeek)
- hosted-провайдер (например, Together AI или Fireworks)
- кастомный корпоративный деплой внутри VPC

Каждый бэкенд отличается по скорости и стоимости, даже если форма API одинаковая.

### Какие модели можно запускать за OpenAI-совместимым API?

Любую современную open-source LLM: Llama, Qwen, Mistral, DeepSeek, Kimi и специализированные дообученные модели.

Если используете фреймворки вроде vLLM и SGLang, они автоматически предоставляют OpenAI-совместимые эндпоинты для этих моделей.

### Обязательно ли нужен OpenAI-совместимый API для self-hosted LLM?

Не строго обязательно, но очень желательно. Без него придётся вручную переписывать интеграции агентов, SDK, совместимость с фреймворками и т.д. Использование схемы OpenAI делает стек проще и переносимее.

### Экономит ли OpenAI-совместимый API деньги?

Сам по себе — нет. Формат API — это только интерфейс, он не удешевляет инференс.

Экономия зависит от того, где работает API:

- **Если вы self-host LLM через vLLM и SGLang**, платите в основном за GPU, а не за токены. Можно применять оптимизации инференса ([выгрузка KV-кэша](../inference-optimization/kv-cache-offloading), [дисагрегация prefill-decode](../inference-optimization/prefill-decode-disaggregation)), чтобы ещё сильнее снизить стоимость. Это обычно намного дешевле для постоянных или крупных нагрузок.
- **Если используете hosted-провайдера (например, Together AI, Fireworks)**, платите за токены или запросы, даже если API «OpenAI-совместимый».
- **Если остаетесь на OpenAI**, платите за токены по их тарифам.

Причина экономии у AI-команд — не OpenAI-совместимый API, а возможность self-host любой модели без переписывания клиентского кода. Подробнее: [serverless vs. self-hosted LLM inference](./serverless-vs-self-hosted-llm-inference).

## Дополнительные ресурсы
>  * [Документация OpenAI](https://platform.openai.com/docs/quickstart?api-mode=chat)
>  * [Примеры: сервинг LLM с OpenAI-совместимыми API](https://github.com/bentoml/BentoVLLM)
