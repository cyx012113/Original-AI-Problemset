# 🌌 Cosmic Probe: Star–Galaxy–Quasar Classification from SDSS Photometry

## Background

Modern wide-field sky surveys generate enormous amounts of imaging and photometric data every night. The fifth-generation Sloan Digital Sky Survey (SDSS-V) has released over $4.6\times10^6$ spectroscopically confirmed celestial objects, providing an irreplaceable data resource for studying the large-scale structure of the Universe, galaxy evolution, and the activity of supermassive black holes.

However, spectroscopic follow-up is both expensive and time-consuming, meaning that only a tiny fraction of the billions of detected sources can receive a definitive classification label. AI for Science offers a breakthrough: by training classification models on the millions of objects that _do_ have spectroscopic labels, astronomers can efficiently sift through survey images to identify high-value targets—especially rare quasars—thus dramatically increasing the scientific return of expensive telescope time.

This challenge is inspired by recent ArXiv research on automated classification of SDSS sources using machine learning (see references below). You will act as a **“digital astronomer”** and build a model that uses photometric magnitudes in five optical bands to classify each object into one of three categories: **STAR**, **GALAXY**, or **QSO** (quasar).

Quasars are distant active galactic nuclei powered by rapidly accreting supermassive black holes; they are key probes of the early Universe. Yet they constitute only about $18.8\%$ of the dataset and are easily confused with stars or compact galaxies. **The sole evaluation metric is the $F_2$ score**, which emphasizes recall—**missing a single quasar carries a far greater scientific cost than misclassifying a few ordinary objects**.

## Task Description

You are required to:

1. Understand and preprocess the SDSS photometric catalog data;
2. Design and train a classification model;
3. Input: model magnitudes in five bands ($u, g, r, i, z$) for each object;
4. Output: the predicted class (`GALAXY`, `STAR`, or `QSO`);
5. **Final ranking is based solely on the $F_2$ score achieved on the B test set**.

The dataset contains $90000$ records, all with spectroscopic confirmation from SDSS. The data have been randomly split into training, A test, and B test sets as described below.

### Dataset

The data are derived from **SDSS Data Release 17** public catalogs, using the `PhotoPrimary` view joined with the `SpecObj` spectral table to select objects with spectroscopic classes `GALAXY`, `STAR`, or `QSO`. The total number of records is $4618598$, randomly partitioned into the following CSV files.

#### File Descriptions

| Filename             | Number of Samples | Description                                                                  |
| :------------------- | :---------------- | :--------------------------------------------------------------------------- |
| `train.csv`          | $30000$           | Training set, containing both features and labels                            |
| `A.csv`              | $30000$           | Public test set (A), containing only features (no labels)                    |
| `B.csv`              | $30000$           | Final test set (B), containing only features (no labels)                     |
| `A_ground_truth.csv` | $30000$           | Ground truth for A (for local validation only; scores not publicly released) |
| `B_ground_truth.csv` | $30000$           | Ground truth for B (used for final ranking; not released)                    |

#### CSV Format

All CSV files are comma-separated, UTF-8 encoded, and include a header row.

**Columns in `train.csv`:**

| Column       | Data Type | Description                                     |
| :----------- | :-------- | :---------------------------------------------- |
| `objid`      | integer   | Unique SDSS object identifier (64-bit integer)  |
| `ra`         | float     | Right Ascension (degrees)                       |
| `dec`        | float     | Declination (degrees)                           |
| `modelMag_u` | float     | $u$-band model magnitude                        |
| `modelMag_g` | float     | $g$-band model magnitude                        |
| `modelMag_r` | float     | $r$-band model magnitude                        |
| `modelMag_i` | float     | $i$-band model magnitude                        |
| `modelMag_z` | float     | $z$-band model magnitude                        |
| `type`       | string    | Spectroscopic class: `GALAXY`, `STAR`, or `QSO` |

**Columns in `A.csv` and `B.csv`:**

| Column       | Data Type | Description                   |
| :----------- | :-------- | :---------------------------- |
| `objid`      | integer   | Unique SDSS object identifier |
| `ra`         | float     | Right Ascension (degrees)     |
| `dec`        | float     | Declination (degrees)         |
| `modelMag_u` | float     | $u$-band model magnitude      |
| `modelMag_g` | float     | $g$-band model magnitude      |
| `modelMag_r` | float     | $r$-band model magnitude      |
| `modelMag_i` | float     | $i$-band model magnitude      |
| `modelMag_z` | float     | $z$-band model magnitude      |

> **Note**: The test files **do not contain** the `type` column. Participants must predict the class for each object.

#### Class Labels

| Label    | Meaning | Scientific Description                                                            |
| :------- | :------ | :-------------------------------------------------------------------------------- |
| `GALAXY` | Galaxy  | A gravitationally bound system of billions of stars                               |
| `STAR`   | Star    | A luminous plasma sphere within the Milky Way, powered by nuclear fusion          |
| `QSO`    | Quasar  | A distant active galactic nucleus with a supermassive black hole accreting matter |

Both training and test splits preserve the original class proportions, making this a typical **imbalanced multi-class classification** problem.

