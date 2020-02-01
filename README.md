# **TMALL Repeat Buyers Prediction**

### **Done by Xinze Tang, Yu Han, Shiqi Tao**

# 1 Business understanding
## 1.1 Business background

Merchants sometimes have big promotions (such as discounts or cash coupons) on specific dates (such as &quot;Boxing-day&quot;, &quot;Black Friday&quot; or &quot;Double 11&quot;) to attract a large number of new buyers. Unfortunately, many attractive buyers are one-time transaction hunters, and these promotions may have a small long-term impact on sales. To alleviate this problem, it is important for merchants to identify who can be converted into repeated buyers. By targeting on these potential loyal customers, merchants can significantly reduce the promotional cost and increase return on investment (ROI).

As we all know, in the field of online advertising, customer targeting is very challenging, especially for new buyers. But through Tmall.com&#39;s long-term accumulated user behavior logs, we may be able to solve this problem.

## 1.2 Task

In this project, we use data from merchants and their corresponding new buyers during the &quot;Double 11&quot; day promotion to predict. We need to predict which new buyers for given merchants will become loyal customers in the future. In other words, we need to predict the probability that these new buyers would buy items from the same merchants again within 6 months.

# 2 Data understanding

This data set contains shopping logs of anonymous users in the past 6 months before and after Double Eleven, and tag information indicating whether they are repeat purchasers. Due to privacy issues, the data is sampled at offsets, so the statistical results of this data set have some deviations from the actual data of Tmall.com. But it does not affect the applicability of the solution. Details of the data format can be found in the table below.

## 2.1 User profile

| **Data Fields** | **Definition** | **Example** |
| --- | --- | --- |
| user\_id | User ID | 376517 |
| age\_range | User age range: 1 corresponds to\&lt;18; 2 corresponds to [18,24]; 3 corresponds to [25,29]; 4 corresponds to [30,34]; 5 corresponds to [35,39]; 6 corresponds to [40,49]; 7 and 8 corresponds to \&gt;= 50; 0 and NULL corresponds to unknown. | 6 |
| gender | User gender: 0 corresponds to female; 1 corresponds to female;  2 and NULL corresponds to unknown. | 1 |

The user profile table has 424170 records in all.

## 2.2 User behavior logs

| **Data Fields** | **Definition** | **Example** |
| --- | --- | --- |
| user\_id | User ID | 328862 |
| item\_id | Item ID | 323294 |
| cat\_id | Item category ID | 833 |
| seller\_id | Merchant ID | 2882 |
| brand\_id | Brand ID | 2661 |
| time\_tamp | Action time | 829 |
| action\_type | Action type {0, 1, 2, 3}0 corresponds to clicking; 1 corresponds to adding to shopping cart;2 corresponds to purchasing; 3 corresponds to favoring. | 0 |

The user log table has 54925330 records in all.

## 2.3 Training and Testing Data

| **Data Fields** | **Definition** | **Example** |
| --- | --- | --- |
| user\_id | User ID | 34176 |
| merchant\_id | Merchant ID | 3906 |
| label | Range  {0, 1}: 1 corresponds to repurchasing;  0 corresponds to non-repurchasing.In the test set this field is NULL. | 0 |

## 2.4 The Statistics about Missing value

Some variables have missing values, and we count the number of missing values for later processing.

> Missing values and missing rates of dataset

| **Data Fields** | **Number of Missing Value** | **Missing Rate** | **Note** |
| --- | --- | --- | --- |
| user\_id | 0 | **-** |   |
| item\_id | 0 | **-** |   |
| cat\_id | 0 | **-** |   |
| seller\_id | 0 | **-** |   |
| brand\_id | 91015 | 0.02% |   |
| time\_tamp | 0 | **-** |   |
| action\_type | 0 | **-** |   |
| age\_range | 2217 | 0.05% |   |
| gender | 6436 | 0.15% |   |


## 2.5 The Statistics about the basic information

We get some basic facts of the dataset listed as followed:

