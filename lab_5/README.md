# Лабораторная работа № 5
Нестеров Павел

# Исследование информации о состоянии беспроводных сетей

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R

## Исходные данные

1.  ОС Windows 11
2.  RStudio Desktop
3.  Интерпретатор R 4.2.2
4.  dplyr 1.1.3
5.  mir.csv-01.csv

**Общая ситуация**

Вы исследуете состояние радиоэлектронной обстановки с помощью журналов
программных средств анализа беспроводных сетей – `tcpdump` и
`airodump-ng`. Для этого с помощью сниффера (микрокомпьютера Raspberry
Pi и специализированного Wi-Fi адаптера, переведенного в режим
мониторинга) собирались данные. Сниффер беспроводного трафика был
установлен стационарно (не перемещался). Какой анализ можно провести с
помощью собранной информации?

## Задание

Используя программный пакет `dplyr` языка программирования R, провести
анализ журналов и ответить на вопросы

## Ход работы

### Шаг 1. Подготовка данных

Для начала установим пакет `dplyr` и `lubridate` (для упрощённой работы
с датами и временем)

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(lubridate)
```

    Warning: пакет 'lubridate' был собран под R версии 4.3.2


    Присоединяю пакет: 'lubridate'

    Следующие объекты скрыты от 'package:base':

        date, intersect, setdiff, union

Импортируем данные из общего файла файла в 2 разных датасета

Датасет 1 – анонсы беспроводных точек доступа (`data_1`)

``` r
data_1 = read.csv("mir.csv-01.csv", nrows = 167)
data_1 %>% glimpse()
```

    Rows: 167
    Columns: 15
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ First.time.seen <chr> " 2023-07-28 09:13:03", " 2023-07-28 09:13:03", " 2023…
    $ Last.time.seen  <chr> " 2023-07-28 11:50:50", " 2023-07-28 11:55:12", " 2023…
    $ channel         <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ Speed           <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ Privacy         <chr> " WPA2", " WPA2", " WPA2", " WPA2", " WPA2", " OPN", "…
    $ Cipher          <chr> " CCMP", " CCMP", " CCMP", " CCMP", " CCMP", " ", " CC…
    $ Authentication  <chr> " PSK", " PSK", " PSK", " PSK", " PSK", "   ", " PSK",…
    $ Power           <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ X..beacons      <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ X..IV           <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ LAN.IP          <chr> "   0.  0.  0.  0", "   0.  0.  0.  0", "   0.  0.  0.…
    $ ID.length       <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ ESSID           <chr> " C322U13 3965", " Cnet", " KC", " POCO X5 Pro 5G", " …
    $ Key             <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
data_1 <- data_1 %>% 
  mutate_at(vars(BSSID, Privacy, Cipher, Authentication, LAN.IP, ESSID), trimws) %>%
  mutate_at(vars(BSSID, Privacy, Cipher, Authentication, LAN.IP, ESSID), na_if, "") %>% 
  mutate_at(vars(First.time.seen, Last.time.seen), as.POSIXct, format = "%Y-%m-%d %H:%M:%S")

data_1 %>% head
```

                  BSSID     First.time.seen      Last.time.seen channel Speed
    1 BE:F1:71:D5:17:8B 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195
    2 6E:C7:EC:16:DA:1A 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130
    3 9A:75:A8:B9:04:1E 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360
    4 4A:EC:1E:DB:BF:95 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360
    5 D2:6D:52:61:51:5D 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130
    6 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130
      Privacy Cipher Authentication Power X..beacons X..IV          LAN.IP
    1    WPA2   CCMP            PSK   -30        846   504   0.  0.  0.  0
    2    WPA2   CCMP            PSK   -30        750   116   0.  0.  0.  0
    3    WPA2   CCMP            PSK   -68        694    26   0.  0.  0.  0
    4    WPA2   CCMP            PSK   -37        510    21   0.  0.  0.  0
    5    WPA2   CCMP            PSK   -57        647     6   0.  0.  0.  0
    6     OPN   <NA>           <NA>   -63        251  3430 172. 17.203.197
      ID.length          ESSID Key
    1        12   C322U13 3965  NA
    2         4           Cnet  NA
    3         2             KC  NA
    4        14 POCO X5 Pro 5G  NA
    5        25           <NA>  NA
    6        13  MIREA_HOTSPOT  NA

