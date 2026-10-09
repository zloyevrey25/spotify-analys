# Курсовой проект по дисциплине «Технологии обработки больших данных»

**Тема:** «Исследование предпочтений в музыке на основе потоковых и текстовых данных»  
**Группа:** ИД24-1  
**Этап:** 2.2. Подбор и изучение источников данных

В этом разделе подбираются открытые источники данных, которые будут использоваться дальше для подготовки, объединения и анализа данных.

По методическим указаниям необходимо использовать:
- не менее одного реляционного набора данных;
- не менее одного иерархического набора данных в формате XML;
- не менее одного API с выдачей в JSON;
- не менее одного источника неструктурированных данных.

Для каждого используемого набора должны быть выделены не менее трех категориальных и трех числовых характеристик. Желательно наличие временного признака.



## 1. Итоговый набор источников

Для проекта выбраны четыре основных источника.

| № | Источник | Формат / тип | Что описывает | Роль в работе |
|---|---|---|---|---|
| 1 | **Spotify Top 200 Dataset** | CSV, реляционный набор | треки, исполнителей, позиции в чарте, стримы и аудиопризнаки | основной количественный источник |
| 2 | **MusicBrainz Web Service** | XML, иерархические данные | метаданные исполнителей, жанры, рейтинги и релизы | обогащение метаданных |
| 3 | **Last.fm API** | JSON API | статистику по трекам: слушатели, проигрывания, длительность, теги | дополнительная оценка интереса аудитории |
| 4 | **The Guardian Open Platform** | неструктурированный текст | статьи о музыке и исполнителях | текстовые признаки и внешний информационный фон |

Такой набор позволяет объединить потоковые показатели, характеристики треков, метаданные исполнителей и текстовую информацию.

### Ссылки на источники

1. Spotify Top 200 Dataset:  
   https://github.com/younver/spotify-top-200-dataset

2. MusicBrainz Web Service:  
   https://musicbrainz.org/doc/MusicBrainz_API

3. Last.fm API, метод `track.getInfo`:  
   https://www.last.fm/api/show/track.getInfo

4. The Guardian Open Platform:  
   https://open-platform.theguardian.com/

Дата проверки доступности источников: **09.10.2026**.



## Перед запуском

Для двух источников потребуются бесплатные API-ключи:

- `LASTFM_API_KEY` — Last.fm;
- `GUARDIAN_API_KEY` — The Guardian Open Platform.

Ключи не следует сохранять в публичном ноутбуке. На время запуска можно добавить их отдельной временной ячейкой:

```python
import os
os.environ["LASTFM_API_KEY"] = "ВАШ_КЛЮЧ"
os.environ["GUARDIAN_API_KEY"] = "ВАШ_КЛЮЧ"
```

После этого ячейки Last.fm и The Guardian можно запустить повторно.



## 2. Реляционный набор данных — Spotify Top 200 Dataset

Основным табличным источником выбран открытый набор **Spotify Top 200 Dataset**. Он содержит недельные данные о глобальном Top-200 Spotify за 2017–2021 годы.

В наборе около **74 тыс. строк и 40 столбцов**. Одна запись описывает трек и исполнителя в определенную неделю. В данных присутствуют показатели стриминга, позиции в чарте, популярность исполнителя и трека, данные об альбоме, жанре, а также аудиохарактеристики.

Этот набор удобнее для проекта, чем первоначально рассмотренный *Spotify Artist Streaming Analytics (2015–2025)*, потому что здесь есть повторные наблюдения по времени (`week`) и показатель `streams`. Поэтому по нему можно изучать не только различия между артистами, но и изменение популярности во времени.

### Выбранные категориальные характеристики

| Характеристика | Смысл |
|---|---|
| `track_name` | название композиции |
| `artist_name` | исполнитель |
| `artist_genres` | жанры исполнителя |
| `album_type` | тип релиза: album, single и др. |
| `album_label` | лейбл |
| `explicit` | наличие explicit-контента |