1. There are 424170 users, 4995 merchants, 1090390 items, 1658 item categories and 8443 brands in the dataset.
2. One user might go to different online shops to purchase different kinds of goods.
3. In the user log, it has been illustrated that there are 48550713 times clicking, 76750 times adding to shopping cart, 3005723 times favoring and 3292144 times purchasing.
4. On double 11 day, there are 10582633 records, including 9188104 clicks, 1223354 purchases, 156450 favors and 14275 adding to shopping cart. While the average data before double 11 day per day is 161518 clicks, 10178 purchases, 15485 favors and 329 adding to shopping cart.

To make it more clear, we display these data in the following table.

|   | **User** | **Merchant** | **Item** | **category** | **brand** |
| --- | --- | --- | --- | --- | --- |
| Number | 424170 | 4995 | 1090390 | 1658 | 8443 |

Now let&#39;s look at Double 11 and non – double&#39;s 11 user behavior statistics

|   | **Click** | **Adding to cart** | **favour** | **purchase** | **Total** |
| --- | --- | --- | --- | --- | --- |
| All period | 48550713 | 76750 | 3005723 | 3292144 | 54925330 |
| 11-11 | 9188104 | 14275 | 156450 | 1223354 | 10582633 |
| Daily average before 11-11 | 161518 | 329 | 15485 | 10178 | 187510 |



# 3 Data preparation
## 3.1 Fill missing values

For the pre-processing, firstly we need to fill the missing values. For missing values in age, we find the label 0 means unknown, which has 92914 records. The number is nearly a quarter of the total population, so we can not replace 0 with mode. Thus, we just fill the missing values with 0, which means unknown. For missing values in gender, we fill them with the mode. For missing values in brand id, we fill them with the mode of this specific brand of a certain merchant.

## 3.2 Feature extraction

Next, we need to extract the features. Features are divided into 3 categories: user related features, seller related features and user-merchant interaction features. Overall, we extracted 127 features.

### 3.2.1 User related features

**User basic features**

The following is users&#39; basic information, mainly provided by dataset. The basic information include some demographics features, such as age and gender. We then transform the age variable into dummy variables, and fill the missing values.

| **Data Fields** | **Type** | **Description** |
| --- | --- | --- |
| User\_id | nominal | Original data provided by dataset |
| age\_0.0 | ordinal | 1 if unknown, 0 otherwise |
| age\_1.0 | ordinal | 1 if the user&#39;s age is less than 18, 0 otherwise |
| age\_2.0 | ordinal | 1 if the user&#39;s age is in the range of 18 to 24, 0 otherwise |
| age\_3.0 | ordinal | 1 if the user&#39;s age is in the range of 25 to 29, 0 otherwise |
| age\_4.0 | ordinal | 1 if the user&#39;s age is in the range of 30 to 34, 0 otherwise |
| age\_5.0 | ordinal | 1 if the user&#39;s age is in the range of 35 to 39, 0 otherwise |
| age\_6.0 | ordinal | 1 if the user&#39;s age is in the range of 40 to 49, 0 otherwise |
| age\_7.0 | ordinal | 1 if the user is older than 50, 0 otherwise |
| age\_8.0 | ordinal | 1 if the user is older than 50, 0 otherwise |
| gender | nominal | 0 for female, 1 for male |

**User behavior features:**

The following are features that correspond to clicking, adding to shop cart, purchasing, favoring and interacting. The data comes from the user\_log table. On one hand, we count the total number of clicking, adding to shop cart, purchasing, favoring and interacting. On the other hand, we calculate the ratio of user&#39;s specific behaviors to the total number of this specific behavior in order to obtain the performance of one specific user in the whole consumer group, given that the total number of users is 424170, total interaction number is 54925330, total click number is 48550713, total adding to shopping cart number is 76750, total purchasing number is 3292144, and total favoring number is 3005723. Also, we calculate the difference between user&#39;s certain behavior and the average number of a certain behavior for the same purpose, given that the average interactions of users are 129.4, the average clicks of users are 114.5, the average purchases of users are 7.8 and the average favors of users are 7.1.