Датасет 2 – запросы на подключение клиентов к известным им точкам
доступа (data_2)

``` r
data_2 = read.csv("mir.csv-01.csv", skip = 170)
data_2 %>% glimpse()
```

    Rows: 12,269
    Columns: 7
    $ Station.MAC     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ First.time.seen <chr> " 2023-07-28 09:13:03", " 2023-07-28 09:13:03", " 2023…
    $ Last.time.seen  <chr> " 2023-07-28 10:59:44", " 2023-07-28 09:13:03", " 2023…
    $ Power           <chr> " -33", " -65", " -39", " -61", " -53", " -43", " -31"…
    $ X..packets      <chr> "      858", "        4", "      432", "      958", " …
    $ BSSID           <chr> " BE:F1:71:D5:17:8B", " (not associated) ", " BE:F1:71…
    $ Probed.ESSIDs   <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

``` r
data_2 <- data_2 %>% 
  mutate_at(vars(Station.MAC, BSSID, Probed.ESSIDs), trimws) %>%
  mutate_at(vars(Station.MAC, BSSID, Probed.ESSIDs), na_if, "")

data_2 <- data_2 %>% 
  mutate_at(vars(First.time.seen, Last.time.seen), 
            as.POSIXct, 
            format = "%Y-%m-%d %H:%M:%S") %>%
  mutate_at(vars(Power, X..packets), as.integer) %>%
  filter(!is.na(BSSID))
```

    Warning: There were 2 warnings in `mutate()`.
    The first warning was:
    ℹ In argument: `Power = .Primitive("as.integer")(Power)`.
    Caused by warning:
    ! в результате преобразования созданы NA
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 1 remaining warning.

``` r
data_2 %>% head
```

            Station.MAC     First.time.seen      Last.time.seen Power X..packets
    1 CA:66:3B:8F:56:DD 2023-07-28 09:13:03 2023-07-28 10:59:44   -33        858
    2 96:35:2D:3D:85:E6 2023-07-28 09:13:03 2023-07-28 09:13:03   -65          4
    3 5C:3A:45:9E:1A:7B 2023-07-28 09:13:03 2023-07-28 11:51:54   -39        432
    4 C0:E4:34:D8:E7:E5 2023-07-28 09:13:03 2023-07-28 11:53:16   -61        958
    5 5E:8E:A6:5E:34:81 2023-07-28 09:13:04 2023-07-28 09:13:04   -53          1
    6 10:51:07:CB:33:E7 2023-07-28 09:13:05 2023-07-28 11:56:06   -43        344
                  BSSID Probed.ESSIDs
    1 BE:F1:71:D5:17:8B  C322U13 3965
    2  (not associated)  IT2 Wireless
    3 BE:F1:71:D6:10:D7  C322U21 0566
    4 BE:F1:71:D5:17:8B  C322U13 3965
    5  (not associated)          <NA>
    6  (not associated)          <NA>

### Шаг 2. Анализ данных

#### Точки доступа

##### Задание 1. Определить небезопасные точки доступа (без шифрования – OPN)

