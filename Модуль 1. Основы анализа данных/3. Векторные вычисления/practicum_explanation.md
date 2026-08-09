# Объяснение input-ячеек практикума

## Input 1

```python
import numpy as np
import pandas as pd
import time
```

Подключаем три библиотеки.

`numpy` и `pandas` нужны для работы с массивами и таблицами.
`time` нужен, чтобы измерять, сколько секунд выполняется код — с его помощью мы будем сравнивать скорость циклов и векторных операций.

## Input 2

```python
df = pd.read_csv('data/retail.csv')
display(df)
df.info()
```

Загружаем файл `retail.csv` в переменную `df` — это данные о ста тысячах транзакций розничного магазина.

`display(df)` показывает таблицу.
`df.info()` показывает типы столбцов и число заполненных значений. Пропусков в данных нет, поэтому сразу переходим к вычислениям.

## Input 3

```python
start_time_for_loop = time.time()

total = 0

for i in range(len(df)):
    total = total + df['unit_price'][i] * df['quantity'][i]

time_for_loop = time.time() - start_time_for_loop

print('total =', round(total, 2))
print('time_for_loop =', time_for_loop)
```

Считаем суммарную выручку «классическим» способом — циклом.

`time.time()` до и после цикла нужен, чтобы узнать, сколько секунд ушло на вычисления.

Внутри цикла для каждой строки берём цену за единицу товара, умножаем на количество и прибавляем к общей сумме `total`.

Такой подход работает, но медленно: Python обрабатывает строки таблицы по одной, а таблица большая — сто тысяч строк.

## Input 4

```python
start_time_vector = time.time()

total = (df['unit_price'] * df['quantity']).sum()

time_vector = time.time() - start_time_vector
if time_vector == 0:
    time_vector = 0.0000005

print('total =', round(total, 2))
print('time_vector =', time_vector)
```

Та же самая сумма, но векторным способом.

`df['unit_price'] * df['quantity']` умножает сразу два столбца целиком, без цикла — pandas делает это внутри на низком уровне (в C), поэтому быстро.

`.sum()` складывает все получившиеся значения.

Если время выполнения оказалось настолько маленьким, что `time.time()` округлил его до нуля, подставляем условную маленькую величину — иначе при делении на неё программа упадёт с ошибкой.

Результат `total` совпадает с результатом цикла — значит, вычисления верны.

## Input 5

```python
speed_up = round(time_for_loop / time_vector)

print('speed_up =', speed_up)
```

Делим время цикла на время векторной операции — получаем, во сколько раз векторный способ быстрее.

## Input 6

```python
start_time_for_loop = time.time()

premium = []

for i in range(len(df)):
    is_expensive = df['unit_price'][i] * df['quantity'][i] > 99000
    is_target_category = df['product_category'][i] == 'Electronics' or df['product_category'][i] == 'Home'
    if is_expensive and is_target_category:
        premium.append(df['transaction_id'][i])

time_for_loop = time.time() - start_time_for_loop

print('premium =', premium)
print('time_for_loop =', time_for_loop)
```

Ищем «премиальные» транзакции циклом.

Транзакция считается премиальной, если одновременно выполняются два условия:

- сумма транзакции (`unit_price * quantity`) больше 99000;
- категория товара — `Electronics` или `Home`.

Для каждой строки проверяем оба условия через `and`. Если оба верны — добавляем `transaction_id` в список `premium`.

## Input 7

```python
start_time_vector = time.time()

revenue = df['unit_price'] * df['quantity']
is_expensive_mask = revenue > 99000
is_target_category_mask = (df['product_category'] == 'Electronics') | (df['product_category'] == 'Home')

mask = is_expensive_mask & is_target_category_mask
premium = df['transaction_id'][mask].values

time_vector = time.time() - start_time_vector
if time_vector == 0:
    time_vector = 0.0000005

print('premium =', premium)
print('time_vector =', time_vector)
```

То же самое, но при помощи булевых масок.

`revenue > 99000` сразу для всей таблицы возвращает столбец из `True`/`False` — это и есть первая маска.

