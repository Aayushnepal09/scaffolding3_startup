# Operational Assignment 3: Text Preprocessing Web Service

A simple web service that fetches and cleans text from Project Gutenberg URLs, built with Flask and Python.

## Setup

1. Make sure you have Python 3.9 or higher installed:
```bash
python --version
```

2. Install the required packages:
```bash
pip install -r requirements.txt
```

3. Verify the environment is working:
```bash
python test_setup.py
```
You should see all tests passing with green checkmarks.

## How to Run

Start the Flask server:
```bash
python app.py
```

Then open your browser and go to: http://localhost:5000

Press `Ctrl+C` to stop the server.

## What I Did

### Part 1: Environment Setup
- Forked the starter repository and set up the development environment
- Ran `test_setup.py` to verify everything was installed correctly
- All 4 checks passed (Python version, packages, basic functionality, Gutenberg access)

### Part 2: Text Preprocessor Methods (starter_preprocess.py)

I implemented three methods in the `TextPreprocessor` class:

**fetch_from_url(url)**
- Takes a URL pointing to a `.txt` file
- Validates that the URL actually ends with `.txt` before making the request
- Uses the `requests` library to download the text with a 30 second timeout
- Raises an error if the URL is bad or the server cant be reached

**get_text_statistics(text)**
- Uses the existing `tokenize_words()` and `tokenize_sentences()` methods from the class
- Calculates total characters, words, and sentences
- Computes average word length and average sentence length (with checks for division by zero)
- Uses Python's `Counter` to find the top 10 most common words
- Returns everything in a dictionary

**create_summary(text, num_sentences=3)**
- Uses `tokenize_sentences()` to split the text into sentences
- Grabs the first N sentences (default is 3)
- Joins them back together with periods and returns the summary string

### Part 3: Flask API Endpoints (app.py)

I implemented two API endpoints:

**POST /api/clean**
- Accepts JSON with a `url` field (e.g. `{"url": "https://www.gutenberg.org/files/1342/1342-0.txt"}`)
- Fetches the text, cleans the Gutenberg headers/footers, normalizes it
- Returns the cleaned text, statistics, and a 3-sentence summary
- Returns an error if the URL is invalid or something goes wrong

**POST /api/analyze**
- Accepts JSON with a `text` field (e.g. `{"text": "some raw text here"}`)
- Runs the statistics on whatever text you give it
- Returns just the statistics (no cleaning or summary)

### Part 4: Web Interface (templates/index.html)

- Implemented the JavaScript form submission handler
- When you submit a URL, it sends a POST request to `/api/clean`
- Shows a loading spinner while the text is being fetched and processed
- Displays the results (statistics, summary, text preview) when done
- Shows an error message if something goes wrong

## Example URLs to Test

- **Pride and Prejudice**: https://www.gutenberg.org/files/1342/1342-0.txt
- **Frankenstein**: https://www.gutenberg.org/files/84/84-0.txt
- **Alice in Wonderland**: https://www.gutenberg.org/files/11/11-0.txt
- **Moby Dick**: https://www.gutenberg.org/files/2701/2701-0.txt

## Testing with curl

You can also test the API from the command line:

```bash
# Test the clean endpoint
curl -X POST http://localhost:5000/api/clean \
  -H "Content-Type: application/json" \
  -d '{"url": "https://www.gutenberg.org/files/11/11-0.txt"}'

# Test the analyze endpoint
curl -X POST http://localhost:5000/api/analyze \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello world. This is a test. Another sentence here."}'

# Health check
curl http://localhost:5000/health
```

## Project Structure

```
OA3/
├── README.md               <- This file
├── requirements.txt        <- Python dependencies
├── test_setup.py           <- Environment validation script
├── app.py                  <- Flask application with API endpoints
├── starter_preprocess.py   <- Text preprocessing class with all methods
└── templates/
    └── index.html          <- Web interface
```

## Notes

- Large books like Moby Dick might take a few seconds to process since the whole text gets downloaded and analyzed
- The text statistics are calculated on the normalized (lowercased, cleaned) version of the text
- The summary is just the first 3 sentences - its a simple extractive approach, nothing fancy
- I had to fix a unicode issue with the smart quotes regex in `normalize_text()` since the original regex characters weren't being parsed correctly on Windows
