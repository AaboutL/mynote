---
title: ORB-SLAM2 跟踪过程
updated: 2022-06-24 07:50:52Z
created: 2022-06-24 01:24:32Z
---

# ORB-SLAM2 跟踪过程

跟踪分为三种：恒速模型跟踪、关键帧跟踪、重定位跟踪
## 恒速模型跟踪：TrackWithMotionModel
**跟踪步骤**：
1. 更新上一帧的位姿；对于双目或rgbd相机，还会根据深度值生成临时地图点
2. 根据上一帧特征点对应地图点进行投影匹配
3. 优化当前帧位姿
4. 提出地图点中的外点
5. 如果匹配数大于10，认为跟踪成功，返回true
流程图：
```mermaid
flowchart TB
	subgraph TrackWithMotionModel
		direction TB
		1.1[创建ORB特征匹配器matcher]-->
		1.2[更新上一帧的位姿:UpdateLastFrame]-->
		1.3[根据之前估计的速度,用恒速模型得到当前帧的初始位姿]-->
		1.4[清空当前帧的地图点]-->
		1.5[设置特征匹配的搜索半径th,单目为15,双目7]-->
		1.6[用上一帧地图点进行投影匹配SearchByProjection,记录匹配成功数目nmatches]-->
		1.7[如果nmatches<20,清空当前帧地图点,th*2重新SearchByProjection]-->
		1.8[[nmatches<20]]
		1.8-->|Yes| 1.13
		1.8-->|No| 1.9
		1.9[利用3D-2D投影关系,优化当前帧位姿PoseOptimization]-->
		1.10[剔除优化后地图中的外点,记录匹配成功的内点数量nmatchesMap]-->
		1.11[[nmatches>=10?]]
		1.11-->|Yes| 1.12
		1.12[匹配成功]
		1.11-->|No| 1.13
		1.13[匹配失败]
		1.14(结束)
		1.12 & 1.13 --> 1.14
	end
	
	subgraph UpdateLastFrame
		direction TB
		2.1[获得上一帧的参考关键帧]-->
		2.2[获得上一帧相对其参考关键帧的位姿变换]-->
		2.3[计算上一帧在世界坐标系下的位姿]-->
		2.4[[上一帧为关键帧或者是单目]]
		2.4-->|Yes| 2.5
		2.5[return]
		2.4-->|No,上一帧不是关键帧,且为双目/rgbd模式| 2.6
		2.6[得到上一帧中具有有效深度值的特征点]-->
		2.7[按特征点深度从小到大排列,至少添加100个点,用nPoints记录]-->
		2.8[遍历这些特征点]-->
		2.9[[当前特征点在上一帧中没有对应的地图点或者有地图点,但是一直没被观测到]]
		2.9-->|Yes| 2.11
		2.9-->|No| 2.10
		2.10[记录有效地图点nPoints++]
		2.11[创建临时地图点,加入到上一帧地图点中]-->2.10
		2.13[[地图点深度值超过设定的阈值且nPoints超过了100个点]]
		2.14(结束)
		2.10 --> 2.13
		2.13-->|Yes| 2.14
		2.13-->|No| 2.8
		2.5-->2.14
	end
1.2-.->UpdateLastFrame
```

## 关键帧跟踪：TrackReferenceKeyFrame
关键帧跟踪的步骤：
1. 将当前普通帧的描述子转化为BoW向量
2. 通过BoW加速当前帧与参考关键帧间的特征点匹配
3. 将上一帧的位姿作为当前帧位姿的初始值
4. 通过优化3D-2D的重投影误差来获得位姿
5. 剔除优化后匹配点中的外点
6. 如果匹配数量超过10，返回true
```mermaid
flowchart TB
	1[将当前帧的描述子转化为BoW向量]-->
	2[创建ORB特征匹配器,和保存成功匹配的地图点的向量]-->
	3[通过BoW加速当前帧与参考关键帧的特征点匹配:matcher.SearchByBow,记录匹配数目nmatches]-->
	4[[nmatches<15]]
	4-->|Yes|10
	4-->|No|5
	5[将上一帧的位姿设为当前帧的初始值]-->
	6[利用3D-2D投影关系,优化当前帧位姿PoseOptimization]-->
	7[剔除优化后匹配点中的外点,记录内点数量nmatchesMap]-->
	8[[nmatchesMap>=10]]
	8-->|Yes|9
	8-->|No|10
	9[匹配成功]
	10[匹配失败]
	11[结束]
	9 & 10 -->11
```

