# How to Scrape Vagas.com.br: Brazilian Job Postings (Vagas de Emprego)

> **Run it now, no code: [vagas-jobs-scraper on the Apify Store](https://apify.com/bibian/vagas-jobs-scraper)** &nbsp;|&nbsp; $1.80 per 1,000 job postings &nbsp;|&nbsp; no login, no browser, no proxy

Vagas.com.br is one of Brazil's largest job boards, with no public export and no official API for third parties. This guide shows how to get its postings as clean structured data: title, company, city, contract type, experience level and salary (when published), with a free Apify Actor, from the console or from code.

Searches this page answers: vagas.com.br scraper, vagas de emprego API, Brazilian job board scraper, Brazil jobs data.

## What you get from the Vagas.com.br scraper

- Every open job posting, with title, company, contract type (CLT, estágio, PJ...) and experience level.
- Location as a normalized `city`/`region`/`country`, plus the site's own display label.
- Salary (`salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`) for the share of postings that publish a real range instead of "a combinar" (negotiable).
- Field names shared with our other job-board Actors (`JobPosting`-based, schema.org names), so a Brazilian source drops into the same pipeline as the others.
- Optional full job description as plain text.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results from a live run) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Vagas.com.br posting id and public link |
| `title` | Job title as published |
| `companyName`, `confidentialCompany` | Hiring company, or `null`/`true` for the postings that keep the employer confidential |
| `locationLabel`, `city`, `region`, `country` | Where the job is based |
| `contractType`, `employmentType` | The site's own contract label (e.g. "Regime CLT") and a normalized type (`full_time`, `internship`...) |
| `experienceLevel` | Seniority level as the site labels it (e.g. "Pleno", "Júnior/Trainee") |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range in BRL, `null` when the posting doesn't publish one |
| `datePosted`, `validThrough` | ISO dates for publication and expiry |
| `skills` | Array of skills, when the posting lists any |
| `description` | Full plain-text description, only when **Include descriptions** is on |

## Use cases

- **Feed a job-aggregation pipeline** with Brazilian postings in the same schema as our other job-board Actors.
- **Market research**: track which companies are hiring, for which roles, in which Brazilian city, over time.
- **Lead generation** for recruiting or HR-tech tools: company names, contract types and vacancy counts, no personal data.
- **Salary benchmarking** for the postings that publish an actual range instead of "a combinar".

## Input

| Field | What it does |
|---|---|
| `city` | Limits results to one Brazilian city as it appears on the site (e.g. `"São Paulo"`, `"Rio de Janeiro"`). Empty browses every city. |
| `keywords` | Keeps only postings whose title contains this text (case-insensitive). |
| `maxItems` | Stops the run once this many postings are saved. Default 50. |
| `includeDescriptions` | Adds the full job description as plain text. Default `true`. |
| `onlyNew` | For scheduled runs: only returns (and charges for) postings a previous run of the same search hasn't already delivered. Default `false`. |

Example input:

```json
{
  "city": "São Paulo",
  "maxItems": 50,
  "includeDescriptions": true
}
```

## Four ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/vagas-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (city, keywords, maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor with **Only new results** turned on.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/vagas-jobs-scraper').call({
    "city": "São Paulo",
    "maxItems": 50,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName} | ${item.locationLabel} | ${item.contractType}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/vagas-jobs-scraper").call(run_input={
    "city": "São Paulo",
    "maxItems": 50,
    "includeDescriptions": True,
})

for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('locationLabel')} | {item.get('contractType')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

### 4. Only new postings every day

Turn on `onlyNew` and schedule the Actor once a day: each run then returns, and charges for, only the postings a previous run of the same search hasn't already delivered.

## Pricing

Pay per result: **$1.80 per 1,000 job postings** (that is $0.0018 per posting), plus a tiny start fee per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/vagas-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- Only what Vagas.com.br publishes is collected. Many postings do not show a salary range, so `salaryMin`/`salaryMax` are often `null` (visible in `sample-output.json`).
- The dataset contains job postings only. No individual candidate data is ever collected.
- Postings are the public ones: anything behind a login is out of scope.
- This is an unofficial tool, not affiliated with Vagas.com.br. Check the site terms that apply to your use.

## FAQ

**Does it need a browser or a proxy?**  
No. The Actor reads the data directly, so runs are fast and cheap.

**How do I only get new postings every day?**  
Turn on `onlyNew` and schedule the Actor once a day.

**Can I filter by city?**  
Yes, set `city` to any Brazilian city as it appears on the site, for example `"Rio de Janeiro"`.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open vagas-jobs-scraper on the Apify Store](https://apify.com/bibian/vagas-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
