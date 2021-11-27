---
title: 'Review on AD Sensor Attack'
date: 2021-11-15
permalink: /posts/2021/11/Security/
tags:
  - autonomous driving
  - security
  - sensing
---
*This post summarizes my understanding of autonomous driving security with a focus on sensor attack.*

What makes AD vulnerable?
===
Compared to conventional vehicles, AD has to communicate with other vehicles and infrastructure with external networks and thus may become a channel for attack. More than that, for AD, vast information has to be processed particularly by DNNs, which are prone to adversarial attacks and the probability of false positives increases with the data size. As the industry is still at the infancy and it is difficult to make AD reliable in all conditions and also they are prone to bugs, which also can be exploited by the attackers.

Background 
===
People have started taking autonomous vehicles to trials and moving towards the plan for larger deployment. However, security is an important consideration. There has been an increasing interest in the attacks and defences as ADs are getting ready for roads. 

Roughly, AD action is determined by the workflow: Perception+Localization->Prediction->Planning->Control. Among these, I think Perception+Localization->Prediction could be seen as the soft point. Because data-driven Perception and Localization highly rely potentially deceiving information gained in real-time. 
 Below I first introduce the three modules before presenting AD vulnerabilities.

**Perception:** processes LiDAR, camera, and radar inputs to detect obstacles. Usually multiple object detection is adopted to avoid false detection.

**Localization**: accepts location measurements from GPS and LiDAR. 

**Prediction**: estimates the future trajectory of the detected obstacles. Neural networks (e.g., MLPs and RNNs) are commonly used to evaluate the probabilities of the possible trajectories.


#Attack on Sensors
Though many attacks make AD produce wrong decisions, sensor attack is the most direct and easy-to-deploy threat. In this section, I am to organize and discuss several methods of sensor attack and propose some areas that have yet to be discovered.

### Methodology
Intuitively, to cause impactful security consequence, the attack goal is usually set as fooling the one or multiple sensors into perceiving fake obstacles in front of a victim AD in order to maliciously alter its driving decisions (Prediction). Therefore, a spoofing model is considered [^lidar42] [^lidar44]. 

Generally, the spoofing model can be head-to-head (attacker in proximity) or an ambush (deceiving obstacle). For the first one, Shin et al. [^lidar44] deploy a relay attack to achieve spoofing that induces illusion (saturation). Prior to this, Petit et al [^black] first injected fake dots to form a virtual wall but with farther distance. However, these attacks are disabled by ML models. Cao et al[^lidar] explored an optimization method to inject faked laser (perturbation) to the victim LiDAR. 

More recently, researchers have come up with an ambush attack (physical attack vector). Sato et al. proposed a new model that attackers should only modify the road surface to make the target car deviate from its current route[^drp] (lateral deviation). This method utilized vehicle motion model and perspective transformation to cast lasting impact on the steering control, i.e., influencing a series of frames. Quite like [^lidar], they set out from the designer's perspective to reveal the security threat by figuring out the lane-bending objective function. Likewise, a multi-sensor fusion attack (camera+LiDAR) was proposed with the same principle, which makes the vehicle ignores an apparent obstacle. This attack is achievable because it optimizes the obstacle's texture to make it "look like" part of the road surface in both LiDAR and camera (attack vector fusion).

### Discussion
1. The physical attack vector idea is brilliant and made feasible by the overfitting nature of DNN. But one of the shortcomings is that operation logic (after perception), e.g., curve parameters in DRP and MSF in Fusion Attack, is still required for iterative optimization. Most manufacturers might have different logic, which might lead to terrible transferability. What if we can scale it to attack on **localization module**?  Though LiDAR spoofing is discussed in perception module, researchers have limited projects done involved with localization. In [^opt], researchers propose a way of interrupting LiDAR localization. However, it is highly feasible that we can fool the localization module into a different map. For example, creating a few meters of drift to force the car make a turn to the wrong block. Nonetheless, there are multiple challenges and maybe the deployment cost is unaffordable...

2. Stationary obstacle is less impactful to the planning module than moving objects. The LiDAR spoofing gave me an idea that maybe moving objects can also be mimicked. I think via the same optimization framework, attackers can also mirror walking pedestrian (VRU)'s echo. However, I consider this type of man-in-the-middle attack fails in some certain situations. For example, beam of a specific LiDAR is unique.


Other Threats
===
Upon summarizing sensor attack on driver's security, I found other aspects interesting.
###Attack on GPS
This is an old topic born far before AV. GPS is facing two threats, jamming and spoofing. Specifically, spoofing is more technical. It falls into two types, relay type and generating type. The former records actual signals from satellites and transmits to the target. The latter generates GPS signals according to real signals via certain programs. Generated signals cannot be distinguished as encryption algorithm and the test matrix algorithm. He et al.[^uav] gave an experiment of GPS spoofing. They input fake location via a program to generate fake signals and then broadcast with USRP, invalidating GPS functions. Thus, an attacker can control a car as he wants with the above technology.

