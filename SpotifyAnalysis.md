<h1 id="spotify-audio-features-and-genre-popularity-analysis">Spotify Audio Features and Genre Popularity Analysis</h1>
<h2 id="a-genre-and-feature-level-analysis-for-independent-artists--marketing-teams">A Genre and Feature-Level Analysis for Independent Artists &amp; Marketing Teams</h2>
<h2 id="business-context">1. Business Context</h2>
<p>This analysis was conducted for a marketing team working with independent music artists to understand what musical characteristics make songs popular on Spotify.</p>
<h2 id="business-question">2. Business Question</h2>
<p>Independent artists and their marketing teams seek to understand how musical characteristics influence listener engagement on Spotify. This analysis addresses the following questions:</p>
<ul>
<li>Which genre categories perform most strongly on Spotify as measured by track popularity?</li>
<li>What shared audio characteristics distinguish higher-performing tracks from lower-performing ones across genres?</li>
</ul>
<h2 id="dataset-overview">3. Dataset Overview</h2>
<ul>
<li>The data was sourced from Maharshipandya’s <a href="https://huggingface.co/datasets/maharshipandya/spotify-tr">spotify-tracks-dataset</a> which was compiled using Spotify’s API to retrieve track-level audio features. The dataset contains approximately <strong>114,000 rows</strong>, representing tracks distributed evenly across <strong>114 genres</strong>, with <strong>1,000 tracks per genre</strong>.</li>
<li>Each record includes both descriptive metadata and audio features, including:<br>
<code>track_id</code>, <code>artists</code>, <code>track_name</code>, <code>album_name</code>, <code>popularity</code>, <code>danceability</code>, <code>energy</code>, <code>acousticness</code>, <code>instrumentalness</code>, <code>valence</code>, and <code>track_genre</code>.</li>
<li>The structure of the dataset supports analysis at both the individual track level and the aggregated genre level.</li>
</ul>
<h2 id="data-preparation">4. Data Preparation</h2>
<h3 id="cleaning">4.1 Cleaning</h3>
<ul>
<li>Two separate tables were created to support different analytical needs: a <strong>deduplicated track-level dataset</strong> and a <strong>genre-normalized dataset</strong>.</li>
<li>Duplicate records were identified using <code>track_id</code>. Multiple entries existed for the same track due to factors such as single releases, album inclusions, featured artist variations, and compilation appearances. In some cases, a single track appeared up to <strong>eight times</strong>.</li>
<li>To address this, tracks were deduplicated by retaining the record with the <strong>highest popularity score</strong> for each <code>track_id</code>. This approach preserved the most representative version of each track while preventing over-representation in analysis. The resulting deduplicated dataset contained approximately <strong>89,000 unique tracks</strong>.</li>
</ul>
<h3 id="genre-normalization">4.2 Genre Normalization</h3>
<ul>
<li>The original dataset included <strong>114 genres</strong>, many of which were highly specific or used virtually identical language (<em>ex: “alt-rock” and “alternative”</em>). To improve interpretability and create stronger comparisons, genres were consolidated into <strong>15 broader genre buckets</strong>.<br>
<a href="https://drive.google.com/file/d/1GBSi4xNQGcFlpxwtaeTdnbp7wndkTKY3/view?usp=sharing">Genre Buckets</a></li>
<li>Genres were grouped based on shared musical characteristics, instrumentation, and stylistic similarities. For example, subgenres such as Alternative, Hardcore, and Metal were grouped under the broader <strong>Rock</strong> category.</li>
<li>This normalization reduced noise caused by overly specific genre labels and allowed for clearer analysis of popularity and audio features at the genre level.</li>
</ul>
<h2 id="exploratory-analysis">5. Exploratory Analysis</h2>
<ul>
<li>Initial exploration revealed a strong weighting toward <strong>Rock</strong>, which consolidated <strong>22 related subgenres</strong> into one genre bucket. <strong>Pop</strong> also proved a dominant category, both in track volume and average popularity, consistent with its broad listener appeal.</li>
<li>When average popularity was aggregated by genre, the distribution largely aligned with expectations: <strong>Pop, Rock, and HipHop</strong> ranked among the most popular genres overall. However, one notable finding was the comparatively high popularity of the <strong>Ambient–Chill</strong> category. Genres such as <em>chill</em>, <em>study</em>, and <em>sleep</em> demonstrated sustained popularity, suggesting that Spotify listeners frequently engage with music intended for background listening, relaxation, or focus rather than active consumption.<br>
<a href="https://drive.google.com/file/d/1o152xjZXEk2ePnOeqUoo9p3ozp1RChQb/view?usp=sharing">Genre Popularity Chart</a>
<ul>
<li>This observation highlights that popularity on Spotify is not solely driven by traditional radio-oriented genres, but also by use-case listening behavior.</li>
</ul>
</li>
</ul>
<h2 id="genre-level-analysis-and-characteristic-comparison">6. Genre-Level Analysis and Characteristic Comparison</h2>
<h3 id="popularity-by-genre">6.1 Popularity by Genre</h3>
<ul>
<li>After establishing how popularity is distributed across normalized genre buckets, the analysis shifted toward identifying <strong>musical characteristics associated with higher- and lower-popularity genres</strong>. Rather than evaluating individual tracks, genre-level averages were calculated to capture broader stylistic tendencies.</li>
</ul>
<p><a href="https://drive.google.com/file/d/1OUuhXS4VgVsZEpmsCku3R4JoJ-SXKU01/view?usp=sharing">Characteristic Scatterplots</a></p>
<ul>
<li>Using SQL, average values for key audio characteristic were computed for each normalized genre bucket. This approach allowed for comparison of musical “profiles” across genres while minimizing the influence of individual outliers.</li>
</ul>
<pre><code>SELECT
	g.normalized_genre,
	COUNT(*)  AS  total_tracks,
	AVG(d.popularity)  AS  avg_popularity,
	AVG(d.danceability)  AS  avg_danceability,
	AVG(d.energy)  AS  avg_energy,
	AVG(d.valence)  AS  avg_valence,
	AVG(d.acousticness)  AS  avg_acousticness,
	AVG(d.instrumentalness)  AS  avg_instrumentalness,
	AVG(d.loudness)  AS  avg_loudness,
	AVG(d.tempo)  AS  avg_tempo
