# ntroduction

In a previous article, we set up **pre-commit** to enforce code quality and formatting standards in our GitLab pipeline. However, ensuring code cleanliness is only half the battle. What’s the point of having pretty code if it doesn’t work? We also need to verify our code functions as expected. This is where [**pytest**](https://github.com/pytest-dev/pytest), a popular testing framework for Python, comes in.

In this article, we’ll configure and integrate pytest into our existing GitLab codebase and pipeline. We’ll create a new stage in our `**.gitlab-ci.yml**` file to run our tests, ensuring our code is both clean and functional.

# Configuring the environment

Before we use pytest, we must ensure it’s installed in our environment. I’ve been using [**uv**](https://docs.astral.sh/uv/) to manage project dependencies lately, so we can install pytest and specify that it’s a development dependency by executing `uv add --dev pytest` in our shell. If you don’t have uv, install pytest using pip by executing `pip install pytest` in your shell.

## Why pytest?

Now that pytest is installed let’s talk about why we used it in the first place. One of the key benefits of pytest is its support for [fixtures](https://docs.pytest.org/en/6.2.x/fixture.html), which allows us to efficiently set up and teardown resources needed for our tests. Fixtures are setup functions that provide a fixed baseline so that tests execute reliably and consistently. We can define fixtures that run before and after each test, or even before and after a group of tests. Fixtures are handy when we need to setup and teardown complex resources, such as database connections or file systems, required for our tests to run.

With pytest, we can define fixtures using the `@pytest.fixture` decorator, which allows us to specify the scope of the fixture (e.g., function, class, module, etc.) and the code that should be executed to setup and teardown the fixture. The `[@pytest.mark.parametrize](https://docs.pytest.org/en/6.2.x/reference.html?highlight=parametrize#pytest-mark-parametrize)` decorator allows you to run the same test function with different input parameters. This decorator is practical when testing a function with multiple inputs or scenarios.

By leveraging pytest’s fixture functionality, we can write more tests that are easier to maintain and extend.

## Folder layout

By default, pytest looks in your current directory and subdirectories for test files and runs any tests it finds. I prefer to keep a folder named “tests” at the root level of the repository to keep these together in a convenient location for all contributors to find.

```sh
.
├── README.md  
├── our_project  
│   ├── __init__.py  
│   └── classification_metrics.py  
├── pyproject.toml  
├── tests  
│   ├── __init__.py  
│   ├── conftest.py  
│   └── test_classification_metrics.py  
└── uv.lock
```

To tell this story, we will create a fictional project that computes two binary classification metrics using **pandas**. We will test that the metric calculations are correct with pytest. We will first add pandas as a project dependency by running `uv add pandas`. This will install it and ensure we can write our classification functions to test against.

## The functions to test against

Functions to compute accuracy and precision scores are easy to recreate and use as examples. We’ll add a file named `**classification_metrics.py**` in the project folder, and add two functions:

```Python 
import pandas as pd  

def accuracy_score(y_true: pd.Series, y_pred: pd.Series) -> float:  
    """Calculate the accuracy of a binary classification model.
    Parameters  
    ----------  
    y_true : pd.Series  
        The true labels.
    y_pred : pd.Series
        The predicted labels.
    Returns  
    -------  
    float  
        The accuracy of the model.
    """
    return (y_true == y_pred).mean()
def precision_score(y_true: pd.Series, y_pred: pd.Series) -> float:
    """Calculate the precision of a binary classification model.  
    Parameters  
    ----------  
    y_true : pd.Series  
        The true labels.  
    y_pred : pd.Series  
        The predicted labels.
    Returns
    -------  
    float  
        The precision of the model.
    """
    tp = ((y_true == 1) & (y_pred == 1)).sum()
    fp = ((y_true == 0) & (y_pred == 1)).sum()
    return tp / (tp + fp)
```

> The docstrings are a great plus for future users as well to understand how these functions can be used. Examples are helpful as well.

These will be easy enough to use with both accepting the same arguments. This is a great use case for a pytest fixture to share a dataset for testing and a pytest marker to parametrize our test to do both tests in one test function.

## Creating a test fixture

In this project, we will likely want a dataframe readily available to test against. This is a great way to ensure code is being tested and evaluated with the same dataset. This fixture will be defined in a file called `**conftest.py**`, located inside our **tests** folder:

```Python 
import pandas as pd
import pytest  
  
  
@pytest.fixture(scope="session")  
def predictions() -> pd.DataFrame:  
    d = {  
         "id": range(1, 13),  
         "actual": [1, 0, 0, 1, 1, 0, 0, 0, 1, 1, 1, 1],  
         "prediction": [1, 0, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1],  
    }  
    yield pd.DataFrame(d)
```

This allows us to pass `predictions` as an argument to any testing function to reference this dataframe. Let’s put together our test function now.

## Creating the test function

We can test against our accuracy and precision metric functions with a single test function.

I want to ensure our code works against scikit-learn’s metrics with the help of parametrize decorators. We will add that as an additional development dependency by running `uv add --dev scikit-learn`.

We can create a new file named `**test_classification_metrics.py**` inside our **tests** folder:

```Python
import pytest  
import sklearn.metrics  
  
import our_project.classification_metrics  
  
  
@pytest.mark.parametrize(    "metric_name",  
    [  
        pytest.param("accuracy_score", id="accuracy_score"),  
        pytest.param("precision_score", id="precision_score"),  
    ],)  
def test_classification_metrics(predictions, metric_name):  
    our_project_func = getattr(our_project.metrics, metric_name)  
    sklearn_func = getattr(sklearn.metrics, metric_name)  
    result = our_project_func(df["actual"], df["prediction"])  
    expected = sklearn_func(df["actual"], df["prediction"])  
    assert result == pytest.approx(expected, abs=1e-4)
```

We can kick off our tests by running `pytest` in our shell. We should see two green dots indicating the tests have passed and your functions are working as intended. This is a convenient way to test both our `accuracy_score` and `precision_score` functions against scikit-learn’s equivalent functions.

# Integrating this to run automatically with GitLab pipelines

This is excellent progress, but we’re not out of the woods yet. We can’t always count on individual contributors to test code locally before submitting a merge request, and even if we could, it’s nice as a maintainer to ensure any merge requests pass tests during review. We explored automating pre-commit with our `**.gitlab-ci.yml**` file. We’ll add a test stage to this file now to run pytest on any commits on the main branch or when any merge request is opened in addition to pre-commit so that the complete file looks like this:

```Yaml
variables:  
  UV_VERSION: 0.5  
  PYTHON_VERSION: 3.12  
  BASE_LAYER: bookworm-slim  
  UV_CACHE_DIR: .uv-cache  
  UV_SYSTEM_PYTHON: 1  
stages:  
  - build  
  - test  
pre-commit:  
  stage: build  
  image: python:3.11  
  script:  
    - pip install pre-commit  
    - pre-commit run --all-files  
  rules:  
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"  
    - if: $CI_COMMIT_BRANCH == "main"  
pytest:  
  stage: test  
  image: ghcr.io/astral-sh/uv:$UV_VERSION-python$PYTHON_VERSION-$BASE_LAYER  
  cache:  
    - key:  
        files:  
          - uv.lock  
      paths:  
        - $UV_CACHE_DIR  
  script:  
    - uv sync --all-extras  
    - uv run pytest  
  rules:  
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"  
    - if: $CI_COMMIT_BRANCH == "main"
```

> We’re using uv to run pytest and sync our environment dependencies.

GitLab will run our tests if the build stage (pre-commit) succeeds.

Successful merge pipeline with three stages, including a compliance stage.


