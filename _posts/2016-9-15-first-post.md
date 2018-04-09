---
layout: post
title: "Behavior based anomaly detection in an Enterprise Network"
---

Network behavioral modeling is a popular approach for attack detection. A novel architecture and implementation of monitoring and securing an Enterprise Network is presented. The implementation  utilizes “Behavior based Anomaly Detection” approach for classifying malicious network traffic from benign network traffic. K-Means clustering algorithm is used for the classification. The unsupervised machine learning model uses features extracted from network flows which evolves over the period of time by observing the network behavior, hence improving the clustering accuracy. More features can be extracted from the flow to make the unsupervised ML model more robust.  Extracting features from these statistics and analyzing the traffic flows in a machine learning based algorithm can enhance the view of the network. From these statistics, one can also infer whether the network is being attacked or it is malfunctioning. 

The proposed solution uses a single-linkage hierarchical clustering method to cluster data from the KDD99 dataset, based on the standard Euclidean distance for inter-patterns similarity. Results are improved  with the same dataset, using three different clustering algorithms: Fixed-Width clustering, an optimized version of K-Nearest Neighbor (KNN), and one class Support Vector Machine (SVM).

Our novel architecture has modules that support scalability by making use of parallelism.
* The unsupervised ML algorithm used in the implementation dynamically evolves by observing the network behavior, which reduces the periodic manual reconfigurations. 
* We have proposed and implemented a real-time classification and analyzing software unlike other solutions that work only on offline datasets. 

## PROPOSED ARCHITECTURE

In an enterprise network, the regulation of traffic is achieved by means of a network firewall. It can be placed where traffic from the Internet enters or exits. It also regulates access to the De-Militarized Zone (DMZ) servers of the enterprise. This firewall can be split in two logical entities having separate policy sets, based on the type of network traffic they handle. The ingress traffic for the enterprise network is handled by “External Firewall” entity. On the other hand, the egress traffic is handled by “Internal Firewall” entity. The policies that control these accesses are decided by the Network Administrators. Fig. 1 depicts the proposed architecture as a mesh of logical connections between such multiple network entities. It is a simplified representation of a complex enterprise network.

