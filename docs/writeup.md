# Hotel Procurement Catalogue Challenge: write-up


## Part 1: Classify the items (macro-F1)

**Approach.** I classify 820 test items into 18 categories from 2,445 labelled ones. Descriptions are short, misspelt and glued together (`Focacciia20x300 g`, `6X2KGREFRIGERATED`). I lowercased them, removed unnecessary token, split glued numbers and words, and used character n-gram TF-IDF (2 to 5 characters) so typos still match. A regex parser added pack count, per-item size, dimensions and storage, and used the unit of measure to turn them into price per kg, litre and item. Supplier, unit of measure, buyer group and Metro SKU segment were one-hot encoded, about 10,000 columns in all. Macro-F1 weights every category equally and the smallest has only 40 rows, so I used balanced class weights. I compared six model families with Optuna (30 trials, 5-fold stratified CV) and kept a ridge classifier. Linear models beat boosting, as is typical for sparse text with few rows. Predictions for all 820 test rows are in the submission format, with ids checked against `sample_submission.csv`.

**Model Results**

- **Model comparison (5-fold CV, macro-F1):** ridge **0.839** (fold SD 0.005), LinearSVC 0.822, SGD 0.805, logistic 0.786, XGBoost 0.756, CatBoost 0.679. Ridge also fits fastest, 0.3 s per fold against 38 s for XGBoost. Boosting got only 2 to 4 trials each, so it is under-tuned.
- **Fit:** train F1 is 0.94 against 0.84 held out. Ridge has the smallest gap of the linear models (0.105, against 0.126 for LinearSVC and 0.186 for logistic). CatBoost's gap is smaller (0.075), but it scores far lower.
- **Per class:** the weakest are Frozen Prepared Foods (F1 0.51) and Dry Goods & Pantry (0.74). The other 16 categories score 0.78 or higher.

**In production, watch:**

- **New suppliers.** 200 of the 820 test rows (24%) come from Orion Food Distributors, which has no labelled rows. Random CV cannot show how the model copes, and some inputs are supplier-specific (supplier one-hot, Metro SKU segment).
- **`buyer_group`.** It is assigned during category review. If it is set after, or because of, the category, the model uses information it will not have at scoring time. 
- **No confidence.** The model returns hard labels, so low-margin items and weak classes need a human review path.

## Part 2: Extract attributes (`size`, `storage`, `pack_qty`)

**Approach.** Regex rules applied one row at a time, adapted from Part 1's parser. Each description is cleaned, storage is read, non-sizes (%, watts, GSM, thread count, garment sizes) are removed, then size and pack patterns are matched and blanked out so no number is read twice. Order matters: `46X33CM` is a dimension before `500X12GM` is a pack. A value is filled only when the description states it, so blank means "not stated". I chose rules over a model because they are deterministic, auditable and free to run. On the 200 items, `size` is filled on 167, `storage` on 47 and `pack_qty` on 114.

**How I checked it works.** 37 tests built from the brief's examples and traps (`10 LAMB` is not 10 L, chilli is not chill, `3.5% fat` and `XL` are not sizes), 29 of the hardest real rows with hand-read answers, and a hand read of all 200 outputs with no disagreements. The output format is validated before writing. **Caveat:** I wrote the rules from these 200 rows, so this says little about unseen descriptions. I have no accuracy figure on new data.

**Cost and time for 10,000 new items.** The extractor runs in 0.30 s without API calls. The real cost is people's time: reviewing about 500 flagged rows (5% at 30 s each) takes 4.2 h, spot-checking 200 random rows 0.8 h, and new-format rules up to 4 h.

**Alternative** For further improvement of this extractor, we could pivot to using a small transformer model (e.g., BERT) to extract the attributes but this would require a labelled dataset of sufficient size.

**In production, watch:**

- **Silent misses.** An unknown storage spelling leaves `storage` blank with no flag, and truncated text (`Butter 82% fa`) cannot be recovered. A miss looks the same as "not stated".
- **Format drift.** Rules decay as suppliers change formats. Track blank and flag rates per supplier, hand-check a weekly sample and version the rules.

