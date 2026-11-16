# Reservoir Facies Classification | Well-Log Machine Learning

**Goal:** Predict **nine depositional lithofacies** from wireline measurements and geological indicators, and evaluate whether a classifier trained on one set of boreholes generalizes to an **unseen well**. The application is subsurface reservoir characterization, not a direct prediction of permeability or reserves.

## Tech stack

- **Python:** Pandas, NumPy, Scikit-learn, Matplotlib, Jupyter.
- **Models:** Random Forest (primary), RBF Support Vector Machine, Logistic Regression.
- **Validation:** Nested well-grouped cross-validation; fixed model settings, **no grid search**.

## Geology and reservoir setting

- **Location:** The **Panoma (Council Grove) gas field**, southwest Kansas, USA, within the **Hugoton Embayment** (northern shelf extension of the Anadarko Basin).
- **Geological age/unit:** **Lower Permian Council Grove Group**. The Panoma production interval lies **below the overlying Hugoton (Chase Group) gas reservoir**. "Hugoton/Panoma" is the broad regional context; the labeled benchmark is specifically associated with the **Panoma/Council Grove** reservoir.
- **Depositional setting:** Repeated marine flooding and regression on a gently dipping shelf/ramp built stacked **marine carbonate** and **nonmarine siliciclastic** intervals. Shallow-water mudstone, wackestone, packstone–grainstone and locally dolomite alternate with siltstone/shale and rare sandstone.
- **Reservoir significance:** Thin carbonate beds may form reservoir flow units, while tighter siliciclastic beds can separate them. Variable facies and rock properties across depth and between wells complicate correlation, reservoir-quality interpretation and 3D modeling.
- **Lateral variation:** The region has thicker nonmarine silt/shale intervals toward the **northwest** and proportionally more marine carbonates toward the **southeast**. Facies transitions are gradational; adjacent-facies misclassifications can be less severe geologically than unrelated rock-type errors.
- **Why well logs?** Core-based facies descriptions are limited to sampled intervals. The exercise learns relationships between **core-labeled facies** and more widely available wireline measurements, as a first step toward extending interpretation to uncored wells.

