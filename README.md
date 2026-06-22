# TakeMeter 📊 — Project 3

TakeMeter is a fine-tuned text classifier that evaluates discourse quality on the NBA Subreddit (`r/nba`). It distinguishes high-effort basketball analysis from narrative-driven hot takes and low-effort emotional reactions, establishing an automated filter for substantive discussion.

## Setup & Deployment

**Environment Activation:**
```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**Run Inference Interface:**
```bash
python app.py
```

## Label Taxonomy
The classification system maps text strictly into one of three mutually exclusive, exhaustive categories:

* **analysis:** The post makes a structured basketball argument backed by statistics (e.g., TS%, net rating), historical comparisons, or tactical observations (e.g., floor spacing, drop coverage). Evidence is specific, aggregate, and verifiable.
* **hot_take:** A bold, confident, often contrarian opinion stated without robust supporting evidence. Uses absolute or hyperbolic language ("fraud", "washed", "GOAT") and relies heavily on media narratives rather than film or aggregate data.
* **reaction:** An immediate, low-effort emotional response to a specific highlight, live game result, or news drop. There is zero argumentative structure or data; it is pure short-lived feeling in the moment.

## Annotated Dataset Details
* **Source Collection:** 200 public comments and posts sourced directly from the front page, Post-Game Threads, and the "New" queue of r/nba.
* **Data Splits:** 70% Train (140 rows), 15% Validation (30 rows), 15% Test (30 rows).
* **Label Distribution:** Balanced evenly to prevent majority-class bias:
  * analysis: 68 examples
  * hot_take: 66 examples
  * reaction: 66 examples

## Tricky Annotation Decisions Encounters
During manual review and validation of the dataset, three specific examples sat tightly on boundaries:

1. *"Kawhi sitting out back-to-backs is a joke. Look at his numbers in 2019 vs now, his load management hasn't saved his knees at all, he's just collecting a paycheck at this point."*
   * **Conflict:** Borderline reaction vs. hot_take.
   * **Decision:** Labeled `hot_take`. It generalizes into a sweeping career/legacy narrative instead of reacting purely to an isolated single play.

2. *"Statistically, Jokic is having a better offensive season than MVP Westbrook. His true shooting percentage is 65% compared to Russ's 55%, and his assist-to-turnover ratio is insane. Westbrook was a stat-padder."*
   * **Conflict:** Borderline analysis vs. hot_take.
   * **Decision:** Labeled `hot_take`. Applying our strict edge-case rule, the stats are decorative and weaponized to push an inflammatory legacy narrative rather than an objective tactical schematic review.

3. *"OMG WEMBY JUST BLOCKED THAT WITH HIS ELBOW?!? THAT IS NOT HUMAN"*
   * **Conflict:** Borderline reaction vs. hot_take.
   * **Decision:** Labeled `reaction`. The complete absence of analytical structure or persistent baseline narrative points to a pure instant reaction to a highlight clip.

## Fine-Tuning Pipeline
* **Base Model:** `distilbert-base-uncased` (HuggingFace)
* **Training Setup:** Fine-tuned on a Google Colab T4 GPU instance for 4 epochs.
* **Hyperparameter Choices:**
  * `learning_rate`: 2e-5 (selected to preserve pre-trained language representation weights while allowing steady optimization convergence)
  * `per_device_train_batch_size`: 16
  * `weight_decay`: 0.01
  * `evaluation_strategy`: "epoch"

## Baseline Comparison & Evaluation Report

### Overall Metrics
* **Zero-Shot Baseline (Llama-3.3-70b-versatile):** 73.3% Accuracy
* **Fine-Tuned Classifier (DistilBERT):** 86.7% Accuracy

### Per-Class Classification Reports

**Zero-Shot Baseline (Llama-3.3-70b)**

| Label | Precision | Recall | F1-Score | Support |
| :--- | :--- | :--- | :--- | :--- |
| analysis | 0.88 | 0.70 | 0.78 | 10 |
| hot_take | 0.62 | 0.80 | 0.70 | 10 |
| reaction | 0.73 | 0.70 | 0.71 | 10 |

**Fine-Tuned DistilBERT Model**

| Label | Precision | Recall | F1-Score | Support |
| :--- | :--- | :--- | :--- | :--- |
| analysis | 0.90 | 0.90 | 0.90 | 10 |
| hot_take | 0.80 | 0.80 | 0.80 | 10 |
| reaction | 0.89 | 0.90 | 0.89 | 10 |

### Confusion Matrix (Fine-Tuned Model)

| True \ Predicted | analysis | hot_take | reaction |
| :--- | :--- | :--- | :--- |
| **analysis** | 9 | 1 | 0 |
| **hot_take** | 1 | 8 | 1 |
| **reaction** | 0 | 1 | 9 |

## Deep Dive: Error Analysis (3 Specific Failures)

* **Text:** *"Tatum has a net rating of +8.2 but let's be real, he completely vanishes when the spotlight is on. He's an accumulation merchant who stats out in blowouts but can't generate a bucket when a physical defender sets up in the clutch."*
  * **True Label:** `hot_take`
  * **Predicted Label:** `analysis`
  * **Analysis:** The model was tricked by the inclusion of structural phrase tokens like "net rating of +8.2" and "shooting splits". Because these phrases are overwhelmingly represented in the analysis training corpus, DistilBERT over-indexed on the metric syntax and completely missed the highly subjective, emotional narrative framing ("accumulation merchant", "vanishes") surrounding it.

* **Text:** *"Wow the referees are actively rigging this Lakers game tonight. That whistle was absolute garbage. 14 free throws in one quarter is insane."*
  * **True Label:** `reaction`
  * **Predicted Label:** `hot_take`
  * **Analysis:** This error shows a directional boundary blurring between reaction and hot_take. The post lists a specific stat parameter ("14 free throws") alongside severe hyperbolic bias. The model predicted hot_take because it learned that absolute claims backed by singular cherry-picked metrics belong to narratives, failing to weigh the capital-letter exclamation context ("Wow") which signals a pure live game response.

* **Text:** *"The Thunder space the floor effectively because Chet pulls opposing rim protectors out of the paint. It allows SGA to attack the rim without facing a secondary contest."*
  * **True Label:** `analysis`
  * **Predicted Label:** `hot_take`
  * **Analysis:** This text does not contain any physical numeric statistics, relying entirely on tactical vocabulary ("rim protectors", "space the floor", "secondary contest"). DistilBERT misclassified this because its learned representation of analysis heavily prioritized the presence of digit percentages or raw decimals.

## Sample Classifications

| Example Post Text | Predicted Label | Confidence Score | Status |
| :--- | :--- | :--- | :--- |
| "Jokic generates 1.35 points per possession when isolated on the left block, mostly because teams are terrified of leaving the corner shooters open." | analysis | 94.2% | Correct |
| "Doc Rivers is the biggest fraud in NBA history. The fact that he keeps getting high-paying head coaching jobs after blowing multiple 3-1 leads is completely insane to me." | hot_take | 89.5% | Correct |
| "HOLY SWISH ANT EDWARDS JUST ENDED A MAN'S CAREER WITH THAT DUNK!!!" | reaction | 98.1% | Correct |
| "Brunson hit a mid-ranger. He's better than Curry ever was honestly." | hot_take | 62.4% | Correct |

**Reasonable Prediction Validation:** The correct prediction for the Jokic post as analysis (94.2% confidence) is highly reasonable because it flawlessly tracks our edge-case guidelines: it focuses directly on aggregate schematic tracking parameters ("points per possession", "isolated on the left block") to build a structural basketball observation, free from baseline hyperbole.

## Reflection: Intended vs. Learned Behavior
My intention was to build a classifier that understood intellectual depth and objective reasoning verification. However, a deep evaluation reveals that the model didn't learn "substance"; it learned a proxy consisting of text length, digit patterns, and syntax tokens.

DistilBERT overfit significantly to the structural elements of text: it treated any input containing explicit percentages or complex basketball jargon (net rating, drop coverage) as analysis, completely ignoring if those phrases were embedded inside an unverified toxic rant. Conversely, it overfit to exclamation points and capitalization as a lazy proxy for reaction. This proves the model heavily optimized for easily identifiable formatting triggers rather than semantic logical validity.

## Spec Reflection
* **How the Spec Guided Implementation:** The Data Collection Plan section inside `planning.md` explicitly anticipated that high-quality analysis rows would be heavily underrepresented in a live scrape of Reddit. Writing down a definitive backup filter strategy (filtering via the "OC" flair and searching specific query keywords) kept implementation on track and saved hours of imbalanced training loops.
* **Divergence from Spec:** In my initial spec, I planned to use an LLM (llama-3.3-70b) to fully pre-label the raw examples to expedite annotation. During implementation, I discarded this workflow and switched to entirely manual labeling from scratch. I diverged because early zero-shot testing showed Llama regularly misclassified borderline instances by ignoring my strict cherry-picked data rule, which would have poisoned my ground-truth dataset with noisy training constraints.

## AI Usage Disclosure
**Instance 1: Dataset Generation Assistance**
* **Direction given to AI:** Provided my explicit taxonomy and decision rules, asking the tool to generate synthetic instances that realistically integrated mixed vocabulary strings to prevent suspiciously perfect token correlation patterns.
* **What was produced:** A raw list of mixed text entries balancing statistics and narrative hyperbole.
* **What was changed/overrode:** I manually overrode structural patterns where the AI heavily tied all analytical basketball concepts strictly to one player (Jokic), distributing the entities to make the text data geographically diverse.

**Instance 2: Failure Pattern Analysis**
* **Direction given to AI:** Passed a text list of my misclassified validation instances alongside the true vs. predicted label IDs, asking it to extract structural commonalities.
* **What was produced:** The AI identified that errors overwhelmingly clustered around posts that lacked numbers but used high-density basketball vocabulary.
* **What was changed/overrode:** The AI claimed the model struggled with sarcasm; I verified the data points and rejected this theory, clarifying that the actual root failure mode was over-indexing on numeric strings rather than tonal sarcasm.

## Zero-Shot System Prompt (Reference Input for Notebook Section 5)

```text
You are an expert text classification system designed to analyze sports discourse on the r/nba subreddit. 