| **Data Fields** | **Type** | **Description** | **Note** |
| --- | --- | --- | --- |
| userTotalAction\_0 | interval | total number of clicks | 用户点击的总次数 |
| userTotalAction\_1 | interval | total number of adding to shopping cart | 用户加购的总次数 |
| userTotalAction\_2 | interval | total number of purchases | 用户购买的总次数 |
| userTotalAction\_3 | interval | total number of collections | 用户收藏的总次数 |
| userTotalAction | interval | count the total number of user interactions | 用户交互的总次数 |
| userTotalActionRatio | interval | the ratio of the number of user interactions to the total number of interactions for all users | 用户交互次数占所有用户的总交互次数的比例 |
| userTotalActionDiff | interval | the difference between the number of user interactions and the average number of interactions for all users | 用户交互次数与所有用户的平均交互次数的差值 |
| userClickRatio | interval | the ratio of the number of user clicks to the total number of clicks for all users | 用户点击次数占所有用户的总点击次数的比例 |
| userClickDiff | interval | the difference between the number of user clicks and the average number of clicks for all users | 用户点击次数与所有用户的平均点击次数的差值 |
| userAddRatio | interval | the ratio of the number of times a user has added to a cart to the total number of times all users have added to a cart | 用户加购次数占所有用户的总加购次数的比例  |
| userAddDiff | interval | the difference between the number of times a user adds to a cart and the average number of times all users add to a cart | 用户加购次数与所有用户的平均加购次数的差值 |
| userBuyRatio | interval | the ratio of the number of user purchases to the total number of purchases for all users | 用户购买次数占所有用户的总购买次数的比例 |
| userBuysDiff | interval | the difference between the number of user purchases and the average number of purchases for all users | 用户购买次数与所有用户的平均购买次数的差值 |
| userSaveRatio | interval | the ratio of the number of user collections to the total number of collections for all users | 用户收藏次数占所有用户的总收藏次数的比例 |
| userSaveDiff | interval | the difference between the number of user collections and the average number of collections for all users | 用户收藏次数与所有用户的平均收藏次数的差值 |

**User habit features:**

In this section we will focus on one specific user&#39;s habits rather than compare individuals&#39; behaviors to the total&#39;s. We will calculate the ratio of certain behavior times to this user&#39;s entire behavior times, e.g. the ratio of the number of user clicks to the total number of interactions by this user. Besides, we will summarize the conversion of one certain step to another. And we also calculated how broad one user would get in touch with certain items or merchants. Whether the user have already repurchased in certain shops before Double 11 Day might also affect the user&#39;s behavior later. We count the number of users&#39; monthly purchases, favors, clicks, adding to cart and total interactions to get their active cycles. Finally, we standardize all the interval features.