*Geological context:* [Kansas Geological Survey — depositional sequences](https://www.kgs.ku.edu/PRS/publication/2003/ofr2003-30/P1-04.html), [regional facies geometry](https://www.kgs.ku.edu/PRS/publication/2003/ofr2003-30/P1-05.html), [Hugoton vs Panoma stratigraphy](https://www.kgs.ku.edu/PRS/publication/ofr2001-33/P1-05.html), [lithofacies prediction study](https://www.kgs.ku.edu/PRS/publication/2003/ofr2003-68/index.html).

## Dataset and wells

- **Source:** Public Kansas Geological Survey/University of Kansas **Council Grove lithofacies teaching benchmark**. This project adapts the data/workflow distributed in [Ryan Mardani's multi-model facies-classification repository](https://github.com/mardani72/Facies-Classification-Machine-Learning) and related work by [Brendon Hall](https://github.com/brendonhall/facies_classification); **no new field measurements or core descriptions were collected**.
- **Labeled data:** **4,149 original rows**, including **9 real wells** — `SHRIMPLIN`, `ALEXANDER D`, `SHANKLE`, `LUKE G U`, `KIMZEY A`, `CROSS H CATTLE`, `NOLAN`, `NEWBY`, `CHURCHMAN BIBLE` — and **80 rows from a recruited `Recruit F9` pseudo-well**.
- **Unlabeled wells:** **STUART** (474 intervals) and **CRAWFORD** (356), totaling **830** predictions. Their accuracy **cannot be measured** without true facies labels.
- **Sampling:** Approximately **half-foot depth intervals**. Labels are **nine facies** spanning sandstone/siltstone/shale and several carbonate textures (mudstone, wackestone, dolomite, packstone–grainstone, bafflestone).
- **Measured log predictors (5):** `GR` (gamma ray), `ILD_log10` (log resistivity), `DeltaPHI` (density–neutron porosity difference), `PHIND` (porosity indicator), and `PE` (photoelectric factor).
- **Geological predictors (2):** `NM_M` (marine/nonmarine indicator) and `RELPOS` (relative stratigraphic position).
- **Missing curve:** `PE` is missing entirely from **ALEXANDER D** and **KIMZEY A**; the other four measured curves and both geological descriptors are available for all real labeled wells.
- **Data QC:** Excluded 80 pseudo-well rows, **1 exact duplicate**, and **4 entries at two contradictory repeated-depth keys**, retaining **4,064 labeled intervals from 9 real wells**. The excluded contradictory rows are exported for auditing; the original CSVs are unchanged.

## Methods

1. **Explore and clean:** Audit PE availability, repeated depths and class imbalance; preserve original five-log + facies plots for **CHURCHMAN BIBLE** and **LUKE G U**.
2. **Choose primary features:** Use **four measured logs + two geological indicators = six features**, excluding PE so both PE-incomplete wells remain in the dataset. A separate paired seven-well ablation checks the effect of including PE.
3. **Test on unseen wells:** Repeat **Leave-One-Well-Out** evaluation across all nine labeled wells. Each of the nine receives predictions from a model trained **without any of its samples**.
4. **Select model safely:** Within each outer fold, run **4-fold GroupKFold** on the other eight wells to compare fixed Logistic Regression, RBF SVM and Random Forest settings by **mean per-well macro-F1**. Scaling/encoding for LR/SVM is fitted only within their training folds. The fixed Random Forest is reported separately.
5. **Evaluate:** Exact facies accuracy, adjacent-facies accuracy (within predefined neighboring classes), macro-F1, per-class performance, confusion matrices, feature sensitivity and held-out-well facies tracks.
6. **Predict unlabelled wells:** Choose a final model by grouped CV on all nine labeled wells, refit it on all **4,064** observations and predict facies for **STUART and CRAWFORD**. These predictions remain **unverified**.

## Results: independent held-out wells

| Model / procedure | Mean exact accuracy across 9 wells | Mean adjacent-facies accuracy | Mean observed-class macro-F1 |
|:--|--:|--:|--:|
| **Fixed Random Forest** | **49.8%** | **89.4%** | **0.403** |
| Nested model selection | 47.7% | 88.2% | 0.390 |
| RBF SVM | 47.0% | 86.6% | 0.397 |
| Logistic Regression | 45.1% | 85.0% | 0.381 |

- **Random Forest pooled accuracy:** **49.8%** across all **4,064 out-of-well predictions** (approximately 49.78% before rounding).
- **Feature sensitivity:** Four measured logs → **40.6%** mean well accuracy; adding the two geological indicators → **49.8%**. On the **same seven PE-complete wells**, six features → **48.3%**; seven features including PE → **49.1%**.
- **Final inference:** Random Forest selected on all labeled wells; facies predicted for **830 unlabeled STUART/CRAWFORD intervals**, with **no ground-truth score**.
- **Important:** These stricter, independent-well figures are **not directly comparable** with the earlier **73.3% random-row test**; nearby intervals in a single well are correlated. Adjacent-facies accuracy is a lenient, geologically informed score—not proof of correct stratigraphic correlation.

## Conclusion and limitations

- **Random Forest** was the strongest fixed candidate on mean held-out-well accuracy; well-to-well differences remain large, especially for uncommon facies such as **bafflestone (class 9)**.
- Well-grouped validation avoids row-wise contamination. With **only nine real labeled wells**, reported results remain uncertain outside the sampled reservoir setting.
- Model quality depends partly on **geological predictors (`NM_M`, `RELPOS`)** being available or defensibly estimated at prediction time. Rock facies predictions alone do **not** establish permeability, RQI, reserves or compliance with Walther's Law.
- Future work: acquire independent core-labeled wells from other areas, verify stratigraphic availability of geological indicators, inspect rare-facies errors and validate predictions with geological experts.

## Reproduce and inspect

```bash
python -m pip install -r requirements.txt
jupyter lab
```

Open **[`facies_classification.ipynb`](facies_classification.ipynb)** and run all cells from this repository's root folder. The notebook generates audit data, well-log figures, confusion matrices, per-well metrics, feature diagnostics, and STUART/CRAWFORD predictions in `outputs/` (created on execution). The original CSV files are deliberately preserved unchanged.

**Sources:** [Bohling & Dubois, KGS OFR 2003-50](https://www.kgs.ku.edu/PRS/publication/2003/ofr2003-50.pdf) · [Dubois et al., 2007, *Computers & Geosciences*](https://doi.org/10.1016/j.cageo.2006.08.011) · [KGS 2003-30](https://www.kgs.ku.edu/PRS/publication/2003/ofr2003-30/P1-02.html) · [Mardani's reference implementation](https://github.com/mardani72/Facies-Classification-Machine-Learning)