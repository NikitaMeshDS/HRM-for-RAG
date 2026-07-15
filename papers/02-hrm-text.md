# HRM-Text: Efficient Pretraining Beyond Scaling

**Авторы:** Guan Wang, Changling Liu, Chenyu Wang, Cai Zhou, Yuhao Sun, Yifei Wu, Shuai Zhen,
Luca Scimeca, Yasin Abbasi Yadkori
**Организация:** Sapient Intelligence + MIT
**Дата:** 20 мая 2026 — свежая статья, #1 Paper of the Day на HuggingFace
**Ссылка:** https://arxiv.org/abs/2605.20613
**Код:** https://github.com/sapientinc/HRM-Text (полный фреймворк, готовые чекпоинты)

---

## TL;DR

Прямое продолжение оригинального HRM от той же команды — применение dual-timescale recurrent
архитектуры к полноценному языковому моделированию (не игрушечные задачи, а настоящая LLM).
1B-модель, обученная с нуля на 40B токенов, конкурирует с моделями, обученными на 130-600x
больше compute и 150-900x больше данных. Готовый чекпоинт на HuggingFace, полный код обучения.

---

## Проблема

Стандартный pretraining LLM — brute-force scaling: массивный compute + интернет-масштаба сырой
текст. Много compute тратится на предсказание нерелевантного текста просто ради общих
представлений. Это закрывает foundational pretraining research для всех кроме гигантских
лабораторий.

## Идея

Снова биологическое вдохновение — **frontoparietal loop** (лобно-теменная петля мозга),
демонстрирующая sample-efficient обучение через multi-timescale processing.

## Три архитектурных нововведения (по сравнению с оригинальным HRM)

1. **MagicNorm** — новая нормализация для стабилизации глубокой рекуррентности именно в
   языковом моделировании (RMSNorm оригинала оказался недостаточен для текста)

2. **Warmup deep credit assignment** — модификация one-step gradient approximation, адаптированная
   для стабильного старта на языковых данных

3. **Task-completion objective + PrefixLM masking** — ключевое отличие: вместо raw-text next-token
   prediction на всём подряд, модель обучается **исключительно на парах инструкция-ответ**.
   Two-pass attention: bidirectional проход по префиксу (инструкции), causal проход по ответу.

## Результаты

| Размер | GPU | Время | Цена | GSM8k | MATH | MMLU | ARC-C |
|--------|-----|-------|------|-------|------|------|-------|
| L (0.6B) | 8×H100 | 50ч | ~$800 | 77.6% | 51.2% | 56.6% | 75.9% |
| XL (1B) | 16×H100 | 46ч | ~$1472 | 84.7% | 56.5% | 60.7% | 81.9% |

Обучение всего на 40B уникальных токенов — против триллионов токенов у обычных LLM сравнимого
качества.

## Инфраструктура (что доступно уже сейчас)

- Готовый чекпоинт `sapientinc/HRM-Text-1B` на HuggingFace (29.7k+ скачиваний)
- Полный тренировочный код: PyTorch FSDP2, FlashAttention 3
- Baseline-архитектуры для сравнения прямо в конфигах: TRM (Tiny Recursive Model), RINS
  (Recursive Inference Scaling), Universal Transformer, обычный Transformer
- Готовый evaluation pipeline (`evaluation/main.py`) под GSM8k, MATH, DROP, ARC, MMLU и др.
- Checkpoint conversion в HuggingFace Transformers формат

---

## Связь с проектом HRM for RAG

**Это меняет практический план проекта.** Вместо адаптации оригинального HRM (заточенного под
судоку/лабиринты) под язык с нуля — можно взять уже готовый HRM-Text-1B и дообучить его под
RAG-специфику.

**Сильная идея для дальнейшего исследования:** PrefixLM-механизм HRM-Text архитектурно идеально
ложится на RAG — retrieved документы естественно становятся bidirectional prefix (модель
свободно смотрит на них туда-сюда), а генерация ответа — causal частью. Это практически
готовая RAG-архитектура, просто нужно направить retrieved context в prefix-слот.

**Практический план:**
1. Взять `sapientinc/HRM-Text-1B` с HuggingFace
2. Дообучить на multi-hop QA данных (HotpotQA/MuSiQue) с retrieved context в prefix
3. Сравнить с обычным LLM аналогичного размера в RAG-режиме
4. Baseline для сравнения архитектур уже есть в репозитории (TRM, RINS, Universal Transformer)

## Открытые вопросы

- Известно ли про HRM-Text? Стоит ли строить эксперимент поверх него вместо оригинального HRM?
- Использовать ли PrefixLM-механизм напрямую для подачи retrieved context?
- Нужна ли XL (1B) версия или B/L достаточно для целей проекта (бюджет GPU)?
