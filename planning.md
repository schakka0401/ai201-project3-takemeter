# TakeMeter Planning: r/soccer Match Threads

## 1. Community
**Community:** `r/soccer` (Live Match Threads)
**Why it's a good fit:** Live match threads are high-volume and highly reactive. The discourse violently swings between objective tactical observation, pure emotional outbursts, and meta-commentary about the broadcast or the subreddit itself. This makes it an excellent environment for a text classifier, as the quality of a "take" depends heavily on whether it is grounded in the reality of the game or just emotional noise.

## 2. Labels
1. **`tactical_analysis`:** The post discusses on-pitch strategy, player positioning, specific game state dynamics, or the mechanics of referee decisions using objective observations.
   * *Example 1:* "I know Cape Verde will play defensively. But the way they defend is too passive. They don't even bother to press Uruguay players until the ball reaches Cape Verde's own third."
   * *Example 2:* "The specific rule is the referee only stops play for suspected head injury or other serious injury. It's to discourage time wasting, which Cape Verde were doing."
2. **`reactive_venting`:** An immediate emotional response, trash-talk, or complaint that asserts a feeling or an opinion without providing specific, verifiable evidence to back it up.
   * *Example 1:* "Sooooo…he isn’t hurt lol. Get the fuck up or leave the pitch. Ruined the game for your team, genuinely"
   * *Example 2:* "Urgay are quite an unlikable team tbh"
3. **`meta_narrative`:** Comments that focus on the broadcast, the stadium, the subreddit's reaction, or the overarching "underdog vs. giant" storyline rather than the actual mechanics of the football match.
   * *Example 1:* "Dolphins' stadium is gaudy as hell. It completely suits Miami."
   * *Example 2:* "Rooting for 2:2 and win over SA, r/soccer would explode lol"

## 3. Hard Edge Cases
**The Ambiguous Post:** "That was a weird sequence: play wasnt stopped for his injury, goal was conceded. If it was stopped, then u could tell the guy to get off the field or get timed out, but he didnt get treatment and when he wanna resume, he STILL got timed out?!"
**Conflict:** Is this complaining about the referee (`reactive_venting`) or analyzing the sequence of events (`tactical_analysis`)?
**Decision Rule:** If a post questions or describes a specific sequence of play, a refereeing protocol, or an on-pitch action—even if the user is clearly frustrated or confused—label it `tactical_analysis`. Reserve `reactive_venting` for comments that rely entirely on insults or pure frustration without detailing *why* the sequence happened.

## 4. Data Collection Plan
*   **Source:** Copying comments directly from public `r/soccer` match threads (e.g., Uruguay vs. Cape Verde, World Cup Group Stage).
*   **Target:** At least 200 comments.
*   **Imbalance Strategy:** Match threads skew heavily toward `reactive_venting`. If I hit 200 comments and `tactical_analysis` is below 20%, I will specifically seek out Post-Match threads or tactical breakdown posts to manually pull more analytical examples until the classes are reasonably balanced.

## 5. Evaluation Metrics
Accuracy alone is not enough, because if 70% of my dataset is `reactive_venting`, the model could achieve 70% accuracy just by guessing that label every time. 
*   **Primary Metric:** Per-class F1 Score. 
*   **Reasoning:** F1 balances precision and recall. I want to know specifically if the model is confusing `tactical_analysis` with `reactive_venting`, so evaluating the F1 score for `tactical_analysis` is the most important metric for this project.

## 6. Definition of Success
The classifier is genuinely useful if it can successfully filter out the emotional noise to find the actual football discussion. I will consider this successful if the fine-tuned model achieves an **F1 score of > 0.70 on the `tactical_analysis` label**, proving it has learned the boundary between an emotional complaint and an objective breakdown.

## 7. AI Tool Plan
*   **Label stress-testing:** Used an LLM to brainstorm edge-case boundaries (e.g., defining the rule for complaints vs. referee analysis).
*   **Annotation assistance:** Used an LLM to parse and format the raw Reddit copy-paste into a structured CSV, while retaining final review authority over all assigned labels.
*   **Failure analysis:** I will feed the model's wrong predictions on the test set into an LLM and ask it to identify structural patterns (e.g., "does it always misclassify short sentences as venting?").