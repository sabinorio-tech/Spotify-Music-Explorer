# Spotify Music Explorer

Spotify Music Explorer is a small Elasticsearch project that indexes Spotify track data and explores how search, filtering, and aggregations work on music metadata and audio features.

The project is based on the Spotify Tracks dataset from Kaggle.

## Project Goal

The goal is to build a searchable music index where tracks can be explored by:

- song title
- artist
- album
- genre
- popularity
- audio features such as danceability, energy, tempo, acousticness, valence, and more

## Current Features

- Loads the Spotify dataset with pandas.
- Cleans the dataset by removing unused columns and missing values.
- Creates an Elasticsearch index called `spotify_tracks`.
- Defines an explicit Elasticsearch mapping for text, keyword, integer, boolean, and float fields.
- Bulk indexes the dataset into Elasticsearch.
- Uses `track_id` as the Elasticsearch document `_id`, so duplicate Spotify track IDs are overwritten instead of duplicated.
- Searches tracks by title.
- Searches tracks by artist.
- Searches tracks by genre.
- Filters tracks by genre and popularity.
- Filters artist results by popularity.
- Runs album aggregations for an artist.
- Calculates average popularity per album using Elasticsearch aggregations.

## Dataset Fields Used

The current indexed documents include:

- `track_id`
- `track_name`
- `artists`
- `album_name`
- `track_genre`
- `popularity`
- `duration_ms`
- `explicit`
- `danceability`
- `energy`
- `key`
- `loudness`
- `mode`
- `speechiness`
- `acousticness`
- `instrumentalness`
- `liveness`
- `valence`
- `tempo`

The original must-have idea included filtering by year. The dataset currently used in this project does not include a `year` column, so year filtering is not implemented yet.

## Elasticsearch Index

Index name:

```text
spotify_tracks
```

The notebook uses an `INDEX_NAME` variable so all indexing, counting, and searching target the same index.

There is also an important distinction between:

```text
spotify_tracks
spotify-tracks
```

The project uses `spotify_tracks` with an underscore.

## Search Examples

Search by track title:

```python
query = {
    "match": {
        "track_name": "Comedy"
    }
}

client.search(index=INDEX_NAME, query=query)
```

Search by artist:

```python
query = {
    "match": {
        "artists": "BTS"
    }
}

client.search(index=INDEX_NAME, query=query)
```

Search by genre:

```python
query = {
    "match": {
        "track_genre": "k-pop"
    }
}

client.search(index=INDEX_NAME, query=query)
```

## Filtering Example

Example: find BTS tracks in the `k-pop` genre with popularity of at least 75.

```python
query = {
    "bool": {
        "must": [
            {"match": {"artists": "BTS"}}
        ],
        "filter": [
            {"term": {"track_genre": "k-pop"}},
            {"range": {"popularity": {"gte": 75}}}
        ]
    }
}

client.search(index=INDEX_NAME, query=query)
```

## Aggregation Example

The project also explores aggregations. For example, it groups The Weeknd tracks by album and calculates average popularity per album.

```python
aggs = {
    "albums": {
        "terms": {
            "field": "album_name.keyword",
            "order": {"avg_popularity": "desc"},
        },
        "aggs": {
            "avg_popularity": {
                "avg": {
                    "field": "popularity"
                }
            }
        }
    }
}
```

To support this, `album_name` is mapped as both text and keyword:

```python
"album_name": {
    "type": "text",
    "fields": {
        "keyword": {"type": "keyword"}
    }
}
```

## Project Structure

```text
.
├── data/
│   ├── raw/
│   │   └── dataset.csv
│   ├── processed/
│   └── samples/
├── notebooks/
│   └── Exploration.ipynb
├── src/
│   ├── analytics/
│   ├── config/
│   ├── indexing/
│   ├── ingestion/
│   ├── search/
│   └── utils/
├── tests/
├── main.py
├── requirements.txt
└── README.md
```

Most of the current work is in `notebooks/Exploration.ipynb`. The `src/` folders are prepared for future refactoring if the notebook code is later moved into reusable Python modules.

## Setup

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Start Elasticsearch locally on port `9200`, then open the notebook:

```bash
jupyter notebook
```

## Notes

- The notebook uses the Elasticsearch 8.x Python client.
- Bulk indexing should capture the return values from `bulk()` so indexing errors are visible.
- A refresh is needed after bulk indexing before newly indexed documents are guaranteed to appear in search.
- The current index count is lower than the cleaned dataframe row count because duplicate `track_id` values overwrite existing documents when used as `_id`.
