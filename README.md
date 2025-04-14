# Product Recommendation System for Bigbasket

<img src="/Users/apple/Desktop/grocery/images/grocery.png" style="zoom:60%;" />

BigBasket is one of India’s largest online grocery delivery platforms, offering a wide range of products including fresh produce, dairy, packaged foods, household items. This project focuses on developing a product recommendation system tailored for BigBasket's members, leveraging **Apriori algorithms** and **similarity measures** to analyze purchase patterns and user behavior. Two key features are introduced:

- **Smart Basket:** Automatically suggests commonly purchased items based on historical purchase data and product associations, helping users quickly rebuild their typical cart.
- **Did You Forget?:** Detects frequently bought items that are missing from the current basket and prompts users with reminders, reducing the chance of accidental omissions.

## Part 1: Exploratory Data Analysis

Explore and visualize BigBasket’s transaction data to uncover business insights.

### Dataset Overview

The dataset comprises 62,141 rows and 5 variables: *Member ID*, *Order ID*, *SKU*, *Timestamp* (when the order was placed), and *Product Description*. It includes 106 unique members who placed a total of 8,387 orders, spanning 1,732 distinct products categorized into 216 unique product types.

### About Members

![](/Users/apple/Desktop/grocery/images/order_placed.png)

**The 106 members placed between 24 and 203 orders on BigBasket.** Notably, 50% of the members placed 75 or more orders, reflecting a high level of user engagement. This dataset offers a robust foundation for analyzing individual shopping habits and purchasing patterns.

![](/Users/apple/Desktop/grocery/images/num_days.png)

**The 106 members have interacted with BigBasket over periods ranging from 501 to 1,334 days.** Notably, 50% of the members have been active for at least 957 days, highlighting sustained, long-term engagement. This dataset enables deep insights into evolving shopping patterns and behavioral trends over time.

<img src="/Users/apple/Desktop/grocery/images/diff_products.png" style="zoom:150%;" />

**Each member purchased between 26 and 94 distinct products**, with an average of 56 products. The distribution is concentrated around 50–60 products, indicating a typical purchasing range. Given the wide variety of items bought by each member, developing an effective personalized recommendation system requires precise pattern analysis to minimize irrelevant suggestions and enhance recommendation relevance.

### About Orders and Products

![](/Users/apple/Desktop/grocery/images/product_by_order.png)

6.86% of orders contain a single product, and 30.63% include fewer than 5 products. Additionally, 75% of orders consist of fewer than 10 items, while the largest order includes 42 products. Despite the capacity to purchase many items in a single transaction, **a significant portion of orders are small, suggesting that many may be made to purchase forgotten items.** This ordering inefficiency can contribute to increased logistics and supply chain costs.

![](/Users/apple/Desktop/grocery/images/product_popularity.png)

Among the 216 product types, vegetables—such as beans, root vegetables, and leafy greens—are the most frequently purchased items, with over 3,000 transactions recorded across 106 members. Other high-demand categories, including organic fruits and vegetables, whole spices, gourds and cucumbers, brinjals, namkeen, and bananas, have each been purchased more than 2,000 times. These trends suggest that **BigBasket is primarily used for grocery shopping, with a strong focus on fresh produce, fruits, and spices.** To maintain high levels of customer satisfaction, BigBasket should prioritize effective inventory and freshness management for these frequently purchased items.

![](/Users/apple/Desktop/grocery/images/most_frequent.png)

Analysis shows consistent shopping patterns across members, with most top-purchased items ranging from 24 to 100 times. **Beans are the most frequently bought item among 18 members, followed by organic fruits and vegetables.** These trends highlight predictable habits, making personalized recommendations both feasible and valuable.

## Part 2: Smart Basket - Apriori Algorithm

I developed a **personalized smart basket generator** for each member using the **Apriori algorithm**, tailored to a specified basket size.

**Data Preparation:** The dataset was structured with each row representing an order, containing a list of purchased product descriptions. Products were then one-hot encoded, with each column indicating the presence of a specific item in the order.

**Support Pruning:** Product combinations were generated from historical purchase data using a **minimum support threshold of 0.05**, meaning each combination must appear in at least 5% of the member's past orders.

**Confidence Pruning:** Association rules were filtered to include only those with **confidence > 0.2**, ensuring the consequent itemsets have at least a 20% likelihood of being purchased given the antecedent.

**Lift Pruning:** Rules were further refined by selecting those with a **lift > 1**, indicating a stronger-than-random relationship between items.

**Smart Basket Generation:** From the remaining rules, baskets matching the desired size were generated. The top combinations—ranked by **support**, **confidence**, or **lift**—provide optimized basket options, enabling post-deployment evaluation of the most effective recommendation criterion.

