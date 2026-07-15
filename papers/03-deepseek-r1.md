# DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning

**Авторы:** DeepSeek-AI (Daya Guo et al., ~100+ авторов)
**Дата:** 22 января 2025 (v2: 4 января 2026)
**Опубликовано:** Nature, том 645, стр. 633-638 (2025) — редкий случай для ML-статьи
**Ссылка:** https://arxiv.org/abs/2501.12948

---

## TL;DR

Reasoning-способности LLM можно получить через чистый reinforcement learning, без единого
человеческого reasoning-примера. R1-Zero, обученная чистым RL поверх DeepSeek-V3-Base (671B),
демонстрирует эмерджентные reasoning-паттерны — self-reflection, verification, "aha moments" —
которые никто явно не программировал.

---

## Проблема

Reasoning в LLM обычно достигают через SFT на человеческих демонстрациях — дорого, ограничено
качеством/объёмом разметки. Вопрос: можно ли получить reasoning вообще без размеченных
человеком reasoning-цепочек — чистым RL?

## Два варианта модели

**DeepSeek-R1-Zero** — эксперимент в чистом виде: DeepSeek-V3-Base + чистый RL (GRPO), ноль
supervised fine-tuning. Модель сама выучилась удлинять рассуждение на сложных задачах,
перепроверять шаги, демонстрировать "aha moment" (буквально "Wait, let me re-check this"
посреди рассуждения). AIME 2024: с 15.6% до 71.0% (сопоставимо с o1), с majority voting — 86.7%.
Проблема: плохая читаемость, language mixing (переключение языка посреди фразы).

**DeepSeek-R1** — та же идея + "холодный старт" (немного SFT перед RL) для исправления
читаемости, затем многоступенчатый pipeline (RL → rejection sampling + SFT → diverse RL phase).

## GRPO (Group Relative Policy Optimization) — ключевая инновация

Убирает отдельную critic-модель (обычно сравнимую по размеру с policy). Вместо этого:
для одного промпта генерируется группа из G ответов, каждый получает reward, награды
нормализуются **относительно группы**:

```
r̄(oᵢ) = (r(oᵢ) - μ) / σ,  где μ, σ — среднее и std по группе
```

Эффект: сокращение памяти на 40-60% против PPO, по некоторым оценкам до 18x дешевле по compute.

**Reward:** rule-based, не LLM-судья — accuracy reward (проверяемый финальный ответ) + format
reward (`<think>...</think>` + `<answer>...</answer>` теги). Избегает reward hacking.

## Дистилляция

Reasoning-паттерны большой R1 систематически передаются в маленькие модели (1.5B-70B, Qwen/Llama
базы) через дообучение на reasoning-траекториях — маленькие модели неожиданно хорошо наследуют
способности.

## Критика (важно!)

Статья "Understanding R1-Zero-Like Training: A Critical Perspective" (arxiv 2503.20783):
- "Aha moment" возможно не такой уж эмерджентный — базовая модель DeepSeek-V3-Base уже
  демонстрирует похожие паттерны даже без RL (pretraining bias?)
- GRPO имеет optimization bias — искусственно удлиняет ответы (особенно неправильные).
  Предложен Dr. GRPO как исправление.

---

## Связь с проектом HRM for RAG

DeepSeek-R1 и HRM решают одну мета-проблему (сильный reasoning без огромных supervised данных)
радикально разными путями:

|  | DeepSeek-R1 | HRM |
|---|---|---|
| База | Готовая большая LLM (671B) | Архитектура с нуля (27M) |
| Reasoning выражается | Явно, текстом (`<think>` теги, long CoT) | Скрыто, в hidden-space |
| Стоимость reasoning | Дорого (тысячи токенов) | Дёшево (фиксированные forward pass) |
| Данные | RL на верифицируемых задачах | 1000 примеров, без CoT |

**Ключевой вопрос для проекта:** если RAG требует multi-hop рассуждения по документам — может ли
HRM-подобный дешёвый latent reasoning заменить дорогой explicit CoT reasoning (как у R1),
сохранив качество? Это и есть reasoning-knowledge decomposition, лежащая в основе проекта.

## Открытые вопросы

- Сравниваем HRM именно с DeepSeek-R1/QwQ как baseline, или с более широким классом моделей?
- Насколько важна интерпретируемость reasoning (explicit CoT у R1 vs latent у HRM) для RAG-задачи?
