# 3778's Machine Learning Challenge

## Dependencies
- Python 3.6 or superior
- Modules listed in `requirements.txt`

## Recommended environment set-up
- Move to the folder you wish to copy the repository
- Run `git clone URL`, `URL` is the url of the current repository (this page's url)
- Create a python environment with the command `virtualenv --python=PYTHON_EXE PROJECT_FOLDER`. `PYTHON_EXE` must be the path to a Python 3.6+ interpreter and `PROJECT_FOLDER`, the folder of your project
- Activate the virtual environment with the command `source bin/activate` in the project folder
- Install dependencies by running `pip install -r PATH_TO_REPOSITORY/requirements.txt`. `PATH_TO_REPOSITORY` is the path to the repository's folder cloned from GitHub
- Install a new kernel for the jupyter notebook, pointing to the current virtual environment by running `python -m ipykernel install --user --name KERNEL_NAME` while the virtual environment is activated. It's recommended to define `KERNEL_NAME` as equal to the name of the virtual environment
- Go to the repository folder and run `jupyter notebook`, open the `Challenge.ipynb` and set the notebook's kernel as equal to the one created for the virtual environment.
- Run the notebook

## Current chosen model

The current model proposal is composed of a map, using a duple of keys (country, indicator), where it's located a dictionary containing a trained Exponential Smoothing model, together with other information necessary to do some operations.\
The model has a correction functionality based on values that appear after the ones that are going to be predicted. To achieve that, it's used an inverse distance weight average together with the difference between the terms predicted by the Exponential Smoothing model and future values.\
The idea was tested using a graph of a particular time series present in the dataset, and the results obtained were better than the one using the Exponential Smoothing model alone:

<div align="center">

 <img src="images/no_correction.jpg?raw=true" width="500"></br>
 Fig 1. Graph containing the predictions without correction.

 <img src="images/with_correction.jpg?raw=true" width="500"></br>
 Fig 2. Graph containing the predictions with correction

</div>

The blue line in the graph represents the past values of the series, used to train the Exponential Smoothing model, the black dots are the true values, the green dashed line is the predictions and the red one is the future values.\
Since not all series have _future_values_, the correction method is not used for those cases.
Because the huge amount of different time series and data in general, a framework for parallel computation and larger-than-memory data handling called __Dask__ was used.\
The framework has a data structure that tries to mimic the functionalities of __Pandas__ called __Dask Dataframe__ that is composed by chunks (or partitions) that maps to original Pandas Dataframes.\
The framework's operations are lazily evaluated and generates a graph trying to optimize the steps to the output. An example of graph, of the training operation performed in this challenge, is shown below.


<div align="center">

 <img src="images/mydask.png?raw=true" width="800" height="220"></br>
 Fig 3. Graph of the training operations.

</div>

The training was performed by partitioning the data into 10 MB chunks containing pandas dataframes, after that the data was grouped by the country and indicator keywords, respectively, and the apply method was called with the train function as argument.\
The __Dask distributed__ module was used to assign jobs (futures) to workers, by creating a local cluster, in order to parallelize the workload and monitor its progression using the __Dask Dashboard__.
It's worth mentioning that, there were some time series containing missing entries, those values were dealt with by performing a simple linear interpolation just before the data was fed to the model.

## Results

After training the models and applying each of them to its respective series, the predictions were evaluated using the official metric of the challenge, resulting in the following:

```
{'explained_variance_score': 0.999911483258075,
 'mean_absolute_error': 72395.64262261496,
 'mean_squared_error': 890276847158.1981,
 'median_absolute_error': 2.1743972778319858,
 'r2_score': 0.9999110712734727}
 ```

 By looking at the _Explained Variance Score_ and the _R2 Score_, they are fundamentally equal to each other, meaning that the model is unbiased (mean error close to 0). Also, they are close to 1.0, which is its maximum, meaning that the variance of the data is almost all explained by the model.\
 Now, when it comes to the _Mean Squared Error_ and the _Mean Absolute Error_, it seems bad at first, but if a closer look to the scale of the data is taken. It's noticed that it varies immensely, so it dominates those two metrics. Also, if the percentual error is checked for those series with larger scales, the majority are quite low.
 In conclusion, the actual proposed model seems to be robust and captures the main characteristics of the series quite well.
