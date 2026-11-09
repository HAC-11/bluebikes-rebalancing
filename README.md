Rebalancing Bluebikes Across Greater Boston
IE 5374 · Module 7 Hackathon

Team: Ahmed, Hetvi, Pooja

An event-aware, GenAI-powered crew planning system for Bluebikes.

We use real Bluebikes trip data (Jan 2024 to Sep 2026) to decide how many full-time and seasonal rebalancing crews Boston, Cambridge, Somerville, Brookline, and the other cities need in each month of 2026. An LLM turns event listings into structured data for the forecast, an optimizer builds the crew plan, and a tool-calling assistant lets the operations team ask questions about it.

What it does
Demand & imbalance model. Every station starts the day half full. We replay pickups and returns hour by hour and count the bikes that would need moving to keep each station between empty and full.
Event awareness (GenAI). Gemini (gemini-3.5-flash-lite) reads free-text event listings (Boston Marathon, Head of the Charles, July 4th, move-in days, Boston Calling) and returns a validated Pydantic EventList with dates, venue coordinates, and crowd size. Red Sox home games come from the MLB Stats API. We calibrate each event type's impact from 2024 and 2025 data.
2026 forecast. Same-weekday history from 2024 and 2025, adjusted for growth, plus calibrated event uplift.
Crew optimizer. A mixed-integer program chooses hires, release notices, and seasonal contracts month by month to minimize crew cost plus lost revenue, following every rule in the brief.
Backtest. The plan (built only from 2024 and 2025 data) is tested against real Jan to Sep 2026 trips.
Sensitivity analysis. Ridership ±20% and e-bike share +10 and +20 points.
Operations Assistant (GenAI). A tool-calling agent answers questions like "What happens if we add one crew to Cambridge in September?" by calling the model, so every number comes from our code, not the LLM.
Course techniques used: API calls, prompt engineering, structured output (Pydantic + LangChain), and agents with tool calling.

Architecture

Key results
Metric (full year 2026, vs. forecast)	Our plan	Keep January crews all year
Crew cost	$878,100	$1,008,000
Lost revenue	$156,475	$182,591
Total cost	$1,034,575	$1,190,591
Rebalancing need served	89.2%	87.4%
Backtest on real Jan to Sep 2026 trips

Metric	Our plan	Keep January crews	Hindsight optimum
Crew cost	$684,900	$756,000	$775,500
Lost revenue	$288,923	$285,117	$163,442
Total cost	$973,823	$1,041,117	$938,942
Need served	78.1%	78.3%	87.6%
Our plan saves about $67,000 versus keeping January crews, at almost the same service level, and lands within 4% of the perfect-hindsight cost.

Where crews fell short: summer 2026 need came in 25% to 49% above forecast (Boston in August was the largest gap: 21,234 actual vs. 14,213 forecast). Rebalancing need fell from 2024 to 2025, the forecast assumed that decline would continue, and 2026 rebounded. The fix is to lean on seasonal contracts, which can be signed monthly once early-summer demand is visible.

Sensitivity: with ridership +20%, the fixed plan serves only 67.3%, and re-planning recovers $157,000. With ridership −20%, re-planning saves $196,000 by avoiding idle crews. Higher e-bike share actually lowers rebalancing need in our simulation (−16% at +20 points).

Hot spots our model found: Ames St at Main St (Kendall) is the top chronic sink, and MIT Vassar St is the top chronic source, matching the stations named in the brief.

Headline decision: full-time crews cover the year-round base load ($6,000/month), and seasonal contracts ($7,800/month, no cost to end) cover the summer peak, because hiring and later releasing a full-time driver adds $18,000 in training and severance.

Setup
Requirements: Python 3.10+, about 1 GB free disk space, and a Gemini API key (OpenAI or Anthropic keys also work).

git clone <your-repo-url>
cd <repo-folder>
pip install -r requirements.txt
Set your API key:

