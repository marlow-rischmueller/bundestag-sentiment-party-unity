# Unified or Divided Through Sentiment?
### A Party Unity Analysis of German Parliamentary Speeches

> Data Science Course Project · M.Sc. Management Information Systems · University of Bremen · 2023/24  
> Team of 3 · Project lead: Marlow Rischmüller (programming & data processing)

Sentiment analysis of 17,198 Bundestag speeches to investigate intra-party unity across the German political spectrum - does sentiment homogeneity differ between governing parties, opposition parties, and fringe parties?

---

## Motivation

Research on party unity traditionally relies on roll-call voting data - the final recorded votes of politicians. This captures *discipline* but not the debates that precede it. Parliamentary speeches may reveal the *true* mood of politicians more openly, since giving a speech is less consequential than casting a vote.

This project shifts the focus from voting records to speech sentiment, using the 20th German legislative period (since November 2021) as its dataset. Two hypotheses are tested:

- **H1:** Governing parties are more unified in sentiment than opposition parties
- **H2:** Fringe parties (positioned at the edges of the political spectrum) are more unified than centrist parties

---

## Methodology

### 1 · Data Collection

All available plenary protocols of the German Bundestag for the 20th legislative period were collected directly from the official Bundestag website as XML files. After removing the first protocol (which contained only procedural content), **139 protocols** from 11 November 2021 to 29 November 2023 were used, containing **17,198 speeches** in total.

### 2 · Parsing & Preprocessing

A custom XML parser was built to extract speeches along with speaker metadata and party affiliation. The parser:
- Drops the final paragraph of each speech (always the session president's acknowledgement)
- Removes interjections, quotes, and comments by other members of parliament
- Handles 31 speeches with malformed or incomplete XML tags (manually corrected)

Speakers with official government roles (e.g. chancellor, federal ministers) were initially untagged - their party affiliations were added manually. Six speeches with ambiguous or dual-party attribution were dropped, resulting in **7 party groups** for analysis.

### 3 · Sentiment Scoring

German stopwords (Snowball stopword list) were removed before tokenisation via NLTK. Each token was looked up in the **SentiWS sentiment dictionary** (v1.8c, University of Leipzig), a validated lexicon for German political language containing positive and negative sentiment scores. Each speech received a cumulative sentiment score.

> **Note:** The paper references the sentiment dictionary by Rauh (2018); the implementation uses SentiWS v1.8c, which served as the practical resource for scoring.

### 4 · Analysis

Intra-party sentiment variation was assessed via:
- **Box plots** per party (score distribution, outliers, spread)
- **Coefficient of Variation (CV)** - standard deviation as a percentage of the mean, enabling comparability across parties with different score magnitudes

---

## Results

### Coefficient of Variation by Party

| Party | CV | Government / Opposition |
|---|---|-------------------------|
| AfD | 146.63% | Opposition              |
| DIE LINKE | 151.64% | Opposition              |
| Fraktionslos | 198.23% | -                       |
| SPD | 449.18% | Government              |
| CDU/CSU | 477.58% | Opposition              |
| FDP | 616.73% | Government              |
| BÜNDNIS 90/DIE GRÜNEN | 9,369.36% | Government              |

### Interpretation

**H1 rejected:** Governing parties (SPD, Grünen, FDP) show substantially *higher* sentiment variation than opposition parties - the opposite of the hypothesis. This is consistent with Sieberer (2020), who found that governmental participation tends to reduce party unity.

**H2 supported:** Fringe parties (AfD, DIE LINKE) have by far the lowest coefficients of variation, indicating more homogeneous sentiment across speeches. Political isolation appears to reinforce internal cohesion.

The extreme CV of Bündnis 90/Die Grünen (9,369%) may be partly explained by the party's internal wing logic ("Realos" vs. "Fundis"), though this remains a hypothesis for future research.

---

## Limitations

- Sentiment scores are derived from lexical content only - tone of voice, facial expression, irony, and sarcasm are not captured
- Sentiment represents only one dimension of party unity; thematic agreement within speeches is not assessed
- Dataset covers November 2021 – November 2023 and does not include later developments (e.g. formation of BSW)

---

## Repository Structure

```
.
├── sentiment_analysis_pipeline.ipynb   # Full pipeline: parsing → preprocessing → scoring → visualisation
├── data_wp20/                          # Bundestag XML protocols (not included - see Setup)
│   └── ReadME.txt                      # Instructions for obtaining and placing the data
├── rating_csvs/                        # Per-party sentiment score CSVs (generated by notebook)
│   ├── ratings0.csv – ratings6.csv     # Individual party ratings
│   ├── combined_ratings.csv            # All parties combined
│   └── combined_stats.csv              # Descriptive statistics per party
├── SnowballStopwordsGerman.txt         # Stopword list (not included - see Setup)
├── SentiWS_v1.8c_Positive.txt          # Sentiment lexicon (not included - see Setup)
└── SentiWS_v1.8c_Negative.txt          # Sentiment lexicon (not included - see Setup)
```

> The `SnowballStopwordsGerman.txt` and `SentiWS_v1.8c_*.txt` files are **not included**. See [Setup](#setup--usage) for download instructions.

---

## Technologies

| Category | Libraries / Tools |
|---|---|
| Parsing | BeautifulSoup4, lxml |
| NLP | NLTK (tokenisation, stopword removal) |
| Data processing | pandas, NumPy |
| Visualisation | seaborn, matplotlib |
| Sentiment lexicon | SentiWS v1.8c (University of Leipzig) |
| Stopwords | Snowball German stopword list |

---

## Setup & Usage

### 1 · Install dependencies

```bash
pip install beautifulsoup4 lxml nltk pandas numpy seaborn matplotlib wordcloud tqdm
```

### 2 · Download required external resources

**SentiWS sentiment dictionary** (University of Leipzig, required for scoring):
- Download from: https://wortschatz.uni-leipzig.de/en/download/
- Place `SentiWS_v1.8c_Positive.txt` and `SentiWS_v1.8c_Negative.txt` in the project root directory

**Snowball German stopword list** (Dr. Martin Porter, University of Cambridge):
- Download from: http://snowball.tartarus.org/algorithms/german/stop.txt
- Save as `SnowballStopwordsGerman.txt` in the project root directory

### 3 · Download Bundestag protocols

Download the XML protocols of the 20th legislative period from the official Bundestag website:
https://www.bundestag.de/dokumente/protokolle

Place all `.xml` files into the `data_wp20/` directory. The notebook expects 139 protocols (11 November 2021 – 29 November 2023).


### 4 · Run the notebook

Open `sentiment_analysis_pipeline.ipynb` and run all cells from top to bottom. The notebook is structured as a linear pipeline:

1. Dependency installation & imports
2. Stopword and sentiment lexicon loading
3. XML parsing & speech extraction
4. Party grouping & manual speaker assignment
5. Tokenisation & sentiment scoring
6. Export to `rating_csvs/`
7. Visualisation (box plots, bar charts)

---

## Citation & Acknowledgements

**SentiWS:**
```
Remus, R., Quasthoff, U., & Heyer, G. (2010). SentiWS - A Publicly Available German-language Resource for Sentiment Analysis.
Proceedings of the 7th International Language Resources and Evaluation (LREC'10).
https://wortschatz.uni-leipzig.de/en/download/German
```

**Sieberer (2020):**
```
Sieberer, U. (2020). Party unity in parliamentary democracies.
```

---

## License

This project is licensed under the [MIT License](LICENSE).  
External resources (SentiWS, Snowball stopwords) are subject to their respective licenses and are not included in this repository.
