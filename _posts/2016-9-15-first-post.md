---
layout: post
title: Network Monitoring and Security 
---
Index Terms: Anomaly detection, Machine Learning.

## Github repo: 

[Network_security_analyser](https://github.com/sunithan29/Network_security_analyser)

In this project a novel architecture to secure an Enterprise Network was implemented. It utilizes “Behavior based Anomaly Detection” approach for classifying malicious network traffic from benign network traffic. The unsupervised machine learning model uses featuresextracted from network flows which evolves over the period of time by observing the network behavior, hence improving the clustering accuracy. K-Means clustering algorithm is used for the above classification.

* The proposed solution falls within the unsupervised attacks detection domain. The vast majority of the unsupervised detection schemes
proposed in the literature are based on clustering and outliers detection. 
* This implementation uses a single-linkage hierarchical clustering method to cluster data from the KDD99 dataset, based on the standard
Euclidean distance for inter-patterns similarity.
