Работу выполнили:
- Ардышев Кирилл Александрович, РИМ-140902
- Баранов Денис Кириллович, РИМ-140903
- Имаев Всеволод Владиславович, РИМ-140903

# Ход работы
## Подключение API
Для реализации мы зашли на 
https://console.cloud.google.com/apis/credentials?hl=ru&project=lateral-pathway-356216
Добваилили для своих аккаунтов API.
После подключили возможность обращаться к "YOUTUBE Data API" v3 в настройках API в console.cloud

## Эксперименты.


### Эксперимент 1: Получение статистики канала
**Цель:** Узнать, как с помощью API получить статистику канала, например количество подписчиков, общее количество видео и количество просмотров.

#### Код на Python:
```python
def get_channel_statistics(channel_id):
    url = f"https://www.googleapis.com/youtube/v3/channels"
    params = {
        "part": "statistics",
        "id": channel_id,
        "key": api_key
    }
    response = requests.get(url, params=params)
    return response.json()

# Получение статистики для канала OpenAI
channel_id = "UCLB7AzTwc6VFZrBsO2ucBMg"
result = get_channel_statistics(channel_id)
stats = result["items"][0]["statistics"]
print(f"Subscribers: {stats['subscriberCount']}, Views: {stats['viewCount']}, Videos: {stats['videoCount']}")

```

#### Запрос в Postman:
- **Method:** GET
- **URL:** `https://www.googleapis.com/youtube/v3/channels?part=statistics&id=UCLB7AzTwc6VFZrBsO2ucBMg&key=API_KEY`

Полученные данные из постмана: 
```
{
    "kind": "youtube#channelListResponse",
    "etag": "1uO_rWT5Ls9zsi0-YpxOXzcE52k",
    "pageInfo": {
        "totalResults": 1,
        "resultsPerPage": 5
    },
    "items": [
        {
            "kind": "youtube#channel",
            "etag": "vEPGlchKxzl4s3NBiF_C0R_V-bM",
            "id": "UCLB7AzTwc6VFZrBsO2ucBMg",
            "statistics": {
                "viewCount": "6870181",
                "subscriberCount": "157000",
                "hiddenSubscriberCount": false,
                "videoCount": "49"
            }
        }
    ]
}

```


#### Вывод:
С помощью этого запроса можно быстро получить основную статистику для анализа популярности и активности канала. Это полезно для маркетинговых исследований или сравнения роста каналов.
Так же мы видим, формат json выходных данных. 



### Эксперимент 2: Поиск видео по ключевым словам и фильтрация по дате
**Цель:** Изучить, как найти видео по ключевым словам, задав фильтрацию по дате публикации, чтобы получить самые новые или самые старые видео по заданной теме.

#### Код на Python:
```python
import requests

api_key = 'API_KEY'

def search_videos(query, order="date"):
    url = f"https://www.googleapis.com/youtube/v3/search"
    params = {
        "part": "snippet",
        "q": query,
        "maxResults": 5,
        "order": order,
        "key": api_key
    }
    response = requests.get(url, params=params)
    return response.json()

# Поиск видео по ключевому слову "Python programming"
result = search_videos("Python programming")
for item in result['items']:
    print(f"Video Title: {item['snippet']['title']}, Published at: {item['snippet']['publishedAt']}")

```

#### Запрос в Postman:
- **Method:** GET
- **URL:** `https://www.googleapis.com/youtube/v3/search?part=snippet&q=Python%20programming&maxResults=5&order=date&key=API_KEY`

#### Вывод:
Этот эксперимент помогает выявить, как с помощью фильтрации можно получить самые свежие видео по интересующей теме. Это полезно для мониторинга новых трендов или появления новых материалов по определенной тематике.


---


---

### Эксперимент 3: Получение комментариев к видео
**Цель:** Исследовать, как извлечь комментарии к конкретному видео, чтобы проанализировать аудиторию и её реакцию на видео.

#### Код на Python:
```python
def get_video_comments(video_id, max_results=5):
    url = f"https://www.googleapis.com/youtube/v3/commentThreads"
    params = {
        "part": "snippet",
        "videoId": video_id,
        "maxResults": max_results,
        "order": "relevance",
        "key": api_key
    }
    response = requests.get(url, params=params)
    return response.json()

# Получение комментариев к видео
video_id = "dQw4w9WgXcQ"
comments = get_video_comments(video_id)
for comment in comments['items']:
    print(f"Comment: {comment['snippet']['topLevelComment']['snippet']['textDisplay']}")

```

#### Запрос в Postman:
- **Method:** GET
- **URL:** `https://www.googleapis.com/youtube/v3/commentThreads?part=snippet&videoId=Ks-_Mh1QhMc&maxResults=5&order=relevance&key=API_KEY`

#### Вывод:
Получение комментариев позволяет лучше понять реакцию зрителей на видео и изучить взаимодействие аудитории. Анализ текстов комментариев может быть полезен для оценки популярности контента и выявления ключевых интересов зрителей.


### Эксперимент 4: Количество видео содержащих слово
**Цель:** Исследовать, как извлечь комментарии к конкретному видео, чтобы проанализировать аудиторию и её реакцию на видео.

#### Код на Python:
```python
import datetime

def get_video_count(query, published_after):
    url = f"https://www.googleapis.com/youtube/v3/search"
    params = {
        "q": query,
        "part": "id",
        "type": "video",
        "publishedAfter": published_after,
        "maxResults": 50,
        "key": api_key
    }
    response = requests.get(url, params=params)
    data = response.json()
    total_results = data['pageInfo']['totalResults']
    return total_results

# Получаем дату 30 дней назад в формате ISO
days_ago_30 = (datetime.datetime.now() - datetime.timedelta(days=30)).isoformat("T") + "Z"

queries = ["backend", "frontend", "data science", "devops", "machine learning"]
results = {}

for query in queries:
    try:
        total_results = get_video_count(query, days_ago_30)
        results[query] = total_results
        print(f"Примерное количество видео по теме '{query}' за последние 30 дней: {total_results}")
    except Exception as e:
        print(f"Ошибка для запроса '{query}': {e}")


```

output 
```
Тематика: backend, Примерное количество видео за последние 30 дней: 548433
Тематика: frontend, Примерное количество видео за последние 30 дней: 113295
Тематика: data science, Примерное количество видео за последние 30 дней: 1000000
Тематика: devops, Примерное количество видео за последние 30 дней: 46419
Тематика: machine learning, Примерное количество видео за последние 30 дней: 1000000
```



#### Запрос в Postman:
- **Method:** GET
- **URL:** `https://www.googleapis.com/youtube/v3/search?part=id&q=backend&type=video&publishedAfter=2024-10-15T00:00:00Z&maxResults=50&key=YOUR_API_KEY`



#### Вывод:
 Полученные данные позволяют оценить популярность определённых тем и ключевых слов. Так можно определить, какие темы вызывают интерес и где существует большой объем контента, а также найти ниши с меньшим количеством контента. 
Так же видно, что за последние 30 дней темы о искусственном ителлекте очень популярны. В УрФУ мы поступили правильно. 


---

### Общие выводы
Эти эксперименты показывают широкие возможности YouTube API для работы с видеоконтентом, метаданными и взаимодействием с аудиторией. Получение данных по запросу, статистике и комментариям открывает возможности для аналитики и глубокого изучения трендов, интересов аудитории и популярности контента.