FROM  `spotifydataset-479318.spotifyDataset.cleaned_deduped`  d
JOIN  `spotifydataset-479318.spotifyDataset.normalizedGenre`  g
ON  d.track_id = g.track_id
GROUP  BY  g.normalized_genre
ORDER  BY  avg_popularity  DESC;
</code></pre>
<ul>
<li>The results showed that genres with higher average popularity generally exhibited <strong>higher energy, greater danceability, and more positive valence (emotion)</strong>, suggesting a preference for rhythmically engaging and emotionally upbeat music. In contrast, genres characterized by high acousticness or instrumentalness tended to exhibit lower average popularity, indicating that more subdued or non-vocal styles may appeal to narrower listener segments. Still, the outlier remained <strong>Ambient-Chill</strong> as it sat high in popularity but low in energy, danceability, and valence.</li>
<li>Tempo was evaluated alongside these features but demonstrated minimal variation across popularity levels and did not display a consistent directional relationship with popularity. As a result, it was deprioritized in subsequent interpretation.</li>
</ul>
<h3 id="genre-weight-in-dataset">6.2 Genre Weight in Dataset</h3>
<ul>
<li>For more complete context in popularity metrics, the dataset was examined for genre representation, referred to here as <em>genre weight</em>. Genre weight reflects the number of tracks assigned to each normalized genre and provides insight into how heavily each genre is represented in the Spotify catalog.</li>
<li>The distribution of tracks across genres was highly uneven. Rock-related tracks accounted for the largest share of the dataset, with over 22,000 tracks falling into the Rock genre bucket (<em>i.e. 22 subgenres converged into one “Rock” genre</em>). Pop and Dance-Electronic were also heavily represented, while genres such as Classical, Comedy, and Jazz-Blues contained significantly fewer tracks.</li>
<li>Understanding genre weight is essential because genres with larger catalogs may experience greater internal competition, while smaller genres may achieve higher average popularity with fewer releases.</li>
</ul>
<h2 id="key-insights">7. Key Insights</h2>
<p>Based on the analysis of musical features, popularity, and genre distribution, several actionable insights emerge that may inform decision-making for independent artists and marketing teams.</p>
<ul>
<li>
<p><strong>Higher-popularity tracks tend to share elevated energy and danceability</strong>, particularly within Pop, HipHop/RnB, and Dance–Electronic genres. This suggests that rhythmically upbeat production and strong dynamic presence are common characteristics among widely consumed tracks, especially in contexts where listener engagement is active.</p>
</li>
<li>
<p><strong>Ambient and low-energy genres demonstrate sustained popularity despite lower energy and danceability scores</strong>, indicating that success on Spotify is not limited to high-intensity music. Genres associated with relaxation, focus, or background listening perform well in playlist-driven environments, highlighting the importance of aligning musical style with listener use cases.</p>
</li>
<li>
<p><strong>Tracks with higher valence are more prevalent among higher-popularity genres</strong>, suggesting a general listener preference for emotionally positive or uplifting sound profiles. Thnis isn’t a requirement for a successful track,  but this trend indicates that mood and emotional tone may influence engagement, particularly in mainstream contexts.</p>
</li>
<li>
<p><strong>Highly instrumental or acoustically dominant tracks tend to cluster at lower popularity levels</strong>, implying a narrower audience reach. For independent artists, this should not signal avoidance, but rather emphasizes the need for targeted marketing strategies when working within more specialized or niche sound profiles.</p>
</li>
</ul>
<p>Collectively, these findings suggest that popularity on Spotify is shaped by a combination of <strong>sonic features, genre context, and listener intent</strong>. Artists and marketing teams may benefit from tailoring production and promotional strategies to align with how listeners are most likely to engage with specific styles of music.</p>
<h2 id="recommendations">8. Recommendations</h2>
<p><strong>Marketing Consultants</strong></p>
<ul>
<li>Position independent artists based on listener use cases rather than genre labels alone. High-energy, danceable tracks align well with active-listening playlists, while lower-energy, ambient tracks perform strongly in focus and relaxation contexts. Marketing strategies should reflect these distinct consumption patterns.</li>
</ul>
<p><strong>Artist Development</strong></p>
<ul>
<li>Artists seeking broader reach may benefit from incorporating higher energy, danceability, or positive emotional tone into their work; however, success is not limited to these characteristics. Artists producing ambient or acoustic music can achieve consistent engagement when their sound aligns with playlist-driven listening behavior.</li>
</ul>
<h2 id="limitations">9. Limitations</h2>
<ul>
<li>The dataset does not include release dates, which prevents analysis of popularity trends over time and limits the ability to account for recency effects or changing listener preferences.</li>
<li>Spotify’s popularity metric is platform-defined and opaque, meaning it reflects relative engagement rather than absolute listen counts or commercial success.</li>
<li>Genre labels were manually normalized into broader buckets to support analysis, which introduces subjectivity and may obscure nuances within subgenres.</li>
</ul>
<h2 id="tableau-dashboards">10. Tableau Dashboards</h2>
<p>This dashboard explores the relationship between musical audio features, genre categories, and track popularity on Spotify. It compares average popularity across normalized genres while highlighting how key features such as energy, danceability, valence, and acousticness vary between genres. The visualizations support an examination of how different listening contexts and musical characteristics align with popularity patterns.</p>
<div class="tableauPlaceholder" id="viz1767812841578"><a href="#"><img alt="Spotify Audio Features and Genre PopularitySource: https://huggingface.co/datasets/maharshipandya/spotify-tr   " src="https://public.tableau.com/static/images/Sp/SpotifyAudioFeaturesandGenrePopularity/Dashboard1/1_rss.png"></a>   </div>                

