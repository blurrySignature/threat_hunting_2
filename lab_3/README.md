# Лабораторная работа № 3
Нестеров Павел

# Основы обработки данных с помощью R

## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R
3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

## Исходные данные

1.  ОС Windows 11
2.  RStudio Desktop
3.  Интерпретатор R 4.2.2
4.  dplyr 1.1.3
5.  nycflights13 1.0.2

## Ход работы

### Шаг 1

Для начала установим пакет `dplyr`

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

Установим пакет `nycflights13` с наборами данных

``` r
library(nycflights13)
```

### Шаг 2

Выполним задания на основе встроенных в `nycflights13` наборов данных

##### Задание 1. Сколько встроенных в пакет nycflights13 датафреймов?

-   nycflights13::airlines
-   nycflights13::airports
-   nycflights13::flight
-   nycflights13::planes
-   nycflights13::weather

##### Задание 2. Сколько строк в каждом датафрейме?

``` r
airlines %>% nrow()
```

    [1] 16

``` r
airports %>% nrow()
```

    [1] 1458

``` r
flights %>% nrow()
```

    [1] 336776

``` r
planes %>% nrow()
```

    [1] 3322

``` r
weather %>% nrow()
```

    [1] 26115

##### Задание 3. Сколько столбцов в каждом датафрейме?

``` r
airlines %>% ncol()
```

    [1] 2

``` r
airports %>% ncol()
```

    [1] 8

``` r
flights %>% ncol()
```

    [1] 19

``` r
planes %>% ncol()
```

    [1] 9

``` r
weather %>% ncol()
```

    [1] 15

##### Задание 4. Как просмотреть примерный вид датафрейма?

``` r
flights %>% head()
```

    # A tibble: 6 × 19
       year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
      <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    1  2013     1     1      517            515         2      830            819
    2  2013     1     1      533            529         4      850            830
    3  2013     1     1      542            540         2      923            850
    4  2013     1     1      544            545        -1     1004           1022
    5  2013     1     1      554            600        -6      812            837
    6  2013     1     1      554            558        -4      740            728
    # ℹ 11 more variables: arr_delay <dbl>, carrier <chr>, flight <int>,
    #   tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>, distance <dbl>,
    #   hour <dbl>, minute <dbl>, time_hour <dttm>

или

``` r
flights %>% glimpse()
```

    Rows: 336,776
    Columns: 19
    $ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2…
    $ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
    $ dep_time       <int> 517, 533, 542, 544, 554, 554, 555, 557, 557, 558, 558, …
    $ sched_dep_time <int> 515, 529, 540, 545, 600, 558, 600, 600, 600, 600, 600, …
    $ dep_delay      <dbl> 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2, -2, -1…
    $ arr_time       <int> 830, 850, 923, 1004, 812, 740, 913, 709, 838, 753, 849,…
    $ sched_arr_time <int> 819, 830, 850, 1022, 837, 728, 854, 723, 846, 745, 851,…
    $ arr_delay      <dbl> 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -3, 7, -1…
    $ carrier        <chr> "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV", "B6", "…
    $ flight         <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 4…
    $ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N668DN", "N394…
    $ origin         <chr> "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR", "LGA",…
    $ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL", "IAD",…
    $ air_time       <dbl> 227, 227, 160, 183, 116, 150, 158, 53, 140, 138, 149, 1…
    $ distance       <dbl> 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 944, 733, …
    $ hour           <dbl> 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5, 6, 6, 6…
    $ minute         <dbl> 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, 0, 59, 0…
    $ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013-01-01 0…

##### Задание 5. Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах данных)?

``` r
airlines %>% nrow()
```

    [1] 16

##### Задание 6. Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

``` r
flights %>% 
  filter(origin=='JFK', month==5) %>%
  nrow()
```

    [1] 9397

##### Задание 7. Какой самый северный аэропорт?

``` r
airports %>%
  filter(lat == max(lat))
```

    # A tibble: 1 × 8
      faa   name                      lat   lon   alt    tz dst   tzone
      <chr> <chr>                   <dbl> <dbl> <dbl> <dbl> <chr> <chr>
    1 EEN   Dillant Hopkins Airport  72.3  42.9   149    -5 A     <NA> 

##### Задание 8. Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

``` r
airports %>% 
  filter(alt == max(alt))
```

    # A tibble: 1 × 8
      faa   name        lat   lon   alt    tz dst   tzone         
      <chr> <chr>     <dbl> <dbl> <dbl> <dbl> <chr> <chr>         
    1 TEX   Telluride  38.0 -108.  9078    -7 A     America/Denver

##### Задание 9. Какие бортовые номера у самых старых самолетов?

``` r
planes %>%
  filter(year == min(year, na.rm = TRUE)) %>%
  select(tailnum)
```

    # A tibble: 1 × 1
      tailnum
      <chr>  
    1 N381AA 

##### Задание 10. Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия).

``` r
weather %>% 
  filter(origin == 'JFK' & month == 9) %>%
  summarise(mean_temp = mean(temp, na.rm = TRUE))
```

    # A tibble: 1 × 1
      mean_temp
          <dbl>
    1      66.9

##### Задание 11. Самолеты какой авиакомпании совершили больше всего вылетов в июне?

``` r
flights %>%
  filter(month == 6) %>%
  group_by(carrier) %>%
  summarize(flights_count = n()) %>%
  arrange(desc(flights_count)) %>%
  head(1)
```

    # A tibble: 1 × 2
      carrier flights_count
      <chr>           <int>
    1 UA               4975

##### Задание 12. Самолеты какой авиакомпании задерживались чаще других в 2013 году?

``` r
flights %>%
  filter(year == 2013 & dep_delay > 0) %>%
  group_by(carrier) %>%
  summarise(delays_count = n()) %>%
  arrange(desc(delays_count)) %>%
  head(1)
```

    # A tibble: 1 × 2
      carrier delays_count
      <chr>          <int>
    1 UA             27261

## Оценка результатов

В ходе выполнения лабораторной работы были изучены функции пакета
`dplyr` и выполнены задания с использованием датафреймов из пакета
`nycflights13`

## Вывод

Были получены базовые навыки обработки данных с помощью языка R
