# VK-Education-Highload Курсовая работа
Шишкин А. Ю. WEB-41/31
# RUTUBE
## 1. Тема и Целевая Аудитория
**Тип сервиса**: Видеохостинг
**Местоположение целевой аудитории**: Россия и СНГ

### Основные страны пользователей
|Страна|% Пользователей|
|-|--------|
|Россия|93.3%|
|Казахстан|1.3%|
|Беларусь|1%|
|Украина|0.9%|
[Источник][3]

__Ключевой функционал__: Размещение видео

__Ключевое продуктовое решение__: Рекомендательная система видео

### MVP
* Авторизация
* Загрузка видео
* Просмотр видео (плеер)
* Просмотры и лайки
* Комментарии
* Админ-панель для модерации
* Рекомендации на главной 
* Поиск
* Главная Страница
* Страница автора

## 2. Расчёт нагрузки

### Предварительные данные
[**MAU**: 80M][1]

[**DAU**: 20M][1]

[4M каналов (зарегестрированных пользователей) всего][10]

**Средняя продолжительность визита: [46 минут][10]**

[**Средняя длина видео**: 12 минут][8]

3.8 Видео за посещение

[**Всего видео на площадке - 388М**][7]

1 минута видео весит 480мбит (0.48гб)(1080p, 30fps, 8Мбит/с битрейта).

Общий вес видео на рутубе на данный момент: 388М * 12 * 0.48 = **2 234 ПБ**

[421М Видео на площадке всего][5]

[33000М просмотров за полгода][5]

33000M / (6 * 30) = **183М / день** или **2К / секунда**


### RPS
| Тип запроса                                  |    Суточно  | Среднее RPS | Пиковое RPS (Среднее * 3) | Пояснение  |
| :------------------------------------------: | :-----------: | ----------: |        ----------:        | :----------:|
| **Просмотр видео**                           |          183M |         ~2K |          ~6K              |            |
| **Стриминг видео**                           |       131760M |       ~1.4M |          ~4.2M            | 2К запусков видео в секунду * 12*60 длительность видео в секундах, получается больно много, но по цифрам всё так будто |
| **Загрузка видео**                           |          370К |        ~4.3 |          ~13              | ~+135M видео за год => 135М/365 = 370K  |
| **Авторизация**                              |            4M |         ~47 |         ~141              | [4M каналов всего][10] |
| **Комментарии**                              |            1M |         ~12 |          ~36              | 1 коммент на 200 просмотров |
| **Главная Страница**                         |           20M |        ~231 |         ~696              | Каждый из DAU видит главную, хотя кто-то же заходит точечно по ссылке на видео... |
| **Профиль**                                  |           20K |        ~0.2 |         ~0.6              | В дальнейшем такие копейки игнорируем... |
| **Рекомендации**                             |          200M |       ~2.3K |          ~7K              | Каждый раз когда смотрят видео получают рекомендации + копейки с главной |
| **Лайки**                                    |            8M |         ~94 |         ~282              | 4 лайка на 100 просмотров |
| **Медиа (аватарки + превьюшки)**             |        2 745M |        ~32K |         ~96K              | На 1м экране просмотра видео умещается 8 рекомендованных видео, округлим до 10 и добавим 5 аватарок пользователей  |

### Трафик
| Тип трафика                                                           | Среднее (Гб/с) | Пиковое (Гб/c) | В сутки (ТБ) | Комментарий                      |
| --------------------------------------------------------------------- | -------------: | -------------: | ------------: | :------------------------------: |
| **Просмотр Видео**                                                    |            ~10 |            ~30 |      864     | 2000 видео/c * 0.005 мб          |
| **Стриминг Видео**                                                    |        ~10 400 |        ~31 200 |  624 000     | 1.4M RPS * 8мбит/c // чёт много              |
| **Загрузка Видео**                                                    |            ~25 |            ~75 |    6 480     | 4.3 * 12 мин * 0.48 гб/мин       |
| **Медиа**                                                             |            ~64 |           ~192 |   16 588     | 32K RPS * 2мб                    |
| **Итого**                                                             |      **~1507** |      **~4521** | **380 000**  |                                  |
 
### Хранилище
|Хранимый контент| Надо сейчас (ТБ) | Надо будет через год (ТБ) |Пояснение          |
|:--------------:|-----------------:|--------------------------:|:-----------------:|
|Видео           |        2 234 000 |                 3 041 280 | Чё так много то...|
|Медиа           |              850 |                      1190 |  4M * 2Мб (аватарки) + 421М * 2Мб (превьюхи) |
|Таблички (Просмотры, лайки, комменты)| | | Сущие копейки на фоне остального которые я даже не представляю как считать|

