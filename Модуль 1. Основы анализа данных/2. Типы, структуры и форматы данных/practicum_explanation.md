# Объяснение input-ячеек практикума

## Input 1

```python
import pandas as pd
```

Подключаем библиотеку `pandas`.

Она нужна для работы с таблицами: читать CSV-файлы, смотреть столбцы, менять типы данных, заполнять пропуски и сохранять результат.

## Input 4

```python
df = pd.read_csv('data/marketing_campaign.csv')
df
```

Загружаем файл `marketing_campaign.csv` в переменную `df`.

`df` — это таблица с данными. Вторая строка просто показывает таблицу на экране.

## Input 7

```python
df.info()
```

Смотрим общую информацию о таблице:

- сколько строк;
- какие есть столбцы;
- какие у них типы данных;
- есть ли пропущенные значения.

Так мы понимаем, что данные «грязные»: некоторые столбцы имеют не тот тип, который нам нужен.

## Input 10

```python
df = pd.read_csv('data/marketing_campaign.csv', dtype = {'user_id':str})#, 'discount':int, 'conversion':bool})
df.head(3)
df.info()
```

Заново читаем файл и сразу говорим: столбец `user_id` нужно читать как строку.

Для `discount` и `conversion` типы пока закомментированы, потому что там есть смешанные значения. Например, в `discount` встречается `15%` и `ten`, поэтому сразу превратить весь столбец в число не получится.

`df.head(3)` показывает первые три строки.  
`df.info()` снова проверяет типы данных.

## Input 12

```python
missing_pages = df['pages_viewed'].isna()
display(df.loc[missing_pages])

df['pages_viewed'] = df['pages_viewed'].fillna(df['pages_viewed'].median()).astype(int)
display(df.head(3))
df.info()
```

Ищем пропуски в столбце `pages_viewed`.

`isna()` отмечает строки, где значения нет.  
`df.loc[missing_pages]` показывает эти строки.

Потом пропуски заменяются медианой.

Медиана — это «серединное» значение. Она часто лучше среднего, если в данных есть выбросы.

После заполнения столбец переводится в целые числа через `astype(int)`.

## Input 14

```python
display(df['conversion'].unique())

conversion_map = {
    'Yes': 'Yes',
    'YES': 'Yes',
    '1': 'Yes',
    'No': 'No',
    '0': 'No',
}

df['conversion'] = df['conversion'].map(conversion_map)
df['conversion'] = [value == 'Yes' for value in df['conversion']]

display(df['conversion'].unique())
df.info()
```

Сначала смотрим, какие разные значения есть в `conversion`.

Там один смысл записан разными способами:

- `Yes`, `YES`, `1` означают «да»;
- `No`, `0` означают «нет».

Через словарь `conversion_map` приводим всё к нормальным значениям `Yes` и `No`.

Потом генератор списков превращает их в логические значения:

- `True`, если было `Yes`;
- `False`, если было не `Yes`.

## Input 16

```python
display(df['discount'].unique())

df['discount'] = df['discount'].fillna(0)
discount_map = {
    0: 0,
    '0': 0,
    '5': 5,
    '10': 10,
    '15': 15,
    '20': 20,
    '15%': 15,
    'ten': 10,
}

df['discount'] = df['discount'].map(discount_map).astype(int)

display(df['discount'].unique())
df.info()
```

Смотрим разные значения в столбце `discount`.

Там скидки записаны не одинаково: например, есть `15`, `15%`, `ten`, а ещё есть пропуск.

Пропуск заменяем на `0`.

Потом через словарь переводим все варианты в обычные числа:

- `15%` становится `15`;
- `ten` становится `10`;
- пустое значение уже стало `0`.

В конце переводим столбец в целый тип.

## Input 18

```python
display(df['gender'].unique())
display(df['age_group'].value_counts())

gender_map = {
    'Male': 'Male',
    'Female': 'Female',
    'M': 'Male',
}

age_group_map = {
    '18-25': '18-25',
    '25-30': '26-35',
    '26-35': '26-35',
    '36-50': '36-50',
    '50+': '50+',
}

df['gender'] = df['gender'].map(gender_map)
df['age_group'] = df['age_group'].map(age_group_map)

display(df[['gender', 'age_group']].head())
df.info()
```

Сначала смотрим уникальные значения пола и частоты возрастных групп.

В `gender` значение `M` заменяем на нормальное `Male`.

В `age_group` значение `25-30` не входит в список разрешённых групп. Поэтому относим его к ближайшей группе `26-35`.

После этого показываем первые строки и снова проверяем таблицу.

## Input 20

```python
df.to_csv('data/marketing_campaign_new.csv', index=False)
df.to_parquet('data/marketing_campaign_new.parquet', index=False)

marketing_csv = pd.read_csv('data/marketing_campaign_new.csv', dtype={'user_id': str})
marketing_csv['conversion'] = marketing_csv['conversion'].map({'True': True, 'False': False, True: True, False: False})
marketing_parquet = pd.read_parquet('data/marketing_campaign_new.parquet')

display(marketing_csv.head(3))
marketing_csv.info()

display(marketing_parquet.head(3))
marketing_parquet.info()
```

Сохраняем очищенную таблицу в двух форматах:

- CSV;
- Parquet.

`index=False` значит: не сохранять отдельный технический индекс строк.