### Выбранные числовые характеристики

| Характеристика | Смысл |
|---|---|
| `streams` | число прослушиваний за неделю |
| `rank` | место композиции в чарте |
| `track_popularity` | показатель популярности трека |
| `artist_popularity` | показатель популярности исполнителя |
| `artist_followers` | число подписчиков исполнителя |
| `danceability` | танцевальность |
| `energy` | энергичность |
| `valence` | эмоциональная позитивность |
| `tempo` | темп композиции |
| `duration` | длительность |

### Временные характеристики

- `week` — неделя нахождения в чарте;
- `release_date` — дата релиза.

**Значимость для гипотезы:** `streams`, `rank` и показатели популярности будут использоваться как основные характеристики интереса аудитории. Аудиопризнаки, жанр, тип релиза и характеристики исполнителя будут рассматриваться как возможные факторы, связанные с популярностью.



```python

# Загрузка основного реляционного набора данных и вывод первых 5 строк

import pandas as pd

SPOTIFY_CSV = (
    "https://raw.githubusercontent.com/younver/"
    "spotify-top-200-dataset/main/spotify-top-200-dataset.csv"
)

spotify_df = pd.read_csv(SPOTIFY_CSV, sep=";")

print("Размер датасета:", spotify_df.shape)
display(spotify_df.head(5))

```

    Размер датасета: (74660, 40)



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>track_id</th>
      <th>track_name</th>
      <th>track_popularity</th>
      <th>track_number</th>
      <th>album_id</th>
      <th>album_name</th>
      <th>album_img</th>
      <th>album_type</th>
      <th>album_label</th>
      <th>album_track_number</th>
      <th>...</th>
      <th>speechiness</th>
      <th>acousticness</th>
      <th>instrumentalness</th>
      <th>liveness</th>
      <th>valence</th>
      <th>tempo</th>
      <th>duration</th>
      <th>pivot</th>
      <th>streams</th>
      <th>track_index</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5aAx2yezTd8zXrkmtKl66Z</td>
      <td>Starboy</td>
      <td>0</td>
      <td>1</td>
      <td>09fggMHib4YkOtwQNXEBII</td>
      <td>Starboy</td>
      <td>https://i.scdn.co/image/ab67616d0000b2730c8599...</td>
      <td>album</td>
      <td>Universal Music Group</td>
      <td>1</td>
      <td>...</td>
      <td>0.2820</td>
      <td>0.165</td>
      <td>0.000003</td>
      <td>0.134</td>
      <td>0.535</td>
      <td>186.054</td>
      <td>230453</td>
      <td>0</td>
      <td>25734078</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5aAx2yezTd8zXrkmtKl66Z</td>
      <td>Starboy</td>
      <td>0</td>
      <td>1</td>
      <td>09fggMHib4YkOtwQNXEBII</td>
      <td>Starboy</td>
      <td>https://i.scdn.co/image/ab67616d0000b2730c8599...</td>
      <td>album</td>
      <td>Universal Music Group</td>
      <td>1</td>
      <td>...</td>
      <td>0.2820</td>
      <td>0.165</td>
      <td>0.000003</td>
      <td>0.134</td>
      <td>0.535</td>
      <td>186.054</td>
      <td>230453</td>
      <td>1</td>
      <td>25734078</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7BKLCZ1jbUBVqRi2FVlTVw</td>
      <td>Closer</td>
      <td>84</td>
      <td>1</td>
      <td>0rSLgV8p5FzfnqlEk4GzxE</td>
      <td>Closer</td>
      <td>https://i.scdn.co/image/ab67616d0000b273495ce6...</td>
      <td>single</td>
      <td>Disruptor Records/Columbia</td>
      <td>1</td>
      <td>...</td>
      <td>0.0338</td>
      <td>0.414</td>
      <td>0.000000</td>
      <td>0.111</td>
      <td>0.661</td>
      <td>95.010</td>
      <td>244960</td>
      <td>0</td>
      <td>23519705</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7BKLCZ1jbUBVqRi2FVlTVw</td>
      <td>Closer</td>
      <td>84</td>
      <td>1</td>
      <td>0rSLgV8p5FzfnqlEk4GzxE</td>
      <td>Closer</td>
      <td>https://i.scdn.co/image/ab67616d0000b273495ce6...</td>
      <td>single</td>
      <td>Disruptor Records/Columbia</td>
      <td>1</td>
      <td>...</td>
      <td>0.0338</td>
      <td>0.414</td>
      <td>0.000000</td>
      <td>0.111</td>
      <td>0.661</td>
      <td>95.010</td>
      <td>244960</td>
      <td>1</td>
      <td>23519705</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5knuzwU65gJK7IF5yJsuaW</td>
      <td>Rockabye (feat. Sean Paul &amp; Anne-Marie)</td>
      <td>75</td>
      <td>1</td>
      <td>3meZFplbMmji648oWUNEfQ</td>
      <td>Rockabye (feat. Sean Paul &amp; Anne-Marie)</td>
      <td>https://i.scdn.co/image/ab67616d0000b2731431c3...</td>
      <td>single</td>
      <td>Atlantic Records UK</td>
      <td>1</td>
      <td>...</td>
      <td>0.0523</td>
      <td>0.406</td>
      <td>0.000000</td>
      <td>0.180</td>
      <td>0.742</td>
      <td>101.965</td>
      <td>251088</td>
      <td>0</td>
      <td>21216399</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 40 columns</p>
