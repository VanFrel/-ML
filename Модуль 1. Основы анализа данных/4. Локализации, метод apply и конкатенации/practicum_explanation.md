# Объяснение input-ячеек практикума

## Input 1

```python
import pandas as pd
```

Подключаем `pandas` — он нужен для всей работы с таблицами в этом практикуме.

## Input 2

```python
df_N = pd.read_csv('data/retail_north.csv')
df_S = pd.read_csv('data/retail_south.csv')
display(df_N.head())
display(df_S.head())
```

Загружаем продажи смартфонов по двум регионам в отдельные таблицы: `df_N` — север, `df_S` — юг.

Столбцы в обоих файлах уже названы с суффиксом (`_N` или `_S`), кроме `Model_ID` — он одинаковый в обеих таблицах, потому что это общий идентификатор модели, по которому мы дальше будем сопоставлять регионы.

## Input 3

```python
models_north = set(df_N['Model_ID'])
models_south = set(df_S['Model_ID'])

common_models = models_north & models_south
north_only = models_north - models_south
south_only = models_south - models_north

print('Общие модели:', common_models)
print('Только на севере:', north_only)
print('Только на юге:', south_only)
```

Переводим столбцы `Model_ID` в `set` — множество уникальных значений. Со множествами удобно сравнивать списки моделей:

- `&` — пересечение: модели, которые есть и там, и там;
- `models_north - models_south` — разность: модели, которые есть только на севере;
- `models_south - models_north` — разность в другую сторону: модели, которые есть только на юге.

Так за три строки получаем ответ на вопрос задания, вместо того чтобы вручную сравнивать списки.

## Input 4

```python
df_N.index = df_N['Model_ID']
df_S.index = df_S['Model_ID']

combine = pd.concat([df_N, df_S], axis=1)
combine = combine.reset_index(drop=True)
combine
```

Чтобы объединить две таблицы по строкам (а не просто «приклеить» одну под другой), сначала делаем `Model_ID` индексом в обеих — тогда `pandas` будет знать, какую строку севера сопоставить с какой строкой юга.

`pd.concat([df_N, df_S], axis=1)` склеивает таблицы «по горизонтали»: для каждого значения индекса (модели) строки из обеих таблиц оказываются в одной строке результата. Если модель встречается только в одной из таблиц, вместо значений из другой появляется `NaN` — их мы заполним позже.

`reset_index(drop=True)` возвращает обычный числовой индекс (0, 1, 2, ...) вместо `Model_ID` — `drop=True` означает «не сохранять старый индекс как новый столбец», ведь `Model_ID` уже есть среди обычных столбцов.

## Input 5

```python
combine = combine.loc[:, ~combine.columns.duplicated()]
combine
```

После конкатенации столбец `Model_ID` оказался в таблице дважды — один раз из `df_N`, второй раз из `df_S` (у них было одинаковое имя, поэтому суффикс не добавился, как у остальных столбцов).

`combine.columns.duplicated()` возвращает булев массив: `True` для каждого повторного вхождения имени столбца, `False` — для первого. `~` разворачивает это в обратную сторону: `True` — оставить, `False` — на дубли. `combine.loc[:, маска]` — это локализация по столбцам (после запятой), где `:` означает «все строки».

## Input 6

```python
combine = combine.fillna(0)

numeric_columns = combine.columns.drop('Model_ID')
combine[numeric_columns] = combine[numeric_columns].astype(int)

def total_revenue(row):
    return row['Units_Sold_N'] * row['Price_N'] + row['Units_Sold_S'] * row['Price_S']

combine['Total'] = combine.apply(total_revenue, axis=1)
combine
```

`fillna(0)` заменяет все `NaN` (модели, которых не было в одном из регионов) на 0 — модель просто не продавалась там, поэтому ноль честно отражает ситуацию.

Проблема: там, где были `NaN`, `pandas` хранил столбец как дробный тип (`float`), потому что `NaN` — это дробное значение по определению. После `fillna` числа выглядят целыми (`245.0`), но тип остался дробным. `numeric_columns` — это все столбцы, кроме `Model_ID` (текстового), а `.astype(int)` приводит их обратно к целому типу.

`combine.apply(total_revenue, axis=1)` — применяет функцию `total_revenue` к каждой строке таблицы (`axis=1` значит «построчно»). Внутри функция получает `row` — одну строку в виде мини-таблицы, и обращается к нужным столбцам по имени. Результат для каждой строки становится новым столбцом `Total`.

## Input 7

```python
def defect_rate(row):
    total_sold = row['Units_Sold_N'] + row['Units_Sold_S']
    total_returns = row['Returns_N'] + row['Returns_S']
    return round(total_returns / total_sold, 2)

combine['Defect'] = combine.apply(defect_rate, axis=1)
combine
```

Похожая идея: для каждой модели складываем продажи и возвраты по обоим регионам, потом делим возвраты на продажи — получаем долю бракованных единиц. `apply(..., axis=1)` снова применяет функцию к каждой строке отдельно, потому что вычисление зависит от нескольких столбцов сразу — просто векторной операцией над столбцом это не сделать одной строкой без явного описания формулы, а через `apply` формула читается как обычная функция.

## Input 8

```python
low_quality = combine.loc[combine['Defect'] > 0.1, ['Model_ID', 'Defect']]
low_quality
```

`combine['Defect'] > 0.1` — булева маска: `True` для моделей с браком выше 10%.

`combine.loc[маска, ['Model_ID', 'Defect']]` — локализация: слева от запятой отбираем нужные строки по маске, справа — только нужные столбцы (не всю таблицу, а лишь ID модели и сам коэффициент брака, остальное отделу контроля качества не нужно).

## Input 9