## 重定位跟踪：Relocalization
重定位跟踪步骤：
1. 计算当前帧特征点的BoW向量
2. 找到与当前帧相似的候选关键帧
3. 通过BoW进行匹配
4. 通过EPnP估计位姿
5. 通过PoseOptimization对位姿进行优化求解
6. 如果内点较少，则通过投影的方式对之前未匹配的点进行匹配，再进行优化求解。
流程图：
```mermaid
flowchart TB
	subgraph Relocalization
	direction TB
	1.1[计算当前帧特征点的BoW向量]-->
	1.2[DetectRelocalizationCandidates根据BoW向量,在关键帧Database中找到当前帧的所有候选关键帧,数量为nKFs]-->
	1.3[[候选关键帧数量nKFs为0?]]
	1.3-->|Yes|1.43
	1.3-->|No|1.4
	1.4[创建ORB匹配器matcher\n创建nKFs个PnP解算器vpPnPsolvers\n创建nKFs个保存特征匹配关系的vector:vvpMapPointMatches\n创建表示放弃关键帧标记的vbDiscarded\n有效候选关键帧数目nCandidates]-->
	1.5[遍历所有候选关键帧i]-->
	1.6[[关键帧i的标记是bad]]
	1.6-->|Yes|1.7
	1.6-->|No|1.8
	1.7[放弃当前关键帧,vbDiscarded.i=true]
	1.8[关键帧i与当前帧用BoW进行匹配:SearchByBoW,匹配结果记录在vvpMapPointMatches.i中,记录匹配数量nmatches]-->
	1.9[[nmatches<15]]
	1.9-->|No|1.11
	1.9-->|Yes|1.10
	1.10[放弃当前关键帧,vbDiscarded.i=true]
	1.11[用匹配结果初始化PnPsolver\n设置PnPsolver参数\n付给vpPnPsolvers.i]-->
	1.12[nCandidates++]
	1.13[[遍历结束?]]
	1.7 & 1.10 & 1.12 --> 1.13
	1.13-->|No|1.5
	1.13-->|Yes|1.14
	1.14[bMatch=false\n创建ORB匹配器matcher2]-->
	1.15[while循环nCandidates>0&&!bMatch]-->
	
	1.16[for遍历vbDiscarded为false的候选关键帧i]-->
	1.17[设置内点标记向量vbInliers\n内点个数nInliers\n是否达到RANSAC最大迭代次数的标记bNoMore]-->
	1.18[获取第i个PnP求解器pSolver\n迭代5次PnP,得到位姿Tcw]-->
	1.19[[如果bNoMore=true\nvbDiscarded.i=true\nnCandidates-1]]-->
	1.20[[Tcw不空?]]
	1.20-->|Yes,PnP算出了位姿,对内点进行BA优化|1.22
	1.20-->|No|1.21
	1.21[[for遍历结束?]]
	1.22[Tcw拷贝到mCurrentFrame.mTcw]-->
	1.23[设置PnP中RANSAC后内点的集合sFound]-->
	1.24[遍历所有内点,把vvpMapPointMatches.i中的内点放到当前帧的地图点和sFound中]-->
	1.25[PoseOptimization对位姿进行优化,输出内点数量nGood:只优化位姿,不优化地图点]-->
	1.26[[nGood<10]]
	1.26-->|Yes|1.21
	1.26-->|No|1.27
	1.27[把当前帧地图点中的外点设置为NULL]-->
	1.28[[nGood<50]]
	1.28-->|Yes|1.29
	1.28-->|No|1.37
	1.29>说明:BoW匹配的内点较少,需通过投影得到更多匹配点,再进行优化]-->
	1.30[matcher2.SearchByProjection\n通过投影将关键帧中未匹配的地图点投影到当前帧生成新的nadditional个匹配]-->
	1.31[[nadditional+nGood>=50]]
	1.31-->|Yes|1.32
	1.31-->|No|1.37
	1.32[根据投影匹配结果,再次采用3D-2D PnP BA优化位姿:PoseOptimization,得到新的内点数目nGood]-->
	1.33[[nGood>30 && nGood<50]]
	1.33-->|Yes|1.34
	1.33-->|No|1.37
	1.34[用BA优化后的位姿在更小窗口内投影搜索匹配nadditional]-->
	1.35[[nGood+nadditional>=50]]
	1.35-->|Yes|1.36
	1.36[再次BA优化位姿PoseOptimization,得到内点数量nGood]-->1.37
	1.35-->|No|1.37
	1.37[[nGood>=50]]
	1.37-->|Yes|1.38
	1.38[bMatch=true\nbreak]-->1.39
	1.37-->|No|1.21
	1.21-->|No|1.16
	1.21-->|Yes|1.39
	1.39[[nCandidates>0&&!bMatch]]
	1.39-->|Yes|1.15
	1.39-->|No|1.40
	1.40[[bMatch==false]]
	1.40-->|Yes|1.41
	1.40-->|No|1.42
	1.41[返回false]
	1.42[重定位成功,mnLastRelocFrameId = mCurrentFrame.mnId\n返回true]
	1.41 & 1.42 -->1.43[结束]
	end
```

