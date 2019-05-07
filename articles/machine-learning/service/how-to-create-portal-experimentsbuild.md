# Create and explore automated machine learning experiments in the Azure portal (Preview)

In this article, you learn how to create, run, and explore automated machine learning experiments in the Azure portal without a single line of code. Automated machine learning automates the process of selecting the best algorithm to use for your specific data, so you can generate a machine learning model quickly.

If you prefer a more code-based experience, you can also configure your automated machine learning experiments in Python with the Azure Machine Learning SDK.

## Use an existing Machine Learning Service Workspace Resource in Azure
In this quickstart, use a Machine Learning Service Workspace resource created for you, found in the [Azure Portal](https://ms.portal.azure.com/). 

## Get started

Navigate to the left pane of your workspace. Select **Automated Machine Learning** under the **Authoring (Preview)** section.

![Azure portal navigation pane](media/how-to-create-portal-experiments/nav-pane.png)

 If this is your first time doing any experiments with Automated Machine Learning, you'll see the following:

![Azure portal experiment landing page](media/how-to-create-portal-experiments/landing-page.png)

Otherwise, you will see your Automated machine learning dashboard with an overview of all of your automated machine learning experiments, including those run with the SDK. Here you can filter and explore your runs by date, experiment name, and run status.

![Azure portal experiment dashboard](media/how-to-create-portal-experiments/dashboard.png)

## Create an experiment

Select the Create Experiment button to populate the following form.

![Create experiment form](media/how-to-create-portal-experiments/create-exp-name-compute.png)

1. Enter your experiment name, such as `Iris Experiment`.

1. Select the compute resource named `aml-compute`, for the data profiling and training job. A list of your existing computes is available in the dropdown. To create a new compute, follow the instructions in step 3.

1. Select a data file, `irisdata.csv`, upload the file from your local computer's desktop to the container.

    ![Select data file for experiment](media/how-to-create-portal-experiments/select-file.png)

1. Use the preview and profile tabs to further configure your data for this experiment.

    1. On the Preview tab, indicate if your data includes headers, and select the features (columns) for training using the **Included** switch buttons in each feature column.

        ![Data preview](media/how-to-create-portal-experiments/data-preview.png)

    1. On the Profile tab, you can view the [data profile](#profile) by feature, as well as the distribution, type, and summary statistics (mean, median, max/min, and so on) of each.

        ![Data profile tab](media/how-to-create-portal-experiments/data-profile.png)

        >[!NOTE]
        > The following error message will appear if your compute context is **not** profiling enabled: *Data profiling is only available for compute targets that are already running*.

1. Select the training job type of classification.

1. Select target column of `class`, which is the column which you would like to do the predictions on.



### (Optional) Data profiling

You can get a vast variety of summary statistics across your data set to verify whether your data set is ML-ready. For non-numeric columns, they include only basic statistics like min, max, and error count. For numeric columns, you can also review their statistical moments and estimated quantiles. Specifically, our data profile includes:

* **Feature**: name of the column which is being summarized.

* **Profile**: an in-line visualization based on the type inferred. For example, strings, booleans, and dates will have value counts, while decimals (numerics) have approximated histograms. This allows you to gain a quick understanding of the distribution of the data.

* **Type distribution**: an in-line value count of types within a column. Nulls are their own type, so this visualization is useful for detecting odd or missing values.

* **Type**: the inferred type of the column. Possible values include: strings, booleans, dates, and decimals.

* **Min**: the minimum value of the column. Blank entries appear for features whose type does not have an inherent ordering (e.g. booleans).

* **Max**: the maximum value of the column. Like "min," blank entries appear for features with irrelevant types.

* **Count**: the total number of missing and non-missing entries in the column.

* **Not missing count**: the number of entries in the column which are not missing. Note that empty strings and errors are treated as values, so they will not contribute to the "not missing count."

* **Quantiles** (at 0.1, 1, 5, 25, 50, 75, 95, 99, and 99.9% intervals): the approximated values at each quantile to provide a sense of the distribution of the data. Blank entries appear for features with irrelevant types.

* **Mean**: the arithmetic mean of the column. Blank entries appear for features with irrelevant types.

* **Standard deviation**: the standard deviation of the column. Blank entries appear for features with irrelevant types.

* **Variance**: the variance of the column. Blank entries appear for features with irrelevant types.

* **Skewness**: the skewness of the column. Blank entries appear for features with irrelevant types.

* **Kurtosis**: the kurtosis of the column. Blank entries appear for features with irrelevant types.

<a name="preprocess"></a>

### (Optional) Advanced preprocessing

When configuring your experiments, you can enable the advanced setting `Preprocess`. Doing so means that the following data preprocessing and featurization steps are performed automatically.

|Preprocessing&nbsp;steps| Description |
| ------------- | ------------- |
|Drop high cardinality or no variance features|Drop these from training and validation sets, including features with all values missing, same value across all rows or with extremely high cardinality (for example, hashes, IDs, or GUIDs).|
|Impute missing values|For numerical features, impute with average of values in the column.<br/><br/>For categorical features, impute with most frequent value.|
|Generate additional features|For DateTime features: Year, Month, Day, Day of week, Day of year, Quarter, Week of the year, Hour, Minute, Second.<br/><br/>For Text features: Term frequency based on unigrams, bi-grams, and tri-character-grams.|
|Transform and encode |Numeric features with few unique values are transformed into categorical features.<br/><br/>One-hot encoding is performed for low cardinality categorical; for high cardinality, one-hot-hash encoding.|
|Word embeddings|Text featurizer that converts vectors of text tokens into sentence vectors using a pre-trained model. Each word’s embedding vector in a document is aggregated together to produce a document feature vector.|
|Target encodings|For categorical features, maps each category with averaged target value for regression problems, and to the class probability for each class for classification problems. Frequency-based weighting and k-fold cross validation is applied to reduce over fitting of the mapping and noise caused by sparse data categories.|
|Text target encoding|For text input, a stacked linear model with bag-of-words is used to generate the probability of each class.|
|Weight of Evidence (WoE)|Calculates WoE as a measure of correlation of categorical columns to the target column. It is calculated as the log of the ratio of in-class vs out-of-class probabilities. This step outputs one numerical feature column per class and removes the need to explicitly impute missing values and outlier treatment.|
|Cluster Distance|Trains a k-means clustering model on all numerical columns.  Outputs k new features, one new numerical feature per cluster, containing the distance of each sample to the centroid of each cluster.|

## Run experiment and view results

To run the experiment, click Start. The experiment preparing process takes a couple of minutes.

### View experiment details

Once the experiment preparation phase is done, you'll see the Run Detail screen. This gives you a full list of the models created. By default, the model that scores the highest based on your parameters is at the top of the list. As the training job tries out more models, they are added to the iteration list and chart. Use the iteration chart to get a quick comparison of the metrics for the models produced so far.

Training jobs can take a while for each pipeline to finish running.

![Run details dashboard](media/how-to-create-portal-experiments/run-details.png)

### View training run details

Drill down on any of the output models to see training run details, like performance metrics and distribution charts. [Learn more about charts](https://docs.microsoft.com/azure/machine-learning/service/how-to-track-experiments#understanding-automated-ml-charts).

![Iteration details](media/how-to-create-portal-experiments/iteration-details.png)
