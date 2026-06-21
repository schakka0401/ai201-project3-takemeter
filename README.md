# ai201-project3-takemeter

# TakeMeter: `r/soccer` Match Thread Discourse Classifier

## 1. Community Choice & Reasoning
**Community:** `r/soccer` (Live Match Threads)
**Reasoning:** Live match threads are high-volume and highly reactive. The discourse swings violently between objective tactical observation, pure emotional outbursts, and meta-commentary about the broadcast or the subreddit itself. This makes it an excellent environment for a text classifier, as the quality of a "take" depends heavily on whether it is grounded in the reality of the game or just emotional noise.

## 2. Label Taxonomy
1. **`tactical_analysis`:** The post discusses on-pitch strategy, player positioning, specific game state dynamics, or the mechanics of referee decisions using objective observations.
   * *Example:* "I know Cape Verde will play defensively. But the way they defend is too passive. They don't even bother to press Uruguay players until the ball reaches Cape Verde's own third."
   * *Example:* "The specific rule is the referee only stops play for suspected head injury or other serious injury. It's to discourage time wasting, which Cape Verde were doing."
2. **`reactive_venting`:** An immediate emotional response, trash-talk, or complaint that asserts a feeling or an opinion without providing specific, verifiable evidence to back it up.
   * *Example:* "Sooooo…he isn’t hurt lol. Get the fuck up or leave the pitch. Ruined the game for your team, genuinely"
   * *Example:* "Urgay are quite an unlikable team tbh"
3. **`meta_narrative`:** Comments that focus on the broadcast, the stadium, the subreddit's reaction, or the overarching "underdog vs. giant" storyline rather than the actual mechanics of the football match.
   * *Example:* "Dolphins' stadium is gaudy as hell. It completely suits Miami."
   * *Example:* "Rooting for 2:2 and win over SA, r/soccer would explode lol"

## 3. Data Collection & Labeling Process
*   **Source:** I manually copy-pasted 202 comments directly from public `r/soccer` match threads (specifically the Uruguay vs. Cape Verde World Cup Group Stage thread). 
*   **Labeling Process:** I used an AI assistant to help format the raw Reddit text into a CSV structure and propose initial labels based on my taxonomy. I manually reviewed and corrected every assigned label to ensure it met my strict definitions, actively balancing the dataset to avoid a `reactive_venting` majority.
*   **Label Distribution:** The final dataset of 202 comments was roughly balanced across the three classes to prevent the model from blindly guessing the majority class.

### Hard Edge Cases Encountered
1. **The Refereeing Dilemma:** *"That was a weird sequence: play wasnt stopped for his injury, goal was conceded..."*
   * *Decision:* `tactical_analysis`. While it complains, it discusses a specific sequence of play and refereeing protocols.
2. **The Couch Coach:** *"If my son's 12yo teammates reacted that way when they are in the wall on a set piece, they'd get benched. That is just unacceptably bad..."*
   * *Decision:* `tactical_analysis`. It sounds like venting, but it directly critiques the physical positioning and mechanics of the defensive wall.
3. **Tournament Rules vs. Game Tactics:** *"These water breaks man. Convinced CV would not be losing if they played the full 45."*
   * *Decision:* `meta_narrative`. It complains about the tournament broadcast/structure (water breaks) rather than tactical choices made by the players.

## 4. Fine-Tuning Approach
*   **Base Model:** `distilbert-base-uncased`
*   **Training Setup:** I used Google Colab with a free T4 GPU. The model was trained using the HuggingFace `transformers` library on 141 training examples, validating on 30.
*   **Hyperparameter Decisions:** I kept the learning rate at `2e-5` and the batch size at `16`, running for `3` epochs. I chose not to increase the epochs because text classification on a small dataset of 200 items is highly prone to overfitting, and DistilBERT learns very quickly.

## 5. Baseline Description
To establish a zero-shot baseline, I used Groq's `llama-3.3-70b-versatile` API. 
*   **Prompt Design:** I provided the Llama model with the exact definitions of my three labels and a strict instruction detailing my edge-case decision rule (e.g., "If a post questions or describes a specific sequence of play... label it tactical_analysis").
*   **Collection:** I forced the LLM to output only the exact category string and evaluated it against the same 31 test examples that my fine-tuned model would face.

---

## 6. Full Evaluation Report

### Overall Metrics
*   **Zero-Shot Baseline (Groq Llama 3.3):** [Insert your Groq baseline accuracy here, e.g., 65%] Accuracy
*   **Fine-Tuned Model (DistilBERT):** 38.7% Accuracy (12/31 correct)

