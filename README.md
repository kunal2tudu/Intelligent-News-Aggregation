# Intelligent News Aggregation

An automated Python script that aggregates news from multiple high-profile sources (The New York Times, The Guardian, BBC News, The Washington Post, and Al Jazeera) using their RSS feeds. 

It generates both a consolidated Microsoft Word document (`.docx`) and a heavily styled "Urban Theme" HTML dashboard report with a timeline of events.

## Features
- **Multi-Source Aggregation**: Fetches and parses RSS feeds from major global news outlets.
- **Automated Reporting**: Automatically generates reports in `reports/` with today's date.
- **Urban Dashboard**: Creates a dynamic, cyber-punk themed HTML dashboard.
- **Timeline View**: Automatically identifies and constructs a cohesive timeline of major geopolitical events from the fetched news.
- **Word Document Output**: Compiles parsed news articles, including fetched inline images, into a neatly formatted `.docx` file.

## Requirements
- Python 3.x
- `feedparser`
- `python-docx`
- `requests`
- `beautifulsoup4`
- `python-dateutil`

## Installation
Ensure you have the required Python packages installed:
```bash
pip install feedparser python-docx requests beautifulsoup4 python-dateutil
```

## Usage
Simply run the Python script:
```bash
python news_scraper.py
```
By default, the script will fetch all of today's news from the configured sources. If you want to grab a specific number of articles (e.g. 5) instead of just today's news, you can use:
```bash
python news_scraper.py --all-today False -n 5
```

The resulting `urban_news_[date].docx` and `urban_news_[date].html` will be saved in the locally created `reports/` folder.
