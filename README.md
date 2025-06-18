# LLM-Benchmark
This project aims to rigorously evaluate and benchmark various LLMs on their ability to accurately translate natural language questions into functional Cypher and SQL queries, using a real-world "Spotify Indonesian Song Dataset."

## Our Dataset
### Relational Database
The dataset provided is a collection of Spotify tracks specifically from Indonesia, consisting of 1,125 records (1.125 Rows). Each record in the dataset contains eight distinct fields: track ID, track name, artist ID, artist name, album ID, album name, release date, and a direct link (8 Columns) to the Spotify API for that track.

All columns are text-based fields. The dataset includes unique identifiers for tracks, artists, and albums, providing clear relational links among these entities. The release date field indicates when each track or album was made available on Spotify. Additionally, each entry includes a direct Spotify API link, facilitating easy integration for further analysis or retrieval of additional information.

### Graph
**Nodes:**
_artist_ ⟶ the music artist that creates the song
_album_ ⟶ the music album
_track_ ⟶ the music track

**Relationships:**
_part_of_ ⟶ shows that a track is a part of an album
_writes_ ⟶ shows that an artist writes a song
_released_ ⟶ shows that an artist released an album

## Canonical Questions
### Level 1 (Simple Lookup) - 1 Question
1. What is the release date of the track "To the Bone"?

### Level 2 (Aggregation) - 2 Questions
1. How many unique artists are in the dataset?			
2. What is the earliest track release date in the dataset?

### Level 3 (Multi-hop Entity) - 5 Questions
1. Which albums contain songs by the artist "Tulus"?			
2. Which tracks are part of the same album as “Hati-Hati di Jalan”?
3. List all tracks by the same artist who released "Monokrom".
4. What are the tracks in the same album as any track released in 2021?
5. List all artists who have more than one album in the dataset.

### Level 4 (Complex Filter) - 7 Questions
1. Which tracks were released after 2020 and are not by "Tulus"?
2. Which artists have tracks from both 2019 and 2022?
3. What are all tracks released before 2018 that are not part of the album "Monokrom"?
4. Which albums have more than 3 tracks in the dataset and were released after 2020?
5. List artists who have released tracks with more than one album ID.
6. Find tracks whose title contains the word "di" and were released in 2022.
7. Which albums include songs from different years?

### Level 5 (Pattern-based) - 6 Questions
1. Which tracks have titles that start with the word "Hati"?
2. Find artists whose names include exactly 2 words.
3. Which albums have names that contain repeated letters (like "oo" or "aa")?
4. List all tracks with titles containing numeric characters.
5. Which tracks have palindromic titles (e.g., same forward and backward)?
6. Which track names follow the pattern "[verb]-[verb]" (like "Runtuh-Bangun")?