重定位中候选关键帧组的选取策略：DetectRelocalizationCandidates
步骤：
1. 找出和当前帧具有公共word的所有关键帧
2. 只和具有公共word较多的关键帧进行相似度计算
3. 将与关键帧相连（权值最高）的前十个关键帧归为一组，计算累计得分
4. 只返回累计得分较高的组中分数最高的关键帧（在每个组中选得分最高的关键帧）。
流程图
```mermaid
flowchart TB
subgraph DetectRelocalizationCandidates
direction TB
1[遍历当前帧的所有word,找到有公共word的KF,把公共word的数量记录在KF中,放到lKFsSharingWords中]-->
2[[lKFsSharingWords为空?]]
2-->|Yes|31[返回空的候选关键帧组]
2-->|No|3
3[遍历lKFsSharingWords,得到其中与当前帧具有共同word最多的单词数量maxCommonWords]-->
4[设置最小公共word数量:minCommonWords = maxCommonWords*0.8f]-->
5[定义保存score和KF的list:lScoreAndMatch]-->
6[遍历lKFsSharingWords]-->
7[[当前KF的共同word数量>minCommonWords?]]
7-->|Yes|8
7-->|No|10
8[计算当前帧和KF的BoW相似度si,付给KF的mRelocScore]-->
9[把<si,KF>放到lScoreAndMatch]-->10
10[[遍历结束?]]
10-->|No|6
10-->|Yes|11
11[[lScoreAndMatch为空]]
11-->|Yes|31[返回空的候选关键帧组]
11-->|No|12
12[定义保存每个组中最优的累计得分和KF的list:lAccScoreAndMatch\nbestAccScore = 0]-->
13[遍历lScoreAndMatch]-->
14[取出当前KF公视程度最高的10个关键帧,作为一组vpNeighs]-->
15[遍历vpNeighs]-->
16[[当前KF不在重定位候选关键帧中]]
16-->|Yes|18
16-->|No|17
17[累加重定位得分accScore+=pKF2->mRelocScore]-->
18[[遍历结束?]]
18-->|No|15
18-->|Yes|19
19[记录组中分数最高的KF,作为当前组的最佳候选帧pBestKF]-->
20[把<accScore,pBestKF>放到lAccScoreAndMatch]-->
21[记录所有组中的最高得分bestAccScore]-->
22[[遍历结束?]]
22-->|No|13
22-->|Yes|23
23[minScoreToRetain = 0.75f*bestAccScore]-->
24[定义set:spAlreadyAddedKF,用于去重\n定义用于输出的候选关键帧vpRelocCandidates]-->
25[遍历lAccScoreAndMatch]-->
26[获取当前KF的得分:si=it->first]-->
27[[si>minScoreToRetain?]]
27-->|No|29
27-->|Yes|28
28[如果KF不在spAlreadyAddedKF中,则插入到spAlreadyAddedKF\n同时放到vpRelocCandidates]-->
29[[遍历结束?]]
29-->|No|25
29-->|Yes|30
30[返回vpRelocCandidates]
30 & 31 -->32
32[结束]
end
```

## 局部地图跟踪
上面三种跟踪只会执行其中一种，获得当前帧比较粗糙的位姿。然后需要通过跟踪局部地图，得到更加精确的位姿。
**局部地图跟踪的步骤**:
1. 更新局部地图（UpdateLocalMap），包括更新局部关键帧和局部地图点
2. 在局部地图中查找与当前帧匹配的MapPoints，即对局部地图点进行跟踪
3. 更新局部所有MapPoint后，对位姿再次优化
4. 更新当前帧的MapPoint被观测程度，并统计跟踪局部地图的效果
5. 决定是否跟踪成功

