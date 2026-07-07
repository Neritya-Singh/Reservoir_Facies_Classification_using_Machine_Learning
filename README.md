# Reservoir Facies Classification using Random Forest Ensembles

**Author:** Neritya Kumar Singh  
**Institution:** Indian Institute of Technology (IIT) Roorkee, India

---

## Project Overview
This project focuses on the automated classification of depositional rock facies using wireline log data. Extending the foundational work by Brendon Hall (Enthought) on the Hugoton and Panoma gas fields, this project replaces a traditional Support Vector Machine (SVM) approach with a **Random Forest Classifier**.

By leveraging ensemble learning, this model addresses the non-linear relationships and severe class imbalances typical of complex, mixed carbonate-siliciclastic systems, resulting in improved predictive accuracy and geological consistency.

## Geological Context
The study area comprises the **Hugoton and Panoma Fields**, the largest natural gas fields in North America. These reservoirs are characterized by complex, heterogeneous lithologies. Accurate facies identification is critical for:
* Building high-fidelity 3D reservoir models.
* Mapping high-permeability flow units.
* Calculating the Reservoir Quality Index (RQI).

## Technical Methodology
The workflow follows a standard petrophysical machine learning pipeline:

1.  **Exploratory Data Analysis (EDA):** Visualization of wireline physics (GR, PE, PHIND) against core-calibrated facies.
2.  **Data Preprocessing:** * Custom masking to handle missing PE log runs.
    * Application of `StandardScaler` to normalize feature distributions across wells.
3.  **Model Training:** Deployment of a Random Forest ensemble using `class_weight='balanced'` to mitigate bias toward dominant background facies (e.g., Marine Shale/Siltstone).
4.  **Validation:** Testing on blind wells (**STUART** and **CRAWFORD**) to verify geological consistency and bed thickness blockiness.

## Performance Metrics
The transition from an SVM-based model to the optimized Random Forest yielded significant improvements:

| Metric | Result |
| :--- | :--- |
| **Exact Facies Accuracy** | ~73.2% |
| **Adjacent Facies Accuracy** | ~93.0% |

> **Note on "Adjacent Accuracy":** The high adjacent accuracy (93%) confirms the model successfully captures the gradational, transitional nature of depositional environments—a vital requirement in geoscience, where misclassifications often occur between geologically similar neighbor facies.

## Repository Contents
* `facies_vectors.csv`: Training dataset (9 core-calibrated wells).
* `validation_data_nofacies.csv`: Blind test data for STUART and CRAWFORD wells.
* `facies_classification.ipynb`: Complete research notebook containing the implementation, geological insights, and log plots.

## References
* Bohling, G. C., and M. K. Dubois (2003). *An Integrated Application of Neural Network and Markov Chain Techniques to Prediction of Lithofacies from Well Logs*, KGS Open-File Report 2003-50.
* Dubois, M. K., G. C. Bohling, and S. Chakrabarti (2007). *Comparison of four approaches to a rock facies classification problem*, Computers & Geosciences, 33 (599-617).

---
*Developed as an independent student research project at IIT Roorkee.*