| **Data Fields** | **Type** | **Description** | **Note** |
| --- | --- | --- | --- |
| userClick\_ratio | interval | The ratio of the number of user clicks to the total number of interactions by this user | 用户点击次数占这个用户总交互次数的比例 |
| userAdd\_ratio | interval | The ratio of the number of times the user added to the cart to the total number of interactions by this user | 用户加购次数占这个用户总交互次数的比例 |
| userBuy\_ratio | interval | The ratio of the number of user purchases to the total number of interactions by this user | 用户购买次数占这个用户总交互次数的比例 |
| userSave\_ratio | interval | The ratio of the number of user collections to the total number of interactions by this user | 用户收藏次数占这个用户总交互次数的比例 |
| userTotalAction\_0\_ratio | interval | the click-to-buy conversion rate | 用户的点击购买转化率 |
| userTotalAction\_0\_ratio\_diff | interval | the difference between a user&#39;s click-to-buy conversion rate and the average click-to-buy conversion rate for all users | 用户的点击购买转化率与所有用户的平均点击购买转化率的差值 |
| userTotalAction\_1\_ratio | interval | the adding to cart-to-buy conversion rate | 用户的加购购买转化率 |
| userTotalAction\_1\_ratio\_diff | interval | the difference between a user&#39;s adding to cart-to-buy conversion rate and the average adding to cart-to-buy conversion rate for all users | 用户的加购购买转化率与所有用户的平均加购购买转化率的差值 |
| userTotalAction\_3\_ratio | interval | the collection-to-buy conversion rate | 用户的收藏购买转化率 |
| userTotalAction\_3\_ratio\_diff | interval | the difference between a user&#39;s collection-to-buy conversion rate and the average collection-to-buy conversion rate for all users | 用户的收藏购买转化率与所有用户的平均收藏购买转化率的差值 |
| item\_cnt | interval | The number of items the user interacts with in six months | 用户六个月中做出行为的不同商品数量 |
| cat\_cnt | interval | The number of cats the user interacts with in six months | 用户六个月中做出行为的不同商品类别数量 |
| seller\_cnt | interval | The number of sellers the user interacts with in six months | 用户六个月中做出行为的不同商家数量 |
| brand\_counts | interval | The number of brands the user interacts with in six months | 用户六个月中做出行为的不同商品品牌数量 |
| active\_days | interval | The number of days the user interacts with in six months | 用户六个月中做出行为的天数 |
| repeat\_seller\_count1 | interval | The number of merchants the user has repeatedly purchased before the double 11. Fill it with 0 if one user hasn&#39;t made a repeat purchase  | 双十一之前，用户重复购买过的商家数量，没有重复购买过则用0填充 |
| repeat\_seller1 | nominal | Before the double 11, whether the user has repeatedly bought | 双十一之前，用户是否重复购买过 |
| &#39;5\_Action\_0&#39;, &#39;5\_Action\_1&#39;, &#39;5\_Action\_2&#39;, &#39;5\_Action\_3&#39;, &#39;5\_Action&#39;,&#39;6\_Action\_0&#39;, &#39;6\_Action\_1&#39;, &#39;6\_Action\_2&#39;, &#39;6\_Action\_3&#39;, &#39;6\_Action&#39;,&#39;7\_Action\_0&#39;, &#39;7\_Action\_1&#39;, &#39;7\_Action\_2&#39;, &#39;7\_Action\_3&#39;, &#39;7\_Action&#39;,&#39;8\_Action\_0&#39;, &#39;8\_Action\_1&#39;, &#39;8\_Action\_2&#39;, &#39;8\_Action\_3&#39;, &#39;8\_Action&#39;,&#39;9\_Action\_0&#39;, &#39;9\_Action\_1&#39;, &#39;9\_Action\_2&#39;, &#39;9\_Action\_3&#39;, &#39;9\_Action&#39;,&#39;10\_Action\_0&#39;, &#39;10\_Action\_1&#39;, &#39;10\_Action\_2&#39;, &#39;10\_Action\_3&#39;, &#39;10\_Action&#39;,&#39;11\_Action\_0&#39;, &#39;11\_Action\_1&#39;, &#39;11\_Action\_2&#39;, &#39;11\_Action\_3&#39;, &#39;11\_Action&#39; | interval | These 35 variables corresponds to the number of clicks, adding to cart, purchases, collections, interactions per month for this users.The first number of the variable name corresponds to month, and the last number of the variable name corresponds to interactions types. | 用户每月的点击次数，每月的加入购物车次数，每月的购买次数，每月的收藏次数，每月的总交互次数     |

### 3.2.2 Seller related features

Now that we have collected the user related features, we need to focus on the merchant related features. The following features will be calculated in the sellers&#39; perspective. The logic behind these features is quite similar to the user related features.

We firstly calculate the sum total of the items, categories and brands to find out the scale of one specific merchant. We then calculate the number of each kind of interaction, the ratio of each kind of interaction of one merchant to the total interactions, the number of users with each kind of interaction of one specific merchant and the number of repurchasers of each merchant to figure out the shop&#39;s attraction to the user group.