### Evaluation Metric

The sole evaluation metric is the **$F_2$ score (F2 Score)** , which is a weighted harmonic mean of precision and recall where recall is weighted four times as much as precision:

$$
F_2 = (1+2^2) \cdot \frac{\text{Precision} \cdot \text{Recall}}{(2^2 \cdot \text{Precision}) + \text{Recall}}
= 5 \cdot \frac{\text{Precision} \cdot \text{Recall}}{4 \cdot \text{Precision} + \text{Recall}}
$$

The $F_2$ score emphasizes recall, meaning the model should **“rather misclassify than miss”**—especially for the rare `QSO` class. In real astronomical surveys, missing a high-redshift quasar could mean losing a rare window into the early Universe.

**Computation**: A confusion matrix is constructed for all test samples. Precision and recall are computed for each class (`GALAXY`, `STAR`, `QSO`), and the per-class $F_2$ scores are aggregated using a **weighted average**, where the weights are the number of true instances of each class in the test set.

### Submission Format

Participants must generate prediction files for `A.csv` and `B.csv`, named `A_predict.csv` and `B_predict.csv` respectively.

Each prediction file must contain exactly two columns:

| Column  | Data Type | Description                                              |
| :------ | :-------- | :------------------------------------------------------- |
| `objid` | integer   | Must match the `objid` in the original test file exactly |
| `type`  | string    | Predicted class, one of `GALAXY`, `STAR`, or `QSO`       |

Example submission snippet:

```csv
objid,type
1237645879551000764,GALAXY
1237645879551066262,STAR
1237645879562862699,QSO
```

> **Note**: The order of `objid` must match the test file row by row. The number of rows must equal the number of test samples; otherwise the submission will be invalid.

### Constraints

1. No pretrained model weights are allowed (including models pretrained on ImageNet or any astronomical dataset). Models must be trained from random initialization;
2. Only third-party libraries permitted by the NOAI syllabus may be used (e.g., `numpy`, `pandas`, `scikit-learn`, `torch`, `matplotlib`);
3. External large language model APIs (e.g., GPT, Claude) may not be used for prediction or feature generation;
4. Model complexity and training time should be kept reasonable (suggested parameter count $\le 10^6$, single training run $\le 10$ minutes);
5. Final submission must include both `A_predict.csv` and `B_predict.csv`.

## References

The dataset originates from SDSS-V Data Release 17. The following ArXiv papers provide background on the data and classification tasks:

- **Paper**: Abdurro'uf, et al. (2022). _The Seventeenth Data Release of the Sloan Digital Sky Surveys: Complete Release of MaNGA, MaStar, and APOGEE-2 Data_. _The Astrophysical Journal Supplement Series_, 259(2), 35.
  ArXiv link: [https://arxiv.org/abs/2112.02026](https://arxiv.org/abs/2112.02026)
  (The PDF of this paper is included in the competition zip archive.)

- **Classification reference**: Clarke, A. O., et al. (2020). _Identifying galaxies, quasars, and stars with machine learning: A new catalogue of classifications for 111 million SDSS sources without spectra_._Astronomy & Astrophysics_, 639, A84.
  ArXiv link: [https://arxiv.org/abs/1909.10963](https://arxiv.org/abs/1909.10963)

Participants may consult these papers to understand the scientific context, but **direct use of pretrained models or external code repositories from these works is prohibited**.

## Acknowledgement

Funding for the Sloan Digital Sky Survey V has been provided by the Alfred P. Sloan Foundation, the Heising-Simons Foundation, the National Science Foundation, and the Participating Institutions. SDSS acknowledges support and resources from the Center for High-Performance Computing at the University of Utah. SDSS telescopes are located at Apache Point Observatory, funded by the Astrophysical Research Consortium and operated by New Mexico State University, and at Las Campanas Observatory, operated by the Carnegie Institution for Science. The SDSS web site is <www.sdss.org>.

SDSS is managed by the Astrophysical Research Consortium for the Participating Institutions of the SDSS Collaboration, including the Carnegie Institution for Science, Chilean National Time Allocation Committee (CNTAC) ratified researchers, Caltech, the Gotham Participation Group, Harvard University, Heidelberg University, The Flatiron Institute, The Johns Hopkins University, L'Ecole polytechnique f\'{e}d\'{e}rale de Lausanne (EPFL), Leibniz-Institut f\"{u}r Astrophysik Potsdam (AIP), Max-Planck-Institut f\"{u}r Astronomie (MPIA Heidelberg), Max-Planck-Institut f\"{u}r Extraterrestrische Physik (MPE), Nanjing University, National Astronomical Observatories of China (NAOC), New Mexico State University, The Ohio State University, Pennsylvania State University, Smithsonian Astrophysical Observatory, Space Telescope Science Institute (STScI), the Stellar Astrophysics Participation Group, Universidad Nacional Aut\'{o}noma de M\'{e}xico, University of Arizona, University of Colorado Boulder, University of Illinois at Urbana-Champaign, University of Toronto, University of Utah, University of Virginia, Yale University, and Yunnan University.
