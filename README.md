# Custom nanoGPT Language Model Experiment

This repository documents two small nanoGPT experiments trained from scratch with whole-word tokens. The goal was not to create a general chatbot. The goal was to observe how a corpus becomes tokens, predictions, loss, gradients, weight updates, embeddings, generated text, and measurable evaluation results.

The first experiment used only the supplied classroom corpus. The second kept that corpus and added original teaching examples for **negation** and **spatial relations**. Both experiments used the same 3,000-step training budget, learning rate, seed, evaluation suite, and generation settings.

## Results at a glance

The evaluation suite contains 48 fixed cases. `Correct / 48` includes every case, while scorable accuracy includes only cases whose prompt and four choices fit the saved vocabulary and context window.

| Experiment | Stage | Correct / 48 | Scorable / 48 | Accuracy among scorable | Coverage | Complete results |
|---|---:|---:|---:|---:|---:|---|
| Classroom baseline | Untrained | 9 | 24 | 37.50% | 50.00% | [CSV](results/baseline/language_evals/untrained/eval_results.csv) / [JSON](results/baseline/language_evals/untrained/eval_results.json) / [summary](results/baseline/language_evals/untrained/eval_summary.json) |
| Classroom baseline | Trained | 20 | 24 | 83.33% | 50.00% | [CSV](results/baseline/language_evals/final/eval_results.csv) / [JSON](results/baseline/language_evals/final/eval_results.json) / [summary](results/baseline/language_evals/final/eval_summary.json) |
| Expanded corpus | Untrained | 10 | 26 | 38.46% | 54.17% | [CSV](results/expanded/language_evals/untrained/eval_results.csv) / [JSON](results/expanded/language_evals/untrained/eval_results.json) / [summary](results/expanded/language_evals/untrained/eval_summary.json) |
| Expanded corpus | Trained | 24 | 26 | 92.31% | 54.17% | [CSV](results/expanded/language_evals/final/eval_results.csv) / [JSON](results/expanded/language_evals/final/eval_results.json) / [summary](results/expanded/language_evals/final/eval_summary.json) |

These scores do not tell the whole story. The expanded model improved the starter-transfer group from 4/8 in the trained baseline to 7/8, and one spatial case was correct. However, the extension group remained only 2/24 scorable because the expanded corpus exceeded the model's 509 retained training-token types. All negation cases remained unscorable. This is a limitation of my data design, not evidence that the model learned negation successfully.

## Experiment choices and prediction

### Experiment 1: classroom baseline

- Corpus: supplied synthetic classroom sentences only
- Training steps: 3,000
- Learning rate: 0.001 with the notebook's warmup and cosine decay
- Seed: 42

I predicted that training and validation loss would fall, generated sentences would begin to resemble the classroom templates, and familiar classroom patterns would improve. I also expected extension categories to remain weak because their words and patterns were absent. That prediction was supported: the final model passed all 16 reserved starter-pattern cases, but all 24 extension cases were unscorable.

### Experiment 2: expanded corpus

I kept the same settings and added two original UTF-8 text files:

- [68 lines of negation examples](teaching_materials/negation_training_examples.txt)
- [60 lines of spatial-relation examples](teaching_materials/spatial_relations_training_examples.txt)

The importer split sentences at boundaries, producing 88 negation passages and 90 spatial passages, or 178 new unique passages total. I chose these categories because the baseline had zero vocabulary coverage for both, and both can be taught with varied, concrete statements rather than copied test stories.

I predicted that the added vocabulary would make both categories scorable. This prediction was only partly supported. Spatial coverage reached 2/3 cases, but negation stayed at 0/3 scorable. The expanded training text contained 543 token types, while the model retains only the 509 most frequent types. Thirty-four types became `UNK`, including words needed by the public evaluation such as `yellow`, `milk`, `bread`, `wide`, and `to`.

### Sources, permissions, and extraction checks

The classroom sentences came from the supplied notebook. I created and manually reviewed the two synthetic extension files for this assignment with AI assistance. They contain no private information or external copyrighted documents and may be published in this repository. No PDFs were used, so OCR and PDF reading-order checks were unnecessary. Both files imported with no warnings; filenames, hashes, previews, and counts are recorded in the [expanded corpus manifest](results/expanded/corpus_manifest.json).

## Run details

