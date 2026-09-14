# Cardiology & Heart Health Research Digest

A GitHub Actions workflow that searches curated cardiovascular medicine, hematology and vascular disease, and pulmonary medicine journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Email notification** - Sends a short email with study count and a dashboard link

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as Everyday Health, Healthline, Prevention, WebMD, Heart.org, STAT, MedPage Today, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Cardiology & Heart Health Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Cardiology | 157 | 2 (chunks 1-2) |
| Hematology & Vascular | 194 | 2 (chunks 1-2) |
| Pulmonary Medicine | 44 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

The journal CSVs in `data/` are now hand-maintained. `scripts/extract_journals.py` originally generated them from a source workbook that no longer exists, so re-running it would wipe hand-added rows. Add new journals by appending rows to the relevant CSV.

## Journal list audit (2026-09-14)

**Method** - Pulled OpenAlex's top sources for this digest's subject areas (cardiology, hematology and vascular, pulmonary) over the prior year, diffed them against the CSVs by ISSN and title, and kept only titles PubMed indexes with at least 20 articles in the past 12 months. Every candidate was then screened by hand for beat fit, pitchable research content, and volume, since adding a journal pulls its entire weekly output into the digest.

**Added (13)**

| Journal | ISSN | Category | PubMed/yr |
|---|---|---|---:|
| JACC: Advances | 2772-963X | Cardiology | 944 |
| JACC: Asia | 2772-3747 | Cardiology | 449 |
| Heart Rhythm O2 | 2666-5018 | Cardiology | 302 |
| European Heart Journal - Digital Health | 2634-3916 | Cardiology | 201 |
| CJC Open | 2589-790X | Cardiology | 195 |
| European Heart Journal Open | 2752-4191 | Cardiology | 191 |
| Cardio-Oncology | 2057-3804 | Cardiology | 164 |
| ERJ Open Research | 2312-0541 | Pulmonary Medicine | 615 |
| BMJ Open Respiratory Research | 2052-4439 | Pulmonary Medicine | 303 |
| Archivos de Bronconeumología | 1579-2129 | Pulmonary Medicine | 288 |
| Pulmonary Circulation | 2045-8940 | Pulmonary Medicine | 230 |
| eJHaem | 2688-6146 | Hematology & Vascular | 229 |
| Blood Vessels, Thrombosis & Hemostasis | 2950-3272 | Hematology & Vascular | 113 |

Archivos de Bronconeumología has a Spanish title, but every PubMed record from the past year carries English-language content, so it was kept.

**Notable exclusions**

- **Mega-journal volume** - Frontiers in Cardiovascular Medicine (~2,500 PubMed articles/yr) would swamp the 30-candidate cap. Journal of Cardiovascular Development and Disease (MDPI) was left out on reputation.
- **Case reports** - JACC Case Reports, European Heart Journal - Case Reports, HeartRhythm Case Reports, Respirology Case Reports, and Journal of Cardiology Cases.
- **Off-beat titles OpenAlex lumped in** - Oncology (Annals of Oncology, Journal of Thoracic Oncology, Lung Cancer, Clinical Lung Cancer, Thoracic Cancer, Translational Lung Cancer Research, International Journal of Radiation Oncology Biology Physics, Blood Neoplasia), urology (BJU International, World Journal of Urology, European Urology Oncology and Open Science, The Prostate, and others), nephrology (JASN, Nephrology Dialysis Transplantation), thoracic surgery (Annals of Thoracic Surgery, Journal of Thoracic Disease, JTCVS Open, JTCVS Techniques), and Bone Marrow Transplantation.
- **Supplements** - European Heart Journal Supplements.
- **Non-English national titles** - Hämostaseologie, Acta Haematologica Polonica, Transfusion Medicine and Hemotherapy.
- **Lower priority** - Imaging-technical titles (Journal of the American Society of Echocardiography, International Journal of Cardiovascular Imaging, EHJ Imaging Methods and Practice) because the list already carries several imaging journals; basic-science titles (American Journal of Respiratory Cell and Molecular Biology, Journal of Molecular and Cellular Cardiology Plus); Pediatric Pulmonology, which the pediatric-health digest already carries; and smaller regional or open-access titles (Circulation Reports, American Heart Journal Plus, IJC Heart & Vasculature, Journal of Arrhythmia, Structural Heart, Indian Journal of Hematology and Blood Transfusion, TH Open, and others). Reviews in Cardiovascular Medicine was left out because it is review-heavy and IMR Press titles are not MEDLINE-indexed.
- **High-volume titles not in PubMed** - Revista Argentina de Cardiología (~780 topic articles/yr in OpenAlex, 0 in PubMed), Journal of Vascular Surgery Cases and Innovative Techniques (0 in PubMed over the year), Russian Journal of Cardiology, Pakistan Heart Journal, Egyptian Journal of Bronchology, and European Heart Journal - Valvular and Structural Heart Disease (a newer ESC title not yet in NCBI's journal list; worth rechecking later).

## Manual Trigger

Go to **Actions -> Cardiology & Heart Health Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Cardiology`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_API_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (dashboard save/delete personalization) |
| `SUPABASE_KEY` | Supabase API key (read-only) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to the shared `research-digest-dashboard` repo |

## Repo Structure

```text
.github/
  workflows/
    cardiology-heart-digest.yml
scripts/
  cardiology_heart_digest.py
  merge_results.py
  extract_journals.py
data/
  Cardiology.csv
  Hematology & Vascular.csv
  Pulmonary Medicine.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for heart health, stroke, blood disease, and cardiovascular risk journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