An example smart basket generated for a member is shown below:

```
The smart baskets for M36876 with size 5:
==============================================================
The data prepared for Apriori Algorithm:
         Agarbatti  Avalakki / Poha  Banana  Beans  Besan  Boiled Rice  
Order                                                                    
6640578      False            False   False   True  False        False   
6678297      False            False   False   True  False        False   
6686874      False            False   False  False  False        False   
6712708      False            False   False   True  False        False   
6808470      False            False    True   True  False        False   

         Brinjals  Cashews  Cookies   Ghee  ...  Raw Rice  Root Vegetables  
Order                                       ...                              
6640578     False    False    False  False  ...     False             True   
6678297     False    False    False  False  ...     False            False   
6686874     False    False    False  False  ...     False             True   
6712708     False    False    False  False  ...     False            False   
6808470     False    False    False  False  ...     False             True   

         Salty Biscuits  Snacky Nuts  Sooji & Rava  Sugar  Toor Dal  
Order                                                                 
6640578           False        False          True  False      True   
6678297           False        False         False  False      True   
6686874           False        False         False  False     False   
6712708           False        False         False  False      True   
6808470           False        False         False  False     False   

         Toothpaste  Urad Dal  Whole Spices  
Order                                        
6640578       False     False         False  
6678297       False      True          True  
6686874       False      True          True  
6712708       False     False         False  
6808470       False     False         False  
==============================================================
Basket with highest support: ['Banana', 'Beans', 'Brinjals', 'Other Rice Products', 'Root Vegetables']
Basket with highest confidence: ['Banana', 'Beans', 'Brinjals', 'Other Rice Products', 'Root Vegetables']
Basket with highest lift: ['Banana', 'Beans', 'Brinjals', 'Other Rice Products', 'Urad Dal']
```

## Part 3: Did You Forget? - Cosine Similarity Measure

This model recommends the top *n* products similar to those previously purchased by a member using **cosine similarity**.

**Data Preparation:** The dataset is structured with product descriptions as rows and order IDs as one-hot encoded columns, where each entry reflects how often a product appears in an order. To ensure fair comparison, values are normalized by total product frequency, reducing the impact of magnitude and improving similarity accuracy.

**Cosine Similarity Matrix:** A **Product-by-Product cosine similarity matrix** is generated per member, where each value captures how frequently two products co-occur across orders.

**Model Output:** The matrix is transformed into a dictionary:

- **Key:** Previously purchased product
- **Value:** Top *n* most similar products with similarity scores

**Interpreting Scores:** Higher scores indicate stronger co-occurrence patterns, implying the products are often bought together.

**Use Case:** When a member has an active basket, the model suggests *n* similar products for each item, helping identify potentially forgotten or complementary items.

An example recommendation output for a member is shown below:

```
The recommended missing 3 products for M32409 are:
==============================================================
The data prepared for Cosine Similarity:
                   6439325   6439360   6457975   6481313   6500391   6532863  
Other Vegetables  0.019417  0.000000  0.009709  0.019417  0.019417  0.019417   
Cream Biscuits    0.019608  0.009804  0.019608  0.019608  0.019608  0.000000   

                   6571342  6572551  6572650   6613908  ...   8289882  
Other Vegetables  0.019417      0.0      0.0  0.019417  ...  0.019417   
Cream Biscuits    0.019608      0.0      0.0  0.019608  ...  0.000000   

                   8294783  8296093   8309883   8319194   8328263   8342751  
Other Vegetables  0.019417      0.0  0.009709  0.009709  0.019417  0.009709   
Cream Biscuits    0.029412      0.0  0.019608  0.000000  0.019608  0.019608   

                   8361465   8373771   8381842  
Other Vegetables  0.019417  0.009709  0.009709  
Cream Biscuits    0.029412  0.019608  0.019608  
==============================================================
The product by product similarity matrix:
                  Other Vegetables  Cream Biscuits  Brinjals     Bread  
Other Vegetables          1.000000        0.795929  0.876816  0.545473   
Cream Biscuits            0.795929        1.000000  0.704248  0.572540   

                  Root Vegetables  Gourd & Cucumber  Other Dals  
Other Vegetables         0.810900          0.880906    0.602406   
Cream Biscuits           0.629221          0.771271    0.468143   

                  Other Rice Products  Avalakki / Poha  Sooji & Rava  ...  
Other Vegetables             0.457047         0.132236      0.406572  ...   
Cream Biscuits               0.377927         0.205527      0.410333  ...   

                  Vermicelli  Other Oils   Jaggery  Milk Drinks  Face Wash  
Other Vegetables    0.209083    0.221766  0.147844     0.073922   0.147844   
Cream Biscuits      0.092848    0.098480  0.131306     0.000000   0.131306   

                  Pain Relievers  Office Stationery   Cookies  Other Flours  
Other Vegetables        0.147844           0.147844  0.147844           0.0   
Cream Biscuits          0.131306           0.131306  0.131306           0.0   

                  Instant Pastas  
Other Vegetables        0.147844  
Cream Biscuits          0.196960  
==============================================================
A dictionary for top 3 similar products for each product is generated.

Preview of the output:

Other Vegetables - Top 3 Similar Products:
  → Similar Product: Gourd & Cucumber, Score: 0.8809
  → Similar Product: Brinjals, Score: 0.8768
  → Similar Product: Beans, Score: 0.8199

Cream Biscuits - Top 3 Similar Products:
  → Similar Product: Other Vegetables, Score: 0.7959
  → Similar Product: Gourd & Cucumber, Score: 0.7713
  → Similar Product: Brinjals, Score: 0.7042

Brinjals - Top 3 Similar Products:
  → Similar Product: Beans, Score: 0.8856
  → Similar Product: Other Vegetables, Score: 0.8768
  → Similar Product: Gourd & Cucumber, Score: 0.863
```

