# Modeling Water Wells in Tanzania

Flatiron Phase 3 Project, by Kelsey Lane, Andy Schmeck and Ted Brandon

This repository has been created to present a data analytical approach to help an NGO predict failure of water wells in Tanzania.  This document is intended to bridge the gap between technical and non-technical audiences.

## Repository Contents:

Technical Jupyter Notebook (.ipynb) containing all data analysis
Matching pdf of the above notebook
Phase 3 presentation (pdf)
[Phase 3 presentation slides](https://docs.google.com/presentation/d/1bLGyPGV1-n9dBV0NYhM2iqxitX8GHsHzPss1DoGjwUw/edit?usp=sharing)

## Overview:
This project uses data about Tanzanian wells available at [DrivenData](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/). Our model seeks to classify wells as functional or not, in order to help a non-government organization (NGO) determine which wells they need to repair. After the data were cleaned, various iterative classification models were run in order to identify a model that has the best mix of a high precision and low complexity. We found that the amount of total static head (tsh) of the well, alongside its installer, extraction type, and if it had a government permit, were all significant predictors in classifying the functionality of a well, with 75% precision.

## Business Problem
An NGO seeks to help Tanzania accomplish its Millenium Development Goal 7C: "halving the proportion of the population without sustainable access to safe drinking water," ([MDG report](https://commonwealthfoundation.com/wp-content/uploads/2013/10/MDG%20Reports%20Tanzania_FINAL_2.pdf)). Tanzania's situation is dire ([World Bank](https://reliefweb.int/sites/reliefweb.int/files/resources/120166.pdf![image.png](attachment:image.png))): 

- Only 60\% of Tanzanians get their drinking water from an improved source.
- Of the 83,000 rural water points recorded in the national water point census, as of 2014, 40\% were found to be non-functional, with the likelihood of failure highest at 20\%, their first year of operation.

Improving water supply, sanitation, and hygeine conditions have been linked with improved human development, reduced poverty, and reduced stunting in early childhood ([World Bank](https://reliefweb.int/sites/reliefweb.int/files/resources/120166.pdf![image.png](attachment:image.png))). 

46% of the wells in Tanzania are in need of repair or nonfunctioning. Instead of building new wells, an NGO can drastically increase clean water supply by simply fixing wells. The urgency of the situation, and an NGO's financial constraints, increase the need for precision in our model. Sending an NGO out to a fully-functional well would be expensive and cost the opportunity of fixing an actual nonfunctioning well.  This problem may be exacerbated since some of these wells are very remote and in mountainous regions. Thus, we will be using the precision metric for evaluating our model. By using our precise model, the NGO can preemptively address probable pump failures, increasing the sustainable, improved well capacity of Tanzania.

## Data Understanding and Preparation
The data used in this analysis is provided by [Tanzania's Ministry of Water](https://www.maji.go.tz/) and compiled by [Taarifa](https://taarifa.org/) available at [DrivenData](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/). The dataset includes information of 59,400 wells, each of which has 41 different features. The target column, `status_group`, indicates if the well is *functional*, *in need of repair*, or *non-functional*. We are able to use the features provided about each well to build a classification model to predict the status of the well, and thus help the NGO determine which wells of unknown status may or may not need repairs.

### Column Inclusion
The dataset has many columns that represent the same information at different levels of specificity. To help illustrate this, the columns in the original dataset are listed out below.

As you can see, there is a lot of overlap, such as `waterpoint_type` and `waterpoint_type_group`, or `extraction_type`, `extraction_type_group`, and `extraction_type_class` (an explanation of all features can be found [here](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/page/25/)). These would pose multicollinearity issues, as, it's the same information represented at different levels, so for all overlaping columns, the most general ones were kept in order to reduce dimensionality in the data. For insurance, each column was analyzed with a $\chi^2$ test to ensure true relation, with an example shown below:

Furthremore, there are other columns which have unclear or ambiguous content. For example, `num_private` is a column with no description given by the DrivenData site, and it is not obvious what this feature indicates. A similar idea holds for `public_meeting`. As a result, these features were dropped. Other unnecessary columns that were dropped include `id`, as it is just the unique identifier for each row, `recorded_by`, as it is the same entry for every row, and `funder`, as it does not seem likely that the funder of the well had any impact on its future functionality. This left us with the following dataset.

As it is unclear from the remaining columns which features may or may not be significant in predicting well failure, the rest of the columns are left in for the time being. These will be paired down later, using a decision tree to find the most significant indicators. The columns remaining, though, all seem different from each other and good potential predictors. The continuous variables were checked against each other using a heatmap to see if there was any multicolinearity, of which there was none. Two important notes: 1) despite the similar column names, `source` and `source_class`, as well as `management` and `management_group` appear to represent different pieces of information, and thus 2) are retained depsite the similar column names.

### Data Cleaning 

One issue in the remaining data has to do with `longitude` and `latitude`. Tanzania does not include null island (0$^{\circ}$ 0$^{\circ}$), but there are occurances of these coordinates in the data. As a result, the rows containing these points were dropped, as they represent only 3\% of the data and the model is only predicting well functionality inside of Tanzania.

The `permit` column indicates if the well is or is not government approved. We elected to fill the missing values in this column with *False* rather than the mode value of *True*, because we did not want to assume government approval from data that are from the government. As a result, these are filled ahead of time before the train-test split.

There are 18,897 zero values in `construction_year`, which represent null values. As this is a high proportion of the column and we didn't want to simply impute the value with another, we instead opted to bin the column into decades and replace the zeroes with *Unknown*.

Finally, `scheme_management` and `installer` both have null values remaining. As they also have a large number of unique entries, these columns were binned into the top five most frequent values and *other*. The null values were filled with *other* as well. Since we did not want to risk data leakage, this process was performed after the trian-test split in order to obtain frequencies based only on the training data.

### Feature Creation

The season in which the well data was recorded may affect the funcitonality of the well, as, intuitively, wells recorded during a rainy season may have more water depending on their source. We elected to add a `season` column based on `date_recorded` and then dropped `date_recorded`.

### Fixing up the Target

In the original dataset, `status_group` is a column containing *functional*, *non functional*, and *functional needs repair*. We are engineering this problem to be binary, so *non functional* and *functional needs repair* were collapsed into one column, as, these are the wells the NGO would want to identify as in need of attention/maintenance. The binary classification eliminates the issues with class imbalance, present in the original distinction, leaving the classes now relatively even with 54% of the wells being functional and 46% needing some attention.

### Limitations

Both `amount_tsh` and `population` have high zero-counts. While zero is a valid potential entry for these columns, it has also been used to represent null values throughout the dataset. Therefore, it is not clear how many of these zeroes represent actual zeroes and how many represent nulls. These columns are also the only ones with outliers that are unaccounted for. However, due to the heavy skew that the zeroes add to the data, it isn't clear where to draw the cutoff for outliers. Therefore, this gums up the data and makes the results of using these columns unclear. 

We were also limited in our ability to utilize two specific columns, as, we did not know what kind of data was recorded:  `num_private` and `public_meeting`. 

Lastly, we were limited to the timeframe the data were collected, between 2003-2013, which could impact our predictive abilities.

## Modeling

Because there is minimal class imbalance present in the data, we opted to use a DummyClassifier to create a baseline model based on the most frequent label.

As expected, the accuracy of the model is 54% (the percentage of zero-labels) and the precision is zero, as the model always predicts zero and never one (hey, no false positives!). The next model we set up was a simple decision tree that was fed all the columns in order to determine which features are most significant in predicting well functionality. We paired down the complexity by selecting the most important features, based on this model. As a result, `max_depth` was only set to five to reduce complexity of the model. The same transformations were performed on the data, despite continuous variables not needing to be scaled, as, this reduced the amount of code needed and had no effect on the function of the decision tree. 

#### All Columns Decision Tree

This first classification model performs well, with training and validation scores of .87, indicating there does not seem to be any overfitting. However, as this model uses all the features, it is more computationally complex than a simpler model that would be easier for an NGO to implement. As a result, we retained the top four, most important features, to build a logistic regression:  `amount_tsh`, `installer`, `extraction_type_class`, and `permit`.

#### Simplified Logistic Regression

#### Logistic Regression Assumption Test

While the precision for the logistic regression model is worse, coming in at 0.75 for both the training and validation sets, this model is less complex and easier to implement than the decision tree. It does not violate the assumptions of logistic regression, nor does it seem to overfit. Therefore, it is a more viable and useable model for the NGO to potentially implement. 

Finally, while a Random Forest Classifier model would be more complex, we were interested in determining if we could get a sizeable increase in precision, based on the four aforementioned most important features, to balance the first model and the second model.

#### Random Forest Classifier
This is only a slight increase in precision for the training score, compared to the first model, but there is also a decrease in the validation score, indicating potential overfitting. In an effort to potentially increase the performance of this model, we tuned its parameters using a gridsearch.

Despite tuning the model, the performance is still only marginally better than the logistic regression model, as well as being more computationally stressful and performing worse on the validation scores. As a result, the final model we decided to go with was the logistic regression model with four features.

## Evaluation
To reiterate from earlier, we have been using precision as our metric; we want to avoid false positives. In this case, false positives are wells that we identify as needing attention when they are actually fully functional, as this would waste resources for the NGO. As a result, the 75% precise logistic regression was our final model. While this is less than the precision of the original decision tree, this model is less computationally strenuous and therefore comparatively easier for the NGO to implement. Similarly, the model does not seem to be overfitting as the scores for both precision and accuracy of the model are close for both the training and validation sets. The assumptions for the logistic regression model are also met, meaning it is a workable model.

While the logistic regression model is worse than the first decision tree model, it outperforms the baseline model which had an accuracy of 54% and a precision of 0, indicating it is still worthwhile to use. Furthermore, it still performs well on the test set, with a precision of 0.73 and accuracy of 0.64, both similar to the training set.

Therefore, the logistic regression model is the one we would put forward as out official model for the NGO. While the precision could be improved, making it a more reliable model, it still operates better than the baseline and meets assumptions, while not seeming to overfit, thereby making it applicable to actual data the NGO would be interested in. Through using this model, the NGO could help identify wells in need of repair or replacement and better allocate their resources without wasting time checking up on functional wells.

## Conclusions

Overall, we would recommend the NGO consider using this model to help them predict which wells to focus their resources on. We would also recommend that if they have any interest in lobbying for better well construction in the future, they focus in on the total static head of the well, the installer, extraction type, and if the well has a government permit, as these were the most important features in predicting well failure. 

One drawback of this model is that it's built using amount_tsh, which is one of the columns with unclear zeroes representing possible nulls. Therefore to improve the model, we could get more accurate data for this column to see how it impacts the model.

Furthermore these data are outdated and contains records that date all the way back to 2003. Between then and now, there could have been changes in the way wells are constructed or maintained that would impact our predictions, but we lack the data needed to see this, so that would be another future improvement we could work on.

Knowing that NGOs often work under tight budget constraints, we would also love to model cost (well parts, replacement, transportation, etc) to help reduce risk of the organization. 

Finally, we would also like to partner with the NGO in disseminating educational materials and instruction on well maintenance, to impower local communities to increase the sustainability of these improved water sources. 

Project Structure
```bash
├──.ipynb_checkpoints
├──Phase 3 presentation
├──data
├──Images
├──Individual_Jupyters
    ├──.ipynb_checkpoints
    ├──Archived
```