</div>


**Вывод.** Источник загружается как табличный набор данных. Первые 5 строк позволяют проверить структуру, названия столбцов и типы признаков, которые будут использоваться в дальнейшем анализе.


## 3. Иерархический источник — MusicBrainz Web Service (XML)

Для выполнения требования по иерархическим данным выбран **MusicBrainz**. Web Service возвращает XML по умолчанию и не требует API-ключа для чтения. Для запросов необходимо указывать корректный `User-Agent` и соблюдать ограничение по частоте запросов.

Объектом в этом источнике будет **исполнитель**. Для исполнителей из основного Spotify-набора будут получаться XML-данные MusicBrainz.

Планируется использовать запросы с дополнительными блоками `aliases`, `genres`, `tags`, `ratings` и `release-groups`.

### Категориальные характеристики

| Характеристика | Смысл |
|---|---|
| `name` | имя исполнителя |
| `type` | тип исполнителя: Person, Group и др. |
| `gender` | пол для персональных исполнителей |
| `country` | страна |
| `genres` | жанры MusicBrainz |

### Числовые характеристики

Часть числовых признаков берется из XML напрямую, часть получается как количество элементов иерархического блока.

| Характеристика | Смысл |
|---|---|
| `rating_value` | средняя пользовательская оценка |
| `rating_votes` | число голосов |
| `release_group_count` | количество групп релизов |
| `genre_count` | количество жанров |
| `tag_count` | количество тегов |
| `alias_count` | количество альтернативных имен |

### Временные характеристики

- начало периода активности исполнителя (`life-span/begin`);
- окончание периода активности (`life-span/end`), если оно указано.

**Значимость для гипотезы:** MusicBrainz дополняет потоковые данные сведениями о жанре, типе и происхождении исполнителя. Эти характеристики можно использовать как факторы при анализе различий в популярности.