`(df['product_category'] == 'Electronics') | (df['product_category'] == 'Home')` — вторая маска: `True` там, где категория подходящая. Знак `|` — это «или» для масок (обычное слово `or` для целых столбцов не работает).

`is_expensive_mask & is_target_category_mask` объединяет обе маски через «и» (`&`) — получаем `True` только там, где выполняются оба условия одновременно.

`df['transaction_id'][mask]` отбирает только те ID транзакций, где маска равна `True`.

Результат совпадает со списком из цикла, но получен намного быстрее.

## Input 8

```python
speed_up = round(time_for_loop / time_vector)

print('speed_up =', speed_up)
```

Сравниваем время двух подходов — получаем ускорение.

## Input 9

```python
start_time_for_loop = time.time()

sum_member = 0
count_member = 0
sum_not_member = 0
count_not_member = 0

for i in range(len(df)):
    amount = df['unit_price'][i] * df['quantity'][i]
    if df['is_member'][i] == 1:
        sum_member = sum_member + amount
        count_member = count_member + 1
    else:
        sum_not_member = sum_not_member + amount
        count_not_member = count_not_member + 1

average_member = round(sum_member / count_member, 2)
average_not_member = round(sum_not_member / count_not_member, 2)

time_for_loop = time.time() - start_time_for_loop

print('average_member =', average_member)
print('average_not_member =', average_not_member)
print('time_for_loop =', time_for_loop)
```

Считаем среднюю сумму транзакции отдельно для участников программы лояльности (`is_member == 1`) и для всех остальных — циклом.

Для каждой строки считаем сумму транзакции и, в зависимости от `is_member`, прибавляем её либо к «сумме участников», либо к «сумме остальных», параллельно считая число строк в каждой группе.

В конце делим сумму на количество — получаем среднее по каждой группе.

## Input 10

```python
start_time_vector = time.time()

amount = df['unit_price'] * df['quantity']
member_mask = df['is_member'] == 1

average_member = round(amount[member_mask].mean(), 2)
average_not_member = round(amount[~member_mask].mean(), 2)

time_vector = time.time() - start_time_vector
if time_vector == 0:
    time_vector = 0.0000005

print('average_member =', average_member)
print('average_not_member =', average_not_member)
print('time_vector =', time_vector)
```

То же самое векторно.

Сначала одним действием считаем сумму каждой транзакции (`amount`) для всей таблицы.

`member_mask` — булева маска участников программы лояльности.

`amount[member_mask].mean()` берёт только суммы участников и сразу считает их среднее.
`~member_mask` — это отрицание маски (значок `~` переворачивает `True` в `False` и наоборот), поэтому `amount[~member_mask]` — это суммы всех, кто не участник.

## Input 11

```python
speed_up = round(time_for_loop / time_vector)

print('speed_up =', speed_up)
```

Сравнение скорости, как и раньше.

## Input 12

```python
start_time_for_loop = time.time()

hours = range(9, 22)
counts_by_hour = {hour: 0 for hour in hours}

for i in range(len(df)):
    if df['is_member'][i] == 1:
        hour = df['hour_of_day'][i]
        counts_by_hour[hour] = counts_by_hour[hour] + 1

hour_max_transactions = max(counts_by_hour, key=counts_by_hour.get)

time_for_loop = time.time() - start_time_for_loop

print('hour_max_transactions =', hour_max_transactions)
print('time_for_loop =', time_for_loop)
```

Ищем час, в который участники программы лояльности совершают больше всего покупок — циклом.

`counts_by_hour` — словарь, где для каждого часа с 9 до 21 хранится счётчик покупок, изначально равный нулю.

В цикле для каждой строки проверяем, что покупатель — участник программы, и если да, увеличиваем счётчик соответствующего часа на 1.

`max(counts_by_hour, key=counts_by_hour.get)` находит час с самым большим значением счётчика.

## Input 13

```python
start_time_vector = time.time()

member_mask = df['is_member'] == 1
hour_max_transactions = df['hour_of_day'][member_mask].value_counts().idxmax()

time_vector = time.time() - start_time_vector
if time_vector == 0:
    time_vector = 0.0000005

print('hour_max_transactions =', hour_max_transactions)
print('time_vector =', time_vector)
```

