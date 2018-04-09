---
layout: post
title: "Enterprise Network Security and Monitoring"
---

Network behavioral modeling is a popular approach for attack detection. A novel architecture and implementation of monitoring and securing an Enterprise Network is presented. The implementation  utilizes “Behavior based Anomaly Detection” approach for classifying malicious network traffic from benign network traffic. K-Means clustering algorithm is used for the classification. The unsupervised machine learning model uses features extracted from network flows which evolves over the period of time by observing the network behavior, hence improving the clustering accuracy. More features can be extracted from the flow to make the unsupervised ML model more robust.  Extracting features from these statistics and analyzing the traffic flows in a machine learning based algorithm can enhance the view of the network. From these statistics, one can also infer whether the network is being attacked or it is malfunctioning. 

The proposed solution uses a single-linkage hierarchical clustering method to cluster data from the KDD99 dataset, based on the standard Euclidean distance for inter-patterns similarity. Results are improved  with the same dataset, using three different clustering algorithms: Fixed-Width clustering, an optimized version of K-Nearest Neighbor (KNN), and one class Support Vector Machine (SVM).

Our novel architecture has modules that support scalability by making use of parallelism.
* The unsupervised ML algorithm used in the implementation dynamically evolves by observing the network behavior, which reduces the periodic manual reconfigurations. 
* We have proposed and implemented a real-time classification and analyzing software unlike other solutions that work only on offline datasets. 

![en1](https://user-images.githubusercontent.com/25291535/38505611-3c02b6aa-3c35-11e8-97c0-eacbe793f5fb.JPG)


