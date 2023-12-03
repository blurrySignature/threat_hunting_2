# Лабораторная работа № 6
Нестеров Павел

# Исследование вредоносной активности в домене Windows

## Цель работы

1.  Закрепить навыки исследования данных журнала Windows Active
    Directory
2.  Изучить структуру журнала системы Windows Active Directory
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R

## Исходные данные

1.  ОС Windows 11
2.  RStudio Desktop
3.  Интерпретатор R 4.2.2
4.  dplyr 1.1.3

**Общая ситуация**

На протяжении долгого времени системные администраторы Доброй
Организации замечали подозрительную активность в домене Windows, но
конкретных доказательств компрометации сети найти не удавалось. К Вам в
руки попал файл с выгрузкой данных из системы SIEM. Помогите выявить
факты компрометации.

## Задание

Используя программный пакет `dplyr` языка программирования R, провести
анализ журналов и ответить на вопросы

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

``` r
library(jsonlite)
library(tidyr)
```

    Warning: пакет 'tidyr' был собран под R версии 4.3.2

``` r
library(xml2)
```

    Warning: пакет 'xml2' был собран под R версии 4.3.2

``` r
library(rvest)
```

    Warning: пакет 'rvest' был собран под R версии 4.3.2

##### 1. Импортируйте данные

``` r
data = stream_in(file('caldera_attack_evals_round1_day1_2019-10-20201108.json'))
```

##### 2. Приведите датасеты в вид “аккуратных данных”, преобразовать типы столбцов в соответствии с типом данных

``` r
data <- data %>%
  mutate(`@timestamp` = as.POSIXct(`@timestamp`, format = "%Y-%m-%dT%H:%M:%OSZ", tz = "UTC")) %>%
  rename(timestamp = `@timestamp`, metadata = `@metadata`)
```

##### 3. Просмотрите общую структуру данных с помощью функции glimpse()

``` r
data %>% glimpse
```

    Rows: 101,904
    Columns: 9
    $ timestamp <dttm> 2019-10-20 20:11:06, 2019-10-20 20:11:07, 2019-10-20 20:11:…
    $ metadata  <df[,4]> <data.frame[26 x 4]>
    $ event     <df[,4]> <data.frame[26 x 4]>
    $ log       <df[,1]> <data.frame[26 x 1]>
    $ message   <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity ID:\…
    $ winlog    <df[,16]> <data.frame[26 x 16]>
    $ ecs       <df[,1]> <data.frame[26 x 1]>
    $ host      <df[,1]> <data.frame[26 x 1]>
    $ agent     <df[,5]> <data.frame[26 x 5]>

### Шаг 2. Анализ данных

##### Задание 1. Раскройте датафрейм избавившись от вложенных датафреймов.

> Для обнаружения таких можно использовать функцию dplyr::glimpse() , а
> для раскрытия вложенности – tidyr::unnest() . Обратите внимание, что
> при раскрытии теряются внешние названия колонок – это можно
> предотвратить если использовать параметр tidyr::unnest(…, names_sep =)

``` r
data_unnested <- data %>%
  unnest(c(metadata, event, log, winlog, ecs, host, agent), names_sep = ".")

data_unnested %>% glimpse
```

    Rows: 101,904
    Columns: 34
    $ timestamp            <dttm> 2019-10-20 20:11:06, 2019-10-20 20:11:07, 2019-1…
    $ metadata.beat        <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…
    $ metadata.type        <chr> "_doc", "_doc", "_doc", "_doc", "_doc", "_doc", "…
    $ metadata.version     <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4…
    $ metadata.topic       <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…
    $ event.created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event.kind           <chr> "event", "event", "event", "event", "event", "eve…
    $ event.code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event.action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log.level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog.event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog.event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
    $ winlog.provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog.api           <chr> "wineventlog", "wineventlog", "wineventlog", "win…
    $ winlog.record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog.computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog.process       <df[,2]> <data.frame[26 x 2]>
    $ winlog.keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog.provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog.channel       <chr> "security", "Security", "Microsoft-Windows-Sysmo…
    $ winlog.task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog.opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog.version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog.user          <df[,4]> <data.frame[26 x 4]>
    $ winlog.activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.user_data     <df[,30]> <data.frame[26 x 30]>
    $ ecs.version          <chr> "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1…
    $ host.name            <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent.ephemeral_id   <chr> "b372be1f-ba0a-4d7e-b4df-79eac86e1fde", "b372be1f…
    $ agent.hostname       <chr> "WECServer", "WECServer", "WECServer", "WECSe…
    $ agent.id             <chr> "d347d9a4-bff4-476c-b5a4-d51119f78250", "d347d9a4…
    $ agent.version        <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4…
    $ agent.type           <chr> "winlogbeat", "winlogbeat", "winlogbeat", "winlog…

