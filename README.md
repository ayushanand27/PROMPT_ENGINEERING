# Does the way you ask an AI change how well it thinks?

**A prompt-engineering research project** (Course: Prompt Engineering, IS4143 · B.Tech CSE (IoT & Intelligent Systems) · Manipal University Jaipur · Semester 7)

**Author:** Ayush Anand (23FE10CII00144)
**Project topic:** *A Comparative Evaluation of Zero Shot and Few Shot Prompting Strategies for Enhancing Complex Reasoning in Large Language Models*

> **In one line:** We asked two AI models 1,164 reasoning questions in four different ways, then measured which way gets the most answers right and what each way costs.

---

## 1. The idea in simple words

When you chat with an AI (like ChatGPT), the words you type are called a **prompt**. Two prompts that ask for the same thing can give answers of very different quality.

Think of a **student taking a maths test**:

| How the teacher asks | What it is called here | Real-life picture |
|---|---|---|
| "Here's the question. Just give the answer." | **Zero-shot** | Asking a friend, "What's a 15% tip on ₹1,240?" and expecting an instant answer |
| "Here's the question. Think step by step." | **Zero-shot CoT** (chain of thought) | Same friend, but you say, "Work it out on paper first" |
| "Here are 4 solved examples. Now do this one." | **Few-shot** | Showing a new employee 4 filled-in forms, then handing over a blank one |
| "Here are 4 solved examples **with the working shown**. Now do this one." | **Few-shot CoT** | Showing 4 filled-in forms *and* how each number was calculated |

**Our question:** which of these four ways gets the best answers, how much extra writing (cost) does each one need, and does the best way depend on the *type* of question?

---

## 2. What we built

A small, complete **experiment machine** in Python:

1. **Gets the questions** from 3 well-known public test sets (see section 3).
2. **Writes the four kinds of prompt** for every question (`prompts_data.py`).
3. **Sends them to the AI models** through the Groq website's API and saves every single answer (`run_experiment.py`). It can be stopped and restarted without losing work.
4. **Marks the answers** automatically: it reads the line "Answer: ..." and compares it with the correct answer.
5. **Does the maths and draws the charts** (`analyze.py`): accuracy, confidence ranges, significance tests and cost.
6. **Writes the final report** as a Word file in the exact course format (`build_report.py`).

Everything can be repeated: the questions are picked by a fixed rule, the AI runs at temperature 0 (no randomness), and **every raw answer the AI gave is saved** in `results/raw.jsonl`.

---

## 3. How we tested (the exam)

**Three kinds of questions**, each needing a different kind of thinking:

| Test set | Kind of thinking | A real example of this kind of question |
|---|---|---|
| **GSM8K** | Maths word problems | *"Marcus is half of Leo's age and five years younger than Deanna. Deanna is 26. How old is Leo?"* (answer: 42) |
| **StrategyQA** | Common-sense yes/no | *"Is the Illuminati card game still popular?"* (answer: No) |
| **Date Understanding** | Calendar reasoning | *"Yesterday was April 30, 2021. What is the date today in MM/DD/YYYY?"* (answer: 05/01/2021) |

**Two AI models** (both open models, used through Groq):
- **ALLaM-2-7B**: a *small* model (7 billion parameters)
- **Qwen3.8-27B**: a *medium* model (27 billion parameters)

**Fair rules:** every method gets the same questions and the same answer format, there is no randomness, the examples used for teaching were *removed* from the test questions (no cheating), and a missing or badly formatted answer counts as wrong.

**How much was run**

| | GSM8K | StrategyQA | Date Understanding |
|---|---|---|---|
| ALLaM-2-7B | 60 questions | 60 | 60 |
| Qwen3.8-27B | 60 | 51 | not finished (see "Honest limits") |

Total: **1,164 scored answers**, each marked right or wrong.

---

## 4. What we found (results in easy words)

Accuracy = % of questions answered correctly. The best score in each row is in **bold**.

