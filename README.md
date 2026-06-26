# Intelligent Candidate Discovery & Ranking System — Nexus Ai

This repository contains the candidate discovery and ranking pipeline built for the **Redrob Hackathon**. The system evaluates 100,000 candidate profiles against the **Senior AI Engineer (Founding Team)** Job Description, guaranteeing a **0% honeypot leak rate** while operating well within the 5-minute CPU and 16GB RAM budget.

---

## 🚀 Key Features & Architecture

### 1. Scale-Adaptive Engine
The system dynamically analyzes the size of the candidate pool ($n$) and auto-tunes its operational knobs to maintain maximum accuracy and speed:
* **Small Scale ($n < 500$):** Focuses on maximum signal extraction, no batching or chunking needed.
* **Medium Scale ($500 \le n < 5000$):** Initiates multi-threaded scoring and optimizes TF-IDF features.
* **Large Scale ($n \ge 5000$):** Activates batched cosine similarity (5,000 candidate chunks) and strict feature pruning (`min_df = 3`) to keep peak memory usage under **2GB RAM** (well below the 16GB limit) and prevents Out-Of-Memory (OOM) errors.

### 2. Gated Honeypot & Anomaly Detection
The ranker runs **9 internal consistency checks** on candidate profiles. If a candidate triggers any of the critical integrity flags, they receive a hard disqualification (score set to `0.0`):
1. **Startup Founding Dates:** Catches fakes claiming roles at startups (like Krutrim, Sarvam AI, OpenAI, etc.) before the companies were officially founded.
2. **Zero-Duration Expert Skills:** Filters keyword-stuffers claiming $\ge 3$ expert skills with `0` months of duration.
3. **Timeline Mismatches:** Detects impossible profiles where graduation year contradicts experience, signup dates are in the future, or active dates precede signup dates.
4. **Implausible Signals:** Flags profiles with perfect signal scores ($\ge 4/5$ at $99\%+$ values).
5. **Employment Overlap:** Flags candidates claiming multiple concurrent `is_current` active roles.

### 3. Multi-Signal Hybrid Scoring
Candidates are scored out of $1.0$ based on a weighted combination of profile features:
* **TF-IDF Semantic Matching (22%):** Computes log-dampened unigram similarities against the JD.
* **Hard Skills (26%):** A 4-tier evidence check (gating advanced/expert skills with a $\ge 6$-month duration and actual usage in job descriptions).
* **Soft Skills (10%):** Validates nice-to-haves (like LLM fine-tuning, ML core, MLOps, and HR-tech domains).
* **Career Trajectory (18%):** Heavy penalties for non-technical current roles, research-only backgrounds, and career-long consulting history (TCS, Infosys, etc.).
* **Experience Fit (8%):** Optimal fit curves for 5–9 years of experience (ideal 6–8 years).
* **Location Match (5%):** Prioritizes Pune/Noida or Tier-1 Indian relocatable candidates.
* **Education Tier (3%):** Degree relevance and institution tier.
* **Engagement Quality (8%):** Activity and responsiveness signals on the Redrob platform.

### 4. Behavioral Availability Multiplier (0.0 – 1.2×)
Behavioral signals (notice period, login recency, average response time, open-to-work flag) are combined into a multiplicative factor. This ensures that unresponsive or inactive candidates are deprioritized, while active, immediately available candidates receive a premium score boost.

---

## 🛠️ Setup & Installation

Ensure you have Python 3.11+ installed.

1. **Clone the repository:**
   ```bash
   git clone https://github.com/aasthamahajan19/job-ai.git
   cd job-ai
   ```

2. **Set up a virtual environment:**
   ```bash
   python -m venv venv
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

---

## 💻 Usage

To run the candidate ranker on the dataset:

```bash
python rank.py --candidates /path/to/candidates.jsonl.gz --out ./Nexus_Ai.csv
```

---

## 📁 Repository Structure

* `rank.py`: The main scale-adaptive ranking and scoring engine.
* `submission_metadata.yaml`: Team identity, project metadata, and reproduction commands.
* `requirements.txt`: Python package dependencies.
* `README.md`: Project documentation and architecture guide.
* `.gitignore`: Git configuration to exclude large datasets and temporary files.