То же самое векторно.

Сначала маской отбираем только строки участников программы лояльности.

`df['hour_of_day'][member_mask]` — часы покупок только этих покупателей.

`.value_counts()` считает, сколько раз встречается каждый час, и сразу сортирует по убыванию частоты.

`.idxmax()` возвращает час с самым большим количеством покупок — то есть индекс (в данном случае — сам час) с максимальным значением.

## Input 14

```python
speed_up = round(time_for_loop / time_vector)

print('speed_up =', speed_up)
```

Финальное сравнение скорости для этого задания.

## Input 15

```python
df = pd.read_csv('data/server_logs.csv')
display(df)
df.info()
```

Загружаем датасет для домашнего задания — `server_logs.csv`.

Это данные о ста пятидесяти тысячах HTTP-запросов к API веб-сервиса за сутки. Пропусков нет, все столбцы прочитались с ожидаемыми типами.

## Input 16 (домашнее задание, только векторно)

```python
# 1. Нагрузка и эффективность
df['is_slow'] = df['response_time_ms'] > 1000
df['is_large_request'] = df['request_size_kb'] > 500

display(df[['response_time_ms', 'is_slow', 'request_size_kb', 'is_large_request']].head())

# 2. Критические запросы по двум критериям
criterion_a = df['status_code'] >= 400
criterion_b = df['is_slow'] & df['is_large_request']

df['is_critical'] = criterion_a | criterion_b

critical_requests = df['request_id'][df['is_critical']]
print('Число критических запросов:', len(critical_requests))
display(critical_requests.head())

# 3.1 Среднее время отклика: mobile_app и остальные
mobile_mask = df['client_type'] == 'mobile_app'
average_time_mobile = df['response_time_ms'][mobile_mask].mean()
average_time_not_mobile = df['response_time_ms'][~mobile_mask].mean()

print('average_time_mobile     =', round(average_time_mobile, 2))
print('average_time_not_mobile =', round(average_time_not_mobile, 2))

# 3.2 Есть ли ошибки среди browser
browser_error_mask = (df['client_type'] == 'browser') & (df['status_code'] >= 400)
has_browser_errors = browser_error_mask.any()

print('has_browser_errors =', has_browser_errors)
print('Число ошибок browser =', browser_error_mask.sum())
```

Домашнее задание нужно было выполнить только векторно, без циклов. Разбираем по шагам.

**Шаг 1 — новые признаки.**
`df['response_time_ms'] > 1000` сразу для всей таблицы даёт булев столбец `is_slow` — `True`, если запрос обрабатывался дольше секунды.
`df['request_size_kb'] > 500` аналогично даёт `is_large_request` — `True` для больших запросов.

**Шаг 2 — критические запросы.**
Критерий A — ошибка на сервере или у клиента: `status_code >= 400`.
Критерий B — запрос одновременно медленный и большой: `is_slow & is_large_request` (объединяем маски через «и»).
Запрос критический, если выполняется хотя бы один критерий — поэтому маски объединяются через «или» (`|`): `criterion_a | criterion_b`.
По этой маске отбираем `request_id` критических запросов.

**Шаг 3.1 — сравнение мобильных и немобильных клиентов.**
`mobile_mask` отмечает строки, где `client_type == 'mobile_app'`.
`df['response_time_ms'][mobile_mask].mean()` — среднее время отклика только для мобильного приложения.
`df['response_time_ms'][~mobile_mask].mean()` — то же самое, но для всех остальных типов клиентов (маска инвертирована через `~`).

**Шаг 3.2 — есть ли ошибки у браузера.**
`browser_error_mask` — `True` там, где клиент — браузер и при этом код ответа означает ошибку (`>= 400`).
`.any()` проверяет, есть ли среди значений маски хотя бы одно `True` — то есть встречается ли такая ситуация вообще.
`.sum()` дополнительно считает, сколько именно таких запросов было (`True` при сложении считается как 1).

Все действия выполнены без единого цикла — только через выражения над столбцами и булевы маски, как и требовалось в задании.