## 3. Глобальная балансировка нагрузки
### Функциональное разбиение по доменам
| Домен                                     | Назначение                                         |
| ----------------------------------------- | -------------------------------------------------- |
| **auth.rutube.ru**                        | Авторизация и управление сессиями                  |
| **upload.rutube.ru**                      | Загрузка видео и обработка                         |
| **video.rutube.ru**                       | Отдача видеопотока (плеер)                         |
| **api.rutube.ru**                         | Просмотры, лайки, комментарии, поиск, рекомендации |
| **admin.rutube.ru**                       | Админ-панель и модерация                           |
| **media.rutube.ru**                       | Превьюхи и аватарки                                |

### Обоснование расположения дата-центров

Москва — основной центр, большая часть аудитории в европейской части РФ.

Владивосток / Хабаровск — снижение задержек для Дальнего Востока.
### Расчет распределения запросов

### DNS


DNS-сервер не эффективен в пределах одной страны


### Anycast

Объявляем одинаковые ip в ДЦ Москвы и Дальнего востока, когда пользователь пытается подключиться к нашему сервису он подключается по ip, остальное дело за BGP который сам выберет ближайший к пользователю ДЦ

# 4. Локальная балансировка нагрузки (Rutube, 1 ДЦ)

## Исходные показатели

| Параметр | Значение |
|-----------|-----------|
| Пиковый трафик (видео) | **31 200 Gbit/s** |
| Пиковый трафик (API / статистика и прочее) | **300 Gbit/s** |
| Архитектура | **L4 (видео)** + **L7 (API, статистика, TLS)** |
| Политика резервирования | **N + 1** |
| Размещение | **все ноды в одном ДЦ** |

---

## Формулы расчёта

1. Количество нод по пропускной способности:  
   **LB_bw = Peak_Gbps / Throughput_per_node (округление вверх)**

2. Применение резервирования (N + 1):  
   **LB_final = LB_bw + 1**

---

## Расчёт для L4 (видеопоток)

| Показатель | Значение |
|-------------|-----------|
| Peak_Gbps_L4 | **31 200 Gbit/s** |
| Пропускная способность одной L4-ноды | **400 Gbit/s** |

LB_bw_L4 = 31 200 / 400 = 78  
LB_final_L4 = 78 + 1 = **79 нод**

**Итого (L4, единый ДЦ): 79 нод (все в одном ДЦ).**

---

## Расчёт для L7 (API / статистика / TLS)

| Показатель | Значение |
|-------------|-----------|
| Peak_Gbps_L7 | **300 Gbit/s** |
| Пропускная способность одной L7-ноды | **40 Gbit/s** |

LB_bw_L7 = 300 / 40 = 7.5 → 8 нод  
LB_final_L7 = 8 + 1 = **9 нод**

**Итого (L7, единый ДЦ): 9 нод (все в одном ДЦ).**

---

## Итоговая сводка (в одном ДЦ)

| Слой | Пиковая нагрузка | Пропускная способность ноды | Резервирование | Всего нод |
|------|------------------|-----------------------------|----------------|----------:|
| **L4 (видео)** | 31 200 Gbit/s | 400 Gbit/s | N+1 | **79** |
| **L7 (API, статистика)** | 300 Gbit/s | 40 Gbit/s | N+1 | **9** |

---

