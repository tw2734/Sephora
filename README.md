# Sephora Fake Reviews Analysis
The purpose of this project is to identify fake reviews for products on Sephora website and address the following issues:
1. What is the difference between the rating distribution of all reviews and that of fake reviews?
2. After fake reviews are removed, how does the rating of a product change?
3. What is the relationship between the number of total reviews and fake reviews?
4. Are there any patterns between the number of fake reviews of one product and its total rating over
time?

## Getting Data
We scraped all product information and reviews of skincare products from Sephora website via Selenium to overcome the problem of lazy load.

## Data Description
In total, there are 791,735 reviews from 3,400 products in our raw dataset. We have two types of datasets: Product Information Data and Reviews Data.
1. Product Information (8 columns):
- product category 
- sub-category 
- brand name
- product name
- product item id
- loves (the number of people who love this product)
- price 
- product total rating
2. Reviews (15 columns):
- product sub-category
- product item id
- number of total reviews for the product
- username 
- user skin type 
- user skin tone 
- user age range 
- review title 
- review content 
- review rating 
- helpfulness (number of people who think this review is helpful)
- unhelpfulness 
- recommend (whether the user would recommend this product to others) 
- review time 
- free product (whether the user received a sponsored free sample)

## Label Fake Reviews
1. Reviews with Same Contents but from Different User ID(s)

Because spammers are usually required to achieve some specific review volumes assigned by their employers, they may post duplicate reviews using different user ID to complete their jobs on time and reduce their workload (Chowdhary & Pandit, 2018).

2. Reviews with High Similarities to the Duplicated Review Contents

Using the abovementioned duplicated reviews contents, we build a corpus and apply Latent Similarity Indexing model to calculate the similarity scores of other remaining reviews to the existing corpus. We label reviews with more than 90% similarity to the corpus as fake.

3. Multiple Reviews from One Reviewer for One Specific Product

It is also unusual for one reviewer to post multiple reviews for a product, especially when the ratings he gives are all extreme ratings (1 or 5).

4. Reviews with Extreme Sentiment Score from Unpopular Products

Suspicious reviews usually have extremely positive or negative sentiment due to merchants’ demand of increasing their own product ratings or decreasing their competitors’ product ratings. However, popular products with high review volumes might have many positive reviews from genuine users. In order to reduce the probability of mislabeling, we only look at products with review volumes below the 25th percentile or loves below the 25th percentile. We implement weighted sentiment analysis using Valence Aware Dict Sentiment Response (VADER) to calculate the compound sentiment score for each review from unpopular products. Since the compound score ranges from -1 to 1, reviews with an absolute compound score over 0.8 are deemed as fake.

5. Reviews Mentioning Brand Name

Some reviews mention the product’s brand multiple times because the reviewer participates in marketing events of the brand and receives a complimentary product for review purposes. There are also reviews comparing the product with competitors’ to boost or damage the reputation of the product. We obtain the brand list of products under the same category from the product dataset and search for each brand in every review to find whether a review mentions a brand name. A mismatch could occur if the brand name appears in a common word. For example, the brand name ‘tarte’ would appear in the word ‘started’ during the search. Therefore, we add whitespace before and after the brand name to improve accuracy for searching. After searching, we find that 87.8% of reviews do not mention any brand, 11.2% mention a brand name once, and a brand is at most mentioned 8 times in a review. We decide that a review is fake if it mentions a brand name more than or equal to 2 times.

6. Reviews with Typos

We use the package pyspellchecker to find typos in a review. The first step is splitting a review into a list of words using regular expression (regex). The reason for using regex is that we do not want words containing numbers, like ‘30ml’, because pyspellchecker would identify these words as misspelled. We further remove abbreviations in the typo list because pyspellchecker sometimes process abbreviations improperly: it would identity ‘can’t’ as correct, but ‘i’ve’ as misspelled. Based on the filtered list, we calculate the ratio of typos for every review. If the typo ratio of a review exceeds the 95% quantile (i.e. more than 6.25% of words in the review are typos), we label it as fake.

7. Time Interval Analysis

Since fake reviews tend to present in short periods where review number increases rapidly (Fornaciari & Poesio, 2014), we carry a weekly time series analysis on review numbers for each product. We define a threshold as 1.96σ above average weekly review counts for that product. Weeks with review number above that threshold will be categorized as rapid growth. We then extract the reviews in those weeks and look at the ‘not helpful’ versus ‘helpful’ of each review, if more than 70% of people consider the review as unhelpful, then the review will be grouped as fake.

## Machine Learning Models
### Two Methods:
1. Without TF-IDF

To better describe the content of the review text, we add several columns as features, including whether the review has a title, the complexity score of each review (length of text, average characters per word, average words per sentence, unique vocabulary percentage), and the user information integrity (whether the user gives her information about her skin type, skin tone, and age).

In total, we have 10 features as input: total_reviews, review_rating, free_product, recommendation, have_title, len_of_review, avg_chars_per_word, avg_words_per_sentence, unique_vocab_percentage, information_integrity. And the target result is the labeled fake reviews.

2. With TF-IDF

We want to further explore whether the frequency of each word will influence the prediction result. Therefore, we use TF-IDF (Term Frequency - Inverse Document Frequency) to represent the relative frequency of each word, besides the original 10 features. To reduce feature dimensionality, we remove those words that only appear once.

We test both methods using Random Forest with a small subset of the Moisturizer category, which is the Decollete and Neckcream subcategory. The accuracy of both are nearly the same at around 0.9. Since the TF-IDF method is both time and space consuming, we decide to choose the 10 original features as the model input for the whole Moisturizer category dataset.

### Seven Models:

We apply 7 classification models to predict fake reviews, including Logistic Regression, Random Forest, Bagging, Neural Network, K-Nearest Neighbors, Multinomial Naive Bayes, and Linear Support Vector Machine. The test accuracy of these 7 models is quite similar ranging from 0.89 to 0.92, among which Bagging gives the best accuracy score of 0.92. We further apply the Bagging model to the whole dataset and use the prediction result to relabel fake reviews for analysis.

## Output Analysis
1. The portion of extreme ratings (1-star and 5-star) in fake reviews are much higher than that in all reviews. Only 63.2% of all reviews have a 5-star review rating while 81.9% of fakes reviews have a 5-star review rating.
2. After removing all fake reviews, 71% of products have rating decreased.
3. There a positive relationship between the number of total reviews and number of fake reviews with a correlation coefficient of 0.51.
4. We have discovered ​THREE​ patterns between the number of fake reviews of one product and its total rating over time:
   - When the rating of a product is stabilized over time, the number of fake reviews of this product tends to decrease.
   - There is a rapid growth in product rating when the number of fake reviews increases over a short period.
   - After the rating of a product decreases, the number of fake reviews increases until the rating of this product reaches a stable and high rating.
