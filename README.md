# NYT-bestsellers-classification-project

## Intro

This classification project aims to predict if a book will appear on any of the New York Times' bestseller lists.
- [Tech Stack](#tech-stack)

- [Process](#process)

- [Data and EDA](#data-and-eda)

    - [Cleaning](#cleaning)

    - [Observations](#observations)

    - [Features](#features)
    
    - [Visuals](#visuals)
    
- [Modeling](#modeling)

    - [Baseline Model](#baseline-dummy-classifier)

    - [Final Model](#final-model-logistic-regression)

- [Future Improvements](#future-improvements)

## Tech Stack

- Python libraries:
    - Requests
    - Selenium
    - Beautiful Soup
    - Pandas
    - Matplotlib
    - Seaborn
    - Scikit-learn

## Process

I gathered data through the New York Times' Book API as well as by scraping GoodReads.com. Bestsellers from 2017 to the present were sourced from the NYT API, while non-bestsellers from 2017-2018 were sourced from GoodReads. Ultimately, features for each book (both bestsellers and non-bestsellers) were also scraped from GoodReads.

## Data and EDA

In order to both prepare and understand the data prior to running models, I completed a number of preprocessing/cleaning steps followed by  exploritory data analysis. 

### Cleaning 

- Remove duplicate books returned by the NYT API 
- Add a column of 0's and 1's for the target variable where 1's reflect bestselling books
- Remove duplicate books from the combined dataframe (books scraped from Goodreads might have been NYT bestsellers which I can determine by identifying books that were returned from both the NYT API and Goodreads)
- Convert non-categorical data types from strings to numbers
- Remove and/or fill in rows with null values
- Group imprints and subsidiaries of the top 5 publishing companies into single groups

### Observations
- 1646 total observations
    - 551 bestsellers
    - 1095 non-bestsellers
    
### Features
- Original dataframe from web scraping:
    - Title
    - Author
    - Goodreads rating (based on user input)
    - Goodreads genre (based on most user tags)
    - Publishing company
    - Publish date
    - Number of pages
    - Format (hardcover, e-book, etc.) 
    
- **Final feature list**:
    - Categorical (dropped columns with < 15 observations):
        - Part of a series
        - Top author (list of Forbes' top earning authors 2017 & 2018)
        - Genre 
        - Publishing company
        - Month of publishing
    - Continuous:
        - Rating

### Visuals

To better understand the distribution of the data, I created a number of visuals using Python's Matplotlib and Seaborn libraries. 

![](/Plots/Top_authors.png)

Of the 1646 total books in the data set, only 50 were written by top authors. However, as indicated by the chart above, the top authors account for more bestsellers than they do non-bestsellers. Of the 50 books written by top authors, 29 were bestsellers, equivalent to 58%. In comparison, of the 1,596 books in the data set that were not written by top authors, 522 were bestsellers, equivalent to 33%. This category turns out to be one of the strongest predictors of being a bestselling book in the final model.

![](/Plots/Publish_month_rating.png)

This chart highlights the distribution of books across ratings and months during which books were published. It is notable that despite the fact that fewer books were published in November and December when compared to other months, a larger proportion of the books sold in those two months were bestsellers. Additionally, the average rating of books published in these two months is higher than that of books published in other months. These two features (published in Nov. or Dec.) end up being two of the strongest predictors of NYT bestsellers in the final model.

![](/Plots/Ratings.png)

The average Goodreads rating for NYT bestsellers is only slightly higher (~4.03) than the average rating of non-bestsellers (~3.96). However, it is notable that the range of ratings is smaller for bestsellers (3.04-4.61) than the range for non-bestsellers (2.79-5.0). Rating does not ultimately have a large predictive impact in the final model.

![](/Plots/Top_5_publishing_companies.png)

Out of the 551 bestselling books in the data set, 307 were published by the Top 5 publishing companies (Penguin Random House, Harper Collins, Macmillan, Simon & Schuster, Hachette Livre). However, the Top 5 also account for a large proportion of the non-bestselling books. It is evident that the Top 5 have a big market share, but ultimately these companies only published 6% more bestsellers out of all the books they account for when compared to the remaining publishing companies in the data set.

## Modeling

I ran a variety of classification models. Here I have only highlighted the baseline model and the final model. For a look at the full set of models, refer to the "Modeling" notebook in this repo. 

In selecting a "final model," I chose to focus on accuracy score as the best measure of evaluating models. This is due to the fact that I view a false positive and a false negative as having equal importance. 
<br>
    - False positive: model would predict a book to be a bestseller when in fact it is not 
<br>
    - False negative: model would predict that a book would not be a bestseller, when in fact it was

From the perspective of a publishing company, a false positive would become a bad investment (the company would make less than expected on a particular book given the incorrect prediction that it would be a bestseller) and a false negative is a missed opportunity (the company would miss out on the chance to make money on a bestselling book given the model's incorrect prediction).

### Baseline: Dummy Classifier

I used Sklearn's Dummy Classifier (parameter for strategy was "most frequent") to create a baseline model. This model looks at the distibution of the data across classes and predicts each observation as the most frequent class. In this case, a larger proportion of the observations were non-bestsellers, so the model always predicted that a book was not a bestseller. As a result, the model had an **accuracy score of 67-68%**, roughly equal to the distribution of the data by class (66% non-bestsellers) with the difference attributed to the fact that the data was split into training and testing sets, so the exact distribution varied.

![](/Plots/Confusion_Matrix_baseline.png)

The confusion matrix shows that no observations were classified as positives (bestsellers) since those made up the less frequent class. The model correctly classified 220 negatives (true negatives) and incorrectly classified 108 negatives (false positives). The latter are the observations in the test set that were in fact bestsellers.

### Final Model: Logistic Regression

The model which performed the best in terms of accuracy was Logistic Regression with Sklearn's default parameters (i.e. C of 1 and penalty parameter of L2 (Ridge)). 

![](/Plots/Confusion_Matrix_Log.png)

The confusion matrix shows that the model correctly predicted 41 positives and 202 negatives, while incorrectly classifying 20 negatives and 67 positives for a **final accuracy score of ~74%**. Given the class imbalance of the data set, the model performed better when classifying the more frequent class (negatives).

![](/Plots/ROC_AUC.png)

The ROC curve plots the true positive rate vs. the false positive rate at different classification thresholds, i.e. the model's ability to distinguish between classes. This model has an AUC score of .779.

#### Coefficients

Features which most contributed to a higher likelihood of being in the positive class (as indicated by a positive, large coefficient):
- Top author: Y
- Genre: politics
- Publish month: December
- Genre: biography
- Publish month: November
- Publish company: Penguin Random House

Features which most contributed to a lower likelihood of being in the positive class (as indicated by a negative, large coefficient):

- Genre: horror
- Genre: science fiction
- Genre: fantasy
- Genre: short story
- Genre: fiction
- Genre: young adult

## Future Improvements

- More bestsellers: The NYT API returned multiple duplicate values and due to time constraints, I was not able to gather additional bestsellers in time for modeling. As a result, there was class imbalance present in my data set (2/3 of the data were not bestsellers) so the model performed better when classifying negative cases. Alternatively, I could implement SMOTE to create additional observations in the positive class.

- Different NYT API link: The link I used returned over 32,000 results, after which point I selected only the books that were on a list between 2017 and the present. This was computationally inefficient as I had to store the entire list from the API and then iterate over it to filter out books that appeared on a list before 2017. Given more time, I would have liked to have investigated the API further to find a url that was better suited for returning books within a certain date range. 

