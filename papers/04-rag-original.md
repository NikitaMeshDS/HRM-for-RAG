# Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

**Авторы:** Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin,
Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel,
Douwe Kiela
**Организация:** Facebook AI Research (FAIR)
**Дата:** 2020
**Ссылка:** https://arxiv.org/abs/2005.11401

---

## TL;DR

Оригинальная статья, давшая имя RAG. Комбинирует pre-trained parametric memory (seq2seq модель)
с non-parametric memory (dense vector index Wikipedia через Dense Passage Retriever) для
knowledge-intensive генерации. Достигает SOTA на open-domain QA, генерирует более специфичный,
разнообразный и фактически точный текст чем чисто parametric подходы.

---

## Проблема

Параметрические знания LLM: (1) сложно расширять/пересматривать напрямую, (2) не дают прямой
инсайт почему модель что-то "решила", (3) могут галлюцинировать. Non-parametric memory (retrieval
из внешней базы) решает всё три проблемы, но раньше исследовалась только для extractive задач,
не для генерации.

## Метод

Два основных компонента:
- **Retriever** — Dense Passage Retriever (DPR), возвращает top-K релевантных пассажей по dense
  vector similarity между query и passage эмбеддингами
- **Generator** — seq2seq модель (BART), генерирует ответ, обусловленный на retrieved пассажи и
  исходный запрос

Два варианта комбинирования:
- **RAG-Sequence** — один retrieved документ используется для генерации всей последовательности
- **RAG-Token** — на каждом шаге генерации можно опираться на разные документы

Retriever и generator обучаются **совместно** (end-to-end), при этом document index остаётся
фиксированным во время обучения.

## Результаты

State-of-the-art на трёх open-domain QA бенчмарках, превосходя как чисто parametric seq2seq
модели, так и task-specific retrieve-and-extract архитектуры. Генерирует более специфичный,
разнообразный и фактически точный текст по сравнению с parametric-only baseline (BART).

---

## Связь с проектом HRM for RAG

Это архитектурная база, от которой отталкивается весь RAG-подход, включая современные варианты.
Ключевая идея, которая сохраняется и сегодня — разделение parametric (генеративные способности) и
non-parametric (фактические знания) memory. Именно эта дихотомия лежит в основе проекта: HRM =
сильный parametric reasoning, RAG добавляет non-parametric knowledge.

Современные RAG-системы (2024-2026) далеко ушли от оригинальной RAG-Sequence/RAG-Token схемы —
используются LLM вместо BART, продвинутый retrieval (hybrid, reranking), agentic RAG паттерны.
Но базовое разделение ролей retriever/generator остаётся тем же.

## Что стоит помнить при чтении современных статей

- RAG-Token vs RAG-Sequence — прообраз современных discussions про "когда именно делать
  retrieval" (single-shot vs iterative retrieval, ближе к HRM-подобным multi-hop сценариям)
- End-to-end joint training retriever+generator — в современных системах retriever и generator
  часто независимы (off-the-shelf embedding модели + любой LLM), это упрощение оригинальной идеи