# Mac / Linux
export GOOGLE_API_KEY="your-key-here"
# Windows
set GOOGLE_API_KEY=your-key-here
Then open the notebook and run all cells:

jupyter notebook Bluebikes_Rebalancing.ipynb
Notes

The first run downloads about 500 MB of trip data from the Bluebikes S3 bucket and takes 15 to 25 minutes. Later runs use the cache in data/processed/.
Download the station list from bluebikes.com/system-data ("Download Station Data") and save it as data/raw/stations.csv.
Without an API key, the notebook still runs end to end using a hand-entered fallback event table.
The Gemini free tier allows about 20 requests per model per day. One full run uses roughly 10. If you hit a 429 error, set os.environ["LLM_MODEL"] to another model your key can use (we used gemini-3.5-flash-lite).
Repo structure
├── Bluebikes_Rebalancing.ipynb   # full pipeline, all cells run with outputs
├── requirements.txt
├── README.md
├── data/                         # created on first run (not committed)
│   ├── raw/                      # trip zips, stations.csv, Red Sox schedules
│   └── processed/                # cached station-hour aggregates
└── outputs/
    ├── crew_plan_2026.csv
    ├── backtest_2026.csv
    ├── sensitivity.csv
    └── analysis_report.md
Key assumptions
Stations reset to half full each night; pickups and returns in the same hour net out (this understates need).
Each unmet bike move is one affected rider, with an expected loss of $5.70 (40% walk to another station, 45% lost ride, 15% lost ride plus lost membership).
Trip station IDs did not match the current station list's IDs, so 631 of 642 stations were matched to the list by location (within 250 m); 11 unmatched stations were assigned to "Other".
Crews work 6 of 7 days on staggered schedules: about 102.9 bikes per crew per day.
New full-time hires are paid during their training month but move no bikes.
The January 2026 starting crews are full-time.
Red Sox attendance is assumed at 32,000 per home game.
Data sources
Bluebikes trip and station data, bluebikes.com/system-data, under the Bluebikes Data License Agreement
Red Sox schedules, MLB Stats API (statsapi.mlb.com)
Event information:
Head of the Charles Regatta 2026 dates: BU Today and meetboston.com
Boston Calling 2026 cancellation: WBUR
Boston Marathon: baa.org
Boston Pops Fourth of July: bso.org
Event data corrections found after the final run. When we checked the event text against these sources, we found two errors: (1) Boston Calling did not take place in 2026 (the festival took a gap year and returns in June 2027), and (2) the 2026 Head of the Charles ran three days, October 16 to 18, not two. Neither changes our results: the calibrated uplift for festivals was 0.00 bikes per 1,000 attendees, so the 2026 Boston Calling entry added nothing to the forecast, and the missing regatta day falls in October, outside the backtest window, with a small sports uplift (0.45 bikes per 1,000 attendees).

Team contributions
All three members contributed equally (about one third each) and every member worked on every deliverable: the notebook, the report, the README, and the demo video. Each member led one part of the pipeline and reviewed the others' work.

Member	Led	Also contributed to
Ahmed	Data pipeline: downloading and aggregating trip files, station matching, and the half-full imbalance simulation	Backtest analysis, chronic sink/source findings, README setup section
Hetvi	GenAI components: LLM event extraction with structured output, prompt design, event calibration, and the tool-calling Operations Assistant	2026 forecast, architecture diagram, AI use documentation, demo video
Pooja	Planning model: 2026 forecast, MILP crew optimizer, backtest, and sensitivity analysis	Analysis report, key results write-up, testing and debugging, demo video
AI use
We took help from Claude (Anthropic) as a coding and writing assistant throughout this project. It helped us plan our approach to the brief, and draft code for the notebook. We then ran everything on the real Bluebikes data, debugged the issues that came up (station ID mismatches, Gemini model changes, and API rate limits), checked the outputs, and decided which results and limitations to highlight in our analysis.

Inside the application itself, we use Google Gemini (gemini-3.5-flash-lite) for two features: extracting structured event data from text, and powering the Operations Assistant.