| Model · Test | Zero-shot | Zero-shot CoT | Few-shot | Few-shot CoT |
|---|---|---|---|---|
| ALLaM-2-7B · Maths (GSM8K) | 11.7% | 46.7% | 13.3% | **56.7%** |
| Qwen3.8-27B · Maths (GSM8K) | 53.3% | 93.3% | 61.7% | **95.0%** |
| ALLaM-2-7B · Yes/No (StrategyQA) | 70.0% | **75.0%** | 66.7% | 68.3% |
| Qwen3.8-27B · Yes/No (StrategyQA) | 76.5% | 80.4% | **86.3%** | 82.4% |
| ALLaM-2-7B · Dates | 36.7% | 30.0% | 45.0% | **56.7%** |

### The five big takeaways

1. **For maths, "show your working" is the magic ingredient.** It lifted the small model from 12% to 57% and the medium model from 53% to 95%. These gains are statistically real, not luck.
2. **Examples alone did not help maths.** Showing solved examples without the working barely moved the score (13% vs 12% for the small model). The AI copies the *format* but still just guesses the number.
3. **For common-sense yes/no questions, no method made a statistically real difference.** Thinking longer can even make the AI talk itself into a wrong answer. Example: for *"Is the Illuminati card game still popular?"* the model wrote a long, confident explanation and still said "Yes" (wrong).
4. **Few-shot CoT is the best all-rounder, and it is cheaper than zero-shot CoT.** For the small model it used about **105 tokens** per answer versus **234** for zero-shot CoT, and was more accurate. The examples teach the AI to be short and to the point.
5. **Bigger model = better, but the same tricks still help.** The medium model beat the small one everywhere we tested it, and "show your working" gave both models a similar boost (about +35 to +45 points on maths).

### A real example of "guessing vs working it out"

The Marcus and Leo question (correct answer **42**):

| Method | ALLaM (small) | Qwen (medium) |
|---|---|---|
| Just the answer | said **20** ❌ | said **62** ❌ |
| Few-shot with working | *"Marcus = 26 − 5 = 21, so Leo = 21 × 2 = **42**"* ✅ | worked step by step → **42** ✅ |

Without working, both models effectively **guessed**. With working, both got it right.

### An interesting failure we found and explained

On the date questions, ALLaM with *"think step by step"* did **worse** than plain zero-shot (30% vs 37%). Why? In 14 of 60 answers it wrote a full date instead of the option letter (for example "01/02/2015" instead of "B"), so the answer could not be marked. Showing examples fixed this completely (0 of 60 such mistakes).

---

## 5. What this means in real life

These are practical ideas based on our results. They are *guidelines*, not guarantees, because we tested only two models and three kinds of questions.

| If you are building... | Our results suggest | Why |
|---|---|---|
| A bill, refund, discount or interest calculator bot | Use **few-shot with working** (few-shot CoT) | Maths improved the most (up to +45 points) |
| A bot that books meetings or works out deadlines and dates | Use **few-shot CoT** and show the exact answer format | Examples also stop format mistakes (tested on the small model) |
| An FAQ / yes-no knowledge bot | A **short, plain prompt** is usually enough | No method won significantly, and long reasoning costs more |
| A bot where cost or speed matters | **Avoid long "think step by step"**; prefer few-shot CoT or plain few-shot | Zero-shot CoT produced up to about 80× more output than a plain answer |
| A study helper for school maths | Always ask the AI to **show its steps**, and check them | The steps are what make the answer right |

**Tip for everyday ChatGPT users:** for any calculation, add *"Show your working step by step."* For a quick factual question, don't bother.

---

## 6. Honest limits (what we did *not* manage)

Good research says what it could not do:

- **Qwen's Date Understanding test was not completed.** The free Groq plan allows only 200,000 tokens per day for that model, and it ran out. Qwen's StrategyQA test was completed for 51 of 60 questions. (The runner can resume later to finish the gap.)
- **Small samples.** 51-60 questions per test means a margin of roughly ±12 points, so only big differences are statistically solid.
- **Different models than planned.** The Llama models we first wanted were no longer offered on Groq.
- **One run, no randomness.** We did not measure run-to-run variation or how much the choice of examples matters.
- The AI may have seen these public tests during its training.