```python
df_start = pd.read_csv('data/health_start.csv')
df_final = pd.read_csv('data/health_final.csv')
display(df_start.head())
display(df_final.head())
```

Загружаем данные обследования пациентов до терапии (`df_start`) и после (`df_final`). Пациенты в двух файлах идут в разном порядке, и часть пациентов не прошла повторное обследование — их не будет в `df_final`.

## Input 10

```python
df_start_indexed = df_start.set_index('Patient_ID')
df_final_indexed = df_final.set_index('Patient_ID')

# Переименовываем столбцы второй таблицы, чтобы не столкнуться с одинаковыми именами после конкатенации
df_final_indexed = df_final_indexed.add_suffix('_2')

combine = pd.concat([df_start_indexed, df_final_indexed], axis=1)
combine = combine.reset_index()
combine
```

`set_index('Patient_ID')` делает ID пациента индексом в обеих таблицах — точно так же, как в задании 3 c `Model_ID`, чтобы при объединении строки правильно сопоставились по пациенту, а не по порядковому номеру строки.

В отличие от задания про магазины, здесь у столбцов `Hemoglobin`, `Vitamin_D`, `Cortisol` изначально нет суффиксов — они одинаково называются в обеих таблицах. Если склеить как есть, `pandas` не сможет их различить, поэтому `add_suffix('_2')` заранее добавляет `_2` ко всем столбцам второй таблицы (включая `Patient_ID`, но это не страшно — этот дублирующий столбец здесь не используется).

`pd.concat(..., axis=1)` и `reset_index()` работают так же, как в задании 3: склеиваем по горизонтали через общий индекс, потом возвращаем `Patient_ID` обратно в обычный столбец (без `drop=True`, потому что тут он и должен остаться единственным идентификатором, а не задваиваться).

Пациенты, которых нет в `df_final`, получат `NaN` в столбцах `_2` — это и есть признак того, что они не прошли повторное обследование.

## Input 11

```python
def get_status(row):
    # Пациент не пришёл на повторное обследование
    if pd.isna(row['Vitamin_D_2']):
        return 'Неизвестно'

    vitamin_up = row['Vitamin_D_2'] > 1.2 * row['Vitamin_D']
    cortisol_down = row['Cortisol_2'] < row['Cortisol']
    hemoglobin_up = row['Hemoglobin_2'] > 1.05 * row['Hemoglobin']

    vitamin_down = row['Vitamin_D_2'] < 0.85 * row['Vitamin_D']
    cortisol_up = row['Cortisol_2'] > 1.2 * row['Cortisol']
    hemoglobin_down = row['Hemoglobin_2'] < 0.9 * row['Hemoglobin']

    if vitamin_up and cortisol_down and hemoglobin_up:
        return 'Улучшение'
    elif vitamin_down or cortisol_up or hemoglobin_down:
        return 'Требуется врач'
    else:
        return 'Без изменений'

combine['Report'] = combine.apply(get_status, axis=1)
combine
```

Функция `get_status` реализует логику медицинского заключения по одному пациенту (одной строке).

Сначала проверка `pd.isna(row['Vitamin_D_2'])` — если повторного обследования не было (в столбцах `_2` стоит `NaN`), сразу возвращаем `'Неизвестно'`, дальше проверять нечего.

Дальше отдельно считаем шесть условий как булевы переменные — по три на «улучшение» и на «требуется врач» — просто чтобы формулы дальше читались короче и понятнее, а не громоздкими выражениями в одной строке.

`if ... and ... and ...` — статус «Улучшение» ставится, только если верны все три условия сразу. `elif ... or ... or ...` — статус «Требуется врач», если верно хотя бы одно из условий ухудшения (проверяется после «Улучшения», то есть если условия улучшения не выполнены). Если не сработало ни одно из двух условий — `'Без изменений'`.

`combine.apply(get_status, axis=1)` применяет эту функцию к каждому пациенту и создаёт столбец `Report`.

## Input 12

```python
improved_count = len(combine.loc[combine['Report'] == 'Улучшение'])
print('Число пациентов со статусом "Улучшение":', improved_count)
```

`combine['Report'] == 'Улучшение'` — маска: `True` там, где статус «Улучшение». `combine.loc[маска]` отбирает только такие строки, `len(...)` считает, сколько их.

## Input 13

```python
combine_vertical = pd.concat([df_start, df_final], axis=0)
combine_vertical = combine_vertical.drop(columns=['Patient_ID'])

def average_change(column):
    before = column[:len(df_start)].mean()
    after = column[len(df_start):].mean()
    return round(after - before, 2)

summary = combine_vertical.apply(average_change)
summary
```

Здесь используются исходные `df_start` и `df_final` (без переиндексации) — конкатенация `axis=0` просто ставит одну таблицу под другой, без сопоставления по пациенту, потому что для этой задачи нужна лишь общая статистика по клинике, а не связь между конкретными «до» и «после».

`pd.concat([df_start, df_final], axis=0)` — склейка сверху вниз: сначала все строки `df_start`, потом все строки `df_final`. `axis=0` можно не указывать — это значение по умолчанию. `drop(columns=['Patient_ID'])` убирает столбец ID, потому что усреднять его бессмысленно.

`combine_vertical.apply(average_change)` — без `axis=1`, поэтому `apply` по умолчанию применяется не к строкам, а к столбцам: функция `average_change` вызывается один раз для каждого столбца (`Hemoglobin`, `Vitamin_D`, `Cortisol`) и получает его целиком.

Внутри функции `column[:len(df_start)]` — первые `len(df_start)` значений столбца, то есть та часть, что пришла из `df_start` («до»); `column[len(df_start):]` — оставшаяся часть, из `df_final` («после»). Считаем среднее по каждой части и разницу между ними — получаем, насколько в среднем изменился каждый показатель по всей клинике.
