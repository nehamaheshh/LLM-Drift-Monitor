# 🚦 LLM Drift Monitor

A practical system to monitor how Large Language Models (LLMs) **change their behavior over time or across models** — even when answers still appear correct.

Instead of focusing on accuracy, this project answers a more important production question:

> **Is the model still behaving the way we expect?**

---

## 🔍 What This Project Does

This system logs LLM interactions and detects different types of drift:

### 1️⃣ Semantic Drift (Meaning)
Checks whether the *meaning* of responses is changing using embedding similarity.

### 2️⃣ Structural Drift (Style & Verbosity)
Detects changes in:
- response length  
- verbosity and formatting  

using statistical tests on response length distributions.

### 3️⃣ Safety Drift
Tracks changes in refusal behavior (for example, when a model starts refusing more queries).

### 4️⃣ Cost Drift (Estimated)
Estimates token usage and cost based on response length to detect silent cost increases.

### 5️⃣ Drift Over Time
Uses rolling windows to show **when** behavior changes started.

### 6️⃣ Auto Alerts
Summarizes drift into:
- 🟢 **PASS** – normal variation  
- 🟡 **WARN** – moderate drift  
- 🔴 **ALERT** – significant behavior change  

---

## 🧠 Why This Matters

In real-world LLM systems:
- Accuracy may stay stable
- But responses can become longer, slower, more restrictive, or more expensive

These changes often go unnoticed until they impact users or cost.  
This project shows why **LLM evaluation needs more than accuracy**.

---

## 🧪 Example Insight

A controlled experiment comparing **LLaMA-3-8B** and **Qwen-2.5-7B** showed:

- Meaning stayed mostly the same
- Response length changed significantly
- Refusal behavior increased
- Estimated cost profile shifted

Traditional evaluation would miss this drift.

---

## 🧱 Tech Stack

- **LLM Runtime:** Ollama  
- **Models:** LLaMA-3-8B, Qwen-2.5-7B  
- **Embeddings:** sentence-transformers (MiniLM)  
- **Database:** SQLite  
- **Statistics:** SciPy (KS test)  
- **Dashboard:** Streamlit  
- **Language:** Python  

---

## 📂 Repository Structure

llm-drift-monitor/
├─ dashboard/
│ └─ app.py
├─ data/
│ └─ llm_logs.db # ignored by git
├─ reports/
│ └─ .gitkeep
├─ scripts/
│ ├─ run_prompt.py
│ ├─ run_daily_monitor.py
│ ├─ run_model_compare.py
│ ├─ check_counts.py
│ └─ inspect_experiments.py
├─ src/
│ ├─ drift/
│ │ └─ detectors.py
│ ├─ features/
│ │ ├─ embed.py
│ │ └─ text_features.py
│ ├─ llm/
│ │ └─ ollama_client.py
│ └─ logging/
│ ├─ logger.py
│ └─ schema.py
├─ .gitignore
├─ LICENSE
├─ requirements.txt
└─ README.md

yaml
Copy code

---

## 🚀 Quickstart

### 1️⃣ Create & activate virtual environment (Windows PowerShell)

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```
### 2️⃣ Install dependencies
```
pip install -r requirements.txt
```
### 3️⃣ Start Ollama & pull models
```
ollama list
ollama pull llama3:8b
ollama pull qwen2.5:7b
```
### 4️⃣ Log LLM interactions
This logs prompts, responses, embeddings, latency, length, and refusal flags.
```
python -m scripts.run_prompt
```
### 5️⃣ Run CLI drift monitor
```
python -m scripts.run_daily_monitor
```
### 6️⃣ Launch Streamlit dashboard
```
streamlit run dashboard/app.py
```
🧪 Controlled Model Switch Experiment (Equal Samples)
This performs a clean A/B test:

-Same prompts
-Same sample size
-Only model changes

```
$env:EXPERIMENT_ID="model_switch_equal_samples"

$env:OLLAMA_MODEL="llama3:8b"
1..31 | % { python -m scripts.run_prompt }   # ≈93 samples

$env:OLLAMA_MODEL="qwen2.5:7b"
1..31 | % { python -m scripts.run_prompt }

python -m scripts.check_counts
python -m scripts.run_model_compare
```

Expected outcome:

✅ Mild semantic drift

⚠️ Strong verbosity / length drift

⚠️ Possible refusal-rate changes

📊 Dashboard Overview
Tabs included:
-Drift Snapshot – current baseline vs recent behavior
-Drift Over Time – rolling-window semantic & length drift
-Model Compare – equal-sample A/B comparison
-Recent Logs – raw interaction inspection

Auto Alert System
Each comparison is labeled:

-🟢 PASS – normal variation
-🟡 WARN – moderate drift detected
-🔴 ALERT – statistically significant behavior change

Alerts are triggered using:

-semantic drift thresholds
-KS statistic + p-value
-refusal-rate increase
-estimated cost increase

📐 Metrics & Interpretation
Semantic Drift (Cosine Distance)
Range	Meaning
0.00 – 0.03	Tiny
0.03 – 0.08	Mild
0.08 – 0.15	Moderate
> 0.15	High

Structural Drift (KS Test)
-KS statistic ↑ → larger distribution shift
-p < 0.01 → statistically significant drift

Why this matters
Even when semantic content stays stable, drift in:

-verbosity
-latency
-refusal behavior
-token cost

can silently degrade UX, increase spend, or break workflows.

🧠 Key Insight Demonstrated
Switching from LLaMA-3-8B to Qwen-2.5-7B caused major verbosity and refusal-rate drift while semantic meaning remained largely stable — showing why accuracy alone is insufficient for LLM evaluation.

🔒 Notes
-data/llm_logs.db is intentionally not committed
-Use EXPERIMENT_ID to keep experiments isolated and reproducible
-Cost estimates are approximate (token ≈ 1.33 × words)



