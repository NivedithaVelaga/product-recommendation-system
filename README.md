# Ecommerce-product-recommendation-system

Product Recommendation System is a machine learning-based project that provides personalized product recommendations to users based on their browsing and purchase history. The system utilizes collaborative filtering and content-based filtering algorithms to analyze user behavior and generate relevant recommendations. This project aims to improve the overall shopping experience for users, increase sales for e-commerce businesses

## Dataset

I have used an amazon dataset on user ratings for electronic products, this dataset doesn't have any headers. To avoid biases,  each product and user is assigned a unique identifier instead of using their name or any other potentially biased information.

* You can find the [dataset](https://www.kaggle.com/datasets/vibivij/amazon-electronics-rating-datasetrecommendation/download?datasetVersionNumber=1) here - https://www.kaggle.com/datasets/vibivij/amazon-electronics-rating-datasetrecommendation/download?datasetVersionNumber=1 

* You can find many other similar datasets here - https://jmcauley.ucsd.edu/data/amazon/


## Approach

### **1) Rank Based Product Recommendation**
Objective -
* Recommend products with highest number of ratings.
* Target new customers with most popular products.
* Solve the [Cold Start Problem](https://github.com/Vaibhav67979/Ecommerce-product-recommendation-system/blob/18d7fb2b8feafd117f7c3f9f859255c2e28cfbe4/ColdStartProblem.md)

Outputs -
* Recommend top 5 products with 50/100 minimum ratings/interactions.

Approach -
* Calculate average rating for each product.
* Calculate total number of ratings for each product.
* Create a DataFrame using these values and sort it by average.
* Write a function to get 'n' top products with specified minimum number of interactions.


### **2) Similarity based Collaborative filtering**
Objective -
* Provide personalized and relevant recommendations to users.

Outputs -
* Recommend top 5 products based on interactions of similar users.

Approach -
# =====================================
# SIMPLE USER-BASED RECOMMENDATION
# =====================================

"""
WHAT THIS DOES (Super Simple):

1. Find users similar to you (who rate things like you)
2. See what THEY bought that YOU didn't
3. Recommend those items!

Like: "Your friend likes same movies → recommend their favorites you missed"
"""

# Step 1: Convert text user_ids to numbers (0,1,2...)
# Why? Math works better with numbers!
user_ids = df['user_id'].astype('category').cat.codes  # "ABC" → 0, "XYZ" → 1

# =====================================
# FUNCTION 1: Find Similar Users
# =====================================
def find_similar_users(user_id, interaction_matrix, n_similar=5):
    """
    Finds TOP n_similar users who like same stuff as user_id
    
    STEPS:
    1. Compare user_id with ALL users (cosine similarity)
    2. Sort: most similar first  
    3. Remove the user himself
    4. Return top similar users + their similarity scores
    """
    
    # Get THIS user's rating row
    user_row = interaction_matrix[user_id:user_id+1].values[0]
    
    # Compare with ALL users
    similarities = []
    for i in range(len(interaction_matrix.index)):
        other_row = interaction_matrix.iloc[i:i+1].values[0]
        score = cosine_similarity([user_row], [other_row])[0][0]
        similarities.append((i, score))  # (user_index, similarity_score)
    
    # Sort: highest similarity first
    similarities.sort(key=lambda x: x[1], reverse=True)
    
    # Remove original user (similarity=1.0 to himself)
    similar_users = [(idx, score) for idx, score in similarities if idx != user_id][:n_similar]
    
    return similar_users  # Returns: [(similar_user1, 0.95), (similar_user2, 0.85)...]

# =====================================
# FUNCTION 2: Get Recommendations
# =====================================
def recommend_products(user_id, interaction_matrix, n_products=10, n_similar=5):
    """
    GET PRODUCT RECOMMENDATIONS:
    
    1. Find similar users
    2. What user ALREADY bought → ignore these
    3. What similar users bought → recommend these!
    """
    
    # Step 1: Get similar users
    similar_users = find_similar_users(user_id, interaction_matrix, n_similar)
    print(f"Top similar users: {similar_users}")
    
    # Step 2: What THIS user already interacted with
    user_interactions = set(interaction_matrix.columns[interaction_matrix.iloc[user_id] > 0].tolist())
    print(f"You already rated/bought: {len(user_interactions)} products")
    
    # Step 3: Collect recommendations from similar users
    recommendations = []
    
    for sim_user_id, sim_score in similar_users:
        # What THIS similar user bought
        sim_user_products = set(interaction_matrix.columns[interaction_matrix.iloc[sim_user_id] > 0].tolist())
        
        # NEW products (similar user bought but YOU didn't)
        new_products = list(sim_user_products - user_interactions)
        
        # Add with similarity weight (higher score = better rec)
        for prod in new_products[:5]:  # Top 5 per similar user
            recommendations.append((prod, sim_score))
    
    # Step 4: Sort by similarity score, get unique top n
    recommendations.sort(key=lambda x: x[1], reverse=True)
    top_products = list(dict.fromkeys([prod for prod, score in recommendations]))[:n_products]
    
    return top_products

# =====================================
# HOW TO USE:
# =====================================
user_id = 42  # Some user
recommendations = recommend_products(user_id, user_item_matrix, n_products=10)

print("YOUR RECOMMENDATIONS:")
for i, prod_id in enumerate(recommendations, 1):
    print(f"{i}. Product: {prod_id}")
