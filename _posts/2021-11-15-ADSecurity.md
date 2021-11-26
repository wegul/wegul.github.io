---
title: 'Review on AD Sensor Attack'
date: 2021-11-15
permalink: /posts/2021/11/Security/
tags:
  - autonomous driving
  - security
  - sensing
---
*This post summarizes my understanding of autonomous driving security.*

Background
===
People have started taking autonomous vehicles to trials and moving towards the plan for larger deployment. However, security is an important consideration. There has been an increasing interest in the attacks and defences as ADs are getting ready for roads. Though many attacks make AD produce wrong decisions, sensor attack is the most direct and easy-to-deploy threat. In this post, I am to organize and discuss several methods of sensor attack and propose some areas that have yet to be discovered.


Sensor Attack
===
### Motivation
Roughly, AV action is determined by the workflow: Perception+Localization->Prediction->Planning->Control. Among these, I think Perception+Localization->Prediction could be seen as the soft point. Because data-driven Perception and Localization highly rely potentially deceiving information gained in real-time. 

 Below I first introduce the three modules before illustrating their vulnerabilities.

**Perception:** processes LiDAR, camera, and radar inputs to detect obstacles. Usually multiple object detection is adopted to avoid false detection.

**Localization**: accepts location measurements from GPS and LiDAR. 

**Prediction**: estimates the future trajectory of the detected obstacles. Neural networks (e.g., MLPs and RNNs) are commonly used to evaluate the probabilities of the possible trajectories.

### Attack Model
Intuitively, to cause impactful security consequence, the attack goal is usually set as fooling the one or multiple sensors into perceiving fake obstacles in front of a victim AV in order to maliciously alter its driving decisions (Prediction). Therefore, a spoofing model is considered [^lidar42] [^lidar44]. 

Generally, the spoofing model can be head-to-head (attacker in proximity) or an ambush (deceiving obstacle). For the first one, Shin et al. [^lidar44] deploy a relay attack to achieve spoofing that induces illusion (saturation). Prior to this, Petit et al [^black] first injected fake dots to form a virtual wall but with farther distance. However, these attacks are disabled by ML models. Cao et al[^lidar] explored an optimization method to inject faked laser (perturbation) to the victim LiDAR. 

More recently, researchers have come up with an ambush attack (physical attack vector). Sato et al. proposed a new model that attackers should only modify the road surface to make the target car deviate from its current route[^drp] (lateral deviation). This method utilized vehicle motion model and perspective transformation to cast lasting impact on the steering control, i.e., influencing a series of frames. Quite like [^lidar], they set out from the designer's perspective to reveal the security threat by figuring out the lane-bending objective function. Likewise, a multi-sensor fusion attack (camera+LiDAR) was proposed with the same principle, which makes the vehicle ignores an apparent obstacle. This attack is achievable because it optimizes the obstacle's texture to make it "look like" part of the road surface in both LiDAR and camera (attack vector fusion).



### Discussion
1. The physical attack vector idea is brilliant and made feasible by the overfitting nature of DNN. However, one of the shortcomings is that operation logic (after perception), e.g., curve parameter and $\theta$ of DNN  in DRP, MSF in Fusion Attack, is still required for iterative optimization. However, most manufacturers might have different logic, which might lead to terrible transferability. What if we can scale it to attack on localization module? To my knowledge, LiDAR-based localization algorithms are not that different among brands. For example, if we can cheat the LiDAR-based localization to render drift, we can have similar attack result. 

2. Stationary obstacle is less impactful to the planning module than moving objects. The LiDAR spoofing gave me an idea that maybe moving objects can also be mimicked. I think via the same optimization framework, attackers can also mirror walking pedestrian (VRU)'s echo. However, I consider this type of man-in-the-middle attack fails in some certain situations. For example, beam of a specific LiDAR is unique.

Other Threats
===
Upon summarizing sensor attack on driver's security, I found other aspects interesting.


### Privacy Leakage
This could threaten both driver and VRU. For the VRU, the sensors are oversensing their behaviors, especially when mmWave can do voice analysis. For the driver, the emitted signal potentially sell out their location. 


### How about Prediction module? -DL Backdoor Attack
Backdoor attack on image and speech recognition had been a heated topic[^target]. However, no one has yet taken it as an attack vector for AD. 

Prediction module estimates the future trajectory of the detected obstacles by neural networks (e.g., RNNs). What if the training set is contaminated? Wenger et al. have considered backdoor attacks in real physical world[^backdoor]. This threat could be a serious issue if trigger is carefully designed to escape tests. Unlike aforementioned "invisible obstacle", this threat is stealthier as attackers don't even have to make up something that might be suspicious. Nevertheless, challenges could be:  1. attackers might have no access to the model and the training set used by the victim system; 2. only a small amount of poisoning samples are allowed; 3. the trigger is hard to notice; etc.




[^lidar42]:Jonathan Petit, Bas Stottelaar, Michael Feiri, and Frank Kargl. 2015. Remote Attacks on Automated Vehicles Sensors: Experiments on Camera and LiDAR. In Black Hat Europe.
[^lidar44]: Hocheol Shin, Dohyun Kim, Yujin Kwon, and Yongdae Kim. 2017. Illusion and Dazzle: Adversarial Optical Channel Exploits Against Lidars for Automotive Applications. In International Conference on Cryptographic Hardware and Embedded Systems (CHES)
[^lidar]: Cao, Yulong, et al. "Adversarial sensor attack on lidar-based perception in autonomous driving." Proceedings of the 2019 ACM SIGSAC conference on computer and communications security. 2019.
[^drp]: Sato, Takami, et al. "Dirty Road Can Attack: Security of Deep Learning based Automated Lane Centering under Physical-World Attack." Proceedings of the 29th USENIX Security Symposium (USENIX Security'21). 2021.
[^fusion]: Cao*, Yulong, Ningfei Wang*, Chaowei Xiao*, Dawei Yang*, Jin Fang, Ruigang Yang, Qi Alfred Chen, Mingyan Liu, and Bo Li. 2021\. “Invisible for Both Camera and LiDAR: Security of Multi-Sensor Fusion Based Perception in Autonomous Driving Under Physical-World Attacks.” *2021 IEEE Symposium on Security and Privacy (SP)*, May, 176–94\.
[^black]: Petit, Jonathan, et al. "Remote attacks on automated vehicles sensors: Experiments on camera and lidar." Black Hat Europe 11.2015 (2015): 995.
[^towards_secure]: Sun, Jiachen, Yulong Cao, Qi Alfred Chen, and Z. Morley Mao. 2020\. “Towards Robust LiDAR-Based Perception in Autonomous Driving: General Black-Box Adversarial Sensor Attack and Countermeasures.” *ArXiv:2006.16974 [Cs]*, June. 
[^comprehensive]: Garcia, Joshua, Yang Feng, Junjie Shen, Sumaya Almanee, Yuan Xia, and and Qi Alfred Chen. 2020\. “A Comprehensive Study of Autonomous Vehicle Bugs.” In *Proceedings of the ACM/IEEE 42nd International Conference on Software Engineering*, 385–96\. Seoul South Korea: ACM. 
[^backdoor]:Yao, Yuanshun, et al. "Latent backdoor attacks on deep neural networks." Proceedings of the 2019 ACM SIGSAC Conference on Computer and Communications Security. 2019.
[^target]: Chen X, Liu C, Li B, et al. Targeted backdoor attacks on deep learning systems using data poisoning[J]. arXiv preprint arXiv:1712.05526, 2017.

