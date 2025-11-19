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

# 6 Физическая схема БД
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


# 7. Алгоритмы

## 7.1. Алгоритм адаптивного стриминга (HLS/DASH)

**Область применения:** Просмотр видео

**Назначение:** Обеспечение плавного просмотра при различном качестве интернет-соединения пользователя.

**Принцип работы:**

### Подготовка контента:
- Видео заранее кодируется в несколько качеств (1080p, 720p, 480p, 360p)
- Каждая версия нарезается на короткие чанки продолжительностью 2-10 секунд
- Формируется манифест-файл с информацией о всех доступных чанках и их качестве

### Инициализация просмотра:
- Плеер загружает манифест и начинает воспроизведение со среднего качества
- Одновременно оценивается скорость сети и стабильность соединения

### Динамическая адаптация:
- Алгоритм постоянно мониторит:
  - Скорость загрузки чанков
  - Уровень заполнения буфера воспроизведения
  - Частоту потерь пакетов
- На основе этих метрик выбирает оптимальное качество для следующего чанка
- При ухудшении связи - переключается на более низкое качество
- При улучшении - постепенно повышает качество

---

## 7.2. Алгоритм автоматического анализа видеоконтента

**Область применения:** Обработка загруженных видео, улучшение поиска и рекомендаций

**Назначение:** Автоматическое извлечение семантической информации из видео для улучшения метаданных.

**Принцип работы:**

### Анализ визуального контента:
- **Детекция объектов и сцен:** автоматическое распознавание людей, животных, транспорта, архитектуры, природы
- **Классификация контента:** определение жанра видео (интервью, влог, обзор, туториал, развлечения)
- **Анализ цветовой палитры:** определение визуального стиля и атмосферы видео
- **Распознавание текста:** извлечение текстовых надписей из кадров для дополнительных ключевых слов

### Анализ аудиодорожки:
- **Транскрибация речи:** автоматическое преобразование речи в текст для создания субтитров
- **Определение языка:** автоматическое определение языка контента
- **Анализ звуковой атмосферы:** распознавание музыки, шумов, типов аудиоконтента
- **Выделение ключевых фраз:** автоматическое извлечение основных тем из распознанной речи

### Обогащение метаданных:
- **Автогенерация тегов:** создание релевантных тегов на основе анализа контента
- **Определение категории:** автоматическая классификация видео по тематическим категориям
- **Извлечение ключевых кадров:** выбор наиболее репрезентативных изображений для превью
- **Оценка содержания:** определение возрастного рейтинга и типа контента

---

## 7.3. Алгоритм персонализированных рекомендаций

**Область применения:** Главная страница, рекомендации, "Следующее видео"

**Назначение:** Максимизация времени на платформе через показ релевантного контента на основе интересов пользователя.

**Принцип работы:**

### Многоуровневая архитектура рекомендаций:

**Первый уровень - отбор кандидатов:**
- **Content-based filtering:** "похожее по семантике" на основе анализа видеоконтента
- **Collaborative filtering:** "похожие пользователи смотрели" с учетом временных паттернов
- **Knowledge-based:** рекомендации по явным предпочтениям (подписки, избранное)
- **Context-aware:** учет времени суток, устройства, локации, дня недели
- **Trend-based:** популярный контент в регионе и социальные тренды
- **Freshness-based:** новые видео от подписанных авторов и актуальный контент

**Второй уровень - интеллектуальное ранжирование:**
- **Deep learning ranking:** нейросетевая модель, учитывающая 100+ признаков
- **Multi-objective optimization:** баланс между релевантностью, разнообразием и новизной
- **Session-aware ranking:** учет текущей сессии просмотра и контекста
- **Exploration vs exploitation:** баланс между проверенным и новым контентом
- **Business rules:** приоритизация партнерского контента и монетизируемых видео

### Метрики качества:
- **Short-term engagement:** CTR, время просмотра, завершение видео
- **Long-term retention:** возвращаемость пользователя, частота визитов
- **Content diversity:** разнообразие рекомендуемого контента
- **Serendipity:** коэффициент "приятных неожиданностей" в рекомендациях

---

## 7.4. Алгоритм полнотекстового поиска

