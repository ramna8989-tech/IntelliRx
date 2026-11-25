# Running TraceDR on a smaller DrugRec subset

This package is based on your current version (AMP, gc and memory flags are kept). Copy these files into `TraceDR-model/`, replacing the old ones.

## 1. Build the subset (one time)

If you already have `intermediate_data/DrugRec0716/*.pkl` (no Neo4j needed):
```bash
python make_subset.py --from_intermediate intermediate_data/DrugRec0716 --out_bench DrugRec_small --frac 0.1
#   or exact sizes: --n_train 2000 --n_dev 250 --n_test 250
```
If you only have `data/DrugRec.json`:
```bash
pip install ijson
python make_subset.py --from_json data/DrugRec.json --out_bench DrugRec_small --n 2500
```
If the JSON records already contain `top_k_drugs`, they go to `intermediate_data/DrugRec_small/` and you can train right away. If they don't, they go to `data/DrugRec_small/`, and the first training run needs `--retrieval` with Neo4j running.

The subset is fixed (seed 1203), and its IDs are saved in `subset_ids.json`. Every model you compare must use this same subset.

## 2. Baseline (paper settings: max_entities 100, max_evidences 50)
```bash
python train_TraceDR.py --benchmark DrugRec_small --model_name TraceDR_small_baseline
python train_TraceDR.py --eval --benchmark DrugRec_small --model_name TraceDR_small_baseline --test_output output/TraceDR_small_baseline
```
Check the line printed while loading the data:
`Answer presence (evidence): … | Gold drug rankable (entity): …`
The second number is the upper limit on P@1 and Hit@5.

## 3. Only if the GPU runs out of memory
Try `--max_entities 60 --max_evidences 30` first, and only then go lower. Whatever values you use, report them next to the baseline and keep them the same for the improved model.

## What changed compared with your current files
| File | Change |
|---|---|
| metrics.py | Removed the hard-coded `intermediate_data/DrugRec0716/dev.pkl` load. It used to crash test evaluation and any other benchmark with a KeyError. It now loads the split being evaluated. The CMS, DDI_rate and group_rate metrics are re-enabled. |
| train_TraceDR.py | `max_entities`/`max_evidences` defaults are back to 100/50 (they were 30/20). `set_patient_data()` is called for dev and test. AMP is enabled only when CUDA is present. |
| data_process.py | Prints entity-level gold coverage. `retrieve_drugs` no longer crashes when diagnosis/symptom are lists (this matters only with `--retrieval`). |
| make_subset.py | New file. |
| model.py, bm25_es.py | Unchanged. Note that the loss weights in `model.py` are 0.7/0.3; the earlier copy had 0.5/0.5. |
