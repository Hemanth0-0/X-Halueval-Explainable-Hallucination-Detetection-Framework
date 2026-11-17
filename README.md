# WebHaluCheck v3

WebHaluCheck is a claim-level factual verification prototype for identifying potentially hallucinated content in LLM responses. It combines question-conditioned claim verification, live evidence retrieval, an NLI baseline, and an LLM-based five-class judge.

## Project workflow

1. **Claim formation** – converts a question-answer pair into a declarative factual claim.
2. **Tier 1 retrieval** – searches Wikipedia, fetches relevant pages, chunks the text, and ranks evidence using TF-IDF similarity.
3. **Tier 1 judging** – classifies the claim as `SUPPORTED`, `CONTRADICTED`, `ABSENT`, `PARTIALLY_SUPPORTED`, or `UNEVALUATABLE`.
4. **Tier 2 retrieval** – escalates inconclusive cases to Firecrawl live web search.
5. **Final verification** – combines retrieved evidence with the judge's verdict and explanation.
6. **Baseline and evaluation** – compares the LLM judge with a DeBERTa NLI baseline and reports classification metrics and paired statistical tests.
7. **Free-form demo** – extracts individual factual claims from a longer response and verifies them separately.

## Repository structure

```text
WebHaluCheck/
│── qa_data.json
│── webhalucheck_v3.ipynb
├── .gitignore
├── requirements.txt
└── README.md
```

## Dataset

The notebook expects `qa_data.json`. Each record contains:

- `knowledge`: reference knowledge/context
- `question`: the question
- `right_answer`: the reference truthful answer
- `hallucinated_answer`: the paired hallucinated answer

The notebook samples `N_PAIRS` unique questions and creates two evaluation examples per question: one truthful and one hallucinated answer. The current notebook configuration uses `N_PAIRS = 50` and `EVAL_SEED = 11`, producing 100 samples.

**Dataset attribution:** Before making this repository public, verify the original dataset's license and redistribution terms. Add the original dataset citation and license information here. If redistribution is not permitted, remove `data/qa_data.json` from the repository and provide instructions/link for users to obtain it from its authorized source.

## Requirements

- Python 3.10+ recommended
- NVIDIA NIM API access
- Firecrawl API access
- Internet connection for Wikipedia and Firecrawl retrieval
- GPU recommended for local DeBERTa inference; CPU fallback is supported but may be slower

Install dependencies:

```bash
pip install -r requirements.txt
```

## API configuration

The notebook requires two credentials:

- `NVIDIA_API_KEY`
- `FIRECRAWL_API_KEY`

Never commit real API keys to GitHub.

### Kaggle

Add both keys under **Add-ons → Secrets** in your Kaggle notebook, using the exact names above.

### Local Jupyter

Set the environment variables before launching Jupyter. For example, in a terminal:

```bash
# macOS / Linux
export NVIDIA_API_KEY="your_key"
export FIRECRAWL_API_KEY="your_key"
jupyter lab
```

On Windows PowerShell:

```powershell
$env:NVIDIA_API_KEY="your_key"
$env:FIRECRAWL_API_KEY="your_key"
jupyter lab
```

The `.env.example` file is only a template. Do not rename it to `.env` and commit real credentials.

## Run the notebook

1. Clone or download this repository.
2. Install dependencies.
3. Configure the two API keys.
4. Open `notebooks/webhalucheck_v3.ipynb`.
5. Run the cells from top to bottom.

The notebook's dataset path is configured for `data/qa_data.json`. If you change the dataset location, update `HALUEVAL_PATHS` in the notebook.

## Important evaluation note

The saved experiment configuration uses 50 question pairs (100 answer samples). Treat these results as an initial experiment, not as proof of generalization or state-of-the-art performance. For research reporting, disclose the sample size, seed, retrieval settings, model identifiers, and evaluation limitations. Do not tune prompts or thresholds on the final test sample.

## Limitations

- Retrieval quality depends on Wikipedia coverage and Firecrawl search results.
- An `ABSENT` verdict means the evidence did not establish the claim; it does not automatically prove the claim is false.
- The current benchmark run is small and should be expanded across more samples and datasets.
- The free-form response demo is not, by itself, a large-scale benchmark.