| Property | Baseline | Expanded |
|---|---:|---:|
| Executed notebook | [open](notebooks/custom_llm_baseline_classroom_3000_executed.ipynb) | [open](notebooks/custom_llm_expanded_negation_spatial_3000_executed.ipynb) |
| Config | [JSON](results/baseline/config.json) | [JSON](results/expanded/config.json) |
| Training summary | [JSON](results/baseline/training_summary.json) | [JSON](results/expanded/training_summary.json) |
| Completed steps | 3,000 | 3,000 |
| Elapsed time | 57.89 seconds | 60.87 seconds |
| Interrupted | No | No |
| Device | Colab CPU | Colab CPU |
| Parameters | 111,872 | 135,936 |
| Vocabulary size | 136 | 512 |
| Training passages | 4,132 | 4,293 |
| Validation passages | 460 | 477 |
| Training unknown-token rate | 0.00% | 0.07% |
| Held-out unknown-token rate | 0.00% | 0.64% |
| Vocabulary report | [JSON](results/baseline/vocabulary_report.json) | [JSON](results/expanded/vocabulary_report.json) |
| Split evidence | [JSON](results/baseline/split.json) | [JSON](results/expanded/split.json) |

The 90/10 split is by deduplicated short passage, not by source file. It measures prediction on held-out combinations from the same kinds of data, not generalization to an unseen document or domain.

## Loss and generated samples

Loss is the model's penalty for assigning low probability to the actual next token. Lower loss means better next-token prediction on that panel. Each row below uses fixed panels of at most 20 training and 20 validation passages and averages non-padding next-token targets.

### Classroom baseline losses

| Step | Training loss | Validation loss |
|---:|---:|---:|
| 0 | 4.9263 | 4.9275 |
| 1,500 | 0.6821 | 0.7182 |
| 3,000 | 0.6783 | 0.7061 |

![Baseline training and validation loss](results/baseline/training_curves.svg)

[Full baseline history](results/baseline/history.json) and [training CSV](results/baseline/training.csv).

### Expanded-corpus losses

| Step | Training loss | Validation loss |
|---:|---:|---:|
| 0 | 6.2436 | 6.2247 |
| 1,500 | 0.7586 | 0.8987 |
| 3,000 | 0.7352 | 0.8919 |

![Expanded training and validation loss](results/expanded/training_curves.svg)

[Full expanded history](results/expanded/history.json) and [training CSV](results/expanded/training.csv).

Both experiments learned their training patterns. The validation loss remained above training loss, especially for the expanded run, showing that held-out prediction was harder. Loss values across these runs are not a ranking because their vocabularies and corpora differ.

The [untrained baseline samples](results/baseline/samples/step_0000.txt) were mostly unrelated words. At [step 1,500](results/baseline/samples/step_1500.txt), complete classroom-style sentences appeared. At [step 3,000](results/baseline/samples/step_3000.txt), examples included:

> our school has a question about the new educator and lesson .

> the report about the nurse explains the health in detail .

The expanded run shows the same timeline in its [untrained](results/expanded/samples/step_0000.txt), [halfway](results/expanded/samples/step_1500.txt), and [final](results/expanded/samples/step_3000.txt) sample files. The final free samples still favored the much larger classroom portion of the corpus, which helps explain why the small extension did not dominate generation.

## Token, embedding, probability, gradient, and update evidence

The baseline [tokenization record](results/baseline/tokenization.json) maps each normalized word or punctuation mark to an integer. These IDs are lookup addresses, not measurements of meaning. In this run, `customer` had token ID **28**. Its embedding was a learned row of 64 numbers. The coordinates do not individually have named meanings; together they are adjusted to make useful predictions.

<details>
<summary>Full 64-number customer embedding before and after baseline training</summary>

Before:

```text
[-0.0575919151, -0.0048099528, 0.0426318869, 0.0193389561, 0.0156431366, -0.0288243648, 0.0256090555, 0.0000524609, 0.0247068163, 0.0206917655, 0.0073690168, -0.0330896154, -0.0535478629, -0.0057429299, -0.0241667628, -0.0147161186, 0.0046857060, -0.0104542682, -0.0083810752, -0.0182585604, -0.0201337002, 0.0050986498, -0.0109165022, -0.0126333516, 0.0283896253, -0.0026312231, -0.0040719286, 0.0136419199, -0.0098917242, -0.0167176239, 0.0019060791, -0.0014535071, 0.0160265211, -0.0056748749, -0.0006723482, -0.0012907272, -0.0073194727, -0.0009307071, 0.0015076200, -0.0049766386, -0.0289870203, 0.0180929881, -0.0073480136, -0.0054402542, 0.0156412050, -0.0045435056, 0.0415679365, 0.0523554347, 0.0226426851, -0.0154142790, -0.0251212027, -0.0067974632, 0.0293527506, -0.0025336796, 0.0298012327, -0.0227970053, -0.0302379131, 0.0064367834, 0.0504908115, 0.0074909981, -0.0107228607, 0.0247373320, -0.0144687248, 0.0132359108]
```

After:

