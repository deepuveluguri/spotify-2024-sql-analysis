# ============================================
# Spotify 2024 Analysis with SQL + Python
# ============================================

# STEP 1: Upload & Load Dataset
from google.colab import files
import pandas as pd

# Upload your CSV file
print("👉 Please upload 'Most Streamed Spotify Songs 2024.csv'")
uploaded = files.upload()

# Read CSV
df = pd.read_csv("Most Streamed Spotify Songs 2024.csv", encoding="latin1")

# Clean "Spotify Streams"
df["Spotify Streams"] = df["Spotify Streams"].str.replace(",", "").astype(int)

# Convert release date & extract year
df["Release Date"] = pd.to_datetime(df["Release Date"], errors="coerce")
df["released_year"] = df["Release Date"].dt.year

print("✅ Dataset loaded & cleaned")
display(df.head())


# STEP 2: Load into SQLite
import sqlite3

# Create SQLite in-memory DB
conn = sqlite3.connect(":memory:")

# Save dataframe as SQL table
df.to_sql("spotify", conn, if_exists="replace", index=False)

print("✅ Dataset loaded into SQLite successfully")


# STEP 3: Run SQL Queries

# 1. Top 10 most streamed songs of 2024
q1 = """
SELECT Track, Artist, [Spotify Streams]
FROM spotify
WHERE released_year = 2024
ORDER BY [Spotify Streams] DESC
LIMIT 10;
"""
top_songs = pd.read_sql(q1, conn)
print("🎶 Top 10 Most Streamed Songs of 2024")
display(top_songs)


# 2. Top 10 single artists (no collab)
q2 = """
SELECT Artist, [Spotify Streams]
FROM spotify
WHERE released_year = 2024
  AND Artist NOT LIKE '%,%'
ORDER BY [Spotify Streams] DESC
LIMIT 10;
"""
print("🎤 Top 10 Single Artists (2024)")
display(pd.read_sql(q2, conn))


# 3. Top 10 collab songs
q3 = """
SELECT Track, Artist, [Spotify Streams]
FROM spotify
WHERE released_year = 2024
  AND Artist LIKE '%,%'
ORDER BY [Spotify Streams] DESC
LIMIT 10;
"""
print("🤝 Top 10 Collab Songs (2024)")
display(pd.read_sql(q3, conn))


# 4. Top 10 artists with most songs in 2024
q4 = """
SELECT Artist, COUNT(*) AS total_songs_released
FROM spotify
WHERE released_year = 2024
GROUP BY Artist
ORDER BY total_songs_released DESC
LIMIT 10;
"""
print("📀 Artists with Most Songs Released (2024)")
display(pd.read_sql(q4, conn))


# 5. Top songs included in Spotify playlists
q5 = """
SELECT Track, Artist, [Spotify Playlist Count] AS playlists
FROM spotify
WHERE released_year = 2024
ORDER BY [Spotify Playlist Count] DESC
LIMIT 10;
"""
print("🎧 Top Songs by Spotify Playlist Count (2024)")
display(pd.read_sql(q5, conn))


# 6. Top songs included in Apple playlists
q6 = """
SELECT Track, Artist, [Apple Music Playlist Count] AS playlists
FROM spotify
WHERE released_year = 2024
ORDER BY [Apple Music Playlist Count] DESC
LIMIT 10;
"""
print("🍎 Top Songs by Apple Playlist Count (2024)")
display(pd.read_sql(q6, conn))


# 7. Percentage of songs per release year
q7 = """
SELECT released_year,
       COUNT(Track) * 100.0 / (SELECT COUNT(Track) FROM spotify) AS percentage
FROM spotify
GROUP BY released_year
ORDER BY percentage DESC;
"""
print("📊 Percentage of Songs by Release Year")
display(pd.read_sql(q7, conn))


# 8. Top songs released in 2024
q8 = """
SELECT Artist, Track, [Spotify Streams]
FROM spotify
WHERE released_year = 2024
ORDER BY [Spotify Streams] DESC
LIMIT 10;
"""
print("🔥 Top Songs Released in 2024")
display(pd.read_sql(q8, conn))


# 9. Average streams of top artists (with >2 songs in 2024)
q9 = """
SELECT Artist, AVG([Spotify Streams]) AS avg_streams, COUNT(*) as song_count
FROM spotify
WHERE released_year = 2024
GROUP BY Artist
HAVING COUNT(*) > 2
ORDER BY avg_streams DESC
LIMIT 10;
"""
print("📈 Average Streams of Top Artists (2024)")
display(pd.read_sql(q9, conn))


# 10. Songs with >1 billion streams
q10 = """
SELECT Track, Artist, [Spotify Streams]
FROM spotify
WHERE released_year = 2024
  AND [Spotify Streams] > 1000000000
ORDER BY [Spotify Streams] DESC;
"""
print("💯 Songs with >1 Billion Streams (2024)")
display(pd.read_sql(q10, conn))


# 11. Songs above average streams (2024)
q11 = """
SELECT Track, Artist, [Spotify Streams]
FROM spotify
WHERE released_year = 2024
  AND [Spotify Streams] > (SELECT AVG([Spotify Streams]) 
                           FROM spotify WHERE released_year = 2024)
ORDER BY [Spotify Streams] DESC;
"""
print("⭐ Songs Above Average Streams (2024)")
display(pd.read_sql(q11, conn))


# STEP 4: Visualization Example
import matplotlib.pyplot as plt

top_songs.plot(kind="bar", x="Track", y="Spotify Streams", legend=False)
plt.title("Top 10 Spotify Songs of 2024")
plt.xticks(rotation=45, ha="right")
plt.show()
