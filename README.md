


### Part I
Given a set of strings (genomes) 𝑆 = 𝑆1… 𝑆𝑛, a string 𝑝 (pattern) and an integer 𝑘. We say that 𝑆𝑖,1≤ 𝑖≤ 𝑛, is a k-instance of 𝑝 if there exists a string 𝑌 such that 𝑌 is a substring of 𝑆𝑖 and 𝑌 can be obtained from 𝑝 by inserting up to 𝑘 characters to 𝑝.
In part I we designed and implemented an algorithm that does the following: Given a set of COG-spelled genomes 𝑆, where each genome is segmented into segments such that each segment could contain one or more operons. Also given parameters 𝑞, ℓ,𝑘 and an “unknown” COG 𝑋: find all strings 𝑐 (over the COG alphabet) of length ℓ that have ≥𝑞
k-instances in S , such that COG 𝑋 appears at least once in 𝑐. 
Then, we chose a certain COG with unknown functionallity called COG0673 and using the algorithm's results we have made a Biological assumption on it's functionality.

### Part II
After analyzing the results of our chosen cog (COG0673), we have deduced a certain biological function for that specific cog.
Now, we would like to test out a hypothesis by designing a supervised machine-learning model based on the data that was used in previous parts.
#### Data Preprocessing:
Our raw data is made of 2 tables. First table maps each cog to its class and its role:
 ![image](https://user-images.githubusercontent.com/81320262/152317811-d21c972e-1ce8-49d9-a735-1a163af6c706.png)

The third column represents the class, and the fourth column represents the role.
There are 4 unique classes and 23 different roles. We have approximately 5500 cogs where each has its own combination of class and role.
Our second table is a database of COG-spelled genomes, where each row contains a list of COGs in the order they appear in the genome:
 ![image](https://user-images.githubusercontent.com/81320262/152317905-ce25cb56-cd41-4c7d-8e7c-9bf3d6395584.png)
In the above example, we can see that COG1776 is adjacent to COG0640. We call them
Tier-1 neighbors. COG003 and COG1393 are Tier-2 neighbors. COGX is unknown.
In our data preprocessing stage, our first step was to iterate over the database and for each COG, count the number of neighbors from every class and role, for each Tier from 1 to 7.
For example, if COG1776 is in class ‘a’, we will add 1 to the value of column ‘class_a’ in the row of COG0640 when iterating over the row specified above.


If a certain cog is unknown, we will add 1 to “role_unknown” and “class_unknown” in its neighbors’ rows:
![image](https://user-images.githubusercontent.com/81320262/152317923-697d6c9d-071a-4cdc-8983-449874a2144c.png)
Eventually we had 7 tables, 1 for each Tier we checked. 
#### Target:
Our objective is to find unknown cogs predicted classes and roles by training a supervised model on our known cogs, and eventually use the fitted model to predict the unknown classes and roles.
Model choosing and measurement:
We have decided to measure our model performance using the accuracy metric, which is calculated in the following way: correctpredictions/totalpredictions.
We have tested numerous different machine learning models, including random forest, logistic regression, boosting trees, k-nearest neighbors and more. Each model was optimized using the randomized grid search algorithm, which saves time and computational power. It samples random hyper-parameters from a large sample space and chooses the parameters with the best results.
In addition, we have used the k-fold cross validation to evaluate our model – in this cross-validation process we divide our data to k folds, train our model on k-1 of them and test it on the remaining fold. We repeat this process k times, and eventually average the accuracy results. This process is important because on some train sets our model reached high accuracy, and fell on others, so this process helps choosing between different models. We chose k=5, which means that the model was trained on 80% of the data and tested on the remaining 20%.
Eventually the open-source gradient boosting trees algorithm “catboost” showed the best results for each one of our tiers.



#### Results Discussion:
For all Tiers 1-7, our model predicts the cog’s class with ~70% success rate. If we tried to predict the class using the general full list (meaning looking at all cogs in the same word), results fall off to ~66%.


| Table          |	Tier 1 | Tier 2	| Tier 3 | Tier 4 | Tier 5 | Tier 6 |	Tier 7 | General |
| -------------  | ------- | ------ | ------ | ------ | ------ | ------ | ------ | ------- |
| Success rate   |	69.9%  |	70.5% |	69.9%  |	70%   |	69.9%  |	69.5% |	69.3%  |	65.9%  |

After viewing the results, and since Tier 2 table had the best result, we have trained the final model using Tier 2 data and the following hyper-parameters:
iterations=1000, learning_rate = 0.01,depth=6, l2_leaf_reg = 1. Rest of the parameters were default catboost 
The graph below shows the model’s training progress. On the x axis we can see the iteration number, and on the y axis the error rate.
The dotted line represents our training set, and the bold line represents the validation set.
We can see that with each iteration, the precision of the validation set improves.

![image](https://user-images.githubusercontent.com/81320262/152319161-7fdbd7cf-10bf-43d2-be5b-9962d1dfacaf.png)
After training the model, we have used an external library called “SHAP” which analyzes and tries to explain the output of any machine learning model.
SHAP assigns each feature an importance value for a particular prediction. On the diagram below we can see the results for the SHAP explainer.

![image](https://user-images.githubusercontent.com/81320262/152319229-254e6a64-f023-4b5f-8036-1dd76bb7dad8.png)
Class 0 – METABOLISM, Class 1 – MOBILOME, class 2 - INFORMATION STORAGE AND PROCESSING, class 3 - CELLULAR PROCESSES AND SIGNALING

We can see that the number of neighbors in each class contributes the most to the cog’s class and is also the model’s main consideration. However, if for each COG we choose the row with most neighbors, we only get 66% success, so clearly there are other factors involved.
After deciding our final model, we can try and predict our cog’s class using the trained model.
As you can see, our model predicted COG0673 to belong in class 0 – which is metabolism.

If we compare it to our own prediction from the previous part of the project, we find it’s very acceptable – we thought it might have something to do with either the sugar transport system or to different permeases in the cell. Out of the four classes it seems most acceptable that it will indeed belong to the metabolism class.

#### Suggestions for future research:
In the same way we ran the model trying to predict the class of each cog, it can be applied to the role of each cog. A model which predicts roles can be trained, and could even take the output of the previous model (the one we showed in this report) as a parameter, meaning we can try and predict the role using the predicted class. 
It is also possible to try and gather some more data – whether more observations and more cog-spelled genomes, which can help improve accuracy without the necessity to change models, or different type of data like the genetic material sequence itself which can be processed and used as input to the model.
