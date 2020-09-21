# AV-Cross-Sell

# What Worked?
* Hyperparameter tuning
* Feature Engineering such as concatenation,aggregation, binning
* Data cleaning steps and most pertinent of them were removing duplicates[more reliable CV] and confusing rows
* Xgboost and CatBoost
* Treating Annual Premium and Vintage as categorical features in catboost
* Blending CatBoost and XgBoost [Blended probablities from 125 CB Models and 125 XgB models]

# What didn't Work?
* NN and LGBM
* Class balancing via oversampler,undersampler,TomkeLinks etc and via hardcoding class weights parameter in individual algorithm

# Algorithms Used

* **LightGBM**

* **XgBoost**---> used for ensemling

* **CatBoost**---> used for ensemling

* **NN via FastAi**---> Poor CV

# Differentiation based on how each algorithm Splits feature

**Catboost** 
* Uses oblivious decision trees
* Before learning, the possible values of each feature are divided into buckets delimited by threshold values, creating feature-split pairs
* Example for such pairs are: (age, <5), (age, 5-10), (age, >10) and so on. In each level of an oblivious tree, the feature-split pair that brings to the lowest loss (according to a penalty function) is selected and is used for all the level’s nodes.

**LightGBM** 
* Uses gradient-based one-side sampling (GOSS) that selects the split using all the instances with large gradients (i.e., large error) and a random sample of instances with small gradients
* In order to keep the same data distribution when computing the information gain, GOSS introduces a constant multiplier for the data instances with small gradients
* GOSS,thus achieves a good balance between increasing speed by reducing the number of data instances and keeping the accuracy for learned decision trees.

**XGboost** 
* Several methods for selecting the best split
* For example, a histogram-based algorithm that buckets continuous features into discrete bins and uses these bins to find the split value in each node
* This method is faster than the exact greedy algorithm, which linearly enumerates all the possible splits for continuous features, but it is slower compared to GOSS that is used by LightGBM



# Differentiation based on how each algorithm grows leaves



![](https://www.riskified.com/wp-content/uploads/2019/11/inner-image-trees-1-1024x386.png)

* **Catboost** grows a balanced tree.


* **LightGBM** uses leaf-wise (best-first) tree growth
    * It chooses to grow the leaf that minimizes the loss, allowing a growth of an imbalanced tree
    * Because it doesn’t grow level-wise, but leaf-wise, overfitting can  happen when data is small
    * It is therefore important to control the tree depth


* **XGboost** splits up to the specified max_depth hyperparameter
    * It then starts pruning the tree backwards and removes splits beyond which there is no positive gain
    * It uses this approach since sometimes a split of no loss reduction may be followed by a split with loss reduction
    
    
 # Differentiation based on how each algorithm handles Missing values

* **Catboost** has two modes for processing missing values, “Min” and “Max”.
    * In “Min”, missing values are processed as the minimum value for a feature (they are given a value that is less than all existing values)
    * This way, it is guaranteed that a split that separates missing values from all other values is considered when selecting splits
    * “Max” works exactly the same as “Min”, only with maximum values



* In **LightGBM** and **XGBoost** missing values will be allocated to the side that reduces the loss in each split


# Differentiation based on how each algorithm calculates feature importance

* **Catboost** has two methods
    * The first is “PredictionValuesChange”. For each feature, PredictionValuesChange shows how much, on average, the prediction changes if the feature value changes. A feature would have a greater importance when a change in the feature value causes a big change in the predicted value
    * The second method is “LossFunctionChange”. This type of feature importance can be used for any model, but is particularly useful for ranking models. For each feature the value represents the difference between the loss value of the model with this feature and without it. 



* **LightGBM** and **XGBoost** have two similar methods
    * The first is “Gain” which is the improvement in accuracy (or total gain) brought by a feature to the branches it is on.
    * The second method has a different name in each package: “split” (LightGBM) and “Frequency”/”Weight” (XGBoost). This method calculates the relative number of times a particular feature occurs in all splits of the model’s trees. This method can be biased by categorical features with a large number of categories. 
    
 
 # Sampling Strategy : Stratified Shuffle Split
![](https://i.stack.imgur.com/AGv9B.png)