## Part 4: Validation Approaches of Product Recommendation System

### **Proposed Methods**

#### **Method 1: Clickstream & Conversion Tracking**

- Track user interactions with the recommended products — such as clicks and add-to-basket actions — using clickstream data.
- Calculate conversion rate of recommended products (e.g., % of recommendations that are eventually added to the basket or purchased).
- A low conversion rate may suggest that the recommended items are not relevant.

#### **Method 2: Run A/B Testing**

Design and run an A/B test comparing two groups:

- Group A: Sees product recommendations
- Group B: Sees no recommendations

Compare key metrics like: add-to-cart rate, average basket size, purchase completion rate

This approach helps validate whether the recommendation system provides measurable value before large-scale deployment.

#### **Method 3: Human Feedback Loop**

- After checkout, prompt users to rate or provide feedback on the recommended products.
- Collect satisfaction scores or open-ended feedback.
- Use this qualitative input to refine the recommendation logic and better align with real user preferences.

### **Validation Analysis**

#### **Feature: Smart Basket**

If poor performance:

- The support threshold may be too low → resulting in noisy rules.
- The confidence or lift threshold may be too low → capturing weak associations.

Adjust thresholds or prune rules to improve recommendation quality.

#### **Feature: Did You Forget**

If poor performance:

- Consider enhancing the dataset with additional product-level features besides the past orders (e.g., product category, brand, price range, seasonality).
- For members with few past orders, consider measuring similarities between members to lev

## Part 5: Challenges & Recommendations

### **Feature: Smart Basket**

#### **Challenges:**

- New users or products may not have enough purchase history to generate reliable rules.
- Association rule mining (e.g., Apriori) can become computationally expensive with large-scale datasets.
- Relying only on historical patterns can miss new trends, seasonal items, or changes in customer preferences.
- Rules may not generalize well across user segments or adapt to real-time cart dynamics.

#### **Recommendations:**

- Combine association rules with collaborative filtering or popularity-based methods to handle cold start cases.
- Generate rules per customer segment (e.g., frequent shoppers, high spenders) to increase relevance.
- Precompute frequent itemsets in batches, then apply them in real-time as users build their baskets.

### **Feature: Did You Forget**

#### **Challenges:**

- The one-hot encoded matrix can become sparse, especially if there are many unique products and few shared orders. This sparsity can limit the ability of cosine similarity to find meaningful relationships.
- Cosine similarity based solely on order frequency may not capture deeper relationships between products, such as how they are used together or their complementary nature in a broader context (e.g., seasonal trends, price ranges).
- Products with higher frequencies may dominate the similarity measure, leading to a bias where only popular products are considered similar, even if they aren't truly complementary or related in meaningful ways.

#### **Recommendations:**

- To improve similarity measures, include more product features such as categories, brand, price, or user ratings. This can help provide more context and lead to more meaningful similarity scores.
- To reduce sparsity and mitigate the impact of irrelevant features, consider using dimensionality reduction techniques on the one-hot encoded matrix.
- Combine content-based and collaborative filtering approaches, especially for members with few past orders. For example, use collaborative filtering to leverage product similarity between members, and content-based similarity to match products with similar features.
- Adjust the cosine similarity formula to account for popularity bias. This can be done by incorporating weighting schemes to reduce the influence of frequently purchased products that dominate similarity calculations.