![en1](https://user-images.githubusercontent.com/25291535/38505611-3c02b6aa-3c35-11e8-97c0-eacbe793f5fb.JPG)

**Functional Aspects: **

The architecture presents a concept of “Smart Router” and “Analyzer Agent”.

* **Smart Router:** As shown in Fig. 1, “Smart Router” is a device in the DMZ. It has a sub-sampling software for analyzing the incoming/outgoing packets. It contacts the “Database” for taking the decision on whether to allow or drop the packets. The “Database” maintains lists of allowed and blocked hosts, defined in Whitelist and Blacklist, respectively. These lists contain IP addresses and a timeout period associated with hosts. An entry in such list is valid till timeout occurs. This ensures smooth network access for benign users while not assuming that they will remain so, forever.
 
 * While analyzing a packet, “Smart Router” tries to find matching entries in the “Database”. If answer to such query is positive, it will either drop the packet and/or reset the connection. If the “Smart Router” is unable to find any matching entry in the “Database”, it will redirect the packets towards appropriate agent(s) for further inspection. The “Database” gets updated every time a malicious activity is detected. Hence, over the course of time, the whole network becomes self updating, eliminating the need for periodic manual reconfigurations.
 
* **Analyzer Agent:** The “Smart Router” takes the initial decision with the help of existing “Database” and IDS. If it is not able to take routing decision after consulting the “Database”, it will redirect the packets towards the “Analyzer Agent”. They are computational units running machine learning based clustering algorithms. Based on the type of data to be analyzed, the algorithms are distributed amongst the “Analyzer Agent”. It forms flows from the received packets. Analysis of such flow can have following responses from the 'Analyzer Agent'.

**True Positive Response:**

  * It means that the redirected flow was malicious.
  * “Analyzer Agent” indicates to the “Smart Router” that it has to drop the packets associated with the flow and terminate the             connection.
  * It then logs this instance in the “Database”.
  *  The IP address, from which the malicious flow originated, is flagged as suspicious and all future traffic from the same IP address     is redirected to the “Analyzer Agent”.

**False Positive Response:**

  * It means that the redirected flow was not malicious.
  * After manual inspection, such instances are logged in “Database” to prevent similar requests from arising in future.
  
**Ambiguous Response:**
 
  * It means that the “Analyzer Agent” was unable to identify the flow as either True or False Positive.
  * The policy for handling such flows can be defined by the network administrators.
  
While examining packets in the flow, the “Analyzer Agent” utilizes information present in the header part of the packets. Here, it does not look at the payload part of the packet but the characteristics of the payload such as the size, number of fragments etc. 

## Process Flowcharts

Implementation of the proposed architecture has two processes that run in parallel which shows the flow of packet in the core part of the “Smart Router”. The IP packet enters the queue managed by NFQUEUE module. The packets arriving in this queue can be taken to the “User-Space” by binding to the queue. The “Analyzer Agent” terminates in case queue is not present. After successfully binding to the queue, a callback function is registered with the Kernel. This function is called everytime a packet arrives in the queue. Management of functionalities, that require meeting timing requirements, is handled by the timers.

Each RAW packet in the queue is converted into a “Packet Object” and matched for Blacklisted IPs in the Database. If a match is found, the packet verdict is set as “DROP” and the Kernel drops the packet. The “Packet Object” goes to a “Packet to Flow Converter” for conversion into “Flow Objects” if no match is found. A “Flow Manager” module manages the sub-sampled network flows in three parallel python dictionary objects, wherein it monitors currently running flows, future flows and finished flows. The time management module handles the interaction between these flow threads. A duration specified by “FLOWDURATION” is used to terminate the sub-sampled flows. The starting time, when the first packet is seen between an IP pair, is noted for each sub-sampled flow. This flow is then observed for the FLOWDURATION period. After that, it is moved to finished flows python dictionary object. Here, it gets analyzed for malicious activity by the behavior based machine learning algorithm. We have used K-Means clustering algorithm for clustering the sub-sampled flows in normal and anomalous flows. If the flow is identified as anomalous, then the packet verdict is set as “DROP” for all the packets belonging to the anomalous flow. The IP addresses of the flow get updated in the “Blacklist IP” database along with a blocking timeout. (For repeatedly offending hosts, this duration increases linearly/exponentially depending on the “Smart Router” settings.)
If the flow is identified as normal, then the packet verdict is set as “ACCEPT” for all the packets belonging to the that flow. These decisions stay in effect for the blocking timeout duration for the flows enqueued in the future flow python dictionary object. 

Fig. 4 shows the flow chart for the RAW network traffic capture process. The packets are initially logged in the Random Access Memory (RAM) of the “Smart Router” to avoid Kernel packet drops.  

![4](https://user-images.githubusercontent.com/25291535/38508126-edf89068-3c3b-11e8-99b6-ab26b192ac01.png)

## Algorithms

The implementation of the “Smart Router” uses conceptsof “Packet Object” and “Flow Object”. Algorithm 1 describes the logic used for generating a “Packet Object” from the RAW packet in the NFQUEUE. 

![algo 1](https://user-images.githubusercontent.com/25291535/38508201-1ff83406-3c3c-11e8-835e-6083e2ad4f8b.png)

Algorithm 2 describes the logic used for generating a Flow Object from a set of Packet Objects in memory. Each flow is identified by <Src.IP, Dst.IP, Protocol>. All “Packet Objects” corresponding to these criteria are gathered and sorted according to the timestamp. “Flow Objects” are created on the basis of criteria along with a FLOWDURATION value. For TCP sessions, in addition to FLOWDURATION criteria, a flow is initiated only after the regular TCP threeway handshake has been established. The termination criteria for a TCP flow is met either by FLOWDURATION or the TCP close connection sequence (in terms of FIN packets or RST packets), whichever is encountered first. In case of UDPs (virtual) flows, only the FLOWDURATION can be employed. FLOWDURATION is a “tunable” parameter, decided by a network administrator based on the understanding of the network.

![algo2](https://user-images.githubusercontent.com/25291535/38508249-4b5ebc14-3c3c-11e8-94da-4bc61d2d6ad8.png)

In the implemented solution, we have used the K-Means clustering ML algorithm for behavior based anomaly detection. It involves following steps.
1) Initialize K Centroids Randomly.
2) Data Assignment Step: Assign each flow object to the group that has the closest centroid.
  
  *  If ci is the collection of centroids in set C, then each data point x is assigned to a cluster based on 
     
     ![eq1](https://user-images.githubusercontent.com/25291535/38508369-a73d2930-3c3c-11e8-99c6-557fd6767731.png)

     
     where dist() is the standard (L2) Euclidean Distance.
     
   * Let the set of data point assignments for each ith cluster centroid be Si.
 
 3) Centroid Update Step: When all the flow objects have been assigned, recalculate the positions of the K centroids.
   
   * This is done by taking the mean of all data points assigned to that centroid’s cluster.
   
   


