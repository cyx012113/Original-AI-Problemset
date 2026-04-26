# Original-AI-Problemset

![CC BY-NC-SA 4.0](http://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-nc-sa.svg)

A curated collection of **original AI competition problems** spanning multiple difficulty levels.
Every problem is self-contained, with bilingual descriptions, baseline code, public/private datasets, and official scoring materials — ready for use in contests, courses, or self-study.

## Difficulty Levels

- **beginner** – Entry-level tasks; minimal background required.
- **intermediate** – Standard machine learning or basic deep learning problems.
- **advanced** – Complex modelling, multimodal data, or advanced architectures.
- **expert** – Research-oriented challenges that may involve domain-specific knowledge (e.g., computational biology, protein folding).

## Repository Structure

Each problem follows a strict directory layout:

```tree
{Problem-Name}/
├── LICENSE
├── Contestant Data/
│   ├── {Problem-Name-EN}.md       # English problem statement (Markdown)
│   ├── {Problem-Name-EN}.pdf      # English problem statement (PDF)
│   ├── {Problem-Name-CN}.md       # Chinese problem statement (Markdown)
│   ├── {Problem-Name-CN}.pdf      # Chinese problem statement (PDF)
│   ├── baseline.ipynb             # Simple baseline solution
│   ├── Data/
│   │   ├── train.csv
│   │   ├── A.csv                  # Public test set
│   │   └── B.csv                  # Private test set
│   ├── Paper_{Reference}.pdf      # Academic paper of dataset
│   ├── README.md                  # Task-specific instructions
│   └── LICENSE
└── Rating Data/
    ├── A_ground_truth.csv
    ├── B_ground_truth.csv
    ├── std.ipynb                  # Reference answer script
    └── LICENSE
```

- **Contestant Data** – Everything provided to participants.
- **Rating Data** – Ground truth and scoring code; normally kept private during a live competition.

## License

All problems in this repository are released under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)** license.
See the `LICENSE` file in the root directory and inside each problem folder for details.

## Usage

- **Competition organisers**: fork the relevant problems, host them on your platform, and use `Rating Data` for evaluation.
- **Learners & practitioners**: clone the repo, read the problem statements, and experiment with your own solutions.
- **Contributors**: you are welcome to submit new original problems via pull request. Please follow the exact directory structure and include both English/Chinese problem statements.

## Contributing

1. Create a new folder under `problems/{difficulty}/`.
2. Populate `Contestant Data` and `Rating Data` exactly as shown above.
3. Ensure all text files are well-formatted and the license is included.
4. Open a pull request with a brief description of the problem.

For any questions, feel free to open an issue.
