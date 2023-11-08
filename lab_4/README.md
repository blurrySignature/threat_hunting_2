# Лабораторная работа № 4
Нестеров Павел

# Исследование метаданных DNS трафика

## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  ОС Windows 11
2.  RStudio Desktop
3.  Интерпретатор R 4.2.2
4.  dplyr 1.1.3
5.  dns.log
6.  header.csv

**Общая ситуация**

Вы исследуете подозрительную сетевую активность во внутренней сети
Доброй Организации. Вам в руки попали метаданные о DNS трафике в
исследуемой сети. Исследуйте файлы, восстановите данные, подготовьте их
к анализу и дайте обоснованные ответы на поставленные вопросы
исследования

## Задание

Используя программный пакет `dplyr`, освоить анализ DNS логов с помощью
языка программирования R

## Ход работы

### Шаг 1. Подготовка данных

Для начала установим пакет `dplyr`

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

1.  Импортируйте данные DNS

    ``` r
    data = read.csv("dns.log", header = FALSE, sep = "\t", encoding = "UTF-8")
    data %>% glimpse()
    ```

        Rows: 427,935
        Columns: 23
        $ V1  <dbl> 1331901006, 1331901015, 1331901016, 1331901017, 1331901006, 133190…
        $ V2  <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7BsbGH", "C…
        $ V3  <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "192.168.20…
        $ V4  <int> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137,…
        $ V5  <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "192.168.2…
        $ V6  <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 5…
        $ V7  <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "ud…
        $ V8  <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62187, 621…
        $ V9  <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x0…
        $ V10 <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "…
        $ V11 <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNE…
        $ V12 <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32", "33", …
        $ V13 <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "SRV"…
        $ V14 <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "…
        $ V15 <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", …
        $ V16 <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FAL…
        $ V17 <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FAL…
        $ V18 <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, FALSE…
        $ V19 <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FAL…
        $ V20 <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, …
        $ V21 <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "…
        $ V22 <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "…
        $ V23 <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FAL…

2.  Добавьте пропущенные данные о структуре данных (назначении столбцов)

    В файле header.csv не хватает некоторых столбцов и данных о них,
    добавим их вручную.

    ``` r
    header = read.csv("header.csv", encoding = "UTF-8", skip = 1, header = FALSE, sep = ',')$V1
    colnames(data) = header
    ```

3.  Преобразуйте данные в столбцах в нужный формат

4.  Просмотрите общую структуру данных с помощью функции glimpse()

    ``` r
    data %>% glimpse()
    ```

        Rows: 427,935
        Columns: 23
        $ ts          <dbl> 1331901006, 1331901015, 1331901016, 1331901017, 1331901006…
        $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
        $ id.orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
        $ id.orig_p   <int> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
        $ id.resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
        $ id.resp_p   <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
        $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
        $ trans_id    <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
        $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
        $ qclass      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
        $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
        $ qtype       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32"…
        $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
        $ rcode       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
        $ rcode_name  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
        $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
        $ TC          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
        $ RD          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
        $ RA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
        $ Z           <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
        $ answers     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
        $ TTLs        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
        $ rejected    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

### Шаг 2. Анализ данных

##### Задание 1. Сколько участников информационного обмена в сети Доброй Организации?

``` r
unique_ips <- union(unique(data$id.orig_h), unique(data$id.resp_h))
unique_ips %>% length()
```

    [1] 1359

##### Задание 2. Какое соотношение участников обмена внутри сети и участников обращений к внешним ресурсам?

Диапазоны частных ip-адресов:

1.  10.0.0.0 - 10.255.255.255
2.  100.64.0.0 - 100.127.255.255
3.  172.16.0.0 - 172.31.255.255
4.  192.168.0.0 - 192.168.255.255

``` r
internal_ip_pattern <- c("192.168.", "10.", "100.([6-9]|1[0-1][0-9]|12[0-7]).", "172.((1[6-9])|(2[0-9])|(3[0-1])).")
internal_ips <- unique_ips[grep(paste(internal_ip_pattern, collapse = "|"), unique_ips)]
count_internal <- sum(unique_ips %in% internal_ips)
count_external <- length(unique_ips) - count_internal

ratio <- count_internal / count_external
ratio
```

    [1] 15.57317