| **Data Fields** | **Type** | **Description** | **Note** |
| --- | --- | --- | --- |
| Seller\_id | nominal |   |   |
| item\_number | interval | the total number of items sold by a seller | 商户的商品总数 |
| cat\_number | interval | the total number of categories of a seller | 商户的商品类别总数 |
| brand\_number | interval | the total number of brands of a seller | 商户的商品品牌总数 |
| item\_ratio | interval | the ratio of the total number of items of a merchant to the total number of items for all merchants | 商户商品数占总的商品数的比例 |
| cat\_ratio | interval | the ratio of the total number of cats of a merchant to the total number of cats for all merchants | 商户商品类别数占总的商品类别数的比例 |
| brand\_ratio | interval | the ratio of the total number of brands of a merchant to the total number of brands for all merchants | 商户商品品牌总数占总的商品品牌数的比例 |
| sellerTotalAction\_0 | interval | Total number of clicks of the merchant | 商户被点击总次数 |
| sellerTotalAction\_1 | interval | Total number of purchases of the merchant | 商户被加入购物车总次数商户被购买总次数 |
| sellerTotalAction\_2 | interval | Total number of purchases of the merchant | 商户被购买总次数 |
| sellerTotalAction\_3 | interval | Total number of collections of the merchant | 商户被收藏总次数 |
| sellerTotalAction | interval | Total number of interactions of the merchant | 商户被交互总次数 |
| sellerTotalAction\_0\_ratio | interval | Merchants&#39; click-to-buy conversion rate | 商户的被点击购买转化率 |
| sellerTotalAction\_1\_ratio | interval | Merchants&#39; adding to cart-to-buy conversion rate | 商户的被加购购买转化率 |
| sellerTotalAction\_3\_ratio | interval | Merchants&#39; collection-to-buy conversion rate | 商户的被收藏购买转化率 |
| seller\_peopleNum\_0 | interval | The number of users with click behavior of the merchant | 商户被点击的用户数 |
| seller\_peopleNum\_1 | interval | The number of users who adding items to cart of the merchant | 商户被加购的用户数 |
| seller\_peopleNum\_2 | interval | The number of users with purchase behavior of the merchant | 商户被购买的用户数 |
| seller\_peopleNum\_3 | interval | The number of users with collection behavior of the merchant | 商户被收藏的用户数 |
| click\_people\_ratio | interval | the ratio of the number of users with click behavior of this merchant to all users with click behavior | 此商户有点击行为的用户数占所有有点击行为用户数的比例 |
| add\_people\_ratio | interval | the ratio of the number of users with adding to cart behavior of this merchant to all users with adding to cart behavior | 此商户有加购行为的用户数占所有有加购行为用户数的比例 |
| buy\_people\_ratio | interval | the ratio of the number of users with purchase behavior of this merchant to all users with purcchase behavior | 此商户有购买行为的用户数占所有有购买行为用户数的比例 |
| save\_people\_ratio | interval | the ratio of the number of users with collection behavior of this merchant to all users with collection behavior | 此商户有收藏行为的用户数占所有有收藏行为用户数的比例 |
| repeatPeoCount | interval | the number of repeater buyers of this merchant before double 11 | 商户的双十一之前的重复买家数量 |

### 3.2.3 User-seller features:

Finally we collected user-seller features, which correspond to the prediction, including times one user will interact with a predicted seller, conversion from one step to the next of a predicted seller, time periods of each conversion, amounts of interaction with each item with a predicted seller, amounts of interaction with each category with a predicted seller, and amounts of interaction with each item with a predicted seller.

| **Data Fields** | **Type** | **Description** | **Note** |
| --- | --- | --- | --- |
| User\_id | nominal |   |   |
| Seller\_id | nominal |   |   |
| userSellerAction\_0 | interval | The number of times the user clicks on the predicted seller | 用户对预测商户的点击次数  |
| userSellerAction\_1 | interval | The number of times the user adds to the shopping cart on the predicted seller | 用户对预测商户的加购次数 |
| userSellerAction\_2 | interval | The number of times the user purchases on the predicted seller | 用户对预测商户的购买次数 |
| userSellerAction\_3 | interval | The number of times the user saves on the predicted seller | 用户对预测商户的收藏次数 |
| userSellerAction | interval | The number of times the user interacts with the predicted seller | 用户对预测商户的总交互次数 |
| userSellerAction\_0\_ratio | interval | the user&#39;s click-to-buy conversion rate for the predicted seller | 用户对预测商户的点击购买转化率 |
| userSellerAction\_1\_ratio | interval | the user&#39;s adding to cart-to-buy conversion rate for the predicted seller | 用户对预测商户的加购购买转化率 |
| userSellerAction\_3\_ratio | interval | the user&#39;s collection-to-buy conversion rate for the predicted seller | 用户对预测商户的收藏购买转化率 |
| click\_days | interval | the number of days when the user clicks on the predicted seller | 用户对预测商户点击的天数 |
| add\_days | interval | the number of days when the user adds to the shopping cart on the predicted seller | 用户对预测商户加购的天数 |
| buy\_days | interval | the number of days when the user purchases on the predicted seller. Since every customer buy items in certain merchant only on double 11, so the day is all 1. After normalization, it is all null, so we delete this variable. | 用户对预测商户购买的天数 |
| save\_days | interval | the number of days when the user saves on the predicted seller | 用户对预测商户收藏的天数 |
| item\_click\_count | interval | The number of items the user clicks on the predicted store | 用户对预测商户的点击商品数量 |
| item\_add\_count | interval | The number of items the user adds to the shopping cart on the predicted store | 用户对预测商户加购的不同商品数量 |
| item\_buy\_count | interval | The number of items the user purchases on the predicted store | 用户对预测商户购买的不同商品数量 |
| item\_save\_count | interval | The number of items the user saves on the predicted store | 用户对预测商户收藏的不同商品数量 |
| cat\_click\_count | interval | The number of cats the user clicks on the predicted store | 用户对预测商户点击的不同商品类别数量 |
| cat\_add\_count | interval | The number of cats the user adds to the cart on the predicted store | 用户对预测商户加购的不同商品类别数量 |
| cat\_buy\_count | interval | The number of cats the user purchases on the predicted store | 用户对预测商户购买的不同商品类别数量 |
| cat\_save\_count | interval | The number of cats the user saves on the predicted store | 用户对预测商户收藏的不同商品类别数量 |
| brand\_click\_count | interval | The number of brands the user clicks on the predicted store | 用户对预测商户点击的不同商品品牌数量 |
| brand\_add\_count | interval | The number of brands the user adds to the cart on the predicted store | 用户对预测商户加购的不同商品品牌数量 |
| brand\_buy\_count | interval | The number of brands the user saves on the predicted store | 用户对预测商户购买的不同商品品牌数量 |
| brand\_save\_count | interval | The number of brands the user saves on the predicted store | 用户对预测商户收藏的不同商品品牌数量 |