###Attack on Prediction module-DL Backdoor Attack
Backdoor attack on image and speech recognition had been a heated topic[^target]. However, no one has yet taken it as an attack vector for AD. 

Prediction module estimates the future trajectory of the detected obstacles by neural networks (e.g., RNNs). What if the training set is contaminated? Wenger et al. have considered backdoor attacks in real physical world[^backdoor]. This threat could be a serious issue if trigger is carefully designed to escape tests. Unlike aforementioned "invisible obstacle", this threat is stealthier as attackers don't even have to make up something that might be suspicious. For another, as inputs for Prediction have no visual characteristics, it could be harder to detect without knowing planning decision consequences. 

Nevertheless, challenges could be:  1. attackers might have no access to the model and the training set used by the victim system; 2. only a small amount of poisoning samples are allowed and 3. Planning unit is transparent, etc.

### Privacy Leakage
Privacy is a serious problem all the time. How to protect everyone's privacy is important. It could threaten both driver and VRU. For the VRU, the sensors are oversensing their behaviors. For example, cars could go in the restricted area and secretly photographing privacy of legal residents. Besides the common camera case, it is more concerning that there are stealthier ones. For example, WaveSpy [^wavespy] can achieve through-wall screen reading via mmWave, not to mention those eavesdropping attacks with mmWave. 

On the flipside, the drivers are not spared. With Internet access, cars can share information with service providers in the cloud,  from which we can get to the classic debate on permission models[^spy]. Furthermore, the emitted signal can potentially sell out their location...






[^lidar42]:Jonathan Petit, Bas Stottelaar, Michael Feiri, and Frank Kargl. 2015. Remote Attacks on Automated Vehicles Sensors: Experiments on Camera and LiDAR. In Black Hat Europe.
[^lidar44]: Hocheol Shin, Dohyun Kim, Yujin Kwon, and Yongdae Kim. 2017. Illusion and Dazzle: Adversarial Optical Channel Exploits Against Lidars for Automotive Applications. In International Conference on Cryptographic Hardware and Embedded Systems (CHES)
[^lidar]: Cao, Yulong, et al. "Adversarial sensor attack on lidar-based perception in autonomous driving." Proceedings of the 2019 ACM SIGSAC conference on computer and communications security. 2019.
[^drp]: Sato, Takami, et al. "Dirty Road Can Attack: Security of Deep Learning based Automated Lane Centering under Physical-World Attack." Proceedings of the 29th USENIX Security Symposium (USENIX Security'21). 2021.
[^fusion]: Cao*, Yulong, Ningfei Wang*, Chaowei Xiao*, Dawei Yang*, Jin Fang, Ruigang Yang, Qi Alfred Chen, Mingyan Liu, and Bo Li. 2021\. “Invisible for Both Camera and LiDAR: Security of Multi-Sensor Fusion Based Perception in Autonomous Driving Under Physical-World Attacks.” *2021 IEEE Symposium on Security and Privacy (SP)*, May, 176–94\.
[^black]: Petit, Jonathan, et al. "Remote attacks on automated vehicles sensors: Experiments on camera and lidar." Black Hat Europe 11.2015 (2015): 995.
[^towards_secure]: Sun, Jiachen, Yulong Cao, Qi Alfred Chen, and Z. Morley Mao. 2020\. “Towards Robust LiDAR-Based Perception in Autonomous Driving: General Black-Box Adversarial Sensor Attack and Countermeasures.” *ArXiv:2006.16974 [Cs]*, June. 
[^comprehensive]: Garcia, Joshua, Yang Feng, Junjie Shen, Sumaya Almanee, Yuan Xia, and and Qi Alfred Chen. 2020\. “A Comprehensive Study of Autonomous Vehicle Bugs.” In *Proceedings of the ACM/IEEE 42nd International Conference on Software Engineering*, 385–96\. Seoul South Korea: ACM. 
[^wavespy]: Li, Zhengxiong, et al. "Wavespy: Remote and through-wall screen attack via mmwave sensing." 2020 IEEE Symposium on Security and Privacy (SP). IEEE, 2020.
[^backdoor]:Yao, Yuanshun, et al. "Latent backdoor attacks on deep neural networks." Proceedings of the 2019 ACM SIGSAC Conference on Computer and Communications Security. 2019.
[^target]: Chen X, Liu C, Li B, et al. Targeted backdoor attacks on deep learning systems using data poisoning[J]. arXiv preprint arXiv:1712.05526, 2017.
[^uav]: He D, Du X, Qiao Y, Zhu Y, Fan Q, Luo W (2017) A Survey on Cyber security of Unmanned Aerial Vehicles, Vol.40, Online Publishing No. 125
[^opt]: Davidson, Drew, et al. "Controlling UAVs with sensor input spoofing attacks." 10th {USENIX} Workshop on Offensive Technologies ({WOOT} 16). 2016.
[^spy]: Pesé, Mert D., Xiaoying Pu, and Kang G. Shin. "SPy: Car Steering Reveals Your Trip Route!." Proc. Priv. Enhancing Technol. 2020.2 (2020): 155-174.