##### Задание 3. Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность

``` r
top_10_activity <- data %>%
  group_by(ip = id.orig_h) %>%
  summarise(activity_count = n()) %>%
  arrange(desc(activity_count)) %>%
  head(10)

top_10_activity
```

    # A tibble: 10 × 2
       ip              activity_count
       <chr>                    <int>
     1 10.10.117.210            75943
     2 192.168.202.93           26522
     3 192.168.202.103          18121
     4 192.168.202.76           16978
     5 192.168.202.97           16176
     6 192.168.202.141          14967
     7 10.10.117.209            14222
     8 192.168.202.110          13372
     9 192.168.203.63           12148
    10 192.168.202.106          10784

##### Задание 4. Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
top_10_domains <- data %>%
  group_by(domain = tolower(query)) %>%
  summarise(request_count = n()) %>%
  arrange(desc(request_count)) %>%
  head(10)

top_10_domains
```

    # A tibble: 10 × 2
       domain                                                          request_count
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "wpad"                                                                  11429
     7 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00…         10401
     8 "isatap"                                                                 9712
     9 "44.206.168.192.in-addr.arpa"                                            7248
    10 "hpe8aa67"                                                               6929

##### Задание 5. Опеределите базовые статистические характеристики (функция summary()) интервала времени между последовательным обращениями к топ-10 доменам.

``` r
top_10_domains_filtered <- data %>% 
  filter(tolower(query) %in% top_10_domains$domain) %>%
  arrange(ts)
time_intervals <- diff(top_10_domains_filtered$ts)

summary(time_intervals)
```

        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.00     0.13     0.85     0.54 49677.59 

##### Задание 6. Часто вредоносное программное обеспечение использует DNS канал в качестве канала управления, периодически отправляя запросы на подконтрольный злоумышленникам DNS сервер. По периодическим запросам на один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP адреса в исследуемом датасете?

``` r
ip_domain_counts <- data %>%
  group_by(ip = tolower(id.orig_h), domain = tolower(query)) %>%
  summarise(request_count = n()) %>%
  filter(request_count > 1)
```

    `summarise()` has grouped output by 'ip'. You can override using the `.groups`
    argument.

``` r
unique_ips_with_periodic_requests <- unique(ip_domain_counts$ip)

unique_ips_with_periodic_requests %>% length()
```

    [1] 240

``` r
unique_ips_with_periodic_requests %>% head()
```

    [1] "10.10.10.10"     "10.10.117.209"   "10.10.117.210"   "128.244.37.196" 
    [5] "169.254.109.123" "169.254.228.26" 

## Шаг 3. Обогащение данных

##### Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов. Для этого можно использовать сторонние сервисы, например https://v4.ifconfig.co.

``` r
top_10_domains
```

    # A tibble: 10 × 2
       domain                                                          request_count
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "wpad"                                                                  11429
     7 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00…         10401
     8 "isatap"                                                                 9712
     9 "44.206.168.192.in-addr.arpa"                                            7248
    10 "hpe8aa67"                                                               6929

1.  teredo.ipv6.microsoft.com

    -   IP: 20.112.250.133
    -   Country: United States
    -   Timezone: America/Chicago
    -   Organization: Microsoft

2.  tools.google.com

    -   IP: 173.194.222.100
    -   Country: United States
    -   Timezone: America/Chicago
    -   Organization: Google

3.  www.apple.com

    -   IP: 17.253.144.10
    -   Country: United States
    -   Timezone: America/Chicago
    -   Organization: Apple-Engineering

4.  safebrowsing.clients.google.com

    -   IP: 64.233.164.100
    -   Country: United States
    -   Timezone: America/Chicago
    -   Organization: Google

5.  44.206.168.192.in-addr.arpa

    -   IP: 44.206.168.192
    -   Country: United States
    -   City: Ashburn
    -   Timezone: America/New_York
    -   Organization: Amazon

## Оценка результатов

В результате были получены ответы на все поставленные вопросы с помощью
языка R и библиотеки `dplyr`

## Вывод

В ходе выполнения лабораторной работы были подготовлены,
проанализированы и обогащены данные DNS трафика
