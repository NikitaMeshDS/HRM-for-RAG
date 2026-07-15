# HRM for RAG — SMILES'26

Проект летней школы SMILES 2026.

## О проекте

Исследование применения Hierarchical Reasoning Models (HRM) к Retrieval-Augmented Generation.
HRM демонстрируют сильные reasoning-способности при скромных фактических знаниях — RAG
компенсирует этот пробел, оставляя reasoning-механизм модели нетронутым.

Два возможных направления:
- **(A) Text RAG** — knowledge-intensive QA (HotpotQA, MuSiQue, 2WikiMultihopQA)
- **(B) Code RAG** — repository-level software engineering repair (SWE-bench-подобные задачи)

## Структура репозитория

```
hrm-rag-project/
├── README.md              # этот файл
├── papers/                 # ревью ключевых статей (см. papers/README.md)
├── src/                     # (будет добавлено) код проекта
├── data/                    # (будет добавлено) датасеты
└── experiments/             # (будет добавлено) логи экспериментов
```

## Ссылки

- HRM оригинальная статья: https://arxiv.org/abs/2506.21734
- HRM-Text (продолжение, май 2026): https://arxiv.org/abs/2605.20613
- HRM-Text код: https://github.com/sapientinc/HRM-Text
- HRM оригинальный код: https://github.com/sapientinc/HRM