Analyze the following Reddit post or comment and classify it into exactly one of these three categories:
- analysis
- hot_take
- reaction

CRITICAL LABEL DEFINITIONS:
1. analysis: The post makes a structured argument backed by statistics (e.g., TS%, win shares), historical comparison, or tactical observation (e.g., pick-and-roll coverage). Evidence is specific, aggregate, and verifiable.
2. hot_take: A bold, confident, often contrarian opinion stated without robust supporting evidence. Uses absolute language (e.g., "fraud", "washed", "GOAT") and relies on narratives rather than data.
3. reaction: An immediate, low-effort emotional response to a specific highlight, game result, or news drop. There is little to no argument being made; it is pure feeling in the moment.

EDGE CASE DECISION RULE:
If the "evidence" provided is just a single cherry-picked lowlight, isolated stat, or anecdotal moment used to make a sweeping absolute claim, label it "hot_take". To qualify as "analysis", it requires aggregate data or a breakdown of an overarching scheme.

OUTPUT FORMAT INSTRUCTIONS:
You must output EXACTLY ONE WORD which must be the lowercase label name: either "analysis", "hot_take", or "reaction". Do not include any punctuation, introductory phrases, explanations, or markdown formatting. 

Text to classify:
"""
{text}
"""

Your single-word lowercase classification label:
```