**名词理解**:
1. 当前帧：mCurrentFrame，是普通帧
2. 参考关键帧：与当前帧共视程度最高的关键帧
3. 父关键帧：和当前关键帧共视程度最高的关键帧
4. 子关键帧：是上述父关键帧的子关键帧
5. 共视关键帧：一级共视和二级共视
	1. 一级共视关键帧：与当前帧有共同的地图点的关键帧
	2. 二级共视关键帧：与一级共视关键帧有共同地图点的关键帧称为当前帧的二级共视关键帧
![7a3ca1e24981d8332ee2f88ef5ca9804.png](../../../_resources/7a3ca1e24981d8332ee2f88ef5ca9804.png)

所以，当前帧的局部关键帧包括：一级共视关键帧、二级共视关键帧、一级共视关键帧的父关键帧和子关键帧。

**流程图**
```mermaid
flowchart
	subgraph TrackLocalMap
	direction TB
	1[开始]-->
	2[UpdateLocalMap\n更新局部关键帧mvpLocalKeyFrames\n和局部地图点MVPLocalMapPoints]-->
	3[SearchLocalPoints\n筛选局部地图中新增的在视野范围内的地图点,投影到当前帧搜索匹配]-->
	4[PoseOptimization\n根据前面跟踪得到的位姿和新的匹配关系,BA优化得到更准确的位姿]-->
	5[遍历当前帧关键点对应的地图点被观测程度\n并统计跟踪局部地图后匹配数目]-->
	6[[当前关键点对应的地图点存在?]]
	6-->|No|11
	6-->|Yes | 7
	7[[当前关键点没有被标记为外点?]]
	7-->|No,外点且是双目模式|8
	8[删除这个点]
	7-->|Yes,不是外点|9
	9[该点的被观测到的数量+1]-->
	10[记录当前帧跟踪到的地图点数目,用于统计跟踪效果]
	11[[遍历结束?]]
	8 & 10 --> 11
	11-->|No|5
	11-->|Yes|12
	12[[刚刚发生重定位]]
	12-->|Yes|13
	13[当前帧跟踪到的地图点数<50\n跟踪失败,返回false]
	12-->|No|14
	14[跟踪到地图点>30\n跟踪成功,返回true]
	15[跟踪到地图点<30\n跟踪失败,返回false]
	16[结束]
	13 & 14 & 15 -->16
	end
```