##### Задание 2. Минимизируйте количество колонок в датафрейме – уберите колоки с единственным значением параметра

``` r
data_clear <- data_unnested %>%
  select(-metadata.beat, -metadata.type, -metadata.version, -metadata.topic, 
         -event.kind, -winlog.api, -agent.ephemeral_id, -agent.hostname, 
         -agent.id, -agent.version, -agent.type)

data_clear %>% glimpse
```

    Rows: 101,904
    Columns: 23
    $ timestamp            <dttm> 2019-10-20 20:11:06, 2019-10-20 20:11:07, 2019-1…
    $ event.created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event.code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event.action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log.level            <chr> "information", "information", "information", "inf…
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
    $ winlog.event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog.event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
    $ winlog.provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
    $ winlog.record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog.computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog.process       <df[,2]> <data.frame[26 x 2]>
    $ winlog.keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog.provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog.channel       <chr> "security", "Security", "Microsoft-Windows-Sysmo…
    $ winlog.task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ winlog.opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog.version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog.user          <df[,4]> <data.frame[26 x 4]>
    $ winlog.activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.user_data     <df[,30]> <data.frame[26 x 30]>
    $ ecs.version          <chr> "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1…
    $ host.name            <chr> "WECServer", "WECServer", "WECServer", "WECSer…

##### Задание 3. Какое количество хостов представлено в данном датасете?

``` r
data_clear %>%
  select(host.name) %>%
  unique
```

    # A tibble: 1 × 1
      host.name
      <chr>    
    1 WECServer

##### Задание 4. Подготовьте датафрейм с расшифровкой Windows Event_ID, приведите типы данных к типу их значений

``` r
webpage_url <- "https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor"
webpage <- xml2::read_html(webpage_url)
event_df <- rvest::html_table(webpage)[[1]]

event_df %>% glimpse
```

    Rows: 381
    Columns: 4
    $ `Current Windows Event ID` <chr> "4618", "4649", "4719", "4765", "4766", "47…
    $ `Legacy Windows Event ID`  <chr> "N/A", "N/A", "612", "N/A", "N/A", "N/A", "…
    $ `Potential Criticality`    <chr> "High", "High", "High", "High", "High", "Hi…
    $ `Event Summary`            <chr> "A monitored security event pattern has occ…

Подготовим данные:

``` r
event_df <- event_df %>%
  mutate_at(vars(`Current Windows Event ID`, `Legacy Windows Event ID`), as.integer) %>%
  rename(c(Current_Windows_Event_ID = `Current Windows Event ID`, 
           Legacy_Windows_Event_ID = `Legacy Windows Event ID`, 
           Potential_Criticality = `Potential Criticality`, 
           Event_Summary = `Event Summary`))

event_df %>% glimpse
```

    Rows: 381
    Columns: 4
    $ Current_Windows_Event_ID <int> 4618, 4649, 4719, 4765, 4766, 4794, 4897, 496…
    $ Legacy_Windows_Event_ID  <int> NA, NA, 612, NA, NA, NA, 801, NA, NA, 550, 51…
    $ Potential_Criticality    <chr> "High", "High", "High", "High", "High", "High…
    $ Event_Summary            <chr> "A monitored security event pattern has occur…

##### Задание 5. Есть ли в логе события с высоким и средним уровнем значимости? Сколько их?

``` r
event_df %>% 
  group_by(Potential_Criticality) %>%
  summarize(count = n()) %>%
  arrange(desc(count))
```

    # A tibble: 4 × 2
      Potential_Criticality count
      <chr>                 <int>
    1 Low                     291
    2 Medium                   79
    3 High                      9
    4 Medium to High            2

Количество событий со средним уровнем значимости: 79
Количество событий с высоким уровнем значимости: 9

## Оценка результатов

В ходе практической работы были получены навыки исследования данных
журнала Windows Active Directory

## Вывод

Были закреплены навыки использования языка программирования R для
обработки данных
