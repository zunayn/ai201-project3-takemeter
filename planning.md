# TakeMeter — Planning Document

## 1. Community and Domain
**Community:** `r/nba` (NBA Subreddit)
**Why this community?** This community is an excellent fit for classification because the discourse is highly active, text-heavy, and incredibly varied. Users frequently debate the quality of posts—complaining about "nephews" posting baseless rumors, while praising high-effort statistical deep dives. This creates a natural, culturally relevant taxonomy of discourse quality (evidence vs. emotion vs. narrative) that is genuinely interesting to categorize.

## 2. Label Taxonomy
My taxonomy consists of three mutually exclusive labels:

* **`analysis`**: The post makes a structured argument backed by statistics (e.g., TS%, win shares), historical comparison, or tactical observation (e.g., pick-and-roll coverage). Evidence is specific, aggregate, and verifiable.
  * *Example 1:* "Looking at the tracking data, Gobert is contesting 12 shots at the rim per game and holding opponents to 48%, which explains the Wolves' defensive rating."
  * *Example 2:* "The Celtics' five-out offense works because Porzingis pulls the opposing rim protector out to the three-point line, completely opening up driving lanes for Tatum and Brown."
* **`hot_take`**: A bold, confident, often contrarian opinion stated without robust supporting evidence. Uses absolute language (e.g., "fraud", "washed", "GOAT") and relies on narratives rather than data.
  * *Example 1:* "Embiid will never win a ring because his playstyle just doesn't translate to the playoffs. He's a regular-season merchant and everyone knows it."
  * *Example 2:* "The Bucks need to trade Giannis right now while he still has value. Their championship window is completely closed."
* **`reaction`**: An immediate, low-effort emotional response to a specific highlight, game result, or news drop. There is little to no argument being made; it is pure feeling in the moment.
  * *Example 1:* "HOLY S*** ANTHONY EDWARDS JUST ENDED JOHN COLLINS' CAREER WITH THAT DUNK."
  * *Example 2:* "I am so absolutely sick of watching this team blow 15-point leads in the 4th quarter. Fade me, see you guys next season."

## 3. Hard Edge Cases
**The Ambiguous Post:** Posts that blend a bold, absolute claim with a single, cherry-picked statistic or video clip (e.g., *"Luka is the worst defender in the league. Look at this clip from last night where he lets them get a free layup."*)
**Handling Strategy:** During annotation, I will apply a strict decision rule: If the "evidence" is just a single cherry-picked lowlight or anecdotal moment used to make a sweeping absolute claim, it will be labeled `hot_take`. To qualify as `analysis`, it requires aggregate data or a breakdown of an overarching scheme, not just a singular video clip used as decoration for a rant.

### Concrete Tricky Examples Encountered During Annotation

1. **Example Post:** *"Kawhi sitting out back-to-backs is a joke. Look at his numbers in 2019 vs now, his load management hasn't saved his knees at all, he's just collecting a paycheck at this point."*
   - **The Conflict:** Borderline `reaction` (emotional venting about an injury report) and `hot_take` (claiming load management hasn't worked and calling him a paycheck collector).
   - **Decision:** Labeled as `hot_take`. While highly emotional, it makes an overarching narrative claim about a player's career trajectory and legacy rather than just reacting to a single game highlight.

2. **Example Post:** *"Statistically, Jokic is having a better offensive season than MVP Westbrook. His true shooting percentage is 65% compared to Russ's 55%, and his assist-to-turnover ratio is insane. Westbrook was a stat-padder."*
   - **The Conflict:** Borderline `analysis` (cites real stats like TS%) and `hot_take` (makes a sweeping narrative claim that Westbrook was a "stat-padder").
   - **Decision:** Labeled as `hot_take`. Following our decision rule, the stats are decorative and cherry-picked to support an inflammatory narrative rather than an objective tactical breakdown.

3. **Example Post:** *"OMG WEMBY JUST BLOCKED THAT WITH HIS ELBOW?!? THAT IS NOT HUMAN"*
   - **The Conflict:** Borderline `reaction` (pure emotional exclamation) and `hot_take` (hyperbolic claim that he is "not human").
   - **Decision:** Labeled as `reaction`. The absolute lack of argumentative structure or persistent narrative means it is purely a short-lived emotional outburst to a live highlight.

## 4. Data Collection Plan
* **Collection Source:** I will manually collect posts and comments from the `r/nba` front page, Post-Game Threads (good for `reaction` and `analysis`), and the "New" queue (good for `hot_take`). 
* **Target Distribution:** I aim for an even split across my 200 examples (roughly 65-70 examples per label).
* **Handling Underrepresentation:** If `analysis` is underrepresented after collecting the first 100 posts (which is likely, as high-effort posts are rarer), I will specifically filter the subreddit using the "OC" (Original Content) flair and search for keywords like "stats", "tracking", or "breakdown" to forcibly balance the dataset.

## 5. Evaluation Metrics
Accuracy alone is insufficient for this task because the boundaries are subjective and class imbalances may occur.
* **Accuracy:** To measure overall correctness across the test set.
* **Macro F1-Score:** Because I care equally about the model's ability to identify all three classes (even if `analysis` ends up slightly smaller in the test set), Macro F1 will balance precision and recall across all labels.
* **Confusion Matrix:** This is critical. I need to see *where* the model is failing. Confusing a `reaction` for a `hot_take` is a mild error, but confusing `analysis` for a `reaction` means the model has fundamentally failed to understand what evidence looks like.

## 6. Definition of Success
For this classifier to be genuinely useful as a community tool (e.g., an automated bot that flags low-effort takes or highlights high-effort breakdowns), it needs to achieve an **Accuracy and F1-Score of > 75%**. More importantly, success means achieving a **Precision of > 80% on the `analysis` label**. If the model claims a post is `analysis`, users need to trust that it actually contains real evidence. I would accept lower performance on distinguishing `hot_take` vs. `reaction`, as human annotators often disagree on those anyway.

---

## 7. AI Tool Plan

### Label Stress-Testing
Before I annotate my 200 examples, I will provide my taxonomy and edge case rules to Claude/ChatGPT. I will ask it to generate 10 synthetic posts that intentionally sit right on the boundary between my labels. I will attempt to classify them; if I struggle to place them cleanly, I will revise and tighten my definitions before moving to actual data collection.

### Annotation Assistance
To speed up the process, I will collect 200 raw text examples into a spreadsheet. I will use Groq (`llama-3.3-70b-versatile`) to do a first-pass pre-labeling of the dataset. I will track this by creating a spreadsheet column named `LLM_Label` and an adjacent column named `Human_Corrected`. I will manually review every single LLM prediction and override it where necessary so the final dataset reflects my own judgment.

### Failure Analysis
Once the model is fine-tuned and tested, I will export the test set rows that the model predicted incorrectly. I will feed these errors and the confusion matrix into an LLM and prompt it to look for systematic blind spots (e.g., "The model consistently flags high word counts as `analysis` even if it's just a long `hot_take`"). I will then manually verify these identified patterns and use them to write the final evaluation report.