# 4 Modeling

After we have generated our feature map, the next step is to choose and train appropriate models for this task. In this part, three sub tasks are conducted.

First, we train six separate models, which consists of logistic regression, linear SVM, multi-layer perceptron, random forest, AdaBoost and XGBoost, and compare them using accuracy score. This part shows that XGBoost performs best among the six models.

Second, we conduct a cross validation on XGBoost and determine the optimal value for three vital parameters in this model. We also do a cross validation on logistic regression to make comparisons.

Finally, we choose the best four models, logistic regression, random forest, AdaBoost and XGBoost to do a stack learning. The model for stacking here we choose perceptron and logistic regression, which is simple and suitable for stack learning.

There is one thing that needed to be specify, for the separate models, the trainset we use here is all the labeled data we have, the testset is the unlabeled one and we need to upload the probability of testset instances to Aliyun and receive the accuracy report from Aliyun.

## 4.1 Separate Models
### 4.1.1 Logistic Regression

Logistic regression is a statistical model that in its basic form uses a logistic function to model a binary dependent variable. It can also deal with multi-class task and penalty.

In our model, we add a &quot;L2&quot; penalty to the model, use the LBFGS algorithm to do the optimization and set the maximum iterations to 100. The results are provided in the following table.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.9386 | 0.6460 | 3.68s |

### 4.1.2 Support Vector Machine

Support Vector Machine (SVM) is a well-known and good model for binary classification task. Here we use a linear kernel SVM to train our model.

We add a &quot;L2&quot; penalty to the model and set the maximum iterations to 1000. The results are showed in the following table.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.9386 | 0.6332 | 247.97s |

### 4.1.3 Multi-layer Perceptron

A multilayer perceptron is a subset of artificial neural network. We establish a multilayer perceptron only with one hidden layer.

Specifically, we use 700 nodes in the hidden layer, use the ReLU activation function. We set the learning rate to 0.001 and the maximum iteration to 200. Here, we set the number of nodes to 700 according to the empirical formula:

where  = number input nodes,  = number of output nodes,  = number of samples in trainset,  = an arbitrary scaling factor usually 2-10.

Multi-layer perceptron has many algorithms to do the optimization. Here we try Adam solver and Stochastic Gradient Descent solver. The following table offers the results.

| **Solver** | **Train accuracy** | **Test accuracy (upload to Aliyun)** | **Train duration** |
| --- | --- | --- | --- |
| Adam | 0.9579 | 0.5759 | 56.2m |
| SGD | 0.9396 | 0.6543 | 13.7m |

### 4.1.4 Random Forest

