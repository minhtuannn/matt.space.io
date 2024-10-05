---
title: Efficient Handling of Sparse Data in Recommendation Systems Using Advanced Hashing Techniques
date: 2024-09-16 12:21:12 # YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [machine_learning, recommendation_system] # [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [cs, production] # [TAG]     # TAG names should always be lowercase
description: This post explores TikTok’s use of advanced hashing techniques to efficiently handle sparse and dynamic data in recommendation systems, reducing collisions, optimizing memory usage, and enabling real-time updates for more accurate recommendations.


---

### Hashing Project Overview

Researchers at TikTok identified a fundamental issue with traditional recommendation systems: the data is mostly **categorical and sparse**, which presents challenges in embedding and processing. When dealing with sparse features like user IDs or item IDs, the number of unique categories can become extremely large. If we use standard ML approaches like word embeddings, these methods become impractical because the number of unique features in recommendation data is much higher compared to language models, which have a limited vocabulary size. To solve this, TikTok researchers introduced a more efficient way to implement embeddings using **collisionless HashTables**.

### Problem: Managing Sparse and Dynamic Data

In typical recommendation systems, embedding tables correspond to sparse features, such as user IDs or product IDs, and are used to generate feature representations. However, these embeddings rely on an assumption that the IDs in the embedding table are evenly distributed in terms of frequency. Unfortunately, real-world data rarely fits this assumption, which can lead to collisions—when different IDs are mapped to the same vector representation. These collisions reduce the model’s ability to distinguish between different categories, leading to inaccurate recommendations and degraded performance.

Additionally, updating models based on static parameters and dense computations is unsuitable for recommendation systems because they need to process dynamic and constantly changing data. In most systems, training and serving phases are completely separate. This means that the model cannot incorporate real-time user feedback, making it challenging to respond to changes in user behavior promptly.

### The Issue of Exploding Feature Space

In deep learning, a common way to handle categorical variables is through embeddings, where each category is represented as an n-dimensional vector. However, the problem arises because new categories can appear at any time, and the number of possible categories is theoretically unlimited. For instance, new users might sign up daily, or new products may be added to a catalog, causing an explosion in the number of unique categories. 

With a finite memory capacity, it’s impossible to have an embedding layer that can handle an infinite number of categories. This limitation leads to **collisions**, where multiple IDs get mapped to the same n-dimensional vector. To understand this, think of it as trying to fit 11 balls into 10 boxes. No matter how you arrange them, at least one box will contain more than one ball. This scenario, called a collision, hampers the model’s ability to correctly differentiate between different categories.

### TikTok’s Solution: HashTables and Filtering Methods

TikTok addressed these issues by designing a set of highly efficient, collisionless, and flexible HashTable operations specifically for sparse parameters. This new approach includes advanced techniques such as **Cuckoo Hashing** and **Filtering and Evicting Categories** to prevent collisions and optimize memory usage.

- **Cuckoo Hashing**: A method used to resolve collisions by moving elements between multiple hash tables until every item has a unique position.
- **Filtering and Evicting Categories**: A strategy to remove less frequently used categories from the embedding table to free up space for new categories, ensuring that only the most relevant categories are retained.

These methods allow the system to better manage the enormous feature space of recommendation data and ensure more accurate and collision-free embeddings.

### Real-Time Application and Training Process

To implement this, the recommendation system follows a process that integrates both batch training and real-time serving:

1. **Input Data**: The input data can include log activities, category IDs, or any other information that can be used to describe user and item interactions.

2. **Batch Training**: In each training step, a training worker reads a mini-batch of data, retrieves parameters from the Parameter Server (PS), performs forward and backward passes to update parameters, and then pushes the updated parameters back to the PS. Unlike many other deep learning tasks, this process only requires one pass over the dataset. Batch training is especially useful when the model architecture is updated or retrained using historical data.

3. **Serving**: The trained parameters are then used to serve real-time recommendations. This process is crucial for models that need to stay up-to-date and responsive to changes in user preferences.

### Concept Drift and Synchronization Between Training and Serving Models

One key challenge in recommendation systems is **concept drift**—the difference between the model used for training and the model used for serving recommendations in real-time. This happens because training the model, which involves gradient computations, is much slower than inference. Consequently, the weights of the Training PS and Serving PS may become out of sync, causing inconsistencies in recommendations.

Key insights to address this problem include:

- **Sparse Features Are Frequently Updated**: In most recommendation models, only a small number of sparse features, like user or item embeddings, are updated frequently. For example, on a platform like TikTok, only a fraction of users might be active at a given time, so only their embeddings need to be updated.

- **Dense Parameters Change Slowly**: Dense parameters that belong to the Deep Neural Network (DNN) part of the model change much more slowly compared to sparse features. This means they can be updated less frequently without affecting performance significantly.

- **Minimizing Update Overhead**: Transferring a multi-terabyte model from the Training PS to the Serving PS can put immense pressure on the network and memory. This is avoided by only updating sparse parameters that have been changed, keeping the Serving PS lightweight and efficient.

### Final Solution

The final approach revolves around tracking which sparse features have been updated during training and only pushing those changes to the Serving PS. This minimizes network load and ensures that the Serving PS can be updated in near real-time without interrupting service.

- **Frequent Updates**: Sparse parameters are updated every minute, which is feasible given the small fraction of IDs that are modified during each training window.
- **Less Frequent Updates for Dense Parameters**: Dense parameters are updated less frequently, as they don’t change as rapidly as sparse parameters.

By leveraging hashing and filtering techniques, this method keeps the Serving PS and Training PS synchronized, even though they might be running in separate clusters. The result is a highly scalable, real-time recommendation system that can handle large-scale categorical data with minimal performance loss.

### Conclusion

This project demonstrates how advanced hashing techniques can optimize the handling of sparse and dynamic data in recommendation systems. By using collisionless HashTables, filtering methods, and smart synchronization strategies, TikTok’s approach ensures that recommendation models can keep up with changes in user preferences in real-time, while minimizing computational and memory overhead. This results in more accurate recommendations and a better user experience, even in systems with massive and evolving data sets.