```text
[0.0366296992, -0.0182185024, 0.1330299377, 0.1059506908, 0.0630148426, 0.0189134274, 0.1523023844, 0.0929090306, -0.0632185191, -0.0172564723, 0.0340957716, -0.0473868959, -0.0645541325, -0.0865872502, -0.1449920833, -0.0358831659, -0.1569090337, -0.1502727568, -0.0076226699, -0.0707449093, -0.0930146873, 0.0091073206, -0.0648097619, 0.0175223872, 0.0039232811, -0.0624555722, 0.1125202551, -0.0643264502, 0.0520459376, -0.1566736698, -0.0706168264, 0.0616783947, -0.0317700915, 0.1413957477, 0.0913106054, 0.0564718805, 0.0196089223, -0.1348370463, 0.1222854555, -0.0338327177, 0.1187400743, 0.0045771315, -0.1344321966, 0.0529430658, -0.0375969820, -0.1031212285, 0.0202728808, 0.0381161012, -0.0198408831, -0.1507370323, 0.0302807782, -0.1205541790, 0.0166181829, 0.0777545646, 0.1180882081, 0.0557370037, 0.0933613256, 0.0026232316, 0.0370569825, 0.0756325200, 0.1185193658, 0.0143766562, 0.0912887752, -0.0746104345]
```

</details>

For the first optimizer update, coordinate 0 of this embedding had:

- Value before: `-0.0575919151`
- Gradient: `0.0006925865`
- Step learning rate during warmup: `0.00001`
- Value after AdamW update: `-0.0576019064`

The gradient indicates how changing that value would affect loss. AdamW combines this signal with its running statistics and weight decay, then updates the parameter. Repeating this across many batches changed the full embedding and the model's predictions.

Before baseline training, the five closest full-space cosine neighbors of `customer` were `bus` (0.213), `educator` (0.203), `helped` (0.202), `bank` (0.201), and `risk` (0.198). After training, they became `shopper` (0.978), `client` (0.977), `buyer` (0.977), `subscriber` (0.971), and `consumer` (0.970). Those final neighbors make sense because the classroom corpus deliberately places those words in similar contexts. The offline viewer compresses 64 dimensions into three with PCA, so visual distance can be distorted; its neighbor ranking uses cosine similarity in the full 64-dimensional space.

For the prefix `the customer`, the untrained model's top predictions were diffuse: `customer` 1.60%, `bus` 1.07%, `educator` 1.04%, `us` 1.03%, and `application` 1.01%. After training, the top predictions became classroom-style verbs: `reviewed` 17.82%, `recommended` 17.12%, `ordered` 16.85%, `selected` 16.34%, and `compared` 15.97%. The complete arrays, attention rows, vectors, and update are in [inspection.json](results/baseline/inspection.json).

This is a neural network because learned matrices transform token and position embeddings through attention, feed-forward layers, residual connections, and normalization. Causal attention combines information from earlier positions but masks future positions. The final scores become probabilities, and sampling chooses the next token from that distribution.

## Temperature

Temperature changes sampling during inference; it does not update the model weights. At 0.3, the baseline output was conservative and template-like. At 0.8, it showed slightly more variety while remaining coherent. At 1.2, this particular baseline seed was still similar, while the expanded run became noticeably noisier and included `UNK`. See the complete [baseline](results/baseline/temperature_comparison.json) and [expanded](results/expanded/temperature_comparison.json) temperature samples.

## Fixed language evaluations

The unchanged public suite is [evals/language_evals.json](evals/language_evals.json), and [run_evals.py](run_evals.py) performs inference and scoring. The runner gives the model only the prompt. It scores whether the correct word has the highest probability among four fixed choices; the separately saved free continuation is evidence for inspection, not the multiple-choice score. Ties, unknown words, and excessive context receive zero in the all-case metric.

### Group and selected-category results after training

| Measurement | Baseline trained | Expanded trained |
|---|---:|---:|
| Starter patterns | 16/16 | 16/16 |
| New-wording transfer | 4/8 | 7/8 |
| Extension group | 0/24, 0 scorable | 1/24, 2 scorable |
| Negation | 0/3, 0 scorable | 0/3, 0 scorable |
| Spatial relations | 0/3, 0 scorable | 1/3, 2 scorable |

The baseline learned its repeated classroom templates well. The expanded model retained that performance and did better on familiar words in new wording. For spatial relations, it answered the inside/contains case correctly, missed the above/below case, and could not score the left/right case because `to` fell outside the retained vocabulary. All three negation cases were unscorable because at least one required prompt or choice word was omitted by the vocabulary cap. I therefore cannot claim that the model learned negation from this run.

Complete comparison and category breakdowns are available in [baseline language_eval_comparison.json](results/baseline/language_eval_comparison.json) and [expanded language_eval_comparison.json](results/expanded/language_eval_comparison.json). Every failure and free continuation remains in the linked per-case JSON and CSV files.

### Evaluation separation and leakage prevention