```mermaid
flowchart TB
	subgraph UpdataLocalMap:UpdateLocalKeyFrames
	direction TB
	1.1(开始)-->
	1.2[定义字典keyframeCounter:\n记录一级共视关键帧和共视程度]-->
	1.3([遍历当前帧的地图点])-->
	1.4[[此地图点是好点?]]
	1.4-->|No|1.5
	1.5[删除此点]-->1.10
	1.4-->|Yes|1.6
	1.6[获取能观测到该地图点的所有关键帧\n和此点在关键帧中的索引observations]-->
	1.7([遍历observations])-->
	1.8[把关键帧插入到map,并累加每个此关键帧的共视程度]-->
	1.9[[遍历结束?]]
	1.9-->|No|1.7
	1.9-->|Yes|1.10
	1.10[[遍历结束?]]
	1.10-->|No|1.3
	1.10-->|Yes|1.11
	1.11[[keyframeCounter为空?]]
	1.11-->|Yes|1.42
	1.11-->|No|1.12
	1.12[定义用于记录具有最大共视程度的关键帧pKFmax]-->
	1.13[清空局部关键帧向量mvpLocalKeyFrames]-->
	1.14([遍历一级共视关键帧keyframeCounter])-->
	1.15[把一级共视关键帧放入mvpLocalKeyFrames]-->
	1.16[用当前关键帧的mnTrackReferenceForFrame记录当前帧id\n表示已经是当前帧的局部关键帧了,防止重复添加]-->
	1.17[记录共视程度最大的关键帧pKFmax]-->
	1.18[[遍历结束?]]
	1.18-->|No|1.14
	1.18-->|Yes|1.19
	1.19([遍历一级共视关键帧mvpLocalKeyFrames\n找到更多局部关键帧])-->
	1.20[[mvpLocalKeyFrames.size>80?]]
	1.20-->|Yes|break跳出遍历-->1.42
	1.20-->|No|1.21
	1.21[获得最多10个与当前一级关键帧共视程度最高的关键帧vNeighs\n作为当前帧的二级共视关键帧]-->
	1.22([遍历二级共视关键帧vNeighs])
	1.23[[遍历结束]]
	1.22 --> 1.24
	1.24[[当前关键帧的mnTrackReferenceForFrame!=当前帧id?]]
	1.24-->|Yes|1.25
	1.25[把当前关键帧放入mvpLocalKeyFrames]-->
	1.26[当前关键帧的mnTrackReferenceForFrame记录当前帧id]-->
	1.27[break跳出循环]-->1.28
	1.24-->|No|1.23
	1.23-->|No|1.22
	1.23-->|Yes|1.28
	1.28[获得当前一级关键帧的子关键帧spChilds]-->
	1.29([遍历子关键帧])
	1.30[遍历结束?]
	1.29-->1.31
	1.31[[当前关键帧的mnTrackReferenceForFrame!=当前帧id?]]
	1.31-->|Yes|1.32
	1.32[把当前关键帧放入mvpLocalKeyFrames]-->
	1.33[当前关键帧的mnTrackReferenceForFrame记录当前帧id]-->
	1.34[break跳出循环]-->1.35
	1.31-->|No|1.30
	1.30-->|No|1.29
	1.30-->|Yes|1.35
	1.35[获得当前一级关键帧的父关键帧pParent]-->
	1.36[[pParent是否存在?]]
	1.36-->|Yes|1.37
	1.37[[父关键帧的mnTrackReferenceForFrame!=当前帧id?]]
	1.37-->|Yes|1.38
	1.38[把父关键帧放入mvpLocalKeyFrames]-->
	1.39[父关键帧的mnTrackReferenceForFrame记录当前帧id]
	1.36-->|No|1.40
	1.37-->|No|1.40
	1.39-->1.40
	1.40[[遍历结束?]]
	1.40-->|No|1.19
	1.40-->|Yes|1.41
	1.41[更新当前帧的参考关键帧为共视程度最高的关键帧]-->
	1.42(结束)
	end
```

```mermaid
flowchart TB
	subgraph UpdataLocalMap:UpdateLocalPoints
	direction TB
	0(开始)-->1
	1[清空局部地图点]-->
	2([遍历当前帧的局部关键帧mvpLocalKeyFrames])
	3[[遍历结束?]]
	2-->4
	4[获得当前关键帧的地图点]-->
	5([遍历当前关键帧的地图点])
	6[[遍历结束?]]
	5-->7
	7[[该地图点的mnTrackReferenceForFrame==当前帧id?]]
	7-->|Yes|6
	7-->|No|8
	8[把该地图点放到局部地图点中MVPLocalMapPoint]-->
	9[该地图点的mnTrackReferenceForFrame设置为当前帧id]-->6
	6-->|No|5
	6-->|Yes|3
	3-->|No|2
	3-->|Yes|10
	10(结束)
	end
```

```mermaid
flowchart TB
	subgraph SearchLocalPoints
	direction TB
	1(开始)-->
	2([遍历当前帧的地图点,标记不参与后面的投影搜索匹配])
	3[[遍历结束?]]
	2-->4
	4[该地图点被观测到的帧数+1]-->
	5[该地图点的mnLastFrameSeen设置为当前帧id]-->
	6[标记该地图点在后面搜索匹配时不被投影mbTrackInView=false]
	6-->3
	3-->|No|2
	3-->|Yes|7
	7[进行投影匹配的点的数目nToMatch=0]-->
	8([遍历当前帧的局部地图点mvpLocalMapPoints])
	9[[遍历结束?]]
	8-->10
	10[[该地图点是否已经被观测到mnLastFrameSeen==当前帧id?]]
	10-->|Yes|9
	10-->|No|11
	11[跳过坏点]-->
	12[[该地图点在当前帧的视野内isInFrustum?]]
	12-->|No|9
	12-->|Yes|13
	13[该地图点被观测到的帧数+1]-->
	14[nToMatch+1,只有视野内的点才投影匹配]-->9
	9-->|No|8
	9-->|Yes|15
	15[[nToMatch>0?]]
	15-->|No|19
	15-->|Yes|16
	16[创建匹配器matcher]-->
	17[设置搜索区域大小阈值:\n正常是1\nRGBD模式为3\n刚经过重定位时为5]-->
	18[投影匹配搜索:matcherSearchByProjection]-->
	19(结束)
	end
```