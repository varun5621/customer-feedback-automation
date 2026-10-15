# Customer Feedback Analysis & Automated Response

Python solution for the Imarticus Data Science Internship assessment (Python Foundations & Gen AI). It finds the critical customer reviews with plain Python rules (no machine learning) and uses Google Gemini to draft a personalised, empathetic apology email for the 3 most critical ones.

## Main deliverable
`feedback_analysis_gemini.ipynb` - the full pipeline with comments, markdown insights and the 3 generated emails.

## Approach

### 1. Data cleaning (Pandas)
- **Dataset:** *Women's E-Commerce Clothing Reviews* from Kaggle (text reviews plus 1-5 star ratings), but the notebook is built to accept almost any review file: Amazon, Yelp, app-store, Twitter, hotel, IMDB-style or a company export.
- **Reading the file:** CSV, TSV, TXT, Excel (uses the sheet with the most rows) and JSON. It detects the separator (comma, semicolon, tab, pipe), the text encoding, files with no header row, and skips broken lines.
- **Finding the columns:** it checks column names and column contents to find the review text, rating, customer name, email and ID. A review title or summary column is added in front of the review text. A one-line override is provided if a guess is wrong.
- **Ratings on any scale:** 1-5, 0-5, 1-10 (also with decimal commas), 0-100 and 0-1 are converted to 1-5 stars, as are text such as "4.0 out of 5 stars", two-value labels (0/4, 0/1) and sentiment labels (negative / neutral / positive). The conversion is shown in a table.
- **Missing values:** rows with no review text or no usable rating are removed. Optional fields (name, email, ID) are filled with empty text.
- **Duplicates:** rows with the same review, customer and order ID are shown, then only the first copy is kept.
- **Text cleaning** (new `clean_text` column; the original text is kept for the emails): HTML entities decoded, lowercase, contractions expanded, links and HTML removed, special characters and digits removed, extra spaces collapsed. Letters from any language are kept.
- Every step prints rows before, after and removed, and a markdown summary table is shown.

### 2. Rule-based logic (no machine learning)
- **Critical reviews:** rating of 1 or 2 stars.
- **Priority by stars (after conversion to a 1-5 scale):** 1 = P1 Critical, 2 = P2 High, 3 = P3 Medium, 4-5 = P4 Low.
- **Ordering inside a priority level:** an urgency score = 2 points per complaint word, 4 per urgent phrase (e.g. "chargeback", "still waiting"), up to 3 for exclamation marks, up to 5 for review length, and 2 per extra problem type.
- **Complaint keywords:** the function `top_keywords()` tokenises the cleaned critical reviews, removes stop-words and counts words with `collections.Counter`.
- **No rating column?** Only reviews with complaint or urgent wording are kept and they are ranked by the urgency score.
- **Complaint themes:** Delivery, Quality/Damage, Customer Service, Wrong Item/Size, Refund/Billing, General Dissatisfaction, found with a keyword list.
- **Top 3 selection:** the 3 highest-ranked critical reviews (lowest stars first, then highest urgency score, duplicates removed).

### 3. Generative AI outreach (Google Gemini)
- Each of the 3 reviews is sent to Gemini with a Customer Support Agent system prompt.
- Gemini reads the **whole** review, chooses the tone itself (empathetic, apologetic, reassuring, friendly, grateful or formal) and says why.
- It returns a subject line, a short email (max 130 words) that apologises, addresses each specific complaint and offers a concrete solution, and a list of steps for the support team.
- If no key is set or the call fails, a clearly labelled rule-based template is used instead (not AI).

## How to run

### 1. Put your Gemini key in a `.env` file (once)
1. Get a free key at https://aistudio.google.com/app/apikey.
2. Copy `.env.example` to a new file named exactly `.env` (same folder as the notebook).
3. Open it and replace `your-key-here` with your key:
   ```
   GEMINI_API_KEY=AIza...your-real-key
   ```
The notebook (and the web app) read this file automatically, so nobody has to type the key again. The key is never written inside the notebook. `.gitignore` already excludes `.env` so it is not uploaded to Git by accident.

Key lookup order in the notebook: `.env` file, Colab secret `GEMINI_API_KEY`, environment variable, then a hidden prompt that can save the key to `.env` for next time.

### 2a. Google Colab
1. Open https://colab.research.google.com and upload `feedback_analysis_gemini.ipynb`.
2. Runtime -> Run all. In the file picker (step 2), select your review file **and** your `.env` file together.

### 2b. Jupyter on your computer
```bash
pip install pandas openpyxl google-genai ipywidgets jupyter
jupyter notebook feedback_analysis_gemini.ipynb
```
Keep `.env` in the same folder as the notebook, run all cells, then choose the review file with the upload button and run the cells after it.

### Sharing the project
Send the notebook, `README.md` and `.env.example`. The other person creates their own free key and pastes it into `.env` once. Do not send your own `.env`, and do not submit it to the assessment or upload it to GitHub: anyone who has the key can use your Gemini quota.

Outputs: `reply_drafts.csv` (the 3 emails and solution steps) and `cleaned_reviews.csv`.

## Optional extras (not required by the assessment)
A web interface (it uses simpler column detection than the notebook): `server.py`, `static/index.html`, `feedback_engine.py` (run `python server.py`), and a Streamlit version `app.py`. Sample data: `sample_reviews.csv`.