``` r
open_wifi <- data_1 %>% 
  filter(grepl("OPN", Privacy)) %>%
  select(BSSID, ESSID) %>%
  arrange(BSSID) %>%
  distinct

open_wifi
```

                   BSSID         ESSID
    1  00:03:7A:1A:03:56       MT_FREE
    2  00:03:7F:12:34:56       MT_FREE
    3  00:25:00:FF:94:73          <NA>
    4  00:26:99:F2:7A:E0          <NA>
    5  00:26:99:F2:7A:EF          <NA>
    6  00:3E:1A:5D:14:45       MT_FREE
    7  00:53:7A:99:98:56       MT_FREE
    8  00:AB:0A:00:10:10          <NA>
    9  02:67:F1:B0:6C:98       MT_FREE
    10 02:BC:15:7E:D5:DC       MT_FREE
    11 02:CF:8B:87:B4:F9       MT_FREE
    12 E0:D9:E3:48:FF:D2          <NA>
    13 E0:D9:E3:49:00:B1          <NA>
    14 E8:28:C1:DC:0B:B2          <NA>
    15 E8:28:C1:DC:33:12          <NA>
    16 E8:28:C1:DC:B2:40 MIREA_HOTSPOT
    17 E8:28:C1:DC:B2:41  MIREA_GUESTS
    18 E8:28:C1:DC:B2:42          <NA>
    19 E8:28:C1:DC:B2:50  MIREA_GUESTS
    20 E8:28:C1:DC:B2:51          <NA>
    21 E8:28:C1:DC:B2:52 MIREA_HOTSPOT
    22 E8:28:C1:DC:BD:50  MIREA_GUESTS
    23 E8:28:C1:DC:BD:52 MIREA_HOTSPOT
    24 E8:28:C1:DC:C6:B0  MIREA_GUESTS
    25 E8:28:C1:DC:C6:B1          <NA>
    26 E8:28:C1:DC:C6:B2          <NA>
    27 E8:28:C1:DC:C8:30  MIREA_GUESTS
    28 E8:28:C1:DC:C8:31          <NA>
    29 E8:28:C1:DC:C8:32 MIREA_HOTSPOT
    30 E8:28:C1:DC:FF:F2          <NA>
    31 E8:28:C1:DD:04:40 MIREA_HOTSPOT
    32 E8:28:C1:DD:04:41  MIREA_GUESTS
    33 E8:28:C1:DD:04:42          <NA>
    34 E8:28:C1:DD:04:50  MIREA_GUESTS
    35 E8:28:C1:DD:04:51          <NA>
    36 E8:28:C1:DD:04:52 MIREA_HOTSPOT
    37 E8:28:C1:DE:47:D0  MIREA_GUESTS
    38 E8:28:C1:DE:47:D1          <NA>
    39 E8:28:C1:DE:47:D2 MIREA_HOTSPOT
    40 E8:28:C1:DE:74:30          <NA>
    41 E8:28:C1:DE:74:31          <NA>
    42 E8:28:C1:DE:74:32 MIREA_HOTSPOT

##### Задание 2. Определить производителя для каждого обнаруженного устройства

-   00:03:7A Taiyo Yuden Co., Ltd.
-   00:03:7F Atheros Communications, Inc.
-   00:25:00 Apple, Inc.
-   00:26:99 Cisco Systems, Inc
-   E0:D9:E3 Eltex Enterprise Ltd.
-   E8:28:C1 Eltex Enterprise Ltd.

##### Задание 3. Выявить устройства, использующие последнюю версию протокола шифрования WPA3, и названия точек доступа, реализованных на этих устройствах

``` r
data_1 %>%
  filter(grepl("WPA3", Privacy)) %>%
  select(BSSID, ESSID, Privacy)
```

                  BSSID              ESSID   Privacy
    1 26:20:53:0C:98:E8               <NA> WPA3 WPA2
    2 A2:FE:FF:B8:9B:C9         Christie’s WPA3 WPA2
    3 96:FF:FC:91:EF:64               <NA> WPA3 WPA2
    4 CE:48:E7:86:4E:33 iPhone (Анастасия) WPA3 WPA2
    5 8E:1F:94:96:DA:FD iPhone (Анастасия) WPA3 WPA2
    6 BE:FD:EF:18:92:44            Димасик WPA3 WPA2
    7 3A:DA:00:F9:0C:02  iPhone XS Max 🦊🐱🦊 WPA3 WPA2
    8 76:C5:A0:70:08:96               <NA> WPA3 WPA2

##### Задание 4. Отсортировать точки доступа по интервалу времени, в течение которого они находились на связи, по убыванию

``` r
data_1_with_intervals <- data_1 %>% 
  mutate(Time.Interval = Last.time.seen - First.time.seen)

data_1_with_intervals %>%
  arrange(desc(Time.Interval)) %>%
  mutate(Time.Interval = seconds_to_period(Time.Interval)) %>%
  select(BSSID, First.time.seen, Last.time.seen, Time.Interval) %>%
  head
```

                  BSSID     First.time.seen      Last.time.seen Time.Interval
    1 00:25:00:FF:94:73 2023-07-28 09:13:06 2023-07-28 11:56:21    2H 43M 15S
    2 E8:28:C1:DD:04:52 2023-07-28 09:13:09 2023-07-28 11:56:05    2H 42M 56S
    3 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38    2H 42M 35S
    4 08:3A:2F:56:35:FE 2023-07-28 09:13:27 2023-07-28 11:55:53    2H 42M 26S
    5 6E:C7:EC:16:DA:1A 2023-07-28 09:13:03 2023-07-28 11:55:12     2H 42M 9S
    6 E8:28:C1:DC:B2:50 2023-07-28 09:13:06 2023-07-28 11:55:12     2H 42M 6S

