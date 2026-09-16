# Machine Learning Fundamentals

> **Purpose:** Personal ML fundamentals notes for recall, revision, interviews, and long-term reference.

---

## Table of Contents

1. [What is Machine Learning?](#1-what-is-machine-learning)
2. [How Machine Learning Can Be Classified](#2-how-machine-learning-can-be-classified)
3. [ML Types Based on Amount of Supervision](#3-ml-types-based-on-amount-of-supervision)
   - [3.1 Supervised Learning](#31-supervised-learning)
   - [3.2 Unsupervised Learning](#32-unsupervised-learning)
   - [3.3 Semi-Supervised Learning](#33-semi-supervised-learning)
   - [3.4 Reinforcement Learning](#34-reinforcement-learning)
   - [3.5 Quick Comparison](#35-quick-comparison)
4. [ML Types Based on How Models Are Served](#4-ml-types-based-on-how-models-are-served)
   - [4.1 Batch Learning](#41-batch-learning)
   - [4.2 Online Learning](#42-online-learning)
   - [4.3 Batch vs Online Learning](#43-batch-vs-online-learning)
5. [ML Types Based on How Models Learn](#5-ml-types-based-on-how-models-learn)
   - [5.1 Instance-Based Learning](#51-instance-based-learning)
   - [5.2 Model-Based Learning](#52-model-based-learning)
   - [5.3 Instance-Based vs Model-Based Learning](#53-instance-based-vs-model-based-learning)
6. [Major Challenges in Machine Learning](#6-major-challenges-in-machine-learning)
7. [Machine Learning Development Life Cycle (MLDLC)](#7-machine-learning-development-life-cycle-mldlc)
8. [A Complete ML Project Example](#8-a-complete-ml-project-example)
9. [Important Fundamental Concepts](#9-important-fundamental-concepts)
10. [Common ML Terms](#10-common-ml-terms)
11. [Interview Revision](#11-interview-revision)
12. [Key Takeaways](#12-key-takeaways)

---

# 1. What is Machine Learning?

**Machine Learning (ML)** is a branch of Artificial Intelligence that allows computers to learn patterns from data and use those patterns to make predictions, decisions, or actions without being explicitly programmed with rules for every possible situation.

### Traditional Programming

In traditional programming:

```text
Rules + Data → Program → Output
```

For example, suppose we want to determine whether an email is spam.

We might manually create rules:

```text
IF email contains "WIN MONEY"
    → Spam

IF email contains "FREE PRIZE"
    → Spam
```

The problem is that spam can be written in thousands of different ways. Creating rules for every case becomes difficult.

### Machine Learning

With ML:

```text
Data + Expected Answers → ML Algorithm → Model
Model + New Data → Prediction
```

Instead of manually writing thousands of rules, we provide examples of spam and legitimate emails. The ML algorithm learns patterns that distinguish them.

---

## Simple Definition

> **Machine Learning is the process of enabling a computer system to learn useful patterns from data and use those learned patterns to perform a task on new data.**

---

# 2. How Machine Learning Can Be Classified

Machine Learning can be classified from different perspectives.

### Classification 1 — Based on amount of supervision

This asks:

> **How much labeled feedback does the model receive?**

The major categories are:

1. Supervised Learning
2. Unsupervised Learning
3. Semi-Supervised Learning
4. Reinforcement Learning

---

### Classification 2 — Based on how the model is trained/updated in production

This asks:

> **Does the model learn from the entire dataset at once or continuously from incoming data?**

The major categories are:

1. Batch Learning
2. Online Learning

---

### Classification 3 — Based on how the model makes predictions

This asks:

> **Does the model compare new examples with previously seen examples, or learn a general mathematical model?**

The major categories are:

1. Instance-Based Learning
2. Model-Based Learning

---

## Important Point

These classifications describe **different dimensions** of an ML system.

For example, one ML system can be:

> **Supervised + Batch + Model-Based**

Another could be:

> **Supervised + Online + Model-Based**

So these categories are **not mutually exclusive across classifications**.

---

# 3. ML Types Based on Amount of Supervision

The amount of supervision refers to how much information we provide about the correct answer during learning.

---

# 3.1 Supervised Learning

## Definition

**Supervised learning** is a type of ML in which the model learns from a dataset containing **input features and corresponding target labels/values**.

The model tries to learn a relationship:

```text
Input X → Target y
```

The target is the answer that the model should learn to predict.

---

## Example: House Price Prediction

Suppose we have:

| Area | Bedrooms | Location | Price |
|---:|---:|---|---:|
| 1000 sq ft | 2 | Delhi | ₹50L |
| 1500 sq ft | 3 | Delhi | ₹75L |
| 2000 sq ft | 4 | Delhi | ₹1Cr |

Here:

### Features

```text
Area
Bedrooms
Location
```

### Target

```text
Price
```

The model learns:

```text
Features → Price
```

Then for a new house:

```text
Area = 1800 sq ft
Bedrooms = 3
Location = Delhi
```

The model might predict:

```text
Price ≈ ₹85L
```

---

## Types of Supervised Learning

Supervised learning is mainly divided into:

### 1. Regression

The target is a **continuous numerical value**.

Examples:

- House price prediction
- Temperature prediction
- Revenue prediction
- Stock price prediction
- Sales forecasting

Example:

```text
Input → House information
Output → ₹85,00,000
```

---

### 2. Classification

The target represents a **category/class**.

Examples:

- Spam vs Not Spam
- Fraud vs Legitimate
- Disease vs No Disease
- Cat vs Dog
- Customer Churn vs No Churn

Example:

```text
Input → Email
Output → Spam
```

---

## Common Supervised Algorithms

### Regression

- Linear Regression
- Polynomial Regression
- Decision Tree Regression
- Random Forest Regression
- Gradient Boosting
- XGBoost

### Classification

- Logistic Regression
- K-Nearest Neighbors
- Decision Trees
- Random Forest
- Support Vector Machines
- Naive Bayes
- Gradient Boosting
- XGBoost
- Neural Networks

---

## Real-World Example

### Credit Card Fraud Detection

Dataset:

```text
Transaction amount
Location
Time
Merchant
Device
Previous transaction behavior
        ↓
    ML Model
        ↓
Fraud / Not Fraud
```

Historical transactions are labeled as fraudulent or legitimate.

The model learns patterns associated with fraud.

---

## Advantages

- Usually easier to evaluate because correct answers are available.
- Excellent for prediction and classification tasks.
- Many mature algorithms are available.

## Challenges

- Requires labeled data.
- Labeling can be expensive and time-consuming.
- Poor labels can produce a poor model.

---

# 3.2 Unsupervised Learning

## Definition

**Unsupervised learning** works with data where there are **no target labels provided**.

The model tries to discover hidden patterns, structures, groups, or relationships within the data.

```text
Input Data → ML Algorithm → Hidden Structure
```

---

## Example: Customer Segmentation

Suppose an e-commerce company has:

| Customer | Spending | Purchases |
|---|---:|---:|
| A | ₹10,000 | 20 |
| B | ₹9,500 | 18 |
| C | ₹1,000 | 2 |
| D | ₹1,500 | 3 |

There is no predefined label such as:

```text
Premium Customer
Regular Customer
```

An unsupervised algorithm can discover groups automatically.

It might identify:

```text
Cluster 1 → High spending / frequent buyers
Cluster 2 → Low spending / occasional buyers
```

---

## Major Types of Unsupervised Learning

### 1. Clustering

Groups similar observations together.

Popular algorithms:

- K-Means
- DBSCAN
- Hierarchical Clustering
- Gaussian Mixture Models

Example:

```text
Customers
    ↓
Clustering
    ↓
Group A | Group B | Group C
```

---

### 2. Dimensionality Reduction

Reduces the number of features while attempting to preserve important information.

Popular methods:

- PCA
- t-SNE
- UMAP

Example:

A dataset has:

```text
100 features
```

Dimensionality reduction might represent it using:

```text
10 important dimensions
```

This can help with visualization, compression, and sometimes model efficiency.

---

### 3. Association Rule Learning

Finds relationships between items.

Example:

A supermarket discovers:

```text
Customers who buy bread
often also buy butter.
```

Algorithms include:

- Apriori
- FP-Growth

---

## Real-World Applications

Unsupervised learning can be used for:

- Customer segmentation
- Anomaly detection
- Recommendation systems
- Market basket analysis
- Document grouping
- Data exploration
- Feature reduction

---

## Important Limitation

Because there are no predefined labels, evaluating unsupervised learning can be more difficult.

For example:

> If an algorithm creates three customer groups, how do we know whether those groups are actually useful?

Business knowledge and additional evaluation methods may be required.

---

# 3.3 Semi-Supervised Learning

## Definition

**Semi-supervised learning** uses a combination of:

- A relatively small amount of labeled data
- A large amount of unlabeled data

```text
Small labeled dataset
        +
Large unlabeled dataset
        ↓
Semi-Supervised Learning
```

---

## Why Is It Useful?

Labeling data can be expensive.

Imagine a company has:

```text
1,000,000 images
```

But manually labeling all of them would require huge amounts of:

- Time
- Human effort
- Money

Instead, humans might label:

```text
10,000 images
```

while:

```text
990,000 images
```

remain unlabeled.

A semi-supervised approach can use both.

---

## Example: Medical Image Classification

Suppose a hospital has:

```text
100,000 X-ray images
```

Only:

```text
5,000
```

have been reviewed and labeled by medical experts.

The remaining:

```text
95,000
```

are unlabeled.

A semi-supervised method can potentially exploit the structure in the unlabeled images along with the labeled examples.

---

## Example: Image Recognition

Suppose:

```text
Labeled:
1,000 images

Unlabeled:
100,000 images
```

The model can use the labeled examples to understand the task while also learning useful patterns from the large unlabeled dataset.

---

## When Is It Useful?

Semi-supervised learning is useful when:

- Unlabeled data is abundant.
- Labeled data is expensive.
- Human experts are required for labeling.
- The available labeled dataset is too small by itself.

---

## Important Idea

Semi-supervised learning sits between supervised and unsupervised learning:

```text
Fully labeled
     ↓
Supervised

Partially labeled
     ↓
Semi-Supervised

No labels
     ↓
Unsupervised
```

---

# 3.4 Reinforcement Learning

## Definition

**Reinforcement Learning (RL)** is a learning approach in which an **agent interacts with an environment**, takes actions, receives rewards or penalties, and learns a strategy for making better decisions.

The key components are:

```text
Agent
  ↓ action
Environment
  ↓
Reward + New State
  ↓
Agent
```

---

## Example: Game Playing

Imagine an AI playing chess.

### Agent

```text
Chess AI
```

### Environment

```text
Chess board
```

### Action

```text
Move a piece
```

### Reward

```text
Win  → Positive reward
Lose → Negative reward
```

The AI repeatedly plays and learns which actions lead to better long-term outcomes.

---

## Another Example: Robot

A robot needs to navigate a room.

```text
State → Robot's current position
Action → Move left/right/forward/backward
Reward → Positive for reaching destination
Penalty → Negative for hitting an obstacle
```

Through repeated interaction, the robot learns a policy.

---

## Important Terms

### Agent

The learner or decision-making system.

### Environment

The world with which the agent interacts.

### State

The current situation of the environment.

### Action

A decision made by the agent.

### Reward

Feedback received after an action.

### Policy

A strategy that determines which action should be taken in a particular state.

---

## Exploration vs Exploitation

One of the fundamental RL problems is balancing:

### Exploration

Trying new actions to discover whether they produce better results.

### Exploitation

Choosing actions that are already known to work well.

Example:

Suppose a robot knows:

```text
Route A → usually gives reward 10
Route B → unknown
```

Always choosing A is exploitation.

Trying B to discover whether it can give reward 20 is exploration.

A good RL system needs to balance both.

---

## Applications

- Robotics
- Game playing
- Resource allocation
- Recommendation
- Autonomous systems
- Control systems
- Some optimization problems

---

# 3.5 Quick Comparison

| Type | Labeled Data | Main Goal | Example |
|---|---|---|---|
| Supervised | Yes | Predict target | House price |
| Unsupervised | No | Find hidden structure | Customer segmentation |
| Semi-Supervised | Some labeled + lots unlabeled | Learn with limited labels | Image classification |
| Reinforcement | Rewards/penalties | Learn actions/policy | Game-playing AI |

---

# 4. ML Types Based on How Models Are Served

Another useful classification is based on **how frequently the model learns from new data**.

The two major approaches are:

1. Batch Learning
2. Online Learning

> Note: In practical ML systems, "batch" and "online" can refer to both training/update strategy and inference/serving patterns. The exact meaning depends on system architecture. Here, the focus is primarily on how the model is updated with data.

---

# 4.1 Batch Learning

## Definition

In **Batch Learning**, the model is trained using a large batch or complete dataset and is not continuously updated every time new data arrives.

A common workflow is:

```text
Historical Data
      ↓
Train Model
      ↓
Deploy Model
      ↓
Collect New Data
      ↓
Retrain Later
      ↓
Deploy Updated Model
```

---

## Example: House Price Prediction

Suppose a company has:

```text
5 years of historical house data
```

It trains a model.

The model is deployed in January.

New data is collected during:

```text
January
February
March
```

At the end of March, the company retrains the model using newer data.

This is a batch-style update process.

---

## Advantages

### 1. Simple

The system is easier to design and maintain.

### 2. Stable

The model does not change after every new data point.

### 3. Suitable for large training jobs

Large datasets can be processed using powerful compute infrastructure.

### 4. Easier testing

A specific model version can be evaluated before deployment.

---

## Disadvantages

### 1. Delayed learning

The model does not immediately learn from new data.

### 2. Can become stale

If the underlying data distribution changes quickly, the model may lose accuracy.

### 3. Retraining can be expensive

Large models and datasets can require significant:

- Compute
- Memory
- Storage
- Cloud resources
- Engineering time

---

# 4.2 Online Learning

## Definition

**Online Learning** updates a model incrementally as new data becomes available.

Instead of waiting for a large dataset to accumulate:

```text
New Data
   ↓
Update Model
   ↓
New Data
   ↓
Update Model
   ↓
...
```

Depending on the implementation, updates can occur one example at a time or in small mini-batches.

---

## Example: Fraud Detection

Financial transaction behavior can change quickly.

Suppose:

```text
Transaction 1 → Model update
Transaction 2 → Model update
Transaction 3 → Model update
...
```

An online learning system can adapt more frequently to new patterns.

---

## Advantages

### 1. Adapts to changing data

Useful when the data distribution changes over time.

### 2. Lower memory requirement in some setups

The system can process smaller portions of data rather than storing the entire training set for every update.

### 3. Suitable for data streams

Useful when data arrives continuously.

---

## Disadvantages

### 1. Sensitive to bad data

If incorrect or malicious data enters the update stream, the model may learn from it.

### 2. More complex

Monitoring and rollback become important.

### 3. Risk of model drift

The model can gradually move away from desired behavior if the incoming data is problematic.

---

# 4.3 Batch vs Online Learning

| Feature | Batch Learning | Online Learning |
|---|---|---|
| Learning style | Periodic retraining | Incremental updates |
| New data | Accumulated before retraining | Used continuously/in small batches |
| Adaptation | Slower | Faster |
| Complexity | Usually simpler | Usually more complex |
| Data stream | Less suitable | Well suited |
| Risk from bad incoming data | Lower during deployment | Higher |
| Typical use | Periodic forecasting | Dynamic environments |

---

## Example Comparison

### Batch

```text
100 GB Data
     ↓
Train
     ↓
Model v1
     ↓
Deploy
     ↓
Wait
     ↓
New 100 GB Data
     ↓
Retrain
     ↓
Model v2
```

### Online

```text
Data 1 → Update
Data 2 → Update
Data 3 → Update
Data 4 → Update
...
```

---

# 5. ML Types Based on How Models Learn

This classification focuses on **how the model uses training examples to make predictions**.

The two major approaches are:

1. Instance-Based Learning
2. Model-Based Learning

---

# 5.1 Instance-Based Learning

## Definition

**Instance-Based Learning** learns by essentially storing examples from the training data and comparing new observations with those examples.

Instead of learning a general mathematical relationship first, the system relies heavily on the similarity between a new example and previously seen examples.

---

## Example: K-Nearest Neighbors (KNN)

Suppose we have customers:

```text
Customer A → Low spending
Customer B → Low spending
Customer C → High spending
Customer D → High spending
```

A new customer arrives.

KNN looks at the most similar existing customers.

If most nearby customers are:

```text
High spending
```

the new customer may be classified as:

```text
High spending
```

---

## Simple Analogy

Imagine you are trying to identify a fruit.

You have previously seen:

```text
Apple → red, round
Apple → green, round
Orange → orange, round
Banana → yellow, long
```

You see a new fruit:

```text
Red + round
```

You compare it with previously seen examples and determine that it resembles apples.

That is the basic intuition behind instance-based learning.

---

## Key Characteristic

The system depends strongly on:

> **Similarity between new data and stored examples.**

---

## Advantages

- Simple concept.
- Can adapt easily when new examples are added.
- Useful when local similarity is meaningful.

## Disadvantages

- Can require substantial storage.
- Prediction can be computationally expensive for large datasets.
- Sensitive to the definition and scaling of distance/similarity.
- Less suitable when comparing every new point with a huge training set is expensive.

---

# 5.2 Model-Based Learning

## Definition

**Model-Based Learning** learns a general mathematical or statistical model from training data.

Instead of simply comparing every new observation with stored examples, the algorithm learns parameters that describe a relationship in the data.

```text
Training Data
     ↓
Learning Algorithm
     ↓
Mathematical Model
     ↓
New Data
     ↓
Prediction
```

---

## Example: Linear Regression

Suppose we want to predict house prices.

The model may learn:

```text
Price = w1 × Area + w2 × Bedrooms + b
```

where:

- `w1` = learned weight for area
- `w2` = learned weight for bedrooms
- `b` = bias/intercept

The algorithm learns values for these parameters from the training data.

---

## Example

Suppose after training:

```text
Price = 5000 × Area + 500000 × Bedrooms + 100000
```

For a new house, the model can plug the new values into the learned equation and produce a prediction.

The model does not need to search through every training example in the same way KNN does.

---

## Goal of Model-Based Learning

The goal is generally to find a model that captures useful relationships in the training data and **generalizes** to unseen data.

---

## Common Model-Based Algorithms

Many algorithms are model-based, including:

- Linear Regression
- Logistic Regression
- Decision Trees
- Random Forest
- Support Vector Machines
- Neural Networks
- Gradient Boosting models

---

# 5.3 Instance-Based vs Model-Based Learning

| Feature | Instance-Based | Model-Based |
|---|---|---|
| Main idea | Compare with stored examples | Learn general model |
| Training | Often little explicit parameter fitting | Learns parameters/structure |
| Prediction | Based on similar instances | Uses learned model |
| Storage | Can require many examples | Usually stores model parameters/structure |
| Example | KNN | Linear Regression |
| Generalization | Based heavily on similarity | Based on learned relationship |

---

## Easy Way to Remember

### Instance-Based

> **"I remember examples and compare."**

### Model-Based

> **"I learn a rule/model from examples."**

---

# 6. Major Challenges in Machine Learning

Building an ML model is not simply:

```text
Get Data → Train Model → Done
```

Real-world ML is an end-to-end engineering process.

A large amount of work can happen before and after model training.

---

# 6.1 Problem Definition

Before collecting data, we must understand:

- What problem are we solving?
- Who will use the model?
- What is the expected output?
- How will success be measured?
- Is ML actually necessary?

---

## Example

Bad problem statement:

> "Build an AI model for our customers."

Better:

> "Predict which customers are likely to cancel their subscription within the next 30 days."

The second problem is measurable and can be converted into an ML task.

---

# 6.2 Data Gathering

An ML model needs useful data.

Possible sources include:

- Databases
- APIs
- CSV files
- Application logs
- Sensors
- Websites
- User interactions
- Third-party datasets
- Internal business systems

---

## Challenges

Data may be:

- Difficult to access
- Expensive
- Incomplete
- Inconsistent
- Biased
- Outdated
- Stored in different systems
- Subject to privacy or legal restrictions

---

# 6.3 Data Quality

A model can only learn from the information it receives.

Common data-quality problems include:

### Missing values

```text
Age = NULL
```

### Incorrect values

```text
Age = 350
```

### Duplicate records

```text
Customer A
Customer A
Customer A
```

### Inconsistent formats

```text
India
IND
IN
```

### Incorrect labels

```text
Actual → Fraud
Label → Legitimate
```

Bad labels can be especially damaging because the model learns the wrong relationship.

---

# 6.4 Insufficient Data

Complex ML problems often require substantial amounts of representative data.

If we have only:

```text
100 examples
```

for a problem with many possible real-world situations, the model may struggle to generalize.

However, more data is not automatically better.

The data must also be:

- Relevant
- Representative
- Correct
- Diverse
- Properly labeled when required

---

# 6.5 Non-Representative Data

A model should be trained on data that reasonably represents the environment in which it will be used.

---

## Example

Suppose a facial recognition system is trained mostly using images captured under:

```text
Bright lighting
```

but deployed in:

```text
Low-light environments
```

Performance may decrease.

This is a form of **distribution mismatch**.

---

# 6.6 Biased Data

Training data can contain systematic biases.

If historical data reflects an existing bias, an ML model may learn and reproduce it.

Therefore, data should be examined carefully for:

- Sampling bias
- Measurement bias
- Labeling bias
- Historical bias
- Representation gaps

---

# 6.7 Data Leakage

**Data leakage** happens when information that would not legitimately be available at prediction time is accidentally used during training.

This can produce unrealistically high evaluation scores.

---

## Example

Suppose we want to predict whether a customer will cancel tomorrow.

But our training features include:

```text
Cancellation confirmation timestamp
```

That information becomes available only after cancellation.

The model effectively gets the answer in advance.

The resulting accuracy may look excellent but will not represent real-world performance.

---

# 6.8 Data Preprocessing

Raw data often cannot be directly given to an ML algorithm.

Typical preprocessing includes:

- Handling missing values
- Removing or investigating duplicates
- Encoding categorical variables
- Scaling numerical features when appropriate
- Parsing dates
- Cleaning text
- Handling outliers
- Feature transformation

---

# 6.9 Feature Engineering

**Feature engineering** means creating useful input features from raw data.

---

## Example

Raw data:

```text
Date of birth
Current date
```

Instead of directly using both, we might create:

```text
Age
```

Another example:

```text
Order Date
Delivery Date
```

can become:

```text
Delivery Time = Delivery Date - Order Date
```

Good features can significantly improve model performance.

---

# 6.10 Choosing the Right Model

There is no universally best ML algorithm.

The appropriate model depends on:

- Problem type
- Dataset size
- Feature types
- Required interpretability
- Training time
- Inference latency
- Available compute
- Accuracy requirements
- Business constraints

A simple model can sometimes outperform a complex model when the data and problem favor it.

---

# 6.11 Underfitting

**Underfitting** occurs when the model is too simple to capture important patterns.

Example:

```text
Complex relationship
       ↓
Very simple model
       ↓
Poor training performance
```

The model has high bias and does not learn enough from the data.

---

# 6.12 Overfitting

**Overfitting** occurs when a model learns the training data too closely, including noise or accidental patterns, and therefore performs poorly on unseen data.

Example:

```text
Training accuracy → 99%
Test accuracy     → 70%
```

This is a warning sign.

The model has learned the training examples very well but does not generalize well.

---

# 6.13 Generalization

A good ML model should perform well not only on training data but also on **unseen data from the same relevant problem distribution**.

This ability is called:

> **Generalization**

The ultimate goal is usually not to memorize the training dataset.

It is to learn useful patterns that transfer to new examples.

---

# 6.14 Training vs Validation vs Test Data

A common split is:

```text
Dataset
   ↓
Training Set
Validation Set
Test Set
```

### Training Set

Used to learn model parameters.

### Validation Set

Used during development for choices such as:

- Hyperparameter tuning
- Model selection
- Threshold selection

### Test Set

Used for final evaluation after development decisions are complete.

---

## Example

Suppose we have:

```text
100,000 records
```

One possible split:

```text
70,000 → Training
15,000 → Validation
15,000 → Test
```

The exact proportions depend on the problem.

---

# 6.15 Model Evaluation

Different problems require different metrics.

### Classification

Common metrics:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- PR-AUC

### Regression

Common metrics:

- MAE
- MSE
- RMSE
- R²

Choosing the wrong metric can lead to optimizing the wrong objective.

---

# 6.16 Computational Cost

ML can be expensive.

Costs can include:

- Data storage
- Data transfer
- CPU
- GPU
- Cloud infrastructure
- Training
- Inference
- Monitoring
- Retraining
- Engineering time

---

## Example

A small linear regression model may train cheaply.

A large deep learning model may require:

```text
Large datasets
+
GPU clusters
+
Long training time
+
High inference costs
```

Therefore, model performance must be considered alongside cost.

---

# 6.17 Deployment Challenges

A model that works in a notebook is not automatically production-ready.

Deployment may involve:

```text
Model
 ↓
API / Service
 ↓
Application
 ↓
Users
```

Challenges include:

- API design
- Latency
- Scalability
- Security
- Versioning
- Resource management
- Error handling
- Dependency management

---

# 6.18 Model Drift

The world can change after a model is deployed.

For example:

```text
Training behavior:
Customers bought X frequently.

One year later:
Customer behavior changed.
```

The relationship between features and target may change.

This can reduce model performance.

---

## Types of Drift

Important concepts include:

### Data Drift

The distribution of input features changes.

### Concept Drift

The relationship between inputs and target changes.

### Prediction Drift

The distribution of model predictions changes.

Monitoring helps detect these changes.

---

# 6.19 Monitoring

After deployment, we should monitor:

- Model performance
- Data quality
- Data drift
- Prediction distributions
- Latency
- Errors
- Resource usage
- Business KPIs

ML is therefore not finished when the model is deployed.

---

# 6.20 Reproducibility

An ML experiment should ideally be reproducible.

Important things to track include:

- Dataset version
- Code version
- Model version
- Hyperparameters
- Random seeds where relevant
- Environment/dependencies
- Evaluation metrics

Tools such as Git and experiment tracking systems can help.

---

# 6.21 Security and Privacy

ML systems may process sensitive or valuable data.

Important concerns include:

- Unauthorized access
- Data exposure
- Adversarial inputs
- Data poisoning
- Model theft
- Privacy violations

Security should be considered throughout the ML lifecycle.

---

# 6.22 Human and Business Challenges

Technical performance is not the only concern.

A model can have excellent metrics but still fail as a product.

Reasons include:

- Users do not trust it.
- Predictions are difficult to explain.
- It does not fit the business workflow.
- The cost is higher than the benefit.
- It solves the wrong problem.
- Stakeholders cannot act on predictions.

---

# 7. Machine Learning Development Life Cycle (MLDLC)

## Definition

The **Machine Learning Development Life Cycle (MLDLC)** is the end-to-end process of developing, deploying, monitoring, and improving an ML system.

A simplified lifecycle is:

```text
Problem Definition
       ↓
Data Collection
       ↓
Data Understanding
       ↓
Data Cleaning & Preparation
       ↓
Feature Engineering
       ↓
Model Selection
       ↓
Model Training
       ↓
Evaluation
       ↓
Deployment
       ↓
Monitoring
       ↓
Retraining / Improvement
       ↺
```

The process is **iterative**, not strictly linear.

---

# 7.1 Step 1 — Problem Definition

First understand the business or real-world problem.

Questions:

- What are we trying to predict?
- What is the unit of prediction?
- What is the target variable?
- What decisions will the prediction support?
- What is the success metric?
- What constraints exist?

---

## Example

Business problem:

> An e-commerce company wants to reduce customer churn.

ML problem:

> Predict whether each active customer will churn within the next 30 days.

Now we have:

```text
Input → Customer information
Target → Churn / No Churn
```

---

# 7.2 Step 2 — Data Collection

Collect relevant data.

Sources may include:

```text
Database
API
CSV
Application logs
Cloud storage
Sensors
External datasets
```

For churn prediction, we might collect:

- Customer age
- Subscription type
- Number of purchases
- Last purchase date
- Customer support interactions
- Total spending
- Login frequency

---

# 7.3 Step 3 — Data Understanding / Exploration

Before modeling, understand the dataset.

Use:

- Descriptive statistics
- Distribution analysis
- Correlation analysis
- Missing-value analysis
- Outlier analysis
- Class distribution
- Visualization

Typical Python tools:

```python
import pandas as pd
import matplotlib.pyplot as plt
```

Questions:

```text
How many rows?
How many columns?
Which columns are missing values?
Are there duplicates?
What is the target distribution?
Are there suspicious values?
```

---

# 7.4 Step 4 — Data Cleaning and Preparation

Clean the data.

Tasks may include:

```text
Handle missing values
Remove duplicates
Fix incorrect values
Standardize formats
Encode categorical variables
Scale features when appropriate
```

Example:

```text
Gender

Male
M
male
MALE
```

could be standardized into:

```text
Male
```

---

# 7.5 Step 5 — Feature Engineering

Transform raw information into useful features.

Example:

```text
Last Purchase Date
Current Date
        ↓
Days Since Last Purchase
```

Another:

```text
Total Spending
Number of Orders
        ↓
Average Order Value
```

Feature engineering can help the model discover useful signals.

---

# 7.6 Step 6 — Train/Validation/Test Split

Separate data appropriately.

```text
Training → Learn
Validation → Tune/select
Test → Final evaluation
```

For time-dependent problems, random splitting may not always be appropriate. A chronological split can better simulate future prediction.

---

# 7.7 Step 7 — Model Selection

Choose candidate algorithms.

For a classification problem:

```text
Logistic Regression
Decision Tree
Random Forest
Gradient Boosting
Neural Network
```

Start with a sensible baseline.

Do not automatically choose the most complicated model.

---

# 7.8 Step 8 — Model Training

The model learns parameters from the training data.

Conceptually:

```text
Training Data
      ↓
Algorithm
      ↓
Parameters
      ↓
Trained Model
```

Example:

```python
model.fit(X_train, y_train)
```

---

# 7.9 Step 9 — Evaluation

Evaluate on validation/test data using appropriate metrics.

For an imbalanced churn problem, accuracy alone may be misleading.

For example:

```text
95% customers → No Churn
5% customers  → Churn
```

A model predicting:

```text
No Churn for everyone
```

would achieve 95% accuracy while detecting zero churners.

This is why metrics such as:

```text
Precision
Recall
F1-score
PR-AUC
```

may be more informative depending on the business objective.

---

# 7.10 Step 10 — Hyperparameter Tuning

**Hyperparameters** are settings chosen outside the normal parameter-learning process.

Examples:

- Tree depth
- Learning rate
- Number of trees
- Regularization strength
- Number of neighbors in KNN

Common approaches:

- Grid Search
- Random Search
- Bayesian Optimization

---

# 7.11 Step 11 — Deployment

Once the model meets requirements, deploy it.

Possible deployment patterns:

### Batch Prediction

```text
Every night
    ↓
Predict all customers
    ↓
Save results
```

### Real-Time Prediction

```text
User Request
    ↓
API
    ↓
Model
    ↓
Prediction
    ↓
Application
```

---

# 7.12 Step 12 — Monitoring

After deployment, monitor the system.

### Technical monitoring

- Latency
- Error rate
- CPU/GPU usage
- Memory
- Throughput

### Data monitoring

- Missing values
- Unexpected values
- Feature distributions
- Data drift

### Model monitoring

- Prediction distribution
- Performance when labels become available
- Drift

### Business monitoring

- Revenue
- Conversion
- Churn
- Fraud loss
- User engagement

---

# 7.13 Step 13 — Retraining

If performance declines or the data changes, retrain the model.

```text
Monitor
   ↓
Performance decreases
   ↓
Collect recent data
   ↓
Retrain
   ↓
Evaluate
   ↓
Deploy new version
```

This creates a continuous improvement loop.

---

# 7.14 MLDLC Is Iterative

One of the most important concepts is:

> **You do not necessarily move through the lifecycle exactly once.**

For example:

```text
Model evaluation
      ↓
Performance poor
      ↓
Investigate data
      ↓
Feature engineering
      ↓
Train again
      ↓
Evaluate again
```

You may repeatedly move backward and forward between stages.

---

# 8. A Complete ML Project Example

Let's combine everything using a **Customer Churn Prediction System**.

---

## Business Problem

An online subscription company loses customers and wants to identify customers who are likely to leave.

---

## Step 1 — Define the Problem

Target:

```text
Will the customer churn within 30 days?
```

Type:

```text
Binary Classification
```

---

## Step 2 — Collect Data

Possible features:

```text
Customer age
Subscription plan
Monthly spending
Number of logins
Number of purchases
Support tickets
Days since last login
```

Target:

```text
Churn = 1
No Churn = 0
```

---

## Step 3 — Explore

We discover:

```text
Missing values
Duplicate customers
Imbalanced target
Outliers
Categorical columns
```

---

## Step 4 — Clean

We:

```text
Handle missing values
Remove/fix duplicates
Correct invalid values
Encode categorical features
```

---

## Step 5 — Feature Engineering

Create:

```text
Days Since Last Login
Average Monthly Spending
Purchases Per Month
Support Tickets Per Month
```

---

## Step 6 — Split

```text
Training
Validation
Test
```

For time-based churn prediction, we should consider a chronological split if it better represents deployment.

---

## Step 7 — Baseline

Train:

```text
Logistic Regression
```

Then compare against:

```text
Random Forest
Gradient Boosting
```

---

## Step 8 — Evaluate

Suppose:

```text
Accuracy = 89%
Precision = 71%
Recall = 78%
F1 = 74%
```

We then ask:

> Are these metrics good enough for the business?

The answer depends on the cost of false positives and false negatives.

---

## Step 9 — Deploy

The company could use:

```text
Customer data
      ↓
Churn model
      ↓
Churn probability
      ↓
CRM system
      ↓
Retention team
```

---

## Step 10 — Monitor

Monitor:

```text
Prediction distribution
Data quality
Data drift
Actual churn rate
Model performance
Business impact
```

---

## Step 11 — Retrain

Every month or when monitoring indicates a need:

```text
New data
   ↓
Retrain
   ↓
Evaluate
   ↓
Deploy if appropriate
```

---

# 9. Important Fundamental Concepts

These concepts are essential for understanding almost every ML project.

---

# 9.1 Features

A **feature** is an input variable used by a model.

Example:

```text
House Price Prediction

Area
Bedrooms
Location
Age of house
```

These are features.

Usually represented as:

```text
X
```

---

# 9.2 Target / Label

The value the model is trying to predict.

Examples:

```text
House Price
Churn
Fraud
Spam
```

Usually represented as:

```text
y
```

---

# 9.3 Training

The process through which an ML algorithm learns from training data.

```text
X_train + y_train
        ↓
     Algorithm
        ↓
   Trained Model
```

---

# 9.4 Prediction / Inference

Using a trained model on new data.

```text
New X
 ↓
Model
 ↓
Prediction
```

---

# 9.5 Parameter

A value learned by the model during training.

For linear regression:

```text
y = w1x1 + w2x2 + b
```

The model learns:

```text
w1
w2
b
```

These are parameters.

---

# 9.6 Hyperparameter

A configuration chosen by the developer or tuning process rather than learned directly as ordinary model parameters.

Examples:

```text
Learning rate
Tree depth
Number of trees
K in KNN
Regularization strength
```

---

# 9.7 Loss Function

A function that measures how far model predictions are from desired targets during training.

Example:

```text
Actual = 100
Predicted = 90

Error = 10
```

The exact loss function depends on the problem.

Examples:

- Mean Squared Error
- Mean Absolute Error
- Cross-Entropy Loss

Training often aims to minimize an objective related to this loss.

---

# 9.8 Training Error vs Generalization Error

### Training Error

Error on examples used to train the model.

### Generalization Error

Error on unseen data from the relevant target distribution.

A model with extremely low training error can still have high generalization error.

This is closely related to overfitting.

---

# 9.9 Bias-Variance Intuition

A useful conceptual framework:

### High Bias

Model is too simple.

```text
Underfitting
```

### High Variance

Model is too sensitive to the training data.

```text
Overfitting
```

The goal is to achieve good generalization rather than simply minimizing training error.

---

# 9.10 Data Distribution

A dataset can be thought of as coming from some underlying distribution.

For example:

```text
P(X)
```

describes the distribution of inputs.

The relationship between inputs and targets can be represented conceptually as:

```text
P(y | X)
```

ML systems often assume that future data is sufficiently related to the data used for development.

When this assumption breaks, performance may degrade.

---

# 10. Common ML Terms

| Term | Meaning |
|---|---|
| Dataset | Collection of data |
| Feature | Input variable |
| Target | Value being predicted |
| Label | Known target/category |
| Sample | One observation/record |
| Training | Learning from training data |
| Inference | Generating predictions |
| Parameter | Value learned during training |
| Hyperparameter | Configuration selected outside ordinary parameter learning |
| Model | Learned representation/relationship used for prediction |
| Epoch | One full pass through training data, common in iterative neural-network training |
| Batch | Group of training examples processed together |
| Overfitting | Model fits training data too closely |
| Underfitting | Model fails to learn enough useful structure |
| Generalization | Performance on unseen relevant data |
| Feature Engineering | Creating useful features |
| Data Leakage | Unfair information entering model development |
| Data Drift | Change in input data distribution |
| Concept Drift | Change in relationship between inputs and target |
| Deployment | Making a model available for real-world use |
| Monitoring | Observing model/system behavior after deployment |
| Retraining | Training a new model using newer or improved data |

---

# 11. Interview Revision

## Q1. What is supervised learning?

Supervised learning is an ML approach where a model learns from labeled examples containing inputs and corresponding targets. It is commonly used for regression and classification.

---

## Q2. What is unsupervised learning?

Unsupervised learning works primarily with unlabeled data and attempts to discover hidden structures, groups, or relationships. Clustering and dimensionality reduction are common examples.

---

## Q3. What is semi-supervised learning?

Semi-supervised learning combines a smaller labeled dataset with a larger unlabeled dataset. It is useful when obtaining labels is expensive but unlabeled data is abundant.

---

## Q4. What is reinforcement learning?

Reinforcement learning involves an agent interacting with an environment, taking actions, receiving rewards or penalties, and learning a policy that improves long-term outcomes.

---

## Q5. What is batch learning?

Batch learning trains or retrains a model using accumulated data periodically rather than continuously updating it with every incoming example.

---

## Q6. What is online learning?

Online learning updates a model incrementally as new data becomes available, making it useful for continuously changing data streams.

---

## Q7. What is instance-based learning?

Instance-based learning makes predictions by relying heavily on similarities between new observations and stored training examples. KNN is a classic example.

---

## Q8. What is model-based learning?

Model-based learning learns a general model or relationship from training data and uses that learned model to make predictions on new data.

---

## Q9. What is overfitting?

Overfitting occurs when a model learns the training data too closely, including noise or accidental patterns, causing poor performance on unseen data.

---

## Q10. What is underfitting?

Underfitting occurs when a model is too simple to capture important patterns in the data, resulting in poor performance even on training data.

---

## Q11. Why is data quality important in ML?

ML models learn from data. Missing, incorrect, biased, inconsistent, or mislabeled data can cause the model to learn incorrect patterns and reduce real-world performance.

---

## Q12. What is data leakage?

Data leakage occurs when information that should not be available at prediction time is used during training or evaluation. It can produce unrealistically good metrics.

---

## Q13. Why can't we simply use accuracy for every classification problem?

Accuracy can be misleading when classes are imbalanced or when the costs of different errors are different. Metrics such as precision, recall, F1-score, and PR-AUC may be more appropriate depending on the problem.

---

## Q14. Is model deployment the end of ML development?

No. A production ML system must usually be monitored for data quality, drift, performance, latency, failures, and business impact. Retraining or other improvements may be required.

---

## Q15. What is the difference between a parameter and a hyperparameter?

A **parameter** is learned by the model during training, while a **hyperparameter** is a configuration chosen outside ordinary parameter learning.

Example:

```text
Linear regression coefficient → Parameter
Learning rate → Hyperparameter
```

---

# 12. Key Takeaways

## The Three Classification Dimensions

Remember these three questions:

### 1. How much supervision?

```text
Supervised
Unsupervised
Semi-Supervised
Reinforcement Learning
```

### 2. How is the model updated?

```text
Batch Learning
Online Learning
```

### 3. How does the model learn from examples?

```text
Instance-Based
Model-Based
```

---

# The Big Picture

A machine learning system is much bigger than an algorithm.

Think:

```text
                 MACHINE LEARNING
                        │
        ┌───────────────┼────────────────┐
        │               │                │
   Data Problem      Learning          System
        │               │                │
   Collection       Algorithm        Deployment
   Cleaning         Training         Monitoring
   Features         Evaluation       Retraining
   Labels           Tuning           Cost
   Quality          Generalization  Reliability
```

---

# The Most Important Mental Model

When starting an ML project, ask:

```text
1. What problem am I solving?
             ↓
2. What data do I need?
             ↓
3. Is the data reliable and representative?
             ↓
4. What exactly am I predicting?
             ↓
5. How should I prepare the data?
             ↓
6. What baseline should I build?
             ↓
7. Which model is appropriate?
             ↓
8. How will I evaluate it?
             ↓
9. Does it generalize?
             ↓
10. Can I deploy it reliably?
             ↓
11. How will I monitor it?
             ↓
12. When and why should I retrain?
```

> **Core principle:** In real-world Machine Learning, the model is only one component. Data quality, problem definition, evaluation, deployment, monitoring, cost, and continuous improvement are equally important.

---

## One-Line Memory Tricks

| Concept | Memory Trick |
|---|---|
| Supervised | **Learn with answers** |
| Unsupervised | **Find patterns without answers** |
| Semi-Supervised | **Few answers + lots of unlabeled data** |
| Reinforcement | **Learn through rewards and penalties** |
| Batch | **Learn periodically from accumulated data** |
| Online | **Learn incrementally from incoming data** |
| Instance-Based | **Remember examples and compare** |
| Model-Based | **Learn a general model** |
| Overfitting | **Memorizes too much** |
| Underfitting | **Learns too little** |
| Generalization | **Works on unseen data** |
| Data Leakage | **Cheating with future/unavailable information** |
| Drift | **The world/data changes** |
| MLDLC | **Problem → Data → Model → Deploy → Monitor → Improve** |

---

# Final Mental Picture

```text
                    MACHINE LEARNING
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
   SUPERVISION         MODEL UPDATE      LEARNING STYLE
         │                 │                 │
   ┌─────┼─────┐       ┌───┴────┐       ┌───┴────┐
   │     │     │       │        │       │        │
Supervised  Unsupervised  Batch   Online  Instance  Model
   │
Semi-Supervised
   │
Reinforcement Learning


              ML DEVELOPMENT LIFECYCLE

Problem Definition
       ↓
Data Collection
       ↓
Data Understanding
       ↓
Data Cleaning
       ↓
Feature Engineering
       ↓
Train / Validation / Test
       ↓
Model Selection
       ↓
Training
       ↓
Evaluation
       ↓
Tuning
       ↓
Deployment
       ↓
Monitoring
       ↓
Retraining
       ↺
```

This is the foundation on which more advanced topics such as **statistics, feature engineering, classical ML algorithms, deep learning, MLOps, and generative AI** are built.