## Prompt
### 1st Prompt
We give these 2 images, accompanied with a prompt.
![image](https://github.com/user-attachments/assets/54c1550e-cc1a-4906-952c-13999f239836)
![image](https://github.com/user-attachments/assets/4a9863c7-17ba-4d02-bc33-cb17fc7b088f)

```
This is the database schema, please remember it. Next i will ask you several questions, please reply with a properly formatted syntax in cypher and SQL query that can be used to execute and return the result that I am asking for so in essence.
```

### 2nd Prompt
```
What is the release date of the track "To the Bone"?                        
How many unique artists are in the dataset?                        
What is the earliest track release date in the dataset?                        
Which albums contain songs by the artist "Tulus"?                        
Which tracks are part of the same album as “Hati-Hati di Jalan”?                        
List all tracks by the same artist who released "Monokrom".                        
What are the tracks in the same album as any track released in 2021?                        
List all artists who have more than one album in the dataset.                        
Which tracks were released after 2020 and are not by "Tulus"?                        
Which artists have tracks from both 2019 and 2022?                        
What are all tracks released before 2018 that are not part of the album "Monokrom"?                        
Which albums have more than 3 tracks in the dataset and were released after 2020?                        
List artists who have released tracks with more than one album ID.                        
Find tracks whose title contains the word "di" and were released in 2022.                        
Which albums include songs from different years?                        
Which tracks have titles that start with the word "Hati"?                        
Find artists whose names include exactly 2 words.                        
Which albums have names that contain repeated letters (like "oo" or "aa")?                        
List all tracks with titles containing numeric characters.                        
Which tracks have palindromic titles (e.g., same forward and backward)?                        
Which track names follow the pattern "[verb]-[verb]" (like "Runtuh-Bangun")?
```
### Query Result on LLM
#### ChatGPT 4.1 (SQL)
```
"SELECT track, release_date
FROM dataset_spotify_indonesia
WHERE track = 'To the Bone'
"	"SELECT COUNT(DISTINCT artist) AS unique_artists
FROM dataset_spotify_indonesia
"	"SELECT DISTINCT album
FROM dataset_spotify_indonesia
WHERE artist = 'Tulus'
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE release_date > '2020' AND artist <> 'Tulus'
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE track LIKE 'Hati%'
"
			"SELECT artist
FROM dataset_spotify_indonesia
WHERE release_date LIKE '2019%' OR release_date LIKE '2022%'
GROUP BY artist
HAVING COUNT(DISTINCT CASE WHEN release_date LIKE '2019%' THEN 1 END) > 0
   AND COUNT(DISTINCT CASE WHEN release_date LIKE '2022%' THEN 1 END) > 0
"	
		"SELECT track
FROM dataset_spotify_indonesia
WHERE album_id = (
  SELECT album_id
  FROM dataset_spotify_indonesia
  WHERE track = 'Hati-Hati di Jalan'
)
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE release_date < '2018' AND album <> 'Monokrom'
"	"SELECT DISTINCT artist
FROM dataset_spotify_indonesia
WHERE LENGTH(TRIM(artist)) - LENGTH(REPLACE(TRIM(artist), ' ', '')) = 1
"
	"SELECT MIN(release_date) AS earliest_release
FROM dataset_spotify_indonesia
"		"SELECT album
FROM dataset_spotify_indonesia
WHERE release_date > '2020'
GROUP BY album
HAVING COUNT(track_id) > 3
"	"SELECT DISTINCT album
FROM dataset_spotify_indonesia
WHERE album REGEXP '(.)\\1'
"
		"SELECT track
FROM dataset_spotify_indonesia
WHERE artist = (
  SELECT artist
  FROM dataset_spotify_indonesia
  WHERE track = 'Monokrom'
)
"	"SELECT artist
FROM dataset_spotify_indonesia
GROUP BY artist
HAVING COUNT(DISTINCT album_id) > 1
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE track REGEXP '[0-9]'
"
		"SELECT DISTINCT d2.track
FROM dataset_spotify_indonesia d1
JOIN dataset_spotify_indonesia d2
  ON d1.album_id = d2.album_id
WHERE d1.release_date LIKE '2021%'
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE LOWER(track) LIKE '% di %' AND release_date LIKE '2022%'
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE track = REVERSE(track)
"
		"SELECT artist
FROM dataset_spotify_indonesia
GROUP BY artist
HAVING COUNT(DISTINCT album_id) > 1
"	"SELECT album
FROM dataset_spotify_indonesia
GROUP BY album
HAVING COUNT(DISTINCT SUBSTR(release_date,1,4)) > 1
"	"SELECT track
FROM dataset_spotify_indonesia
WHERE track REGEXP '^[A-Za-z]+-[A-Za-z]+$'
"
```
### Final Score Benchmarking LLM (ChatGPT, Gemini, Claude)
![image](https://github.com/user-attachments/assets/9f758fc1-0d84-4b56-af0c-3aead0c4cce7)

### Conclusion

From the testing results, we can conclude that:
Gemini 2.5 Flash is best for generating Cypher Query, followed by ChatGPT 4.1 on the second position.
In terms of generating SQL Query, ChatGPT 4.1 returns the best result than the others.