##### Задание 5. Обнаружить топ-10 самых быстрых точек доступа

``` r
top_10_fastest_spots <- data_1 %>%
  arrange(desc(Speed)) %>%
  select(BSSID, ESSID, Speed, Privacy) %>%
  head(10)

top_10_fastest_spots
```

                   BSSID              ESSID Speed   Privacy
    1  26:20:53:0C:98:E8               <NA>   866 WPA3 WPA2
    2  96:FF:FC:91:EF:64               <NA>   866 WPA3 WPA2
    3  CE:48:E7:86:4E:33 iPhone (Анастасия)   866 WPA3 WPA2
    4  8E:1F:94:96:DA:FD iPhone (Анастасия)   866 WPA3 WPA2
    5  9A:75:A8:B9:04:1E                 KC   360      WPA2
    6  4A:EC:1E:DB:BF:95     POCO X5 Pro 5G   360      WPA2
    7  56:C5:2B:9F:84:90         OnePlus 6T   360      WPA2
    8  E8:28:C1:DC:B2:41       MIREA_GUESTS   360       OPN
    9  E8:28:C1:DC:B2:40      MIREA_HOTSPOT   360       OPN
    10 E8:28:C1:DC:B2:42               <NA>   360       OPN

##### Задание 6. Отсортировать точки доступа по частоте отправки запросов (beacons) в единицу времени по их убыванию

``` r
data_1_with_beacon_frequency <- data_1_with_intervals %>% 
    mutate(beacon_rate = as.double(X..beacons) / as.integer(Time.Interval))

data_1_with_beacon_frequency %>%
  select(BSSID, ESSID, Privacy, X..beacons, Time.Interval, beacon_rate) %>%
  filter(!is.infinite(beacon_rate)) %>%
  arrange(desc(beacon_rate)) %>%
  head
```

                  BSSID               ESSID   Privacy X..beacons Time.Interval
    1 F2:30:AB:E9:03:ED     iPhone (Uliana)      WPA2          6        7 secs
    2 B2:CF:C0:00:4A:60 Михаил's Galaxy M32      WPA2          4        5 secs
    3 3A:DA:00:F9:0C:02   iPhone XS Max 🦊🐱🦊 WPA3 WPA2          5        9 secs
    4 02:BC:15:7E:D5:DC             MT_FREE       OPN          1        2 secs
    5 00:3E:1A:5D:14:45             MT_FREE       OPN          1        2 secs
    6 76:C5:A0:70:08:96                <NA> WPA3 WPA2          1        2 secs
      beacon_rate
    1   0.8571429
    2   0.8000000
    3   0.5555556
    4   0.5000000
    5   0.5000000
    6   0.5000000

#### Данные клиентов

##### Задание 1. Определить производителя для каждого обнаруженного устройства

