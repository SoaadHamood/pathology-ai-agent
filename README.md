# Pathology AI: from Multiple Instance Learning to an AI Review Assistant

Three notebooks on deep learning for whole-slide pathology images. Each one builds on the one before: a simple baseline, a harder genomic prediction task, and finally an agent that reviews a slide the way a pathologist does and shows its evidence instead of just giving an answer.

## Kaggle notebooks

- Prostate baseline: https://www.kaggle.com/code/soaadhamood34/01-prostate-baseline-mil
- Lung EGFR: https://www.kaggle.com/code/soaadhamood34/02-lung-egfr-gated-attention
- EGFR review assistant: https://www.kaggle.com/code/soaadhamood34/03-egfr-review-assistant
- 
| # | Notebook | What it does | Main result |
|---|---|---|---|
| 1 | [Prostate baseline](01-prostate-baseline-mil.ipynb) ([run on Kaggle](https://www.kaggle.com/code/soaadhamood34/01-prostate-baseline-mil)) | Predicts clinically significant prostate cancer (ISUP 2 or higher) from 300 PANDA slides with a simple max-pooling MIL model | AUROC 0.79 (95% CI 0.73 to 0.83); 0.72 and 0.63 when tested on the other hospital |
| 2 | [Lung EGFR](02-lung-egfr-gated-attention.ipynb) ([run on Kaggle](https://www.kaggle.com/code/soaadhamood34/02-lung-egfr-gated-attention)) | Predicts EGFR driver mutations in lung adenocarcinoma from 75 TCGA slides, with clean driver-mutation labels, the Phikon pathology encoder and CLAM-style gated attention | AUROC 0.65 (95% CI 0.51 to 0.78) |
| 3 | [EGFR review assistant](03-egfr-review-assistant.ipynb) ([run on Kaggle](https://www.kaggle.com/code/soaadhamood34/03-egfr-review-assistant)) | An agent that scans a slide at 5x, zooms into chosen regions at 20x, retrieves similar past cases, and writes a checked LLM summary for a pathologist | Reads 2.4% of the tiles for AUROC 0.60 (whole slide 0.65); after fixing the prompt, all 30 LLM summaries passed the automatic checks |

## What the review assistant shows a pathologist

For each slide, one review page with:

- **Where the agent looked**, in order, and **which regions drove its prediction**
- **20x close-ups** of those regions
- **Similar past cases**, compared tile to tile with this slide, with their known EGFR mutation (for example "exon 19 deletion")
- **A short summary** written by an LLM (Claude Haiku 4.5)

The recommendation comes from a safety rule in code, not from the LLM, and every summary is checked automatically for invented numbers and a missing disclaimer. A worklist puts the cases that most need a human at the top. The decision always stays with the pathologist.

## Things I found along the way

- **Domain shift is real.** The prostate model dropped from 0.79 to 0.72 and 0.63 when tested on a hospital it never trained on.
- **Pen marks.** Many TCGA slides have marker ink drawn around the tumor, and my brightness-based tissue mask counts it as tissue. The attention model learned to give ink tiles almost zero weight, but a color filter would be cleaner.
- **The agent saves time, not accuracy.** Choosing regions by attention did not beat choosing them at random, because the 5x model that picks the regions is weak (AUROC 0.53). Improving it is the clearest next step.
- **LLM checks matter.** In my first run, only 57% of the summaries passed the number check, because the model calculated its own counts. After making the case file clearer and the prompt stricter, all 30 passed.

## How to run

The notebooks run on Kaggle with a GPU. Open a Kaggle link and click **Copy & Edit**; the data inputs are already attached there.

- Notebook 1 needs the PANDA competition data (accept the competition rules on Kaggle once).
- Notebook 2 downloads the TCGA slides from the GDC, which takes a few hours.
- Notebook 3 uses the saved output of notebook 2. The LLM summaries need your own Claude or Gemini API key as a Kaggle Secret; without one, the notebook uses a template summary and everything else still runs.

## Data and tools

- [PANDA Challenge](https://www.kaggle.com/c/prostate-cancer-grade-assessment): prostate biopsies from Karolinska and Radboud
- [TCGA-LUAD](https://portal.gdc.cancer.gov/) whole-slide images from the NIH Genomic Data Commons, and mutation calls from [cBioPortal](https://www.cbioportal.org/)
- [Phikon](https://huggingface.co/owkin/phikon) pathology encoder (Owkin)
- PyTorch, OpenSlide, scikit-learn, Claude API

## Limitations

These are research prototypes on small public datasets, not clinical tools. Each notebook lists its own limitations in detail.

## How I built this

I designed the projects, chose the methods and the evaluation, ran every notebook and wrote the interpretations. Most of the code was written with Claude (Anthropic's AI assistant) based on my design and guidance, and I went through each part so I can explain what it does and why.
