# Анализ банковских транзакций и выявление мошенничества

[Русская версия](#русская-версия) | [English version](#english-version)

---

# Русская версия

## О проекте

Проект посвящён анализу банковских транзакций и выявлению потенциально мошеннических операций на основе данных BankSim.

Основная задача — исследовать поведение клиентов, найти признаки подозрительных транзакций и разработать набор антифрод-правил. На основе этих правил формируется скоринговая оценка риска, позволяющая выделять операции для дополнительной проверки.

Проект носит исследовательский характер: разработанные правила не являются готовой банковской антифрод-системой.

## Данные

Используется датасет [BankSim](https://www.kaggle.com/datasets/ealaxi/banksim1).

Датасет содержит около 595 тысяч синтетических транзакций, среди которых примерно 1,2% отмечены как мошеннические.

Основные признаки:

- `step` — временной шаг симуляции;
- `customer` — идентификатор клиента;
- `age` — категория возраста;
- `gender` — категория пола;
- `merchant` — идентификатор торговой точки;
- `category` — категория покупки;
- `amount` — сумма транзакции;
- `fraud` — целевой признак (1 — мошенническая операция, 0 — обычная).

Данные являются синтетическими. Валюта и единицы измерения сумм не указаны.

## Структура проекта

```text
bank-fraud-analysis/
├── data/
│   └── raw/
├── notebooks/
│   ├── 01_eda.ipynb
│   └── 02_fraud_detection_rules.ipynb
├── reports/
├── README.md
└── requirements.txt
```

## Этапы работы

### 1. Исследовательский анализ данных (EDA)

- Изучение структуры и качества данных.
- Анализ распределения сумм транзакций.
- Исследование доли мошеннических операций.
- Сравнение обычных и мошеннических транзакций.
- Анализ мошенничества по категориям покупок, торговым точкам и клиентам.
- Исследование временных закономерностей.

В ходе анализа обнаружено, что мошеннические операции в среднем имеют значительно более высокие суммы, чем обычные. Также выявлены различия в уровне риска между категориями покупок и торговыми точками.

### 2. Разработка антифрод-правил

Разработаны три правила для выявления подозрительных операций:

1. **Крупная сумма** — транзакция превышает 95-й перцентиль сумм обычных операций.
2. **Нетипичная сумма для клиента** — сумма транзакции значительно превышает среднюю сумму операций клиента.
3. **Частые транзакции** — клиент совершает две или более операций за один временной шаг симуляции.

Для оценки качества отдельных правил использованы метрики Precision и Recall.

| Правило | Precision | Recall |
|---|---:|---:|
| Крупная сумма | 0.178 | 0.881 |
| Нетипичная сумма | 0.121 | 0.563 |
| Частые транзакции | 0.095 | 0.575 |

На основе трёх правил сформирован `risk_score` — количество сработавших правил для каждой транзакции. Значение может находиться в диапазоне от 0 до 3.

Результаты оценки для разных порогов риска:

| Порог риска | Precision | Recall |
|---|---:|---:|
| `risk_score >= 1` | 0.096 | 0.953 |
| `risk_score >= 2` | 0.167 | 0.723 |
| `risk_score >= 3` | 0.242 | 0.344 |

Чем выше порог, тем меньше операций попадает на проверку, но тем больше мошеннических транзакций система может пропустить.

В рамках разработки антифрод-правил также проведён анализ подозрительных операций:

- исследование отдельных транзакций с максимальной оценкой риска;
- анализ категорий покупок и торговых точек с повышенной долей мошенничества;
- выявление клиентов с большим количеством подозрительных операций;
- разбор отдельных случаев для понимания причин срабатывания правил.

Анализ показал, что подозрительная операция не обязательно должна иметь экстремально большую сумму. Важными сигналами могут быть существенное отклонение от привычного поведения клиента и высокая частота операций.

## Основные выводы

- Сумма транзакции — значимый признак для выявления мошенничества.
- Сравнение операции с историей клиента позволяет обнаруживать нетипичные расходы.
- Частота транзакций может выступать дополнительным сигналом риска.
- Комбинация нескольких правил позволяет гибко настраивать чувствительность системы.
- Категории покупок и торговые точки дают дополнительную информацию для оценки риска.

Срабатывание правила не доказывает факт мошенничества. Такие операции следует рассматривать как кандидатов на дополнительную проверку.

## Ограничения

- Используется синтетический датасет, который может не отражать реальные закономерности банковских операций.
- Разработанные пороги и правила нельзя напрямую переносить на реальные банковские данные.
- При расчёте средней суммы клиента использовалась информация из всего датасета. Для реальной системы необходимо учитывать только историю, доступную на момент совершения операции, чтобы избежать утечки данных.
- Правила требуют дальнейшей проверки и настройки на новых данных.

## Технологии

- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- Jupyter Notebook

## Запуск проекта

1. Клонировать репозиторий:

```bash
git clone https://github.com/alphakentavra/bank-fraud-analysis.git
cd bank-fraud-analysis
```

2. Установить зависимости:

```bash
pip install -r requirements.txt
```

3. Скачать датасет [BankSim](https://www.kaggle.com/datasets/ealaxi/banksim1) и поместить файл `bs140513_032310.csv` в папку `data/raw/`.

4. Запустить JupyterLab:

```bash
jupyter lab
```

---

# English version

## About the project

This project focuses on analyzing banking transactions and identifying potentially fraudulent operations using the BankSim dataset.

The main goal is to explore customer behavior, identify suspicious transaction patterns, and develop a set of rule-based fraud detection methods. These rules are combined into a risk score that helps prioritize transactions for further investigation.

This is an exploratory project, not a production-ready fraud detection system.

## Dataset

The project uses the [BankSim dataset](https://www.kaggle.com/datasets/ealaxi/banksim1).

It contains approximately 595,000 synthetic transactions, with around 1.2% labeled as fraudulent.

Key features:

- `step` — simulation time step;
- `customer` — customer identifier;
- `age` — age category;
- `gender` — gender category;
- `merchant` — merchant identifier;
- `category` — purchase category;
- `amount` — transaction amount;
- `fraud` — target variable (1 — fraudulent, 0 — legitimate).

The dataset is synthetic. The currency and units of transaction amounts are not specified.

## Project structure

```text
bank-fraud-analysis/
├── data/
│   └── raw/
├── notebooks/
│   ├── 01_eda.ipynb
│   └── 02_fraud_detection_rules.ipynb
├── reports/
├── README.md
└── requirements.txt
```

## Workflow

### 1. Exploratory Data Analysis (EDA)

- Exploring data structure and quality.
- Analyzing transaction amount distributions.
- Examining the fraud rate.
- Comparing legitimate and fraudulent transactions.
- Investigating fraud across purchase categories, merchants, and customers.
- Exploring temporal patterns.

The analysis showed that fraudulent transactions have substantially higher average amounts than legitimate ones. Fraud rates also vary across purchase categories and merchants.

### 2. Rule-based fraud detection

Three rules were developed to identify suspicious transactions:

1. **Large transaction amount** — the transaction amount exceeds the 95th percentile of legitimate transactions.
2. **Unusual amount for a customer** — the transaction amount is significantly higher than the customer's average transaction amount.
3. **Frequent transactions** — a customer makes two or more transactions within the same simulation time step.

The rules were evaluated using Precision and Recall.

| Rule | Precision | Recall |
|---|---:|---:|
| Large transaction amount | 0.178 | 0.881 |
| Unusual customer amount | 0.121 | 0.563 |
| Frequent transactions | 0.095 | 0.575 |

A `risk_score` was created by counting the number of triggered rules for each transaction. The score ranges from 0 to 3.

Evaluation results for different risk thresholds:

| Risk threshold | Precision | Recall |
|---|---:|---:|
| `risk_score >= 1` | 0.096 | 0.953 |
| `risk_score >= 2` | 0.167 | 0.723 |
| `risk_score >= 3` | 0.242 | 0.344 |

A higher threshold reduces the number of transactions flagged for review, but also increases the number of fraudulent transactions that may be missed.

The rule development stage also includes suspicious transaction analysis:

- Investigating individual transactions with the highest risk scores.
- Analyzing purchase categories and merchants with elevated fraud rates.
- Identifying customers with multiple suspicious transactions.
- Reviewing individual cases to understand why the rules were triggered.

The analysis showed that a suspicious transaction does not necessarily have to involve an extremely large amount. A significant deviation from a customer's usual behavior or an unusually high transaction frequency can also be important risk indicators.

## Key findings

- Transaction amount is an important indicator of potential fraud.
- Comparing transactions with customer history helps identify unusual spending patterns.
- Transaction frequency can provide an additional risk signal.
- Combining multiple rules allows the sensitivity of the detection system to be adjusted.
- Purchase categories and merchants provide additional information for risk assessment.

A triggered rule does not prove that a transaction is fraudulent. Flagged transactions should be treated as candidates for further investigation.

## Limitations

- The dataset is synthetic and may not represent real-world banking patterns.
- The developed rules and thresholds cannot be directly applied to real banking data.
- Customer average amounts were calculated using the full dataset. A production system should use only historical information available at the time of each transaction to avoid data leakage.
- The rules require further validation and tuning on new data.

## Technologies

- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- Jupyter Notebook

## How to run

1. Clone the repository:

```bash
git clone https://github.com/alphakentavra/bank-fraud-analysis.git
cd bank-fraud-analysis
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Download the [BankSim dataset](https://www.kaggle.com/datasets/ealaxi/banksim1) and place `bs140513_032310.csv` in the `data/raw/` directory.

4. Start JupyterLab:

```bash
jupyter lab
```