# 5. Логическая схема БД
``` mermaid
erDiagram
    users {
        int id PK
        varchar username
        varchar email
        varchar password_hash
        timestamp created_at
        varchar avatar_url
    }

    videos {
        int id PK
        int user_id FK
        varchar title
        text description
        varchar file_path
        varchar thumbnail_url
        date upload_date
        int duration
        bool is_moderated
    }

    views {
        int id PK
        int video_id FK
        int user_id FK
        timestamp view_date
    }

    likes {
        int id PK
        int video_id FK
        int user_id FK
        timestamp created_at
    }

    comments {
        int id PK
        int video_id FK
        int user_id FK
        int parent_comment_id FK "nullable"
        text text
        bool is_banned
        timestamp created_at
    }

    moderators {
        int id PK
        int user_id FK
        varchar role
    }

    sessions {
        varchar session_id PK
        int user_id FK
        timestamp created_at
        timestamp expires_at
        text session_data
    }

    video_storage {
        int id PK
        int video_id FK
        varchar storage_path
        varchar resolution
        varchar format
        int size_mb
        varchar checksum
        timestamp uploaded_at
    }

    video_stats {
        int id PK
        int video_id FK
        bigint total_views
        bigint total_likes
        timestamp updated_at
    }

    search_index {
        int id PK
        int video_id FK
        text keywords
        text tags
        tsvector search_vector
        timestamp updated_at
    }

    subscriptions {
        int id PK
        int subscriber_id FK
        int subscribed_to_id FK
        timestamp created_at
    }

    reports {
        int id PK
        int reporter_id FK
        int target_user_id FK "nullable"
        int target_video_id FK "nullable"
        int target_comment_id FK "nullable"
        text reason
        bool is_resolved
        timestamp created_at
    }

    users ||--o{ videos : "загружает"
    users ||--o{ views : "просматривает"
    users ||--o{ likes : "ставит лайки"
    users ||--o{ comments : "пишет"
    users ||--o{ moderators : "может быть модератором"
    users ||--o{ sessions : "имеет сессии"
    users ||--o{ subscriptions : "подписывается"
    users ||--o{ reports : "отправляет"

    videos ||--o{ views : "просматривается"
    videos ||--o{ likes : "лайкается"
    videos ||--o{ comments : "комментируется"
    videos ||--o{ video_storage : "имеет файлы"
    videos ||--|| video_stats : "имеет статистику"
    videos ||--|| search_index : "имеет индекс"
    videos ||--o{ reports : "может быть объектом жалобы"

    comments ||--o{ comments : "ответ на"
    comments ||--o{ reports : "может быть объектом жалобы"

    moderators ||--o{ reports : "обрабатывает"

```

# 6 пункт какой он там
```mermaid
graph LR

subgraph PostgreSQL_Auth["Auth DB : PostgreSQL"]
    U[users]
    SUB[subscriptions]
    M[moderators]
end

subgraph PostgreSQL_Video["Video Metadata DB : PostgreSQL"]
    V[videos]
    VS[video_stats]
    VST[video_storage_metadata]
end

subgraph ClickHouse["Activity DB : ClickHouse"]
    VW[views]
    L[likes]
    C[comments]
    R[reports]
end

subgraph Redis["Redis : Sessions / Cache"]
    S[(sessions)]
end

subgraph MinIO["Object Storage : MinIO / S3"]
    OBJ[(video_files)]
end

subgraph Elasticsearch["Search Engine : Elasticsearch"]
    SI[(video_search_index)]
end

%% Relations
U --> V
U --> VW
U --> L
U --> C
U --> SUB
U --> R
V --> VW
V --> L
V --> C
V --> VS
V --> R
V --> VST
V --> SI
VST --> OBJ
C --> R
M --> R
S --> U

classDef shard fill:#ffcc00,stroke:#333,stroke-width:1px,color:#000
VW:::shard
L:::shard
C:::shard
VS:::shard


```

| Хранилище                   | Таблица / Индекс         | Назначение                                                                                 |
| --------------------------- | ------------------------ | ------------------------------------------------------------------------------------------ |
| **PostgreSQL (AuthDB)**     | `users`                  | Профили пользователей, авторизация, связь с сессиями                                       |
|                             | `subscriptions`          | Подписки на других пользователей                                                           |
|                             | `moderators`             | Роли и доступы модераторов                                                                 |
| **PostgreSQL (VideoDB)**    | `videos`                 | Метаданные видео (название, описание, ID владельца, дата загрузки, флаги модерации)        |
|                             | `video_stats`            | Счётчики лайков, просмотров и комментариев по видео (для ускоренного доступа)              |
|                             | `video_storage_metadata` | Служебные данные о хранении файлов в MinIO: путь, версия, формат, thumbnail                |
| **ClickHouse (ActivityDB)** | `views`                  | Подробные события просмотров (user_id, video_id, timestamp)                                |
|                             | `likes`                  | События лайков с точными временными метками                                                |
|                             | `comments`               | Комментарии и иерархия ответов                                                             |
|                             | `reports`                | Репорты на контент или пользователей                                                       |
| **Redis**                   | `sessions`               | Авторизационные токены / refresh-токены / CSRF-защита / TTL                                |
| **MinIO (S3 совместимое)**  | `video_files`            | Файлы видео, превью, thumbnails — хранятся по UUID                                         |
| **Elasticsearch**           | `video_search_index`     | Полнотекстовый индекс: названия, описания, теги, имена авторов (для поиска и рекомендаций) |


## Индексы

### PostgreSQL
| Таблица         | Индекс                              | Тип   |
| --------------- | ----------------------------------- | ----- |
| `users`         | `email`, `username`                 | BTREE |
| `videos`        | `upload_date`                       | BTREE |
| `video_stats`   | `video_id`                          | BTREE |
| `subscriptions` | `(subscriber_id, subscribed_to_id)` | BTREE |