```python

# Получение 5 объектов из MusicBrainz в формате XML
# и преобразование XML в таблицу для наглядного просмотра

import requests
import xml.etree.ElementTree as ET
import pandas as pd

url = "https://musicbrainz.org/ws/2/artist/"
params = {
    "query": 'artist:"Taylor Swift"',
    "limit": 5
}

headers = {
    "User-Agent": "TOBD-course-project/1.0 (student educational project)"
}

response = requests.get(url, params=params, headers=headers, timeout=30)
response.raise_for_status()

print("Первые 1000 символов исходного XML:")
print(response.text[:1000])

root = ET.fromstring(response.text)

ns = {
    "mb": "http://musicbrainz.org/ns/mmd-2.0#"
}

artists = []

for artist in root.findall(".//mb:artist", ns)[:5]:
    aliases = artist.findall("mb:alias-list/mb:alias", ns)
    tags = artist.findall("mb:tag-list/mb:tag", ns)

    artists.append({
        "id": artist.attrib.get("id", ""),
        "type": artist.attrib.get("type", ""),
        "name": artist.findtext("mb:name", default="", namespaces=ns),
        "country": artist.findtext("mb:country", default="", namespaces=ns),
        "alias_count": len(aliases),
        "tag_count": len(tags),
        "score": artist.attrib.get(
            "{http://musicbrainz.org/ns/ext#-2.0}score", ""
        )
    })

musicbrainz_df = pd.DataFrame(artists)

print("\nПервые 5 объектов после разбора XML:")
display(musicbrainz_df.head(5))

```

    /Users/Arina/Library/Python/3.9/lib/python/site-packages/urllib3/__init__.py:35: NotOpenSSLWarning: urllib3 v2 only supports OpenSSL 1.1.1+, currently the 'ssl' module is compiled with 'LibreSSL 2.8.3'. See: https://github.com/urllib3/urllib3/issues/3020
      warnings.warn(


    Первые 1000 символов исходного XML:
    <?xml version="1.0" encoding="UTF-8" standalone="yes"?><metadata created="2026-10-09T05:58:31.859Z" xmlns="http://musicbrainz.org/ns/mmd-2.0#" xmlns:ns2="http://musicbrainz.org/ns/ext#-2.0"><artist-list count="5" offset="0"><artist id="20244d07-534f-4eff-b4d4-930878889970" type="Person" type-id="b6e035f4-3ce9-331c-97df-83397230b0df" ns2:score="100"><name>Taylor Swift</name><sort-name>Swift, Taylor</sort-name><gender id="93452b5a-a947-30c8-934f-6a4056b151c2">female</gender><country>US</country><area id="489ce91b-6658-3307-9877-795b68554c98" type="Country" type-id="06dd0ae4-8c74-30bb-b43d-95dcedf961de"><name>United States</name><sort-name>United States</sort-name><life-span><ended>false</ended></life-span></area><begin-area id="bfb8ed5a-7342-4a01-a1f2-94e620c3416b" type="City" type-id="6fd8f29a-3d0a-32fc-980d-ea697b69da78"><name>West Reading</name><sort-name>West Reading</sort-name><life-span><ended>false</ended></life-span></begin-area><ipi-list><ipi>00454808047</ipi><ipi>00454808145</i
    
    Первые 5 объектов после разбора XML:



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>type</th>
      <th>name</th>
      <th>country</th>
      <th>alias_count</th>
      <th>tag_count</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20244d07-534f-4eff-b4d4-930878889970</td>
      <td>Person</td>
      <td>Taylor Swift</td>
      <td>US</td>
      <td>57</td>
      <td>37</td>
      <td>100</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4fb7be42-860c-401c-8ec3-0163a93018ff</td>
      <td>Person</td>
      <td>Taylor Swift</td>
      <td></td>
      <td>0</td>
      <td>0</td>
      <td>63</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0ea4e9a4-4da5-45c3-a1fe-a6d9461a157b</td>
      <td>Group</td>
      <td>Fearless: The Taylor Swift Experience</td>
      <td></td>
      <td>0</td>
      <td>0</td>
      <td>52</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a26ca924-c7ba-4ab3-92e0-929145382fca</td>
      <td>Group</td>
      <td>Almost Eras: The Taylor Swift Experience</td>
      <td></td>
      <td>0</td>
      <td>0</td>
      <td>50</td>
    </tr>
    <tr>
      <th>4</th>
      <td>790624d8-8c6e-4163-a920-297d3debaa35</td>
      <td>Group</td>
      <td>Rikki Lee Wilson - Love Story - Taylor Swift t...</td>
      <td></td>
      <td>0</td>
      <td>0</td>
      <td>44</td>
    </tr>
  </tbody>
</table>
</div>


**Вывод.** MusicBrainz возвращает иерархический XML. Для проверки формата выше выводится фрагмент исходного XML, а первые 5 объектов дополнительно представлены в табличном виде.


## 4. API с выдачей JSON — Last.fm API

Для источника с API в формате JSON выбран **Last.fm**, метод `track.getInfo`.

Он подходит для проекта, потому что объектом является музыкальный трек и его можно объединять с основным Spotify-набором по паре **исполнитель + название трека**.

Для метода необходим бесплатный API key, но пользовательская авторизация не требуется.

### Категориальные характеристики

| Характеристика | Смысл |
|---|---|
| `name` | название трека |
| `artist.name` | исполнитель |
| `album.title` | альбом |
| `toptags` | пользовательские теги / жанровые обозначения |
| `mbid` | идентификатор MusicBrainz |

### Числовые характеристики

| Характеристика | Смысл |
|---|---|
| `duration` | длительность трека в миллисекундах |
| `listeners` | число слушателей |
| `playcount` | число проигрываний |
| `album.position` | позиция трека в альбоме, если указана |

**Значимость для гипотезы:** Last.fm дает дополнительную оценку интереса аудитории, независимую от показателя `streams` в основном наборе. Теги также можно использовать для дополнительной категоризации музыкального контента.



```python

# Получение 5 треков из Last.fm в формате JSON
# Перед запуском задайте бесплатный API-ключ Last.fm.

import os
import requests
import pandas as pd

LASTFM_API_KEY = os.getenv("LASTFM_API_KEY", "")

if not LASTFM_API_KEY:
    print(
        "Не задан LASTFM_API_KEY.\n"
        "Сначала выполните временную ячейку:\n"
        'import os\nos.environ["LASTFM_API_KEY"] = "ВАШ_КЛЮЧ"'
    )
else:
    params = {
        "method": "chart.getTopTracks",
        "api_key": LASTFM_API_KEY,
        "format": "json",
        "limit": 5
    }

    response = requests.get(
        "https://ws.audioscrobbler.com/2.0/",
        params=params,
        timeout=30
    )
    response.raise_for_status()

    data = response.json()
    tracks = data["tracks"]["track"]

    lastfm_df = pd.DataFrame([
        {
            "track": track.get("name", ""),
            "artist": track.get("artist", {}).get("name", ""),
            "listeners": int(track.get("listeners", 0)),
            "playcount": int(track.get("playcount", 0)),
            "mbid": track.get("mbid", "")
        }
        for track in tracks
    ])

    print("Первые 5 объектов Last.fm:")
    display(lastfm_df.head(5))

```

    Не задан LASTFM_API_KEY.
    Сначала выполните временную ячейку:
    import os
    os.environ["LASTFM_API_KEY"] = "ВАШ_КЛЮЧ"


**Вывод.** Last.fm используется как API с выдачей JSON. В демонстрационной ячейке выводятся 5 треков и основные числовые показатели `listeners` и `playcount`. API-ключ не хранится в публичном ноутбуке.


## 5. Неструктурированные данные — The Guardian Open Platform

В качестве текстового источника выбраны музыкальные статьи **The Guardian** через Open Platform.

Developer-доступ предназначен в том числе для учебных и некоммерческих проектов. Для работы нужен бесплатный API key. Через API можно получать полный текст статьи.

Для проекта будут запрашиваться статьи из раздела `music` по исполнителям, присутствующим в основном наборе. Период текстовых материалов можно ограничить 2015–2025 годами.

Объектом является **статья**.

### Категориальные характеристики

| Характеристика | Смысл |
|---|---|
| `webTitle` | заголовок статьи |
| `sectionName` | раздел |
| `type` | тип материала |
| `byline` | автор |
| `tags` | тематические теги |

### Числовые характеристики

Для неструктурированного текста числовые признаки будут сформированы на этапе аннотирования и подготовки данных, что соответствует требованиям методических указаний.

| Характеристика | Смысл |
|---|---|
| `wordcount` | число слов, возвращаемое API при наличии поля |
| `text_length` | длина текста в символах |
| `keyword_count` | количество тематических тегов / ключевых слов |
| `artist_mention_count` | число упоминаний анализируемого исполнителя |
| `sentiment_score` | оценка тональности текста после аннотирования |

### Временная характеристика

- `webPublicationDate` — дата публикации статьи.

**Значимость для гипотезы:** статьи дают внешний текстовый контекст вокруг исполнителя. После очистки и векторизации можно проверить, связаны ли текстовые признаки и интенсивность медийного внимания с показателями популярности.



```python

# Получение 5 музыкальных статей The Guardian
# Перед запуском задайте бесплатный Developer API key.

import os
import requests
import pandas as pd

GUARDIAN_API_KEY = os.getenv("GUARDIAN_API_KEY", "")

if not GUARDIAN_API_KEY:
    print(
        "Не задан GUARDIAN_API_KEY.\n"
        "Сначала выполните временную ячейку:\n"
        'import os\nos.environ["GUARDIAN_API_KEY"] = "ВАШ_КЛЮЧ"'
    )
else:
    params = {
        "section": "music",
        "from-date": "2015-01-01",
        "to-date": "2025-12-31",
        "page-size": 5,
        "show-fields": "headline,byline,bodyText,wordcount",
        "show-tags": "keyword",
        "api-key": GUARDIAN_API_KEY
    }

    response = requests.get(
        "https://content.guardianapis.com/search",
        params=params,
        timeout=30
    )
    response.raise_for_status()

    results = response.json()["response"]["results"]

    guardian_df = pd.DataFrame([
        {
            "title": item.get("webTitle", ""),
            "date": item.get("webPublicationDate", ""),
            "section": item.get("sectionName", ""),
            "type": item.get("type", ""),
            "author": item.get("fields", {}).get("byline", ""),
            "wordcount": item.get("fields", {}).get("wordcount", ""),
            "text_preview": item.get("fields", {}).get("bodyText", "")[:200]
        }
        for item in results
    ])

    print("Первые 5 статей The Guardian:")
    display(guardian_df.head(5))

```

    Не задан GUARDIAN_API_KEY.
    Сначала выполните временную ячейку:
    import os
    os.environ["GUARDIAN_API_KEY"] = "ВАШ_КЛЮЧ"


**Вывод.** The Guardian используется как источник неструктурированных текстовых данных. Для проверки выводятся 5 статей с метаданными и фрагментом текста. Полный текст будет использован на следующем этапе для очистки, аннотирования и векторизации.


## 6. Проверка соответствия источников требованиям

| Источник | Множество объектов | ≥3 категориальных | ≥3 числовых | Время | Требование |
|---|---:|---:|---:|---:|---|
| Spotify Top 200 CSV | да | да | да | да | реляционный набор |
| MusicBrainz XML | да | да | да | да | иерархический XML |
| Last.fm API | да | да | да | частично | API JSON |
| The Guardian | да | да | да, после аннотирования текста | да | неструктурированный текст |

Таким образом, все четыре обязательных типа источников закрыты.

Для текстового источника часть числовых характеристик формируется при подготовке данных (`text_length`, число тегов, число упоминаний, sentiment). Это необходимо, потому что исходный объект представляет собой неструктурированный текст, а методические указания отдельно требуют его аннотирования и векторизации.



## 7. План объединения данных

Основным уровнем анализа будет **трек**, а при работе с текстами дополнительно будет использоваться уровень **исполнителя**.

План слияния:

1. **Spotify Top 200 + Last.fm**  
   Общие атрибуты: нормализованные `artist_name` + `track_name`.  
   Результат: к потоковым данным добавляются `listeners`, `playcount`, `duration` и Last.fm-теги.

2. **Spotify Top 200 + MusicBrainz**  
   Общий атрибут: исполнитель.  
   Сначала по имени исполнителя определяется MusicBrainz ID, после чего добавляются жанр, страна, тип исполнителя, рейтинг и сведения о релизах.

3. **Spotify + The Guardian**  
   Статьи ищутся по имени исполнителя. Затем статьи агрегируются до уровня исполнителя: число публикаций, средняя тональность, суммарное число слов, среднее число упоминаний и другие текстовые признаки.

4. Перед слиянием названия исполнителей и треков будут нормализованы: перевод в единый регистр, удаление лишних пробелов и технических символов. Для неоднозначных совпадений будут дополнительно использоваться MBID и другие идентификаторы, если они доступны.

Такой подход позволяет связать внутренние показатели музыкального контента с внешними факторами: пользовательским интересом Last.fm, метаданными MusicBrainz и текстовым информационным фоном.



## 8. Аналитический портрет объекта исследования

В объединенном наборе планируется использовать следующие группы признаков.

### Показатели состояния объекта
- число стримов;
- позиция в чарте;
- популярность трека;
- популярность исполнителя;
- число слушателей и проигрываний Last.fm.

### Факторы, которые могут быть связаны с популярностью
- жанр;
- тип релиза;
- наличие коллаборации;
- explicit-контент;
- число подписчиков исполнителя;
- danceability;
- energy;
- valence;
- tempo;
- длительность;
- страна и тип исполнителя;
- количество публикаций в медиа;
- тональность текстовых материалов;
- текстовые признаки после TF-IDF / векторизации.

### Временные признаки
- неделя нахождения в чарте;
- дата релиза;
- дата публикации статьи;
- начало периода активности исполнителя.

Выбранные признаки позволяют проверить гипотезу о связи популярности музыкального контента с потоковыми, содержательными и текстовыми характеристиками.



## 9. Почему первоначальные источники были скорректированы

На первом этапе рассматривались Spotify Web API, Last.fm API, Billboard, Million Song Dataset и Kaggle-набор *Spotify Artist Streaming Analytics (2015–2025)*.

После проверки требований список был изменен:

- **Spotify Web API** не используется как обязательный источник: для проекта уже есть открытый табличный набор с нужными потоковыми и аудиопризнаками, поэтому дополнительная авторизация Spotify только усложнила бы сбор данных.
- **Billboard scraping** не выбран как основной источник, так как структура сайта может изменяться, а воспроизводимость парсинга ниже, чем у API.
- **Million Song Dataset** не используется, поскольку он значительно тяжелее по объему и формату хранения, но не нужен для закрытия обязательных типов источников.
- **Spotify Artist Streaming Analytics (2015–2025)** оставлен как возможный резервный источник. По доступному описанию он хорошо подходит для сравнения исполнителей, но содержит агрегированные показатели и хуже подходит для анализа изменения стримов по времени.

В результате выбран набор источников, который проще воспроизводится и лучше подходит для последующего слияния и проверки гипотезы.



## 10. Вывод по этапу 2.2

Для исследования выбраны четыре открытых источника разной структуры: реляционный CSV, XML, JSON API и неструктурированный текст.

Основным источником является Spotify Top 200 Dataset, так как он содержит потоковые показатели, временной признак и большое количество характеристик треков и исполнителей. MusicBrainz будет использоваться для обогащения метаданных, Last.fm — для получения дополнительной статистики интереса аудитории, а The Guardian — для формирования текстовых признаков.

Все выбранные источники описывают множество музыкальных объектов и позволяют выделить не менее трех категориальных и трех числовых характеристик. На следующем этапе данные будут загружены, проверены на пропуски и дубликаты, очищены, приведены к общей структуре и объединены.