Random forest is an ensemble learning method for classification, regression and other tasks that operates by constructing a multitude of decision trees at training time and outputting the class that is the mode of the classes (classification) or mean prediction (regression) of the individual trees.

As for this task, we set the number of estimators (trees) to be trained as 100 using gini index as criterion. Meanwhile, we set the maximum depth for each tree as 8 and maximum features for each tree as . Random forest trains very fast and the result is as following.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.9389 | 0.6563 | 8.85s |

### 4.1.5 AdaBoost

Adaptive Boosting (short as AdaBoost), is a machine learning meta-algorithm which can be used in conjunction with many other types of learning algorithms to improve performance. The output of the other learning algorithms (&quot;weak learners&quot;) is combined into a weighted sum that represents the final output of the boosted classifier.

Here we set the number of estimators (trees) as 50, set the learning rate as 1.0 and use SAMME.R algorithm. We get a result as follows.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.9388 | 0.6588 | 37.34s |

### 4.1.6 XGBoost

XGBoost is a decision-tree-based ensemble machine learning algorithm that uses a gradient boosting framework. XGBoost has been utilized in many data science competitions and performs well on structured data.

Specifically, we use 300 estimators (trees) and set the learning rate to 0.1 and the maximum depth for each tree to 8. Then we get the best score so far as 0.6734.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.9671 | 0.6734 | 53.94s |

## 4.2 Cross Validation

Cross validation can help us to determine the optimal value of some vital parameters and deal with the problem of training overfitting.

We set the number of subset splits as 10. Here we can only do cross validation on all the labeled data we have. We use the area under curve score (AUC) for the receiver operating characteristic (ROC) curve as the measuring metrics because we are more concerned about precision ratio, which affects most of the actions company will take for the repurchase buyers.

### 4.2.1 XGBoost

We conduct cross validation on three important parameters: number of estimators, maximum depth for each tree, learning rate. For explanation, we plot out the AUC score with variances. As the flowing figurers shows, the optimal parameter values are 67 estimators, learning rate: 0.1, maximum depth for each tree: 8.

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/xgboost_nest.png)

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/xgboost_maxdepth.png)

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/xgboost_lr.png)

Specify the optimal parameters, we get the score better than the last best one. This score help our group to rank the 86th of the leaderborder of this competition.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.7971 | 0.6787 | 19.34s |

### 4.2.2 Logistic Regression

We also conduct a cross validation on logistic regression to determine the maximum iterations as 138. As we can see in the following figure, we find that the lines for trainset and test are almost the same. It is because that logistic regression is such a simple model that there is little room for complexity and overfitting on trainset.

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/lr_lr.png)

### 4.2.3 Random Forest

Here we do cross validation on four parameters: &#39;n\_estimators&#39;, &#39;max\_depth&#39;, &#39;criterion&#39;, &#39;max\_features&#39;. As what we have done on XGBoost, we set value of the four parameters as 133, 14, &#39;entropy&#39; and 17. The following table shows the result after cross validation.

| **Train accuracy** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| 0.9703 | 0.6715 | 25.59s |

## 4.3 Stack Learning

We do stack learning in order to learn the ability from different models. Stack learning means that we use the output probability of each separate model as the input feature map of the stack model. Compared among the above six models, we choose the best and fastest four separate model for this step: logistic regression, random forest, AdaBoost and XGBoost.

Since the input values are only the probabilities therefore there are not many data. A complex model is not needed in this situation, so we choose perceptron and logistic regression as the stack models.

As shown in the following table, we found that stack learning doesn&#39;t improve the accuracy. The results of cross validation are also not very good. The mean AUC never exceeds 0.6. Actually, there are few parameters in these two simple models needed to be set by cross validation.

| **Stack Model** | **Train accuracy** | **Test accuracy**** (upload to Aliyun)** |
| --- | --- | --- |
| Perceptron | 0.9435 | 0.6750 |
| Logistic Regression | 0.9444 | 0.6360 |

# 5 Evaluation
## 5.1 Overall Comparison

We make an overall comparison of all the separate models as shows in the following table. We can conclude that:

- XGBoost performs the best.
- MLPs are too time-consuming and cannot guarantee good accuracy.
- Logistic Regression is the fastest model.