### Fine-Tuned Model Confusion Matrix
| True Label \ Predicted Label | `meta_narrative` | `tactical_analysis` | `reactive_venting` |
| :--- | :--- | :--- | :--- |
| **`meta_narrative`** | 5 | 6 | 0 |
| **`tactical_analysis`** | 3 | 7 | 0 |
| **`reactive_venting`** | 9 | 1 | 0 |

### Error Analysis (3 Wrong Predictions)
Based on the confusion matrix from image_1224f6.png, our model completely lost the ability to predict the `reactive_venting` class, and struggled to draw the line between meta-commentary and tactical observation.

1. **True Label:** `reactive_venting` | **Predicted:** `meta_narrative`
   * **Text:** *"Say what you will about the US team at least we don’t play dirty"*
   * **Why it failed:** This comment is pure emotional trash-talk with no evidence. However, because it mentions a country's team identity ("US team") and compares it to another in a broad sense, the model likely triggered on the structural keywords of a "storyline" rather than picking up on the emotional frustration.
2. **True Label:** `reactive_venting` | **Predicted:** `meta_narrative`
   * **Text:** *"Urgay are quite an unlikable team tbh"*
   * **Why it failed:** This is an explicit emotional outburst. However, the model failed to learn the boundary of "emotional tone." It learned that anytime a comment makes a broad, generalized statement about a team as an entity, it belongs in `meta_narrative`. It overfit to the subject matter (the team) rather than the intent (complaining).
3. **True Label:** `meta_narrative` | **Predicted:** `tactical_analysis`
   * **Text:** *"These water breaks man. Convinced CV would not be losing if they played the full 45."*
   * **Why it failed:** The true label is meta-narrative because it complains about the tournament's broadcast/structure. The model predicted `tactical` because the second half of the sentence sounds structurally like a tactical observation about game state. The model confused an observation about the rules with an observation about the players' execution.

### Sample Classifications
Here is a look at how the model handles new inputs based on its learned boundaries:

| Text | Predicted Label | Confidence |
| :--- | :--- | :--- |
| *"The gap between the midfield and the defense for Cape Verde is getting way too large. They are exhausted."* | `tactical_analysis` | 88% |
| *"Miami crowd is dead, this feels like a friendly."* | `meta_narrative` | 92% |
| *"Trash referee."* | `meta_narrative` | 61% |

**Correct Prediction Explanation:** The model correctly identified *"The gap between the midfield and the defense..."* as `tactical_analysis`. It successfully learned that objective vocabulary related to pitch geography ("midfield", "defense", "gap") belongs in the tactical bucket, independent of whether the user is criticizing a team's performance.

---

## 7. Reflections

**What the model learned vs. what I intended:**
I intended for the model to learn the difference between emotional tone (`reactive_venting`), objective game observation (`tactical_analysis`), and overarching storylines (`meta_narrative`). Instead, the model's decision boundaries overfit entirely to structural keywords. It learned that words related to the pitch and formations equal "tactical". It learned that *everything else* equals "meta-narrative". It completely failed to learn what "emotion" looks like in text, resulting in a 0% prediction rate for the venting class in image_1224f6.png. To fix this, I would need to provide drastically more examples of short, angry outbursts in the training data so the model is forced to recognize negative sentiment as a distinct feature.

**Spec Reflection:**
*   **How the spec helped:** The requirement to define "hard edge cases" in Milestone 2 before starting saved me massive amounts of time. By deciding beforehand that referee complaints counted as tactical analysis, I avoided second-guessing myself during annotation.
*   **How I diverged:** I diverged slightly from strict manual annotation by using an AI workflow to structure and pre-label my CSV rows. Doing it strictly by hand in a raw text file was too prone to breaking the Colab CSV parser with rogue commas.

**AI Usage Disclosure:**
1.  **Dataset Pre-Labeling & Formatting:** I provided an LLM with raw chunks of Reddit match-thread text and my strict taxonomy. I directed the AI to parse the text into a CSV format and assign a preliminary label. *What I overrode:* I manually reviewed all 202 rows and overrode the AI's labels on edge cases (like the referee/injury posts) to ensure they matched my specific decision rules.
2.  **Failure Analysis Formatting:** I provided an LLM with the raw output of my confusion matrix and my 31 test predictions. I directed it to help me identify the structural pattern behind why `reactive_venting` got 0 predictions. *What I revised:* I took the AI's high-level observation (that the model overfit to team names) and explicitly rewrote the analysis to match my specific Reddit context and tone.