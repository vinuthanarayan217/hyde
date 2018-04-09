---
layout: post
title: "Behavior based anomaly detection in an Enterprise Network"
---

Network behavioral modeling is a popular approach for attack detection. A novel architecture and implementation of monitoring and securing an Enterprise Network is presented. The implementation  utilizes “Behavior based Anomaly Detection” approach for classifying malicious network traffic from benign network traffic. K-Means clustering algorithm is used for the classification. The unsupervised machine learning model uses features extracted from network flows which evolves over the period of time by observing the network behavior, hence improving the clustering accuracy. More features can be extracted from the flow to make the unsupervised ML model more robust.  Extracting features from these statistics and analyzing the traffic flows in a machine learning based algorithm can enhance the view of the network. From these statistics, one can also infer whether the network is being attacked or it is malfunctioning. 

The proposed solution uses a single-linkage hierarchical clustering method to cluster data from the KDD99 dataset, based on the standard Euclidean distance for inter-patterns similarity. Results are improved  with the same dataset, using three different clustering algorithms: Fixed-Width clustering, an optimized version of K-Nearest Neighbor (KNN), and one class Support Vector Machine (SVM).

Our novel architecture has modules that support scalability by making use of parallelism.
* The unsupervised ML algorithm used in the implementation dynamically evolves by observing the network behavior, which reduces the periodic manual reconfigurations. 
* We have proposed and implemented a real-time classification and analyzing software unlike other solutions that work only on offline datasets. 

PROPOSED ARCHITECTURE

In an enterprise network, the regulation of traffic is achieved by means of a network firewall. It can be placed where traffic from the Internet enters or exits. It also regulates access to the De-Militarized Zone (DMZ) servers of the enterprise. This firewall can be split in two logical entities having separate policy sets, based on the type of network traffic they handle. The ingress traffic for the enterprise network is handled by “External Firewall” entity. On the other hand, the egress traffic is handled by “Internal Firewall” entity. The policies that control these accesses are decided by the Network Administrators. Fig. 1 depicts the proposed architecture as a mesh of logical connections between such multiple network entities. It is a simplified representation of a complex enterprise network.

![en1](https://user-images.githubusercontent.com/25291535/38505611-3c02b6aa-3c35-11e8-97c0-eacbe793f5fb.JPG)

Functional Aspects: The architecture presents a concept of “Smart Router” and “Analyzer Agent”.

 Smart Router:: As shown in Fig. 1, “Smart Router” is a device in the DMZ. It has a sub-sampling software for analyzing the incoming/outgoing packets. It contacts the “Database” for taking the decision on whether to allow or drop the packets. The “Database” maintains lists of allowed and blocked hosts, defined in Whitelist and Blacklist, respectively. These lists contain IP addresses and a timeout period associated with hosts. An entry in such list is valid till timeout occurs. This ensures smooth network access for benign users while not assuming that they will remain so, forever.
 
 While analyzing a packet, “Smart Router” tries to find matching entries in the “Database”. If answer to such query is positive, it will either drop the packet and/or reset the connection. If the “Smart Router” is unable to find any matching entry in the “Database”, it will redirect the packets towards appropriate agent(s) for further inspection. The “Database” gets updated every time a malicious activity is detected. Hence, over the course of time, the whole network becomes self updating, eliminating the need for periodic manual reconfigurations.
 
 Analyzer Agent:: The “Smart Router” takes the initial decision with the help of existing “Database” and IDS. If it is not able to take routing decision after consulting the “Database”, it will redirect the packets towards the “Analyzer Agent”. They are computational units running machine learning based clustering algorithms. Based on the type of data to be analyzed, the algorithms are distributed amongst the “Analyzer Agent”. It forms flows from the received packets. Analysis of such flow can have following responses from the 'Analyzer Agent'
 
 

