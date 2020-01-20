# Sephora Fake Reviews Analysis
The purpose of this project is to identify fake reviews for products on Sephora website and address the following issues:
1. What is the difference between the rating distribution of all reviews and that of fake reviews?
2. After fake reviews are removed, how does the rating of a product change?
3. What is the relationship between the number of total reviews and fake reviews?
4. Are there any patterns between the number of fake reviews of one product and its total rating over
time?

## Getting Data
We scraped all product information and reviews of skincare products from Sephora website via Selenium.
In total, there are 791,735 reviews from 3,400 products in our raw dataset.

## Label Fake Reviews
1. Reviews with Same Contents but from Different User ID(s)
2. Reviews with High Similarities to the Duplicated Review Contents
3. Multiple Reviews from One Reviewer for One Specific Product
4. Reviews with Extreme Sentiment Score from Unpopular Products
5. Reviews Mentioning Brand Name
6. Reviews with Typos
7. Time Interval Analysis

## Machine Learning Models
### Two Methods:
1. Without TF-IDF
2. With TF-IDF
### Seven Models:
Logistic Regression, Random Forest, Bagging, Neural Network, K-Nearest Neighbors, Multinomial Naive Bayes, and Linear Support Vector Machine

## Output Analysis
1. The portion of extreme ratings (1-star and 5-star) in fake reviews are much higher than that in all reviews. Only 63.2% of all reviews have a 5-star review rating while 81.9% of fakes reviews have a 5-star review rating.
2. After removing all fake reviews, 71% of products have rating decreased.
3. There a positive relationship between the number of total reviews and number of fake reviews with a correlation coefficient of 0.51.
4. We have discovered ​THREE​ patterns between the number of fake reviews of one product and its total rating over time:
   - When the rating of a product is stabilized over time, the number of fake reviews of this product tends to decrease.
   - There is a rapid growth in product rating when the number of fake reviews increases over a short period.
   - After the rating of a product decreases, the number of fake reviews increases until the rating of this product reaches a stable and high rating.