Потом оба файла читаются обратно.

Для CSV дополнительно указываем тип `user_id` и явно восстанавливаем логический столбец `conversion`, потому что CSV хранит данные проще и может забыть часть типов.  
Parquet обычно лучше сохраняет типы данных.

## Input 23

```python
df['is_mobile'] = [os_type != 'Desktop' for os_type in df['os_type']]

display(df[['os_type', 'is_mobile']].head())
df.info()
```

Создаём новый столбец `is_mobile`.

Он показывает, мобильный ли пользователь.

Логика простая:

- если `os_type` равен `Desktop`, значит `False`;
- если это `Android` или `iOS`, значит `True`.

## Input 25

```python
age_activity_map = df.groupby('age_group')['session_duration'].mean().round(2).to_dict()
df['age_activity'] = df['age_group'].map(age_activity_map)

print(age_activity_map)
display(df[['age_group', 'session_duration', 'age_activity']].head())
df.info()
```

Считаем среднюю длительность сессии для каждой возрастной группы.

Например, отдельно для `18-25`, отдельно для `26-35` и так далее.

`groupby('age_group')` группирует строки по возрасту.  
`mean()` считает среднее.  
`round(2)` округляет до двух знаков.  
`to_dict()` превращает результат в словарь.

Потом этот словарь используется, чтобы создать новый столбец `age_activity`.

## Input 30

```python
churn = pd.read_csv('data/churn.csv')
churn
```

Загружаем датасет `churn.csv`.

Это таблица про клиентов телеком-оператора и их отток.

## Input 31

```python
churn.info()
```

Смотрим общую информацию о таблице `churn`.

Здесь проверяем типы столбцов и пропуски.

## Input 32

```python
churn['customer_id'] = churn['customer_id'].astype(str)

churn['lifetime'] = churn['lifetime'].fillna(churn['lifetime'].median()).astype(int)

contract_mode = churn['contract_type'].mode()[0]
churn['contract_type'] = churn['contract_type'].fillna(contract_mode)

display(churn.head())
churn.info()
```

Приводим несколько столбцов в порядок.

`customer_id` переводим в строку, потому что ID — это не число для расчётов, а идентификатор.

В `lifetime` есть пропуски. Их заменяем медианой, а потом переводим столбец в целые числа.

В `contract_type` пропуски заменяем самым частым значением.

`mode()[0]` берёт самое популярное значение в столбце.

## Input 33

```python
display(churn['tech_support'].unique())
display(churn['satisfaction'].unique())
display(churn['churn'].unique())

tech_support_map = {'Yes': True, '1': True, 'No': False, '0': False}
satisfaction_map = {'good': 10, 'bed': 0}
churn_map = {'Yes': True, 'True': True, '1': True, 'No': False, 'False': False, '0': False}

churn['tech_support'] = churn['tech_support'].map(tech_support_map)
churn['satisfaction'] = churn['satisfaction'].replace(satisfaction_map).astype(int)
churn['churn'] = churn['churn'].map(churn_map)

display(churn.head())
churn.info()
```

Сначала смотрим, какие странные значения есть в трёх столбцах.

`tech_support` должен быть логическим:

- `Yes` и `1` превращаем в `True`;
- `No` и `0` превращаем в `False`.

`satisfaction` должен быть числом от 0 до 10.

В нём есть текстовые значения:

- `good` заменяем на `10`;
- `bed` заменяем на `0`.

`churn` тоже должен быть логическим:

- `Yes`, `True`, `1` означают, что клиент ушёл;
- `No`, `False`, `0` означают, что клиент не ушёл.

## Input 34

```python
loyal_share_by_contract = (1 - churn.groupby('contract_type')['churn'].mean()).round(2).to_dict()
churn['churn_by_contract'] = churn['contract_type'].map(loyal_share_by_contract)

print(loyal_share_by_contract)
display(churn[['contract_type', 'churn', 'churn_by_contract']].head())

churn.to_csv('data/churn_new.csv', index=False)
churn.to_parquet('data/churn_new.parquet', index=False)

churn_csv = pd.read_csv('data/churn_new.csv', dtype={'customer_id': str})
bool_map = {'True': True, 'False': False, True: True, False: False}
churn_csv['tech_support'] = churn_csv['tech_support'].map(bool_map)
churn_csv['churn'] = churn_csv['churn'].map(bool_map)
churn_parquet = pd.read_parquet('data/churn_new.parquet')

display(churn_csv.head(3))
churn_csv.info()

display(churn_parquet.head(3))
churn_parquet.info()
```

Создаём новый признак `churn_by_contract`.

Сначала считаем среднюю долю ушедших клиентов для каждого типа контракта.

Поскольку `churn=True` означает «клиент ушёл», среднее по этому столбцу даёт долю ушедших.

Но в задании нужна средняя доля лояльных клиентов, то есть тех, кто не ушёл.

Поэтому используется:

```python
1 - средняя доля ушедших
```

После этого каждому клиенту записывается доля лояльных клиентов для его типа контракта.

В конце очищенная таблица сохраняется в CSV и Parquet, а потом оба файла читаются обратно для проверки.

Для CSV отдельно восстанавливаются логические столбцы `tech_support` и `churn`, потому что CSV не так хорошо сохраняет типы данных, как Parquet.