``` r
data_2 %>%
  filter(grepl("(..:..:..:)(..:..:..)", BSSID)) %>%
  distinct(BSSID)
```

                   BSSID
    1  BE:F1:71:D5:17:8B
    2  BE:F1:71:D6:10:D7
    3  1E:93:E3:1B:3C:F4
    4  E8:28:C1:DC:FF:F2
    5  00:25:00:FF:94:73
    6  00:26:99:F2:7A:E2
    7  0C:80:63:A9:6E:EE
    8  E8:28:C1:DD:04:52
    9  0A:C5:E1:DB:17:7B
    10 9A:75:A8:B9:04:1E
    11 8A:A3:03:73:52:08
    12 4A:EC:1E:DB:BF:95
    13 BE:F1:71:D5:0E:53
    14 08:3A:2F:56:35:FE
    15 6E:C7:EC:16:DA:1A
    16 2A:E8:A2:02:01:73
    17 E8:28:C1:DC:B2:52
    18 E8:28:C1:DC:C6:B2
    19 E8:28:C1:DC:C8:32
    20 56:C5:2B:9F:84:90
    21 9A:9F:06:44:24:5B
    22 12:48:F9:CF:58:8E
    23 E8:28:C1:DD:04:50
    24 AA:F4:3F:EE:49:0B
    25 3A:70:96:C6:30:2C
    26 E8:28:C1:DC:3C:92
    27 8E:55:4A:85:5B:01
    28 5E:C7:C0:E4:D7:D4
    29 E2:37:BF:8F:6A:7B
    30 96:FF:FC:91:EF:64
    31 CE:B3:FF:84:45:FC
    32 00:26:99:BA:75:80
    33 76:70:AF:A4:D2:AF
    34 E8:28:C1:DC:B2:50
    35 00:AB:0A:00:10:10
    36 E8:28:C1:DC:C8:30
    37 8E:1F:94:96:DA:FD
    38 E8:28:C1:DB:F5:F2
    39 E8:28:C1:DD:04:40
    40 EA:7B:9B:D8:56:34
    41 BE:FD:EF:18:92:44
    42 7E:3A:10:A7:59:4E
    43 00:26:99:F2:7A:E1
    44 00:23:EB:E3:49:31
    45 E8:28:C1:DC:B2:40
    46 E0:D9:E3:49:04:40
    47 3A:DA:00:F9:0C:02
    48 E8:28:C1:DC:B2:41
    49 E8:28:C1:DE:74:32
    50 E8:28:C1:DC:33:12
    51 92:F5:7B:43:0B:69
    52 DC:09:4C:32:34:9B
    53 E8:28:C1:DC:F0:90
    54 E0:D9:E3:49:04:52
    55 22:C9:7F:A9:BA:9C
    56 E8:28:C1:DD:04:41
    57 92:12:38:E5:7E:1E
    58 B2:1B:0C:67:0A:BD
    59 E8:28:C1:DC:33:10
    60 E0:D9:E3:49:04:41
    61 1E:C2:8E:D8:30:91
    62 A2:64:E8:97:58:EE
    63 A6:02:B9:73:2F:76
    64 A6:02:B9:73:81:47
    65 AE:3E:7F:C8:BC:8E
    66 B6:C4:55:B5:53:24
    67 86:DF:BF:E4:2F:23
    68 02:67:F1:B0:6C:98
    69 36:46:53:81:12:A0
    70 E8:28:C1:DC:0B:B0
    71 82:CD:7D:04:17:3B
    72 E8:28:C1:DC:54:B2
    73 00:03:7F:10:17:56
    74 00:0D:97:6B:93:DF

-   00:03:7F Atheros Communications, Inc.
-   00:0D:97 Hitachi Energy USA Inc.
-   00:23:EB Cisco Systems, Inc
-   00:25:00 Apple, Inc.
-   00:26:99 Cisco Systems, Inc
-   08:3A:2F Guangzhou Juan Intelligent Tech Joint Stock Co.,Ltd
-   0C:80:63 Tp-Link Technologies Co.,Ltd.
-   DC:09:4C Huawei Technologies Co.,Ltd
-   E0:D9:E3 Eltex Enterprise Ltd.
-   E8:28:C1 Eltex Enterprise Ltd.

##### Задание 2. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