**Область применения:** Поиск по видеоплатформе

**Назначение:** Обеспечение точного и быстрого поиска видео по всем метаданным, включая автоматически извлеченные.

**Принцип работы:**

### Индексация данных:
**Структура поискового индекса включает:**
- **Базовые метаданные:** названия, описания, теги автора
- **Автоматически извлеченные данные:** транскрипция речи, распознанный текст из кадров
- **Визуальные дескрипторы:** объекты, сцены, активности, выявленные алгоритмом анализа
- **Контекстуальные признаки:** продолжительность, качество, язык, возрастной рейтинг
- **Социальные сигналы:** лайки, комментарии, просмотры, репосты

### Ранжирование результатов:

**Многофакторная модель релевантности:**
- **Текстовая релевантность:** BM25 + семантическое сходство через эмбеддинги
- **Визуальное соответствие:** совпадение по выявленным объектам и сценам
- **Популярностные метрики:** просмотры, вовлеченность, социальные сигналы
- **Свежесть контента:** приоритет новым и актуальным видео
- **Персонализация:** учет истории поисков и просмотров пользователя
- **Качество контента:** оценка технического и содержательного качества

### Возможности поиска:

**Гибридный поиск:**
- **Semantic search:** поиск по смыслу, а не точным словам
- **Visual search:** поиск по визуальным характеристикам ("видео с закатами")
- **Faceted search:** фильтрация по продолжительности, качеству, дате, языку
- **Voice search:** оптимизация для голосовых запросов

**Поисковые подсказки:**
- Автодополнение на основе популярных запросов и истории пользователя
- Исправление опечаток и орфографических ошибок
- Синонимизация и расширение запросов

---

## 7.5. Алгоритм сегментации и кодирования видео

**Область применения:** Обработка загружаемых видео

**Назначение:** Подготовка видеофайлов к эффективному хранению и адаптивному стримингу.

**Принцип работы:**

### Прием загрузки:
- Пользователь загружает видеофайл
- Файл проверяется на безопасность и валидность
- Видео помещается в очередь обработки

### Параллельная обработка:
- **Кодирование в multiple bitrate:** создание версий разного качества
- **Контент-анализ:** запуск алгоритма автоматического анализа видеоконтента
- Обе процессы выполняются параллельно для ускорения обработки

### Сегментация на чанки:
- Каждая версия качества нарезается на сегменты по 2-10 секунд
- Формируются плейлисты и манифесты для адаптивного стриминга

### Синхронизация метаданных:
- Результаты контент-анализа сохраняются в базу данных
- Обновляются поисковые индексы
- Видео становится доступным для поиска и рекомендаций
# 8 Технологии

| Технология | Область применения | Обоснование выбора |
|------------|-------------------|-------------------|
| **Go (Golang)** | Бэкенд-сервисы, API | Высокая производительность, простота разработки микросервисов, эффективное потребление ресурсов |
| **Nginx** | Балансировщик нагрузки (L7), раздача статики | Высокая производительность, гибкая конфигурация, проверенная надежность |
| **Docker, Kubernetes** | Контейнеризация, оркестрация | Изоляция сервисов, масштабируемость, упрощение деплоя |
| **Apache Kafka** | Очереди событий, стриминг данных | Высокая пропускная способность, отказоустойчивость, гарантированная доставка сообщений |
| **Grafana + Prometheus** | Мониторинг и визуализация | Мониторинг метрик в реальном времени, гибкие дашборды, оповещения |
| **HAProxy** | Балансировщик нагрузки (L4) | Эффективная балансировка TCP-трафика, стабильность, мониторинг |
| **FFmpeg** | Обработка видео | Промышленный стандарт для кодирования/декодирования видео, поддержка всех форматов |
| **hls.js** | Клиентский плеер (браузер) | Библиотеки для адаптивного стриминга в браузере, поддержка HLS/DASH |
| **Jaeger** | Распределенный трейсинг | Отслеживание запросов в микросервисной архитектуре, диагностика проблем |  
| **ArgoCD** | GitOps деплой | Автоматизация деплоя приложений, синхронизация с git-репозиторием |