All of this is also written in the report (section 7).

---

## 7. What is inside this repository

```
├── README.md                 ← you are here
├── report/
│   ├── Prompt_Engineering_IS4143_Research_Report.docx   ← the final 12-page report (Word)
│   └── Prompt_Engineering_IS4143_Research_Report.pdf    ← the same report as PDF
├── prompts_data.py           ← picks questions, writes the 4 prompt styles, marks answers
├── run_experiment.py         ← sends questions to the AI, saves every answer (restartable)
├── analyze.py                ← accuracy, confidence ranges, significance tests, charts
├── build_report.py           ← builds the Word report from the results
├── report_static.py          ← report text that does not depend on results (intro, method, references)
├── docx_helpers.py           ← formatting helpers (Times New Roman, 1.5 spacing, etc.)
├── make_pipeline_fig.py      ← draws the pipeline figure used in the report
├── data/                     ← the public question sets (GSM8K, StrategyQA, Date Understanding)
└── results/
    ├── raw.jsonl             ← EVERY answer the AI gave (the evidence)
    ├── accuracy.csv          ← accuracy per model / test / method, with confidence ranges
    ├── overall.csv           ← average output length and overall accuracy
    ├── mcnemar.csv           ← significance tests (all questions pooled)
    ├── mcnemar_by_dataset.csv← significance tests per test set
    └── *.png                 ← the charts used in the report
```

---

## 8. How to run it yourself

You need Python 3.10+ and a free [Groq API key](https://console.groq.com).

```bash
pip install requests pandas scipy matplotlib python-docx

# 1) run the experiment (the key is read from your environment and is never saved in any file)
export GROQ_API_KEY="your-key-here"
python run_experiment.py --n 60 --models allam-2-7b                                   # small model
python run_experiment.py --n 60 --workers 1 --otpm 950 --models qwen/qwen3.8-27b     # medium model (paced to its rate limit)

# 2) analyse and draw the charts
python analyze.py

# 3) build the Word report
python build_report.py "Your Name" "Your Registration Number"
```

Tips
- `run_experiment.py` **skips finished work**, so you can stop and re-run it safely. Re-running it later is also how to complete the missing Qwen runs.
- Free-plan limits are strict. `--otpm 950` makes the runner wait so it never gets blocked.
- If you ever share a key by accident, create a new one and delete the old one. Never commit a key to GitHub.

---

## 9. Words you may not know

| Word | Simple meaning |
|---|---|
| **LLM** (large language model) | The AI that predicts and writes text, e.g. ChatGPT, Qwen |
| **Prompt** | What you type to the AI |
| **Zero-shot / few-shot** | Asking with 0 examples / with a few examples |
| **Chain of thought (CoT)** | Making the AI write its reasoning steps before the answer |
| **Token** | A small piece of text (roughly ¾ of a word); AI services are priced and limited by tokens |
| **Accuracy** | % of questions answered correctly |
| **Confidence interval (Wilson)** | A range that says "the true accuracy is probably in here" |
| **McNemar test / p-value** | A fairness check: "could this difference just be luck?" A p-value below 0.05 means *probably not luck* |
| **Temperature 0** | Turns off randomness so the same question gives the same answer |

---

## 10. Credits and sources

- **GSM8K**: Cobbe et al., *Training Verifiers to Solve Math Word Problems* (OpenAI), 2021.
- **StrategyQA** and **Date Understanding**: from **BIG-bench** (Srivastava et al., 2022; StrategyQA by Geva et al., 2021).
- Ideas tested: *Language Models are Few-Shot Learners* (Brown et al., 2020), *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models* (Wei et al., 2022), *Large Language Models are Zero-Shot Reasoners* (Kojima et al., 2022). The full list of 17 references is in the report.
- Models: ALLaM-2-7B and Qwen3.8-27B, accessed through the Groq API.

*Submitted for the Research Based Project, Prompt Engineering (IS4143), July-Nov 2026.*