### Clickhouse
| Таблица    | PRIMARY KEY              | ORDER BY                 |
| ---------- | ------------------------ | ------------------------ |
| `views`    | `(video_id, user_id)`    | `(video_id, view_date)`  |
| `likes`    | `(video_id, user_id)`    | `(video_id, created_at)` |
| `comments` | `(video_id, created_at)` | `(video_id, created_at)` |
| `reports`  | `(video_id, created_at)` | `(video_id, created_at)` |

### Elasticsearch
| Индекс               | Поля                                          | Назначение                          |
| -------------------- | --------------------------------------------- | ----------------------------------- |
| `video_search_index` | `title`, `description`, `tags`, `author_name` | Полнотекстовый поиск и рекомендации |


## Денормализация
| Таблица                  | Денормализованные поля                         | Причина                                  |
| ------------------------ | ---------------------------------------------- | ---------------------------------------- |
| `video_stats`            | `total_views`, `total_likes`                   | Быстрые выборки без агрегации            |
| `views` / `likes`        | `video_id`, `user_id`                          | Упрощение JOIN-ов в ClickHouse           |
| `comments`               | `parent_comment_id`                            | Быстрая иерархия комментариев            |
| `video_storage_metadata` | `bucket`, `object_key`, `resolution`, `format` | Быстрый доступ к файлам без JOIN с MinIO |


## Выбор БД

| Назначение                    | СУБД              | Причина выбора                     |
| ----------------------------- | ----------------- | ---------------------------------- |
| Пользователи, метаданные      | **PostgreSQL**    | ACID, транзакции                   |
| Активность (просмотры, лайки) | **ClickHouse**    | Аналитика и real-time              |
| Видео-файлы                   | **MinIO (S3)**    | Масштабируемое объектное хранилище |
| Сессии / кеш                  | **Redis Cluster** | In-memory, TTL                     |
| Поиск                         | **Elasticsearch** | Быстрый полнотекстовый поиск       |


## Шардирование
| Компонент                 | Метод шардинга                     | Репликация / резервирование   |
| ------------------------- | ---------------------------------- | ----------------------------- |
| PostgreSQL (Auth / Video) | Hash по `user_id` или `video_id`   | Master + 2 replicas (async)   |
| ClickHouse                | Distributed table по `video_id`    | ReplicatedMergeTree (2 копии) |
| Redis                     | А он тут нужен?                    | Cluster mode + Sentinel       |
| MinIO                     | Erasure coding + replication       | 4–8 нод, 2 parity блока       |
| Elasticsearch             | Shards по `video_id` + replica     | primary + replica shards      |


## Клиентские библиотеки и интеграции
| Компонент         | Клиент (Go)                   | Заметки                  |
| ----------------- | ----------------------------- | ------------------------ |
| PostgreSQL        | `pgx`                         | connection pooling       |
| ClickHouse        | `clickhouse-go/v2`            | async вставка батчами    |
| Redis             | `go-redis/v9`                 | для сессий и кеша        |
| MinIO             | `minio-go/v7`                 | multipart upload         |
| Elastic           | `elastic/go-elasticsearch/v8` | если нужен внешний поиск |

## Резервное копирование
| Компонент           | Метод                | Частота                | Хранение           |
| ------------------- | -------------------- | ---------------------- | ------------------ |
| PostgreSQL          | `pg_dump` + WAL      | каждые 6 ч (инкремент) | S3 / MinIO         |
| ClickHouse          | `BACKUP TABLE` → S3  | 1 раз в сутки          | S3 snapshot        |
| MinIO               | Snapshot replication | раз в сутки            | отдельный кластер  |
| Конфиги (YAML, env) | Git + S3 backup      | при деплое             | Git / S3           |


[1]: https://tass.ru/ekonomika/24311321 "Источник"
[2]: https://inclient.ru/rutube-stats/#rutube3 "Не уверен верить ли источнику"
[3]: https://www.similarweb.com/ru/website/rutube.ru/#demographics "Трафик по странам"
[5]: https://www.cnews.ru/news/line/2025-09-04_dnevnaya_auditoriya_rutube_vyrosla "Последняя статистика по рутубу"
[6]: https://affmaven.com/ru/youtube-statistics/ "Табличка статистики рутуба которую можем спроецировать на рутуб"
[7]: https://habr.com/ru/news/896842/ "388кк видео на рутубе"
[8]: ... "Источника нет, предположение нейронки"
[9]: https://skillbox.ru/media/marketing/mediascope-opublikovala-issledovanie-auditorii-sotsialnykh-media-youtube-poka-eshchye-lider/ "Инфа от медиаскоупа"
[10]: https://habr.com/ru/news/926262/
