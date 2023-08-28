# Matching, Поиск ближайшего соседа  
<div style='text-align: justify;'>В данном проекте было необходимо разработать алгоритм который для всех товаров из списка может
предложить несколько вариантов наиболее похожих товаров из базы данных. В моём распоряжении
находилась база данных с 3 миллионами товаров и 72 признаками для каждого, тренировочный набор данных (100000 строк) и
данные для валидации (100000 строк).  Заказчик предварительно зашифровал все данные, поэтому анализ признаков не представлялся возможным.</div>

## Данные

- *base.csv* - анонимизированный набор товаров. Каждый товар представлен как уникальный id (0-base, 1-base, 2-base) и вектор признаков размерностью 72.
- *target.csv -* обучающий датасет. Каждая строчка - один товар, для которого известен уникальный id (0-query, 1-query, …) , вектор признаков И id товара из *base.csv*, который максимально похож на него (по мнению экспертов).
- *validation.csv* - датасет с товарами (уникальный id и вектор признаков), для которых надо найти наиболее близкие товары из *base.csv*
- *validation_answer.csv* - правильные ответы к предыдущему файлу.  

Задача была разбита на два этапа. В первом была применена библиотека от разработанная Facebook -
"Facebook AI Similarity Search". Необходимо было выбрать n кандидатов для каждого запроса из target и на втором
этапе с помощью модели CatBoostClassifier проранжировать их.
<div style='text-align: justify;'>На первом этапе (Файл Matching.ipynb) было проанализировано несколько вариантов индексации и квантования признаков:
инвертированный индекс (IVF), product quantization (PQ), инвертированный мульти индекс (IMI), их различные комбинации,
а также методы Refine, RFlat и OPQ. Наилучший результат в index_factory показала строка "OPQ32,IMI2x8,PQ32" со значением метрики 
accuracy@5 = 82 для 200 ближайших соседей и 84.5 для 500 и nprobe = 10000. Хочется отметить несколько интересных 
фактов: </div>
1) Применение метода масштабирования признаков StandardScaler в последствии даёт меньший результат, чем MinMaxScaler.
2) Для IVFPQ можно разбить вектор признаков на две части и для каждой обучить свою модель faiss. После объединения предсказаний и удаления 
дубликатов, можно получить метрику на 1% выше, чем для одной модели faiss для одинакового выходного количества соседей. 
Данный метод плохо работает для IVFFlat и HNSWFlat.  
3) Некоторые признаки сильно ухудшают работу faiss, поэтому первый этап проводился без них.

## Итоги

| Количество соседей, n | Accuracy@n после первого этапа | Время выполнения валидации | Accuracy@5 |
|:----------------:|:----------------:|:---------:|:----------------:|
| 500 | 84.073 | 48m 35s | 81.713 |
| 200 | 82.115| 22m 9s | 80.148 |
| 50 | 78.67| 8m 44 | 77.4 |


Время выполнения первого этапа не зависит от количества соседей и равно примерно 250 секунд. Самый большой вклад во время работы запроса - ранжирование CatBoost