``` r
data_2 %>%
  filter(grepl("(..:..:..:)(..:..:..)", BSSID) & !is.na(Probed.ESSIDs)) %>%
  select(BSSID, Probed.ESSIDs) %>%
  group_by(BSSID, Probed.ESSIDs) %>%
  filter(n() > 1) %>%
  arrange(BSSID) %>%
  unique()
```

    # A tibble: 16 × 2
    # Groups:   BSSID, Probed.ESSIDs [16]
       BSSID             Probed.ESSIDs   
       <chr>             <chr>           
     1 00:26:99:BA:75:80 GIVC            
     2 00:26:99:F2:7A:E2 GIVC            
     3 0C:80:63:A9:6E:EE KOTIKI_XXX      
     4 1E:93:E3:1B:3C:F4 Galaxy A71      
     5 6E:C7:EC:16:DA:1A Cnet            
     6 8E:55:4A:85:5B:01 Vladimir        
     7 AA:F4:3F:EE:49:0B Redmi Note 8 Pro
     8 BE:F1:71:D5:17:8B C322U13 3965    
     9 E8:28:C1:DC:B2:50 MIREA_HOTSPOT   
    10 E8:28:C1:DC:B2:50 MIREA_GUESTS    
    11 E8:28:C1:DC:B2:52 MIREA_HOTSPOT   
    12 E8:28:C1:DC:C6:B2 MIREA_HOTSPOT   
    13 E8:28:C1:DC:F0:90 MIREA_GUESTS    
    14 E8:28:C1:DD:04:40 MIREA_HOTSPOT   
    15 E8:28:C1:DD:04:52 MIREA_HOTSPOT   
    16 E8:28:C1:DE:74:32 MIREA_HOTSPOT   

##### Задание 3. Кластеризовать запросы от устройств к точкам доступа по их именам. Определить время появления устройства в зоне радиовидимости и время выхода его из нее

``` r
clustered_data <- data_2 %>%
  filter(!is.na(Probed.ESSIDs)) %>%
  group_by(Station.MAC, Probed.ESSIDs) %>%
  arrange(First.time.seen)

cluster_summary <- clustered_data %>%
  summarise(Cluster_Start_Time = min(First.time.seen),
            Cluster_End_Time = max(Last.time.seen),
            Total_Power = sum(Power))
```

    `summarise()` has grouped output by 'Station.MAC'. You can override using the
    `.groups` argument.

``` r
cluster_summary %>% head(10)
```

    # A tibble: 10 × 5
    # Groups:   Station.MAC [10]
       Station.MAC Probed.ESSIDs Cluster_Start_Time  Cluster_End_Time    Total_Power
       <chr>       <chr>         <dttm>              <dttm>                    <int>
     1 00:90:4C:E… Redmi         2023-07-28 09:16:59 2023-07-28 10:21:15         -65
     2 00:95:69:E… nvripcsuite   2023-07-28 09:13:11 2023-07-28 11:56:13         -55
     3 00:95:69:E… nvripcsuite   2023-07-28 09:13:15 2023-07-28 11:56:17         -33
     4 00:95:69:E… nvripcsuite   2023-07-28 09:13:11 2023-07-28 11:56:07         -69
     5 00:F4:8D:F… Redmi 12      2023-07-28 10:45:04 2023-07-28 11:43:26         -73
     6 02:00:00:0… xt3 w64dtgv5… 2023-07-28 09:54:40 2023-07-28 11:55:36         -67
     7 02:06:2B:A… Avenue611     2023-07-28 09:55:12 2023-07-28 09:55:12         -65
     8 02:1D:0F:A… iPhone (Дима… 2023-07-28 09:57:08 2023-07-28 09:57:08         -61
     9 02:32:DC:5… Timo Resort   2023-07-28 10:58:23 2023-07-28 10:58:24         -84
    10 02:35:E9:C… iPhone (Макс… 2023-07-28 10:00:55 2023-07-28 10:00:55         -88

##### Задание 4. Оценить стабильность уровня сигнала внури кластера во времени. Выявить наиболее стабильный кластер. Для оценки стабильности оценить математическое ожидание и среднеквадратичное отклонение для каждого найденного кластера.

``` r
stability_metrics <- clustered_data %>%
  group_by(Station.MAC, Probed.ESSIDs) %>%
  summarise(Mean_Power = mean(Power))
```

    `summarise()` has grouped output by 'Station.MAC'. You can override using the
    `.groups` argument.

``` r
stability_metrics %>%
  arrange((Mean_Power)) %>% head(1)
```

    # A tibble: 1 × 3
    # Groups:   Station.MAC [1]
      Station.MAC       Probed.ESSIDs  Mean_Power
      <chr>             <chr>               <dbl>
    1 8A:45:77:F9:7F:F4 iPhone (Дима )        -89

## Оценка результатов

В ходе практической работы были импортированы, подготовлены и
проанализированы данные трафика Wi-Fi сетей

## Вывод

Были закреплены навыки работы с пакетом `dplyr`
