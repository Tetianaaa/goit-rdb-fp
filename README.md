# goit-rdb-fp
# Фінальний проєкт: інфекційні захворювання (infectious_cases)

## Датасет

`infectious_cases.csv`: статистика інфекційних захворювань за країнами, регіонами та роками (12 колонок: `Entity`, `Code`, `Year` і дев'ять показників хвороб: `Number_yaws`, `polio_cases`, `cases_guinea_worm`, `Number_rabies`, `Number_malaria`, `Number_hiv`, `Number_tuberculosis`, `Number_smallpox`, `Number_cholera_cases`).

Особливості даних:

- пропущені значення показників (наприклад, `Number_rabies` порожній приблизно у 36% рядків) імпортуються як `NULL`;
- у `Code` немає значення для регіонів і груп країн (`Africa`, `G20`, групи за доходом тощо): такі `Code` зберігаються як `NULL`;
- у топі за середнім `Number_rabies` значна частина позицій це агрегати (`World`, регіони), а не окремі країни.

## Як запустити

1. Відкрийте `final_prj_tetiana_kravchuk_rdb.ipynb` у Google Colab.
2. Покладіть `infectious_cases.csv` у `/content` (або в `data/` клонованого репозиторію). Якщо файлу немає, Colab запропонує завантажити його вручну.
3. Виконайте **Runtime → Restart session and run all**.

## Що зроблено після завантаження даних

### Нормалізація до 3НФ

Ключ вихідної таблиці — пара (`Entity`, `Year`), але `Code` залежить лише від `Entity`. Тому `Entity` та `Code` повторювалися б у кожному рядку для кожного року. Таблицю розділено на дві:

| Таблиця | Вміст | Ключі |
|---|---|---|
| `entities` | унікальні комбінації `Entity` + `Code` (245 рядків) | PK `entity_id`, `UNIQUE (entity)` |
| `disease_stats` | показники по роках, замість `Entity` і `Code` — `entity_id` | PK `(entity_id, year)`, FK `entity_id` → `entities` |

Кожен `Entity` має рівно один `Code`, тому `entity` унікальний. Кількість рядків у `disease_stats` дорівнює кількості рядків вихідної таблиці, дані не втрачено й не продубльовано.

<img width="1255" height="565" alt="image" src="https://github.com/user-attachments/assets/8d2a3a5a-54cd-4a08-beed-101de29e9901" />

### Аналіз `Number_rabies`

Для кожної комбінації `Entity` + `Code` (через `entity_id`) обчислено середнє, мінімум, максимум і суму. Результат відсортовано за середнім за спаданням, виведено 10 рядків. Записи без значення `Number_rabies` виключено, щоб `NULL` не потрапив на початок списку при сортуванні за спаданням.

<img width="820" height="365" alt="image" src="https://github.com/user-attachments/assets/fb920cb4-1a48-4013-b3b7-cc568f697c34" />

### Різниця в роках (вбудовані функції)

```sql
MAKE_DATE(year, 1, 1)                                         AS year_start,
CURRENT_DATE                                                  AS today,
DATE_PART('year', AGE(CURRENT_DATE, MAKE_DATE(year, 1, 1)))   AS years_diff
```
<img width="451" height="365" alt="image" src="https://github.com/user-attachments/assets/9df490b5-2df3-4f85-aab7-2d5c237dfad0" />

### Власні функції

| Функція | Параметри | Що повертає |
|---|---|---|
| `years_diff(p_year)` | рік (`INTEGER`) | різницю в повних роках між поточною датою та `р-01-01` |
| `avg_cases_per_period(p_cases_per_year, p_divisor)` | кількість на рік, дільник (`NUMERIC`) | середню кількість за період; дільник `0` дає `NULL` |

<img width="565" height="108" alt="image" src="https://github.com/user-attachments/assets/e4d3edbc-5b16-44b4-8a11-e4eaec6da27c" />

Результат `years_diff` порівняно з колонкою із завдання 4: значення збігаються.

<img width="450" height="350" alt="image" src="https://github.com/user-attachments/assets/ce84ee2b-78db-414c-afe0-b5f2c271a30b" />

Перевірка `avg_cases_per_period` виконана на `Number_rabies` лише для записів із числовим значенням (`IS NOT NULL`) із дільниками `12` (місяць), `4` (квартал) і `2` (півріччя). 

<img width="723" height="350" alt="image" src="https://github.com/user-attachments/assets/5868b4b5-18e7-4683-9813-d1d8f6268ca0" />

## Примітка

`CURRENT_DATE` залежить від дня запуску, тому значення різниці в роках у скриншотах відповідатимуть даті виконання notebook.