# 10 Схема проекта
```mermaid
---
config:
  layout: dagre
---
flowchart TB
    EXT_USER["Пользователь"] --> ANYCAST["Anycast IP<br>Ближайший ДЦ"]
    ANYCAST --> L4_EXT["L4 Балансировщик<br>HAProxy"]
    L4_EXT --> CDN["CDN<br>Статика, чанки видео"] & L7_AUTH["L7: auth.rutube.ru"] & L7_API["L7: api.rutube.ru"] & L7_REC["L7: rec.rutube.ru"] & L7_SEARCH["L7: search.rutube.ru"] & n1["L7: video.rutube.ru"] & UPLOAD_SERVICE["Upload Service<br>Go"]
    L7_AUTH --> AUTH_SERVICE["Auth Service<br>Go"]
    AUTH_SERVICE --> REDIS[("Redis<br>sessions")] & KAFKA_AUTH[["Kafka Auth"]]
    L7_API --> API_SERVICE["API Service<br>Go"]
    API_SERVICE --> REC_SERVICE["Recommendation Service<br>Python/Go"] & SEARCH_SERVICE["Search Service<br>Go"] & KAFKA_API[["Kafka API"]]
    VIDEO_SERVICE["Video Service<br>Go"] --> MINIO[("MinIO<br>video_files")] & KAFKA_VIDEO[["Kafka Video"]]
    UPLOAD_SERVICE --> MINIO & KAFKA_UPLOAD[["Kafka Upload"]]
    L7_REC --> REC_SERVICE
    REC_SERVICE --> REC_ENGINE_SINGLE["Single Rec Engine"]
    L7_SEARCH --> SEARCH_SERVICE
    SEARCH_SERVICE --> KAFKA_SEARCH[["Kafka Search"]] & REQUEST_FILLER["Search Request Filler App"]
    KAFKA_AUTH --> PG_AUTH[("PostgreSQL<br>users")]
    KAFKA_API --> ITEM_FEATURES_DB[("PostgreSQL<br>item_features")]
    KAFKA_VIDEO --> PG_VIDEO[("PostgreSQL<br>videos")]
    KAFKA_UPLOAD --> PG_VIDEO
    KAFKA_SEARCH --> SEARCH_INDEX_DB[("PostgreSQL<br>search_index")]
    SEARCH_APP["Simple Search App"] --> SEARCH_RANKER["Simple Ranker App"] & SEARCH_INDEX_DB & ES[("Elasticsearch<br>search_index")]
    SEARCH_RANKER --> RANKING_MODELS_DB[("PostgreSQL<br>ranking_models")] & SEARCH_SERVICE & USERS_HISTORY_DB[("PostgreSQL<br>users_history")]
    REQUEST_FILLER --> RECENT_SEARCHES_DB[("Redis<br>recent_searches")] & SEARCH_APP
    KAFKA[["Kafka<br>События"]] --> ITEM_FEATURES_WORKER["Item Features Worker<br>Python"] & USER_ACTIVITY_WORKER["User Activity Worker<br>Python"] & RECENT_SEARCH_WORKER["Recent Search Worker<br>Python"] & TRAFFIC_MARKER_WORKER["Traffic Marker Worker"] & EVENTS_SERVICE["Events Service"] & PG_VIDEO
    ITEM_FEATURES_WORKER --> ITEM_FEATURES_DB
    ITEM_FEATURES_DB --> SEARCH_INDEX_WORKER["Search Index Worker"] & TRAIN_RANKER_WORKER["Train Ranker Worker<br>Python"]
    SEARCH_INDEX_WORKER --> SEARCH_INDEX_DB & ES
    USER_ACTIVITY_WORKER --> USERS_HISTORY_DB
    RECENT_SEARCH_WORKER --> RECENT_SEARCHES_DB
    TRAFFIC_MARKER_WORKER --> RANKING_MODELS_DB
    USERS_HISTORY_DB --> TRAIN_RANKER_WORKER
    TRAIN_RANKER_WORKER --> RANKING_MODELS_DB
    EVENTS_SERVICE --> DWH_DB[("ClickHouse<br>analytics")]
    ITEM_SERVER["Item Server"] --> SAMPLE_RANKER_APP["Sample Ranker App"]
    PROMETHEUS["Prometheus<br>Сбор метрик"] --> GRAFANA["Grafana<br>Дашборды"]
    n1 --> VIDEO_SERVICE
    n2["Api Upload Video<br>Services"] --> KAFKA & PROMETHEUS
    REC_ENGINE_RANKER["Simple Ranker App"] --> REC_SERVICE
    REC_ENGINE_SINGLE --> REC_ENGINE_RANKER & REDIS_REC[("Redis<br>rec_cache")] & ITEM_FEATURES_DB
    REDIS_REC --> REC_ENGINE_RANKER
    REC_ENGINE_SINGLE --> USERS_HISTORY_DB
    n1@{ shape: rect}
    n2@{ shape: diam}
     EXT_USER:::external
     ANYCAST:::external
     L4_EXT:::loadbalancer
     CDN:::external
     L7_AUTH:::loadbalancer
     L7_API:::loadbalancer
     L7_REC:::loadbalancer
     L7_SEARCH:::loadbalancer
     n1:::loadbalancer
     UPLOAD_SERVICE:::service
     AUTH_SERVICE:::service
     REDIS:::storage
     KAFKA_AUTH:::queue
     API_SERVICE:::service
     REC_SERVICE:::recommendation
     SEARCH_SERVICE:::service
     KAFKA_API:::queue
     VIDEO_SERVICE:::service
     MINIO:::storage
     KAFKA_VIDEO:::queue
     KAFKA_UPLOAD:::queue
     REC_ENGINE_SINGLE:::recommendation
     KAFKA_SEARCH:::queue
     REQUEST_FILLER:::search
     PG_AUTH:::storage
     ITEM_FEATURES_DB:::search
     PG_VIDEO:::storage
     SEARCH_INDEX_DB:::search
     SEARCH_APP:::search
     SEARCH_RANKER:::search
     ES:::storage
     RANKING_MODELS_DB:::search
     USERS_HISTORY_DB:::search
     RECENT_SEARCHES_DB:::search
     KAFKA:::queue
     ITEM_FEATURES_WORKER:::ml
     USER_ACTIVITY_WORKER:::ml
     RECENT_SEARCH_WORKER:::ml
     TRAFFIC_MARKER_WORKER:::ml
     SEARCH_INDEX_WORKER:::ml
     TRAIN_RANKER_WORKER:::ml
     DWH_DB:::search
     ITEM_SERVER:::service
     SAMPLE_RANKER_APP:::search
     PROMETHEUS:::monitoring
     GRAFANA:::monitoring
     n2:::service
     REC_ENGINE_RANKER:::recommendation
     REDIS_REC:::storage
    classDef external fill:#e1f5fe
    classDef loadbalancer fill:#f3e5f5
    classDef storage fill:#fff3e0
    classDef queue fill:#fce4ec
    classDef monitoring fill:#e8f5e8
    classDef ml fill:#e1f5fe
    classDef recommendation fill:#f3e5f5
    classDef search fill:#fff0f0
    classDef service fill:#e8f5e8
    style n2 stroke:#000000,fill:#C8E6C9

```

[1]: https://tass.ru/ekonomika/24311321 "Источник"
[2]: https://inclient.ru/rutube-stats/#rutube3 "Не уверен верить ли источнику"
[3]: https://www.similarweb.com/ru/website/rutube.ru/#demographics "Трафик по странам"
[5]: https://www.cnews.ru/news/line/2025-09-04_dnevnaya_auditoriya_rutube_vyrosla "Последняя статистика по рутубу"
[6]: https://affmaven.com/ru/youtube-statistics/ "Табличка статистики рутуба которую можем спроецировать на рутуб"
[7]: https://habr.com/ru/news/896842/ "388кк видео на рутубе"
[8]: ... "Источника нет, предположение нейронки"
[9]: https://skillbox.ru/media/marketing/mediascope-opublikovala-issledovanie-auditorii-sotsialnykh-media-youtube-poka-eshchye-lider/ "Инфа от медиаскоупа"
[10]: https://habr.com/ru/news/926262/