The fixed suite stayed in `evals/`, while teaching files stayed in `corpus/` only during the expanded Colab run. Evaluation prompts, choices, correct answers, generated outputs, and chat transcripts were never used to build vocabulary or update weights. The notebook rejected exact reserved prefixes and saved [baseline](results/baseline/eval_separation.json) and [expanded](results/expanded/eval_separation.json) separation records.

I also reviewed the two teaching files against all 48 cases. They contain no exact evaluation prompt, no four-token-or-longer prompt match, and none of the test-specific entity/answer pairings. Ordinary vocabulary and the underlying concepts overlap because otherwise the model could not learn the skills. Exact-match checks cannot detect every paraphrase, so manual separation was still necessary. Because the public suite guided the choice of extension categories, these results are a development benchmark, not proof of unseen generalization.

## Chat interface

The demonstrated model is the expanded run `20260922T055941_580949Z`, trained for 3,000 steps. Its model SHA-256 recorded in the transcript is `1869513ad6c693fbcb8568cf92b1e212f6e3d114a2a1a2de2be44dd3f2d7b7f2`.

| Prompt | Actual model response | Observation |
|---|---|---|
| `the spaceship landed on mars` | `right .` | `spaceship`, `landed`, and `mars` were unknown. |
| `keira did not carry boots . she carried a scarf . keira carried` | `.` | All words were known, but the model failed to supply the corrected object. |
| `the balcony is above the stream . the stream is` | `above the .` | All words were known, but it failed to reverse the relation to `below`. |

![Three expanded-model chat interactions](evidence/expanded_chat_3_interactions.png)

The complete transcript is [chat_transcript.json](results/expanded/chat_transcript.json). Each prompt starts fresh, the model uses at most 48 tokens of context, and messages are not added to the corpus or used for retraining. The interface generates from the saved nanoGPT model; it does not use canned answers or another model API.

To run the terminal interface:

```bash
python -m pip install -r requirements.txt
python chat.py --model results/expanded/model.pt --transcript chat_transcript.json
```

To rerun the fixed evaluations on the saved expanded model:

```bash
python run_evals.py --model results/expanded/model.pt --output rerun-evals
```

## What I learned

- A corpus is the collection of examples used for weight updates. It controls what patterns and vocabulary the model can learn.
- A token is a word or punctuation unit. Its ID is only a lookup index. Its embedding is the learned 64-number vector stored at that index.
- The model predicts the next token, compares the probabilities with the actual next token, and converts the error into loss.
- Backpropagation calculates gradients. AdamW uses those gradients to update neural-network parameters, including embeddings.
- Held-out loss matters because falling training loss alone can reflect memorization. This split is still limited because both sides share templates and source styles.
- Attention lets each position combine earlier context. Causal masking prevents it from looking at future tokens.
- Temperature changes the randomness of sampling without changing any learned weight.
- Plausible sentences and high scores on repeated templates do not mean that this tiny model understands language generally.

## Limitation and next experiment

The clearest limitation is the expanded vocabulary design. I added too many low-frequency decorative words, so the 509-token retention limit removed words needed to score the chosen skills. The model also produced incorrect relation reversals even when every prompt word was known.

My next experiment would use a smaller, more focused extension corpus with fewer unique nouns and more varied repetitions of the same negation and inverse-relation patterns. I would keep the public evaluation suite unchanged, add a separate unseen set that did not guide corpus design, and predict lower unknown-token rates and more scorable extension cases. I would change only the corpus so the result remains interpretable.

## Reproduce and inspect

1. Install `requirements.txt` and open [custom_llm.ipynb](custom_llm.ipynb), or open it in Colab.
2. For the baseline, leave `corpus/` empty except for its README, set `CORPUS = "classroom"`, `TRAINING_STEPS = 3000`, and `LEARNING_RATE = 0.001`, then run all cells.
3. For the expanded run, copy both files from [teaching_materials](teaching_materials/) into `corpus/`, keep the same settings, and run all cells from a fresh model.
4. Download the results ZIP and executed notebook separately. Do not clear outputs.
5. Open [embedding-viewer.html](embedding-viewer.html) locally and load a run's `checkpoint.json` to inspect initial/final embeddings and full-space cosine neighbors.

The original result archives are preserved as [baseline ZIP](results/custom_llm_baseline_classroom_3000_results_20260922T052027Z.zip) and [expanded ZIP](results/custom_llm_expanded_negation_spatial_3000_results_20260922T055941Z.zip).

## Repository map

- `notebooks/`: both executed experiments with visible outputs
- `teaching_materials/`: the two publishable extension sources
- `results/baseline/` and `results/expanded/`: models, plots, inspections, samples, complete eval outputs, and chat transcripts
- `evidence/`: chat screenshot
- `evals/`: unchanged public evaluation suite and guide
- `chat.py`, `run_evals.py`, `nanogpt_model.py`: runnable interface, evaluator, and pinned nanoGPT model
