# Documentation for Term Project 1
**Course: Data Engineering 1**

**Program: Business Analytics MS**

**Author: Márton Nagy**

## Executive summary

The `spotify` [database](#data), sourced from [Kaggle](https://www.kaggle.com/datasets/tonygordonjr/spotify-dataset-2023), holds data on a number of tracks, albums, artists and genres available on Spotify. I have performed some normalization tasks on the original database to end up with the following database structure:

![The normalized structure of the spotify database, EER graph](/Term1/assets/OLTP_structure_normalized.png)

If you import the provided [SQL dump](/Term1/normalized_data_dump.zip) file, you should end up with this structure, populated with data. Alternatively, you can create the database structure from scratch and populate it with data from the raw files, then perform the normalization tasks, all using the provided [SQL script](/Term1/MartonNagy_term1_from_scratch.sql).

The aim of the project was to provide data that may be used to answer a set of [12 analytical questions](#analytical-questions). To do so, first, 4 [data warehouses](#data-warehouses) have been created by separate ETL pipelines. The warehouses relate to facts on albums, tracks, artists and genres. Then, from these warehouses tailor-made [data marts](#data-marts) have been created as views to provide data related to each question.

To maintain the up-to-dateness of data warehouses, a few triggers and a scheduled event have been [implemented](#extra-features-triggers-events-and-materialized-views) in my solution. In addition, some views have also been implemented in a materialized form (with a scheduled daily update).

All the above functionalites have been implemented in a single [SQL script](/Term1/MartonNagy_term1_ETLs.sql) (pre-supposing having loaded the database).

### Quick reproduction guide

You can create the normalized database in two ways: either download the raw files and paste them into your MySQL Uploads folder to import and normalize the raw CSVs, or you can import the normalized schema by running the dump file.

#### Option 1: Importing from CSVs

1. Download and unzip the raw [CSV files](/Term1/spotify_raw_data.zip) from this repository.
2. Paste the unzipped CSVs into you MySQL Uploads folder.
3. Download the [`MartonNagy_term1_from_scratch.sql`](/Term1/MartonNagy_term1_from_scratch.sql) SQL script and open it in MySQL Workbench.
4. **Modify the `load data` statements so that the path points to your specific MySQL Uploads folder**.
5. Run the whole script (might take up to 30 minutes depending on you set-up).
6. You should end up with the normalized `spotify` database.

#### Option 2: Importing from the dump file

1. Download and unzip the [`normalized_data_dump.zip`](/Term1/normalized_data_dump.zip) file.
2. Open it in MySQL Workbench.
3. Run the whole script (this also might take a good 15 minutes).
4. You should end up with the normalized `spotify` database.

From this point on, you can follow this roadmap:

1. Download the [`MartonNagy_term1_ETLs.sql`](/Term1/MartonNagy_term1_ETLs.sql) SQL script.
2. Open it in MySQL Workbench.
3. Run the whole script.
4. You should end up with the appropriate data warehouses, data marts, as well as some extre features: triggers, events, and some materialized views.
5. Download the [`views_chart_generator.py`](views_chart_generator.py) Python script.
6. Please verify that the following packages are installed in your environment: `pymysql`, `seaborn`, `matplotlib`, `mpl_toolkits`, `pandas`, `numpy`, `warnings`, `textwrap`, `os`, `getpass`.
   * If you are using Anaconda, everything other than `mysql.connector` should be installed by default.
   * If something is missing, please install it before running the script.
7. Verify that your MySQL local server is running.
   * If you have created the database in a remote server, please adjust the following part of code accordingly:
```python
spotify = mysql.connector.connect(
host='localhost',
user= username,    # your MySQL username
password= password, # your MySQL password
database='spotify'
```
9. Execute the script from a location that is convenient for you.
10. You should end with charts and tables (12 in total) in a `charts` directory in the location from where you have executed the script. Each chart is named `qX.png`, where `X` stands for the number of question the chart is related to.

## Introduction

Spotify holds a vast array of quantitative data on all the tracks, albums and artists uploaded there (e.g. valence of every song, that is their overall positiveness score). These variables can help as draw useful insights on the determinants of e.g. the popularity of a song or an album, the number of followers of an artist, or we can even determine trending genres.

### Analytical questions

Having taken a look at the dataset, I would like to answer the following questions in this project:

1. What are the determinant factors of an album's popularity in the pop genres?
2. How does albums' popularity differ between between songs from 2010 to 2015 and 2016 to 2013?
3. Is there a relationship between an album's duration and the characteristics of its songs?
4. What are the determinant factors of Taylor Swift's songs - that is what kind of songs should she produce to maximize popularity?
5. How did the average valence of songs evolve over time? Is there a pattern, or at least some bumps that we might attribute to major world events?
6. Which songs were one-time hits - that is, which songs have a much higher popularity than the average popularity of songs on the album?
7. Does an artist's follower count influence the popularity of their songs?
8. Does having a high-follower count featuring artist increase the popularity of an artist's song relative to where there is no featuring artist?
9. Does the high popularity of songs where the artist only features increase their main songs' popularity as well?
10. What are the genres that are very popular but don't have many songs - that is, what kind of genres should we produce if we want high popularity and low competition?
11. Are certain genres associated with more explicit language?
12. What is the relationship between genres' popularity aggregated on 3 different levels: songs, albums, and artists? Does the level of aggregation have an effect on popularity?

Notice, that these questions can be grouped in 3s: #1-3 relates to albums, #4-6 to tracks, #7-9 to artists and #10-12 to genres. This is intententional, as I wanted analyze every type of fact from my database. I will get back to these facts and their possible dimensions in the [Data warehouses](#data-warehouses) chapter.

### Submitted project artifacts and their description

[`spotify_raw_data.zip`](/Term1/spotify_raw_data.zip): The original source CSV files as downloaded from Kaggle. The compressed folder contains the following files: 
* `spotify_artist_data_2023.csv`,
* `spotify_features_data_2023.csv`,
* `spotify_albums_data_2023.csv`,
* `spotify_tracks_data_2023.csv`.

> [!Note]
> On Kaggle, there is one more CSV titled spotify_data_12_20_2023.csv. This is already a merged table of the above data, so this was not used in my project.

[`normalized_data_dump.zip`](/Term1/normalized_data_dump.zip): Contains the `normalized_data_dump.sql` dump file. When executed, it creates the structure of the normalized database (see in [Figure 2](#figure-2)) and populates it with data. It is recommended to run this script rather than to import all data from the raw files.

[`MartonNagy_term1_from_scratch.sql`](/Term1/MartonNagy_term1_from_scratch.sql): This SQL-script initializes the database structure, populates the original tables with data imported from local CSV files, and then performs some normalization tasks to alter the database structure.
> [!Important]
> If you decide to run this script, please make sure to change the path to the imported files according to your local setup! However, I recommend simply loading the database through the provided SQL dump file for performance purposes.

[`MartonNagy_term1_ETLs.sql`](/Term1/MartonNagy_term1_ETLs.sql): This is the main solution file that
* creates ETLs to data warehouses based on the OLTP data,
* maintains ETLs up-to-date,
* and creates views (data mart) based on the data warehouses to provide to for answering the analytical questions.

[`views_chart_generator.py`](views_chart_generator.py): This script connects to the local MySQL server, extracts the views from the `spotify` database to Pandas dataframes and creates charts/tables for each analytical question.
> [!Note]
> This charting code stemmed from my need to practice charting for one of my other courses, so this is not an integral part of my term project. However, as I used the `spotify` dataset, I decided to include this script and the resulting charts in my project here as well.

## Data
### Sources

The database has a single source: a Kaggle dataset repository by the name _Spotify Dataset 2023_. The dataset has been compiled on 2023-12-20 using the Spotify API. It contains data on 438,938 tracks and their respective artists and albums. Note, that this dataset contains only a fraction of the tracks, artists and albums currently on Spotify, and the selection criteria of each observation is unknown. This means that while the dataset may be a good tool to practice SQL, the conclusions drawn from the dataset are bound to be biased because of the unknown (and possibly arbitrary) selection criteria.

The raw CSVs were downloaded directly from Kaggle: https://www.kaggle.com/datasets/tonygordonjr/spotify-dataset-2023

I have downloaded the files on 2024-10-07.

### Variable description

Variable descriptions were taken directly from Kaggle.

> [!Note]
> The fields and tables listed here correspond to the original dataset structure, as downloaded from the source. I have performed some normalization on these tables, which will be described in detail in the [Database structure](#database-structure) chapter. There, I will give a brief description of all the new tables and fields and their rationale.

`albums`:

- `track_name`: Name of the track.

- `track_id`: The Spotify ID for the track.

- `track_number`: The number of the track. If an album has several discs, the track number refers to the number on the specified disc.

- `duration_ms`: The track length in milliseconds.

- `album_type`: The type of the album. Allowed values include: `album`, `single`, `compilation`.

- `artists`: The artists who performed the track.

- `total_tracks`: The number of tracks in the album.

- `album_name`: The name of the album. In case of an album takedown, the value may be an empty string.

- `release_date`: The date the album was first released.

- `label`: The label associated with the album.

- `album_popularity`: The popularity of the album, ranging between 0 and 100, with 100 being the most popular.

- `album_id`: The Spotify ID for the album.

- `artist_id`: The Spotify ID for the artist.

- `artist_0`: Main artist.

- `artist_1`: Featuring artist.

- `artist_2`: Featuring artist.

- `artist_3`: Featuring artist.

- `artist_4`: Featuring artist.

- `artist_5`: Featuring artist.

- `artist_6`: Featuring artist.

- `artist_7`: Featuring artist.

- `artist_8`: Featuring artist.

- `artist_9`: Featuring artist.

- `artist_10`: Featuring artist.

- `artist_11`: Featuring artist.

- `duration_sec`: Track length in seconds.

`artist`:

- `id`: The Spotify ID of the artist.

- `name`: Name of the artist.

- `artist_popularity`: The popularity of the artist, ranging between 0 and 100, with 100 being the most popular. The artist's popularity is calculated based on the popularity of all the artist's tracks.

- `artist_genres`: A list of the genres the artist is associated with. If the artist is not yet classified, this array will be empty.

- `followers`: The total number of followers.

- `genre_0`: Main genre.

- `genre_1`: Sub-genre.

- `genre_2`: Sub-genre.

- `genre_3`: Sub-genre.

- `genre_4`: Sub-genre.

- `genre_5`: Sub-genre.

- `genre_6`: Sub-genre.

`tracks`:

- `id`: The Spotify ID for the track.

- `track_popularity`: The popularity of a track, ranging between 0 and 100, with 100 being the most popular. The popularity is calculated by an algorithm based on the total number of plays and how recent those plays are. Duplicate tracks (e.g., the same track from a single and an album) are rated independently. Artist and album popularity are derived from track popularity.

- `explicit`: Whether the track contains explicit lyrics (`true` = yes, `false` = no or unknown).

`features`:

- `danceability`: Danceability describes how suitable a track is for dancing based on a combination of musical elements including tempo, rhythm stability, beat strength, and overall regularity. A value of 0.0 is least danceable and 1.0 is most danceable.

- `energy`: Energy is a measure from 0.0 to 1.0 and represents a perceptual measure of intensity and activity. Typically, energetic tracks feel fast, loud, and noisy. For example, death metal has high energy, while a Bach prelude scores low on the scale. Perceptual features contributing to this attribute include dynamic range, perceived loudness, timbre, onset rate, and general entropy.

- `key`: The key the track is in. Integers map to pitches using standard Pitch Class notation. E.g., 0 = C, 1 = C♯/D♭, 2 = D, and so on. If no key was detected, the value is -1.

- `loudness`: The overall loudness of a track in decibels (dB). Loudness values are averaged across the entire track and are useful for comparing relative loudness of tracks. Values typically range between -60 and 0 dB.

- `mode`: Mode indicates the modality (major or minor) of a track. Major is represented by 1 and minor by 0.

- `speechiness`: Speechiness detects the presence of spoken words in a track. The more exclusively speech-like the recording (e.g., talk show, audiobook, poetry), the closer to 1.0 the attribute value. Values above 0.66 describe tracks made entirely of spoken words, values between 0.33 and 0.66 describe tracks that may contain both music and speech (e.g., rap music), and values below 0.33 likely represent music or non-speech-like tracks.

- `acousticness`: A confidence measure from 0.0 to 1.0 of whether the track is acoustic. A value of 1.0 represents high confidence that the track is acoustic.

- `instrumentalness`: Predicts whether a track contains no vocals. "Ooh" and "aah" sounds are treated as instrumental. Rap or spoken word tracks are clearly vocal. The closer the value is to 1.0, the greater the likelihood the track contains no vocal content. Values above 0.5 are intended to represent instrumental tracks, with higher confidence as the value approaches 1.0.

- `liveness`: Detects the presence of an audience in the recording. Higher liveness values represent an increased probability that the track was performed live. A value above 0.8 indicates a strong likelihood the track is live.

- `valence`: A measure from 0.0 to 1.0 describing the musical positiveness conveyed by a track. High valence indicates positive tracks (e.g., happy, cheerful, euphoric), while low valence indicates negative tracks (e.g., sad, depressed, angry).

- `tempo`: The overall estimated tempo of a track in beats per minute (BPM). In musical terms, tempo is the speed or pace of a given piece and derives directly from the average beat duration.

- `type`: The object type.

- `id`: The Spotify ID for the track.

- `uri`: The Spotify URI for the track.

- `track_href`: A link to the Web API endpoint providing full details of the track.

- `analysis_url`: A URL to access the full audio analysis of this track (an access token is required to retrieve data).

- `duration_ms`: The duration of the track in milliseconds.

- `time_signature`: An estimated time signature. The time signature ranges from 3 to 7, corresponding to time signatures from 3/4 to 7/4.


## OLTP layer
### Database structure
The initial database (that is, the OLTP layer) has 4 tables with structure presented in Figure 1.

<a name="figure-1"></a>
***Figure 1: The initial structure of the `spotify` database***

![The initial structure of the spotify database, EER graph](/Term1/assets/OLTP_structure.png)

Note, that the database is clearly not in a normal form because:

* Genres could be better represented in a separate table connected to `artist` by a junction table (currently they are represented in 8 different fields of `artist`, which makes it hard to work with).
* `albums` is currently very redundant: it holds information on every album as many times as there are tracks in the album. So, `albums` should be splitted into 2 separate tables: one describing only album-related info, and one junction table connecting albums to tracks.
* Information on tracks is scattered around in 3 different tables: `tracks`, `features` and `albums`. These fields should be merged into one table containing all track-specific fields (for the next point, note that artists performing a track is also a track-specific information, though an album should have a main artist as well).
* Performing artists of a track could be better represented by a junction table connecting `artist` and the `tracks` (now containing all track-specific fields) tables.

So, I performed some normalization tasks on the original tables, and the resulting structure is presented in Figure 2. I will perform all future tasks of Term Project 1 on this normalized database rather than on the original.

<a name="figure-2"></a>
***Figure 2: The normalized structure of the `spotify` database***

![The normalized structure of the spotify database, EER graph](/Term1/assets/OLTP_structure_normalized.png)

> [!Note]
> During the process of normalization, I have encountered a small number of duplicate entries in the database (this was possible, as I was performing the load data statements with `unique_checks` set to 0 for performance reasons). Only one instance of a duplicate has been kept. Thus, the normalized database has 438,102 tracks, 78,172 albums, 37,012 artists and 3,959 genres in it.

> [!Important]
> When normalizing the tables, there was one major issue I could not resolve. In the original `albums` table, artists were presented only by name rather than ID (only one artist ID has been given for the main artist of the album). Thus, I had to join artists to tracks by artists' name rather than a unique ID which lead to some incorrect connections. For main artists, this was later resolved by overwriting the main artist of each song to the main artist of the album. However, I have found no feasible solution (other than manually looking up every record) to correct featuring artists' relations, so they were left in the database in the incorrect way. So, if we were to derive some insights from the data, this is a major limitation one must pay close attention to!

## Data warehouses

As I have noted in the introduction, we need four different data warehouses based on the facts we seek to examine: one on albums, one on tracks, one on artists and one on genres. Then, we can easily derive views related to each question based on the respective data warehouse. Below, I present briefly each data warehouse in a chapter, outlining the facts and their dimensions in each one. Note, that neither of the warehouses are exhaustive: many more fields may have been added for each dimension. I aimed for a golden mean between what was strictly necessary to answer my questions and the endless possibilites I could have had.

All data warehouses are implemented as stored procedures. They are called once explicitly to initialize the warehouses. Then, a scheduled event (see later) re-builds all warehouses once a day to keep them up-to-date. The stored procedures also log some messages into the `messages` table (the message differs whether the stored procedure is called to initalize the warehouse with a 0 input parameter, or when it is called by an event, with a 1 input parameter).

### Data warehouse on albums (`albums_dw`)

This table stores aggregated album data, including album facts, artist dimensions, genre dimensions, and track dimensions. Below is a detailed list of the fields and their content.

#### Table Structure

##### Primary Key

> [!Note]
> A primary key was needed so that deletion triggers work even if `safe updates` is turned on in MySQL settings.

- `album_id`: Unique identifier for the album.

##### Album Facts
- `album_name`: Name of the album.
- `total_tracks`: Total number of tracks in the album.
- `album_popularity`: Popularity score of the album.
- `release_date`: Date when the album was released.
- `album_type`: Type of album (e.g., album, single, compilation).
- `label`: Name of the record label that released the album.
- `total_duration_s`: Total duration of the album in seconds.

##### Artist Dimension
- `artist_id`: Unique identifier for the main artist.
- `artist_name`: Name of the main artist.
- `artist_popularity`: Popularity score of the main artist.
- `followers`: Number of followers of the main artist.
- `avg_feat_artist_popularity`: Average popularity score of featuring artists on the album.
- `avg_feat_artist_followers`: Average number of followers of featuring artists on the album.
- `count_feat_artist`: Count of featuring artists on the album.

##### Genre Dimension
- `artist_main_genre`: Main genre of the album's primary artist.

##### Track Dimension (Aggregated Track-Level Data)
- `explicit_tracks_pct`: Percentage of explicit tracks in the album.
- `avg_danceability`: Average danceability score of tracks in the album.
- `avg_energy`: Average energy score of tracks in the album.
- `avg_loudness`: Average loudness level of tracks in the album.
- `avg_speechiness`: Average speechiness score of tracks in the album.
- `avg_acousticness`: Average acousticness score of tracks in the album.
- `avg_instrumentalness`: Average instrumentalness score of tracks in the album.
- `avg_liveness`: Average liveness score of tracks in the album.
- `avg_valence`: Average valence score (musical positivity) of tracks in the album.

##### Subqueries
- Subqueries are used to calculate aggregate values, such as:
  - Featured artist popularity and follower statistics.
  - Total album duration (in seconds).
  - Track-level averages for metrics like energy, danceability, and explicitness.

### Data warehouse on tracks (`tracks_dw`)

This table stores aggregated track data, including track facts, album dimensions, artist dimensions, and genre dimensions. Below is a detailed list of the fields and their content.

#### Table Structure

##### Primary Key

> [!Note]
> A primary key was needed so that deletion triggers work even if `safe updates` is turned on in MySQL settings.

- `id`: Unique identifier for the track.

##### Track Facts
- `track_name`: Name of the track.
- `track_popularity`: Popularity score of the track.
- `is_explicit`: Indicates whether the track contains explicit content.
- `danceability`: Danceability score of the track.
- `energy`: Energy level of the track.
- `key_signature`: Key signature of the track.
- `loudness`: Loudness level of the track.
- `mode`: Mode of the track (e.g., major or minor).
- `speechiness`: Speechiness score of the track.
- `acousticness`: Acousticness score of the track.
- `instrumentalness`: Instrumentalness score of the track.
- `liveness`: Liveness score of the track.
- `valence`: Valence score (musical positivity) of the track.
- `tempo`: Tempo of the track in beats per minute (BPM).
- `duration_s`: Duration of the track in seconds (converted from milliseconds).
- `time_signature`: Time signature of the track.

##### Album Dimension
- `album_name`: Name of the album containing the track.
- `total_tracks`: Total number of tracks in the album.
- `release_date`: Release date of the album.
- `album_popularity`: Popularity score of the album.
- `album_duration_s`: Total duration of the album in seconds.
- `albumtracks_avg_popularity`: Average popularity of tracks in the album.

##### Artist Dimension
- `main_artist_name`: Name of the main artist.
- `main_artist_popularity`: Average popularity score of the main artist.
- `main_artist_followers`: Average number of followers of the main artist.
- `feat_artist_avg_popularity`: Average popularity score of featured artists.
- `feat_artist_avg_followers`: Average number of followers of featured artists.
- `feat_artist_count`: Count of featured artists on the track.

##### Genre Dimension
- `main_artist_genre`: Main genre of the track’s primary artist.

##### Subqueries
- Subqueries are used to calculate aggregate values, such as:
  - Total album duration (in seconds).
  - Average track popularity within the album.
  - Main and featuring artist popularity and follower statistics.
  - Genre data linked to the main artist.

### Data warehouse on artists (`artist_dw`)

This table stores aggregated artist data, including artist facts, tracks dimensions, albums dimensions, and genres dimensions. Below is a detailed list of the fields and their content.

#### Table Structure

##### Primary Key

> [!Note]
> A primary key was needed so that deletion triggers work even if `safe updates` is turned on in MySQL settings.

- `artist_id`: Unique identifier for the artist.

##### Artist Facts
- `artist_name`: Name of the artist.
- `artist_popularity`: Popularity score of the artist.
- `followers`: Number of followers of the artist.

##### Tracks Dimension
- `main_songs_popularity`: Average popularity score of tracks where the artist is the main artist.
- `feat_songs_popularity`: Average popularity score of tracks where the artist is a featured artist.
- `main_songs_count`: Count of tracks where the artist is the main artist.
- `feat_songs_count`: Count of tracks where the artist is a featured artist.
- `avg_popularity_no_feat`: Average popularity score of tracks without featuring artists.
- `avg_popularity_with_feat`: Average popularity score of tracks with featuring artists.
- `avg_popularity_with_high_follower_feat`: Average popularity score of tracks where there is a highly followed featuring artist (defined as having at least twice the main artist's followers).

##### Albums Dimension
- `albums_count`: Total number of albums by the artist.
- `avg_albums_popularity`: Average popularity score of the artist's albums.

##### Genres Dimension
- `sub_genres_count`: Count of the artist's subgenres.
- `main_genre`: Main genre of the artist.

##### Subqueries
- Subqueries are used to calculate aggregate values, such as:
  - The artist's track and feature statistics, including counts and average popularity.
  - The artist's album counts and average album popularity.
  - Genre details like subgenre counts and the main genre.
  - Track popularity in relation to featured artists, including tracks with highly followed featuring artists.

### Data warehouse on genres (`genres_dw`)

This table stores aggregated genre data, including album, artist, and track statistics in relation to their association with genres as either the main or subgenre. Below is a detailed list of the fields and their content.

#### Table Structure

##### Primary Key

> [!Note]
> A primary key was needed so that deletion triggers work even if `safe updates` is turned on in MySQL settings.

- `genre_id`: Unique identifier for the genre.

##### Genre Facts
- `genre`: Name of the genre.

##### Album Dimension
- `albums_in_genre_main`: Number of albums where the genre is the main genre of the album's artist.
- `albums_in_genre_sub`: Number of albums where the genre is a subgenre of the album's artist.
- `albums_in_genre_main_popularity_avg`: Average popularity score of albums where the genre is the main genre.
- `albums_in_genre_sub_popularity_avg`: Average popularity score of albums where the genre is a subgenre.

##### Artist Dimension
- `artist_in_genre_main`: Number of artists that have the genre as their main genre.
- `artist_in_genre_sub`: Number of artists that have the genre as a subgenre.
- `artist_in_genre_main_popularity_avg`: Average popularity score of artists with the genre as their main genre.
- `artist_in_genre_sub_popularity_avg`: Average popularity score of artists with the genre as a subgenre.
- `artist_in_genre_main_foll_sum`: Total number of followers of artists with the genre as their main genre.
- `artist_in_genre_sub_foll_sum`: Total number of followers of artists with the genre as a subgenre.

##### Track Dimension
- `tracks_in_genre_main`: Number of tracks where the genre is the main genre of the track's artist.
- `tracks_in_genre_sub`: Number of tracks where the genre is a subgenre of the track's artist.
- `tracks_in_genre_main_popularity_avg`: Average popularity score of tracks where the genre is the main genre.
- `tracks_in_genre_sub_popularity_avg`: Average popularity score of tracks where the genre is a subgenre.
- `tracks_in_genre_main_explicit_pct`: Percentage of explicit tracks where the genre is the main genre.
- `tracks_in_genre_sub_explicit_pct`: Percentage of explicit tracks where the genre is a subgenre.

##### Subqueries
- Subqueries are used to calculate aggregate values, such as:
  - Counts and average popularity of albums, artists, and tracks in main and subgenres.
  - Total follower counts for artists in each genre.
  - Explicit content percentages for tracks within each genre.

## Data marts

I have created views providing selected data to answer my 12 analytical questions. For most of the questions, I provide fact-level data in each view so that further statistical analysis (e.g. a simple regression) may be performed. Note, however, that for the 2nd question, I went with a different approach: I aggrageted the data for the 2 periods and the view presents some of the key statistical properties of the groups. This helps answering the question without any more sophisticated approach, but cuts the possibility of further analysis.

### What are the determinant factors of an album's popularity in the pop genres?

This view, `pop_albums`, provides insights into what influences an album's popularity within the pop genre. By selecting albums where the artist's main genre is related to "pop," the view helps identify patterns and factors that might correlate with higher album popularity.

#### Selected Fields

- **Album Details**: 
  - `album_id`, `album_name`, `album_type`, and `label` provide core album information.
  - Date-related fields (`release_year`, `release_month`, `release_day`, `release_dayofweek`, and `release_time`) offer insights into the release timing of the album, which might influence its popularity.
  - `total_tracks` and `total_duration_s` give an idea of the album's structure and length.

- **Popularity Metrics**:
  - `album_popularity` is the key dependent variable being analyzed.
  - The view includes `artist_popularity` and `followers` for the main artist, as well as the average popularity and follower count of any featuring artists (`avg_feat_artist_popularity`, `avg_feat_artist_followers`, and `count_feat_artist`).

- **Genre**: Only albums from artists whose main genre includes "pop" are included in the view (`artist_main_genre`).

- **Track Characteristics**:
  - Track-level data is aggregated to give averages for explicit tracks (`explicit_tracks_pct`), danceability, energy, loudness, and other musical attributes like speechiness, acousticness, instrumentalness, liveness, and valence. These features can be useful in understanding the sound profile that correlates with popular pop albums.

#### Ordering
The results are ordered by `album_popularity` in descending order, so the most popular albums appear first in the dataset.

#### Charts
I have charted a few regression plots based on the view to uncover which factors influence most the albums' popularity (see in Figure 3).

***Figure 3: Regressions of albums_popularity on certain quantitative variables***

![albums_popularity_regressions](/Term1/assets/q1.png)

Based on this, we can see that average song level variables do not really have an influence on album popularity. However, artists' popularity and follower counts show a strong correlation with the popularity of the albums. This means that more popular artists tend to have more popular albums.

### How does albums' popularity differ between songs from 2010 to 2015 and 2016 to 2023?

The `albums_popularity_date` view analyzes the differences in album popularity between two distinct time periods: 2010-2015 and 2016-2023. This view aggregates album popularity data to identify trends and patterns based on the release year of the albums.

#### Selected Fields

- **Release Year Category**:
  - The view categorizes albums into two groups:
    - `'2010-2015'`: For albums released between 2010 and 2015.
    - `'2016-2023'`: For albums released between 2016 and 2023.
  
- **Popularity Metrics**:
  - `avg_album_popularity`: The average popularity score of albums within each release year category.
  - `std_album_popularity`: The standard deviation of album popularity, indicating the variability within each category.
  - `min_album_popularity` and `max_album_popularity`: These fields provide insight into the range of album popularity within each time frame.
  - `count_album_popularity`: The total count of albums considered in each release year category.

#### Grouping and Filtering
- The results are grouped by `release_year_category`, allowing for a comparison of album popularity between the two defined periods.
- The `having` clause ensures that only non-null categories are included in the final output, eliminating any rows without valid release year categories.

#### Output table

***Figure 4: Album popularity aggregate differences between 2010-2015 and 2016-2023***

![album popularity in two eras](/Term1/assets/q2.png)

From the output table, we can see that more recent songs tend to be slightly more popular then older ones (this is the same tendency we can see on Figure 3's last plot).

### Is there a relationship between an album's duration and the characteristics of its songs?

The `duration_determinants` view explores the potential relationship between an album's duration and various musical characteristics of its tracks. By analyzing these attributes, this view aims to uncover insights regarding how song characteristics may correlate with the length of albums.

#### Selected Fields

- **Album Identification**:
  - `album_id`: Unique identifier for the album.
  - `album_name`: The name of the album.
  
- **Album Duration**:
  - `total_duration_s`: Total duration of the album in seconds.

- **Musical Characteristics**:
  - `avg_danceability`: Average danceability score of tracks in the album.
  - `avg_energy`: Average energy score of tracks in the album.
  - `avg_loudness`: Average loudness level of tracks in the album.
  - `avg_speechiness`: Average speechiness score of tracks in the album.
  - `avg_acousticness`: Average acousticness score of tracks in the album.
  - `avg_instrumentalness`: Average instrumentalness score of tracks in the album.
  - `avg_liveness`: Average liveness score of tracks in the album.
  - `avg_valence`: Average valence score (musical positivity) of tracks in the album.

#### Filtering Criteria
The view only includes albums where `total_duration_s` is not null, ensuring that all analyzed albums have defined durations for accurate analysis.

#### Charts

Just like for question #1, I have plotted a few regression plots of albums' duration against different factors that might influence it. Note, that the dependent variable has been log-transformed.

***Figure 5: Regressions of ln(total_duration_s) on certain quantitative variables***

![albums_duration_regressions](/Term1/assets/q3.png)

From the results, we can see some interesting tendencies. For example more danceable albums tend to be shorter, and the same stands for loudness and speechiness. However, the results from these regressions are very noisy, so no clear-cut conclusions can be drawn.

### What are the determinant factors of Taylor Swift's songs - that is what kind of songs should she produce to maximize popularity?

The `taylor_swift_songs` view analyzes the characteristics of Taylor Swift's tracks to identify which factors contribute to maximizing their popularity. By examining various song attributes, this view aims to provide insights into the kind of songs she should produce to enhance their appeal.

#### Selected Fields

- **Track Information**:
  - `track_name`: The name of the track.
  - `track_popularity`: Popularity score of the track.
  - `is_explicit`: Indicates if the track is explicit.
  
- **Musical Characteristics**:
  - `danceability`: Danceability score of the track.
  - `energy`: Energy score of the track.
  - `key_signature`: Key signature of the track.
  - `loudness`: Loudness level of the track.
  - `mode`: Musical mode of the track.
  - `speechiness`: Speechiness score of the track.
  - `acousticness`: Acousticness score of the track.
  - `instrumentalness`: Instrumentalness score of the track.
  - `liveness`: Liveness score of the track.
  - `valence`: Valence score (musical positivity) of the track.
  - `tempo`: Tempo of the track.
  - `duration_s`: Duration of the track in seconds.
  - `time_signature`: Time signature of the track.

- **Album Context**:
  - `album_duration_s`: Total duration of the album containing the track.
  
- **Featured Artist Metrics**:
  - `feat_artist_avg_popularity`: Average popularity of featuring artists on the track.
  - `feat_artist_avg_followers`: Average number of followers of featuring artists on the track.
  - `feat_artist_count`: Count of featuring artists on the track.

- **Release Date Information**:
  - `release_year`: Year the track was released.
  - `release_month`: Month the track was released.
  - `release_day`: Day the track was released.
  - `release_dayofweek`: Day of the week the track was released.
  - `release_time`: Time the track was released.

#### Ordering
The results are ordered by `track_popularity` in descending order, allowing for easy identification of the most popular tracks.

#### Charts

***Figure 6: Regressions of Taylor Swift's track popularity on certain quantitative variables***

![taylor_swift_regressions](/Term1/assets/q4.png)

Some of the most important facts we can derive from the above graphs are:
* more energetic songs tend to be more popular;
* the less speechy a song is, the more popular it tends to be;
* the later a song is released during the week, the more popular it gets on average;
* Taylor Swift's songs tend to get more popular over time.

### How did the average valence of songs evolve over time?

The `valence_ts` view investigates the evolution of the average valence score of songs released over time. Valence is a measure of musical positivity or emotional value, and this analysis aims to identify trends and potential correlations with significant world events.

#### Selected Fields

- **Release Date Information**:
  - `release_year`: Year of the song's release.
  - `release_month`: Month of the song's release.

- **Song Metrics**:
  - `count_songs`: Total number of songs released in that specific year and month.
  - `avg(valence)`: Average valence score of the songs released during that time period.

#### Grouping and Ordering
- The results are grouped by `release_year` and `release_month`, allowing for a monthly overview of how average valence scores change over time.
- The results are ordered by `release_year` and `release_month` to maintain a chronological sequence.

#### Purpose
This view enables an analysis of trends in the emotional content of music over time, potentially highlighting patterns or significant changes that may correspond with major world events.

#### Chart

The below figure might give us an idea on how the average valence of songs evolved over time (the count of songs has also been plotted to show that data availibility is very much skewed towards more recent songs). One of the key message of this chart is that songs tend to be less and less positive since the 2008 financial crises.

***Figure 7: Average valence and song count over time (monthly, with 12-month moving averages)***

![valence_time_series](/Term1/assets/q5.png)

### Which songs were one-time hits - that is, which songs have a much higher popularity than the average popularity of songs on the album?

The `one_time_hits` view identifies songs that have achieved exceptionally high popularity compared to the average popularity of tracks on their respective albums. This analysis focuses on isolating tracks that stand out significantly from their peers within the same album.

#### Selected Fields

- **Track Information**:
  - `id`: Unique identifier for the track.
  - `track_name`: Name of the track.
  - `track_popularity`: Popularity score of the track.
  - `albumtracks_avg_popularity`: Average popularity score of tracks on the same album.

#### Filtering Criteria

- **Popularity Threshold**: The view includes only those tracks whose popularity is at least **25 times higher** than the average popularity of their album's tracks.
  
- **Album Track Count**: Only albums with at least **10 tracks** are considered to ensure a substantial dataset for comparison.

- **Exclusion of Zero Popularity**: Tracks and album averages with a popularity score of **zero** are excluded from the results to avoid misleading data.

#### Ordering
The results are ordered by `track_popularity` in descending order, highlighting the most popular one-time hits at the top of the list.

#### Purpose
This view allows for the identification of tracks that became standout hits, providing insights into songs that significantly exceeded the popularity expectations set by their albums.

#### Chart

Below there is a scatterplot of the songs that qualify as one time hits (track popularity against albums' songs average popularity).

***Figure 8: Songs qualifying as one-time-hits***

![one time hits](/Term1/assets/q6.png)

### Does an artist's follower count influence the popularity of their songs?

The `artist_followers_popularity` view examines the relationship between an artist's follower count and the popularity of their songs. This analysis seeks to determine whether there is a correlation between the number of followers an artist has and their overall popularity within the music industry.

#### Selected Fields

- **Artist Information**:
  - `artist_id`: Unique identifier for the artist.
  - `artist_name`: Name of the artist.
  - `artist_popularity`: Popularity score of the artist.
  - `followers`: Total number of followers the artist has.

#### Ordering
The results are ordered first by `artist_popularity` in descending order and then by `followers` in descending order. This arrangement highlights the most popular artists at the top of the list, allowing for easy comparison of popularity against follower counts.

#### Purpose
This view aims to provide insights into whether having a higher follower count translates into greater song popularity, allowing for further analysis of trends and correlations in the music industry.

#### Chart

The results were plotted on the below regression plot. For better visibility, I only included artists with more than 1M followers. From the plot, it is clearly visible, that there is a relationship between artists' follower count and popularity (just as common sense would suggest).

***Figure 9: Regression of artists' popularity against follower counts***

![artist popularity vs followers](/Term1/assets/q7.png)

### Does having a high-follower count featuring artist increase the popularity of an artist's song relative to where there is no featuring artist?

The `feat_effects` view investigates whether collaborating with a featuring artist who has a high follower count positively influences the popularity of an artist's song compared to songs without a featuring artist. This analysis provides insights into the potential benefits of collaborations in the music industry.

#### Selected Fields

- **Artist Information**:
  - `artist_id`: Unique identifier for the artist.
  - `artist_name`: Name of the artist.
  
- **Popularity Metrics**:
  - `avg_popularity_no_feat`: Average popularity of songs where there are no featuring artists.
  - `avg_popularity_with_feat`: Average popularity of songs that feature artists.
  - `avg_popularity_with_high_follower_feat`: Average popularity of songs featuring artists with a high follower count.

#### Ordering
The results are ordered by `avg_popularity_no_feat` in descending order, followed by `avg_popularity_with_feat`, and then `avg_popularity_with_high_follower_feat`. This order helps to highlight the differences in song popularity based on the presence and follower count of featuring artists.

#### Purpose
This view aims to clarify the relationship between collaboration with high-follower count artists and the overall popularity of an artist's songs, allowing for informed decisions about collaborations in future music projects.

#### Charts

The three scenarios in the view (no feat. artist, with feat. artist, with high follower-count feat artist) were plotted on three violin plots to compare the distributions.

***Figure 10: Violin plots of the three scenarios***

![feat effects](/Term1/assets/q8.png)

From the plot, it is clearly visible, that featuring artists contribute heavily to song's popularity. However, having a high-follower count feat. does not necessarily provide even more popularity. 

### Does the high popularity of songs where the artist only features increase their main songs' popularity as well?

The `feature_spillovers` view examines the potential relationship between the popularity of songs in which an artist is featured and the popularity of their main songs. This analysis aims to determine if there is a "spillover" effect, where high-performing collaborative tracks contribute to the overall popularity of an artist's primary work.

#### Selected Fields

- **Artist Information**:
  - `artist_id`: Unique identifier for the artist.
  - `artist_name`: Name of the artist.

- **Popularity Metrics**:
  - `feat_songs_popularity`: Average popularity of songs where the artist is featured.
  - `main_songs_popularity`: Average popularity of the artist's main songs.

#### Ordering
The results are ordered by `feat_songs_popularity` in descending order, followed by `main_songs_popularity`. This ordering highlights any potential relationships between the popularity of featured songs and the main songs of an artist.

#### Purpose
This view seeks to understand whether the success of an artist in collaborative roles translates into increased popularity for their solo work, providing insights into the dynamics of featuring in songs within the music industry.

#### Chart

The question may be examined using a regression plot.

***Figure 11: Regression of main songs' popularity agains feat. songs' popularity***

![main vs feat](/Term1/assets/q9.png)

From the figure, we can easily deduce that high popularity featured songs correlate strongly with the popularity of main songs.

### What are the genres that are very popular but don't have many songs - that is, what kind genres should we produce if we want high popularity and low competition?

The `genre_niche` view aims to identify genres that exhibit high popularity while having a relatively low number of songs. This analysis is valuable for understanding which genres might be less competitive yet still appealing to audiences, guiding production efforts toward potentially successful niches.

#### Selected Fields

- **Genre Information**:
  - `genre_id`: Unique identifier for the genre.
  - `genre`: Name of the genre.

- **Song Metrics**:
  - `tracks_in_genre_main`: Number of songs classified as the main genre.
  - `tracks_in_genre_sub`: Number of songs classified as subgenres.

- **Popularity Metrics**:
  - `tracks_in_genre_main_popularity_avg`: Average popularity of songs in the main genre.
  - `tracks_in_genre_sub_popularity_avg`: Average popularity of songs in the subgenres.

#### Filtering Criteria
- The view filters genres to include only those where the total number of songs (both main and subgenres) is between 50 and the average number of songs per genre across the dataset. This definition of "not many songs" ensures that we are looking for genres that have a good amount of popularity but are not oversaturated with content.

#### Ordering
The results are ordered by `tracks_in_genre_main_popularity_avg` in descending order, followed by `tracks_in_genre_sub_popularity_avg`. This ordering highlights the most promising genres based on their average popularity metrics.

#### Purpose
This view provides insights into potential opportunities within the music industry, highlighting genres that may offer high popularity without significant competition. It serves as a strategic tool for artists and producers looking to explore less crowded musical territories while still appealing to a broad audience.

#### Charts

The below chart shows the identified niche genres for both main and sub-genres.

***Figure 12: Identified niche genres***

![niche genres](/Term1/assets/q10.png)

### Are certain genres associated with more explicit language?

The `explicit_genres` view investigates the relationship between musical genres and the prevalence of explicit language in their songs. This analysis helps identify which genres are more likely to feature explicit content, providing insights for artists, producers, and audiences.

#### Selected Fields

- **Genre Information**:
  - `genre_id`: Unique identifier for the genre.
  - `genre`: Name of the genre.

- **Explicit Content Metrics**:
  - `tracks_in_genre_main_explicit_pct`: Percentage of explicit tracks in the main genre.
  - `tracks_in_genre_sub_explicit_pct`: Percentage of explicit tracks in the subgenres.

#### Ordering Criteria
The view orders genres by a calculated metric that represents the overall prevalence of explicit content: the formula combines the explicit percentages of main and subgenre tracks, weighted by the number of tracks in each category. This allows for a comprehensive assessment of explicit language usage across genres.

#### Purpose
This view aims to provide insights into the explicit content associated with various musical genres. By identifying genres with higher rates of explicit language, stakeholders in the music industry can make informed decisions about marketing, production, and audience targeting.

#### Chart

The below chart shows the identified explicit genres.

***Figure 13: Identified explicit genres***

![explicit genres](/Term1/assets/q11.png)

### What is the relationship between genres' popularity aggregated on 3 different levels: songs, albums, and artists? Does the level of aggregation have an effect on popularity?

The `genre_aggregation` view explores the relationship between the popularity of musical genres at three distinct levels: songs, albums, and artists. This analysis aims to understand whether the level of aggregation affects the perceived popularity of a genre.

#### Selected Fields

- **Genre Information**:
  - `genre_id`: Unique identifier for the genre.
  - `genre`: Name of the genre.

- **Popularity Metrics**:
  - `tracks_in_genre_main_popularity_avg`: Average popularity of tracks within the main genre.
  - `albums_in_genre_main_popularity_avg`: Average popularity of albums associated with the main genre.
  - `artist_in_genre_main_popularity_avg`: Average popularity of artists whose primary genre is the main genre.

#### Ordering Criteria
- The view orders genres based on the following criteria, in descending order:
  - Average track popularity within the main genre.
  - Average album popularity within the main genre.
  - Average artist popularity associated with the main genre.

#### Purpose
This view is designed to provide insights into how the popularity of genres varies when measured at the levels of songs, albums, and artists.

#### Charts

The below violin plots show that theres is a clear difference in the average popularity of genres when aggregated on different levels.

***Figure 14: Distribution of genres' popularity aggrageted on three levels***

![genre aggregation](/Term1/assets/q12.png)

## Extra features: triggers, events and materialized views

### Triggers

The following triggers are designed to automate the logging and deletion of corresponding data from various data warehouse tables when records are deleted from the main tables. Each trigger performs the following actions:

- **Trigger on Albums Deletion** (`on_albums_delete`): 
  - Activates after a deletion occurs on the `albums` table.
  - Logs a message into the `messages` table indicating which album ID was deleted.
  - Deletes the corresponding entry from the `albums_dw` table.

- **Trigger on Artist Deletion** (`on_artist_delete`): 
  - Activates after a deletion occurs on the `artist` table.
  - Logs a message into the `messages` table indicating which artist ID was deleted.
  - Deletes the corresponding entry from the `artist_dw` table.

- **Trigger on Tracks Deletion** (`on_tracks_delete`): 
  - Activates after a deletion occurs on the `tracks` table.
  - Logs a message into the `messages` table indicating which track ID was deleted.
  - Deletes the corresponding entry from the `tracks_dw` table.

- **Trigger on Genres Deletion** (`on_genres_delete`): 
  - Activates after a deletion occurs on the `genres` table.
  - Logs a message into the `messages` table indicating which genre ID was deleted.
  - Deletes the corresponding entry from the `genres_dw` table.

These triggers help maintain data integrity and provide a history of deletions.

### Events

The following events are set up to automate the re-initialization of data warehouses (DW) and the updating of materialized views on a daily schedule. These events help maintain the accuracy and relevance of the data in the system:

- **Update All DWs Event (`UpdateAllDWEvent`)**: 
  - This event is scheduled to run every day at 12:30 PM, starting from `2024-10-11`, and will continue for two months.
  - The purpose of this event is to re-initialize all data warehouses to track changes effectively. 
  - When the event is triggered, it informs the user that the update is in progress, displaying a message directly on the screen to alert them.
  - The event logs the start and successful completion of the update in the `messages` table and calls procedures to update the `albums_dw`, `artist_dw`, `tracks_dw`, and `genres_dw` tables.

- **Update Materialized Views Event (`UpdateMVs`)**: 
  - This event is scheduled to run every day at midnight, starting from `2024-10-11`, and will also continue for two months.
  - It is designed to update materialized views, ensuring they reflect the most recent data.
  - Similar to the previous event, it provides immediate feedback to the user about the update status, logs the progress in the `messages` table, and calls procedures to update the relevant materialized views.

These events are essential for maintaining the data integrity and performance of the data warehouse system by ensuring that data is consistently updated and accurate.

### Materialized views

The views for the first two questions are also implemented in a materialized view form. The materialized views are implemented using stored proceedures. They are updated (more precisely, re-built from scratch) by the `UpdateMVs` event scheduled once a day at midnight.

### Charts

As you could see above, I have created a couple of charts to provide intuitive answers to my analytical questions. Note however, that these answers are merely intuitive - that is they are just there to provide a first impression on what might be going on, but for a clear answer, more sophisticated analysis would be needed. Also, these charts were created mainly for me to practice charting in Python for one of my other courses. So they are included here only because they use the same data as my term project.

# A note on what happened to IMDb...

You might remember that I wanted to do Term Project 1 using IMDb data. I have collected all in all 9 tables from IMDb itself, from Kaggle and from scraping The-Numbers.com. Now, it turned out that these tables are rather large in size, so importing them into a database was really time consuming (and also stressful when a `load data` statement throw an error after an hour of running...). Even though I managed to import the tables to a database at the end, I have decided to drop that project, as working with such large files meant I was spending most of my time waiting for a response rather than practicing SQL. Thus, I have chosen the above described Spotify dataset to process in my term project instead.

Nevertheless, I have uploaded 2 files related to my IMDb endeavours:
* [`imdb.sql`](/Term1/imdb_legacy_files/imdb.sql): the script I have used to set up my tables for the IMDb project and to populate them with data from downloaded raw files,
* [`the_numbers_scraper.py`](/Term1/imdb_legacy_files/the_numbers_scraper.py): a Python script I have used to scrape the The-Numbers.com site for budget and box office data.

> [!Important]
> These files are uploaded solely FYI, that is they do not constitute a part of my Term Project 1 submission!
