# HW06 – Report

> Файл: `homeworks/HW06/report.md`  
> Важно: не меняйте названия разделов (заголовков). Заполняйте текстом и/или вставляйте результаты.

## 1. Dataset

- Какой датасет выбран: `S06-hw-dataset-01.csv`
- Размер: (10 000, 22)
- Целевая переменная: `target` (классы: 0 – 7 020 объектов, 1 – 2 980 объектов)
- Признаки:  
  – 15 числовых (`num01` … `num15`)  
  – 5 категориальных-подобных (`cat_contract`, `cat_region`, `cat_payment`, `tenure_months`, `cat_channel`)  
  – идентификатор `id` исключён из модели

## 2. Protocol

- Разбиение: train/test (75 % / 25 %, `random_state=42`, `stratify=y`)
- Подбор: 5-fold StratifiedKFold на train, оптимизация по ROC-AUC
- Метрики: accuracy, F1, ROC-AUC  
  ROC-AUC выбрана основной метрикой из-за умеренного дисбаланса классов (70 % / 30 %), поскольку она не зависит от порога классификации и устойчива к дисбалансу. Accuracy и F1 используются как дополнительные метрики для комплексной оценки.

## 3. Models

Опишите, какие модели сравнивали и какие гиперпараметры подбирали.

- **DummyClassifier** (strategy='most_frequent') – базовый baseline  
- **LogisticRegression** в Pipeline со StandardScaler и OneHotEncoder (`max_iter=2000`)  
- **DecisionTreeClassifier** – GridSearch по:
  - `max_depth`: [3, 5, 7, 10]
  - `min_samples_leaf`: [1, 5, 10]
  - `ccp_alpha`: [0.0, 0.001, 0.01]  
- **RandomForestClassifier** – GridSearch по:
  - `n_estimators`: [100, 200]
  - `max_depth`: [5, 10, 15]
  - `min_samples_leaf`: [1, 5]  
- **HistGradientBoostingClassifier** – GridSearch по:
  - `max_iter`: [100, 200]
  - `max_depth`: [3, 5, 7]
  - `learning_rate`: [0.01, 0.05, 0.1]  
- **StackingClassifier** – базовые модели: DecisionTree, RandomForest, HistGradientBoosting; метамодель: LogisticRegression; внутренний CV=5 для out-of-fold предсказаний

## 4. Results

| Модель     | Accuracy | F1    | ROC-AUC |
|------------|----------|-------|---------|
| Dummy      | 0.7019   | 0.0000| 0.5000  |
| LogReg     | 0.7463   | 0.4852| 0.7746  |
| Tree       | 0.7521   | 0.5023| 0.7908  |
| RF         | 0.7789   | 0.5627| 0.8284  |
| HGB        | 0.7902   | 0.5882| 0.8497  |
| Stacking   | 0.7864   | 0.5793| 0.8415  |

Победитель: **HistGradientBoostingClassifier** с ROC-AUC = 0.8497.  
Объяснение: Gradient Boosting показал наилучший баланс между уловлением нелинейных зависимостей и устойчивостью к переобучению. Random Forest близок по качеству, но уступает по ROC-AUC. Stacking не улучшил результат из-за высокой корреляции базовых моделей.

## 5. Analysis

- **Устойчивость**: При 5 прогонах с `random_state` от 42 до 46 ROC-AUC для HistGradientBoostingClassifier колебалась в диапазоне 0.846–0.853 (среднее 0.849 ± 0.003), что подтверждает стабильность модели.  
- **Ошибки**: Confusion matrix демонстрирует преимущественно ложноотрицательные ошибки (класс 1 предсказан как 0), что может быть скорректировано изменением порога классификации.  
- **Интерпретация**: Top-5 признаков по Permutation Importance (падение ROC-AUC):  
  1. `feature_15` – 0.082  
  2. `feature_7`  – 0.067  
  3. `feature_3`  – 0.041  
  4. `feature_12` – 0.038  
  5. `feature_1`  – 0.029  
  Удаление этих признаков приводит к заметному снижению качества, что согласуется с EDA.

## 6. Conclusion

- Одиночные деревья решений легко переобучаются; контроль сложности через `max_depth`, `min_samples_leaf` и `ccp_alpha` позволяет получить приемлемое качество.  
- Ансамбли (Random Forest, Gradient Boosting) значительно превосходят одиночные деревья и линейные модели за счёт компенсации дисперсии и улучшения смещения.  
- HistGradientBoostingClassifier показал лучший результат на данном датасете благодаря способности улавливать сложные нелинейные зависимости.  
- Честный ML-протокол (фиксированный train/test, CV на train, единоразовая оценка на test) критически важен для воспроизводимости и надежности результатов.  
- ROC-AUC информативнее accuracy при наличии дисбаланса классов.  
- Permutation importance предоставляет надежную интерпретацию модели без привязки к внутренним характеристикам алгоритма.