| **Model** | **Test accuracy** **(upload to Aliyun)** | **Train duration** |
| --- | --- | --- |
| Logistic Regression | 0.6460 | 3.68s |
| Linear SVM | 0.6332 | 247.97s |
| MLP-adam | 0.5759 | 56.2m |
| MLP-sgd | 0.6543 | 13.7m |
| Random Forest | 0.6715 | 25.59s |
| AdaBoost | 0.6588 | 37.34s |
| XGBoost | **0.6787** | 19.34s |

## 5.2 Feature Importance

In order to understand model behavior and interpret the predictions of a model, we try to get insight from feature importance. We extract the feature importance on logistic regression and XGBoost.

### 5.2.1 Logistic Regression

For logistics regression, the bigger of the absolute coefficient value is, the more important the feature is to the model. And the positive value means that the corresponding feature helps to predict a buyer as repurchase buyer, the negative one helps to predict a buyer not as repurchase buyer.

The following figure shows the top 20 positive and negative features.

We do some detailed analysis on the positive ones: The most important variable is the user&#39;s adding to cart-to-buy conversion rate for the predicted seller (用户对预测商户的加购购买转化率）. This variable reflects the users&#39; purchasing power and the preference for predicted merchant. The second important variable is the number of days the user acts in six months (用户六个月内做出交互行为的天数). This variable reflects whether the user interacts in a long term. The third important variable is the ratio of the number of users with adding to cart behavior of this merchant to all users with adding to cart behavior (此商户有加购行为的用户数占所有有加购行为用户数的比例).  This variable reflects the popularity of the merchant in the current market. If the merchant is popular, users are more likely to purchase repeatedly.

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/coef_lr_top20.png)

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/coef_lr_last20.png)

### 5.2.2 XGBoost

The &#39;weight&#39; importance of XGBoost is the number of times a feature appears in a tree estimator as showed in the following figure.

The most frequently used variable is the number of repeater buyers of this merchant (这个商户的重复买家数量). This is not difficult to understand, if a merchant in the past was easy to attract customers to repurchase, then new users are also more likely to repeatedly purchase. The second frequently used variable is the adding to cart-to-buy conversion rate of the merchant. This variable reflects the degree of the merchant&#39;s attraction.  The third one is  the ratio of the number of user clicks to the total number of interactions of this user (用户点击次数占这个用户总交互次数的比例). This variable reflects the user&#39;s purchasing power. If a user has a higher proportion of clicks and a lower proportion of other behaviors, the user may be less likely to make a second purchase.

![image](https://github.com/CaptainCandy/TMALL_Repurchase/blob/master/images/XGBoost_weight.png)

## 5.3 Future Improvement

First, in our current models, we have 123 features in total, which may be too many. There are two ways we can improve this:

- For a specific model, we can apply forward, backward or stepwise filtering to select the features that will improve the model accuracy.
- There exists some of the features that are highly correlated, which can be filtered by conducting correlation analysis.

Second, we only conduct cross validation on the best model XGBoost and on the simplest model logistic regression. Currently, many of our models have been overfitted, which need to be addressed. In the future, we should do cross validation on each model and conduct stack learning afterwards.

Third, if we want to get a wonderful model, we need to do cross validation on other parameters. For example, in XGBoost, the parameters like minimum sum of instance weight(hessian) needed in a child or minimum loss reduction required to make a further partition on a leaf node of the tree are all interesting parameter worth exploring.

Forth, there exists many interpretable methods to explain the model, such as Permutation Importance, Partial Plots and SHAP values. We can use different methods to understand a model for better implementation.

# 6 Deployment

The results of our analysis can benefit both the merchants and the customers.

For the merchants, they can achieve their goal of identifying the one-time buyers and the repeated buyers. As the former user group will consume the resources offered by the merchants but pay little back and the latter user group will possibly become loyal buyers and make profits for this merchant in the long-term, the merchants can thus make strategies to attract the potential loyal buyers and keeping away from the one-time buyers. For example, they can offer coupons to the potential loyal buyers only, eliminating the one-time buyers. In this way, merchants can reduce their promotion costs and improve their ROI at the same time.

For customers, once the merchants identified who will be their potential loyal buyers, they will send messages and pushes to this specific user group. Therefore, users can get rid of the endless annoying advertisements sent by merchants, and just receive their most interested messages instead.
