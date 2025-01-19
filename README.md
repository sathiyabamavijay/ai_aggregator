# Install necessary libraries
!pip install transformers feedparser

# Import required modules
from transformers import pipeline
import feedparser

# Initialize the summarizer model
summarizer = pipeline("summarization", model="sshleifer/distilbart-cnn-12-6")

# Function to fetch news from an RSS feed
def fetch_news(feed_url):
    # Parse the RSS feed
    feed = feedparser.parse(feed_url)
    articles = []
    for entry in feed.entries:
        articles.append({
            'title': entry.title,
            'summary': entry.summary if 'summary' in entry else '',  # Use empty string if no summary
            'link': entry.link
        })
    return articles

# Provide a valid RSS feed URL
rss_url = "https://news.google.com/rss?hl=en-US&gl=US&ceid=US:en"  # Google News RSS feed
news = fetch_news(rss_url)

# Summarize and display the fetched articles
if news:
    print(f"Fetched {len(news)} articles.\n")
    for i, article in enumerate(news[:5]):  # Summarize the first 5 articles
        print(f"Article {i+1}: {article['title']}")
        print(f"Link: {article['link']}\n")

        # Check if the article has content to summarize
        if not article['summary']:
            print("No summary available for this article.\n")
            continue

        # Truncate text to the maximum model input length
        truncated_summary = article['summary'][:1024]
        try:
            # Summarize the article
            summary = summarizer(truncated_summary, max_length=50, min_length=20, do_sample=False)
            print(f"Summary: {summary[0]['summary_text']}\n")
        except Exception as e:
            print(f"Error summarizing article {i+1}: {e}\n")
else:
    print("No articles were fetched. Check the RSS feed URL.")
