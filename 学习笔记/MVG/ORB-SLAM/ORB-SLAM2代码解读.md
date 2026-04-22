---
title: ORB-SLAM2代码解读
updated: 2022-06-06 08:53:41Z
created: 2022-02-15 04:02:43Z
latitude: 39.92850000
longitude: 116.38500000
altitude: 0.0000
---

# System类：
主要完成系统初始化配置，包括：
### 成员变量
mpVocabulary：指向orb字典的指针
mpKeyFrameDatabase：用于重定位和回环检测的关键帧数据库的指针
mpMap：所有关键帧和地图点的地图指针
mpTracker：追踪器：在主进程中运行，进行运动追踪，以及负责创建关键帧、地图点和重定位。
mpLocalMapper：局部建图器：管理局部地图，并进行局部BA
mpLoopCloser：回环检测器：有新关键帧时，进行回环检测，并开一个新的线程，进行全局BA
mpViewer：可视化器
mpFrameDrawer：帧绘制器
mpMapDrawer：地图绘制器
三个线程：
mptLocalMapping：局部建图线程
mptLoopClosing：回环检测线程
mptViewer：可视化线程

mTrackingState：追踪状态标志

### 成员函数
```
// TrackMonocular是主线程的入口函数
cv::Mat System::TrackMonocular(const cv::Mat &im, const double &timestamp)
{
    if(mSensor!=MONOCULAR)
    {
        cerr << "ERROR: you called TrackMonocular but input sensor was not set to Monocular." << endl;
        exit(-1);
    }

    // Check mode change
	  // 模式会因为用户在可视化界面中手动设置而改变。
    {
        // 独占锁，主要是为了mbActivateLocalizationMode和mbDeactivateLocalizationMode不会发生混乱
        unique_lock<mutex> lock(mMutexMode);
        // mbActivateLocalizationMode为true会关闭局部地图线程
        if(mbActivateLocalizationMode)
        {
            mpLocalMapper->RequestStop();

            // Wait until Local Mapping has effectively stopped
            while(!mpLocalMapper->isStopped())
            {
                usleep(1000);
            }

            // 局部地图关闭以后，只进行追踪的线程，只计算相机的位姿，没有对局部地图进行更新
            // 设置mbOnlyTracking为真
            mpTracker->InformOnlyTracking(true);
            // 关闭线程可以使得别的线程得到更多的资源
            mbActivateLocalizationMode = false;
        }
        // 如果mbDeactivateLocalizationMode是true，局部地图线程就被释放, 关键帧从局部地图中删除.
        if(mbDeactivateLocalizationMode)
        {
            mpTracker->InformOnlyTracking(false);
            mpLocalMapper->Release();
            mbDeactivateLocalizationMode = false;
        }
    }

    // Check reset
    {
    unique_lock<mutex> lock(mMutexReset);
    if(mbReset)
    {
        mpTracker->Reset();
        mbReset = false;
    }
    }

    //获取相机位姿的估计结果
	  // tracking的入口
    cv::Mat Tcw = mpTracker->GrabImageMonocular(im,timestamp);

    unique_lock<mutex> lock2(mMutexState);
    mTrackingState = mpTracker->mState;
    mTrackedMapPoints = mpTracker->mCurrentFrame.mvpMapPoints;
    mTrackedKeyPointsUn = mpTracker->mCurrentFrame.mvKeysUn;

    return Tcw;
}
```

# Tracking 类
## 1. 成员变量
eTrackingState mState ：当前帧的跟踪状态，SYSTEM_NOT_READY：系统没有准备好，一般是加载配置文件和词典文件；NO_IMAGES_YET：当前没有图像；NOT_INITIALIZED：有图像但是没有完成初始化；OK：正常工作状态；LOST：系统跟踪丢失
eTrackingState mLastProcessedState：上一帧的跟踪状态
Frame mCurrentFrame：追踪线程中的当前帧
std::vector<int> mvIniLastMatches：初始化过程中，上一帧的匹配？
std::vector<int> mvIniMatches：初始化中，当前帧的特征点和参考帧的特征点的匹配关系
std::vector<cv::Point2f> mvbPrevMatched：初始化中，保存参考帧的特征点
std::vector<cv::Point3f> mvIniP3D：初始化中，匹配后进行三角化得到的空间点
Frame mInitialFrame：初始化中的参考帧
list<cv::Mat> mlRelativeFramePoses：所有参考关键帧的位姿，是相对位姿
list<KeyFrame*> mlpReferences：所有参考关键帧
list<double> mlFrameTimes：所有帧的时间戳
list<bool> mlbLost：是否跟丢
LocalMapping* mpLocalMapper：局部建图器
LoopClosing* mpLoopClosing：回环检测器
ORBextractor* mpORBextractorLeft, *mpORBextractorRight：orb特征点提取器
ORBextractor* mpIniORBextractor：初始化阶段用到的提取器，会提取更多的特征点
ORBVocabulary* mpORBVocabulary：orb特征字典
KeyFrameDatabase* mpKeyFrameDB：当前系统运行时，关键帧产生的数据库
Initializer* mpInitializer：单目初始化器
KeyFrame* mpReferenceKF：参考关键帧
std::vector<KeyFrame*> mvpLocalKeyFrames：局部关键帧集合
std::vector<MapPoint*> mvpLocalMapPoints：局部地图点集合
System* mpSystem：系统实例指针
Map* mpMap：全局地图

cv::Mat mK：相机内参矩阵
cv::Mat mDistCoef：相机的畸变参数
float mbf：相机的基线长度*相机的焦距
int mMinFrames：
int mMaxFrames：新建关键帧和重定位中用来判断最小最大时间间隔，与帧率有关
float mThDepth：用于区分远点和近点的阈值，认为近点可信度较高；远点要求在两个关键帧中得到匹配
float mDepthMapFactor：深度缩放因子，链接深度值和具体深度值的参数，只对RGBD输入有效
int mnMatchesInliers：当前帧中的进行匹配的内点，将会被不同的函数反复使用。
KeyFrame* mpLastKeyFrame：上一个关键帧
Frame mLastFrame：上一帧
unsigned int mnLastKeyFrameId：上一关键帧的ID
unsigned int mnLastRelocFrameId：上次重定位的那一帧的ID
cv::Mat mVelocity：运动模型
list<MapPoint*> mlpTemporalPoints：临时的地图点，用于提高双目和RGBD摄像头的帧间效果，用完就丢

## 2. tracking流程
1. mState==NOT_INITIALIZED时，进行初始化
2. mState==OK时进入跟踪建图模式，否则跟踪失败，进行重定位
	2.1 检查并更新上一帧被替换的MapPoints：CheckReplacedInLastFrame()
	2.2 运动模型为空或者刚完成重定位时，跟踪参考关键帧；否则用恒速模型跟踪，其中TrackReferenceKeyFrame()和TrackWithMotionModel()会计算出当前帧的mvpMapPoints
3. 当跟踪得到当前帧的初始状态后，对local map进行跟踪得到更多的匹配，并优化当前位姿。第二步只是跟踪一帧得到的初始位姿，这里会搜索局部关键帧、局部地图点，和当前帧进行投影匹配，得到更多匹配的MapPoints后进行Pose优化：bOK = TrackLocalMap()
4. 更新显示线程中的图像、特征点、地图点等信息mpFrameDrawer->Update(this)
5. 跟踪成功时，更新恒速运动模型
6. 清除观测不到的地图点
7. 清除恒速模型跟踪中UpdateLastFrame中为当前帧临时添加的MapPoints(仅双目和RGBD)，6中只是在当前帧中将这些MapPoints剔除，这里从MapPoints数据库中删除。
8. 检测并插入关键帧，对于双目和RGBD会产生新的地图点。
9. 删除在BA中检测为outlier的地图点
10. 如果初始化不久就跟踪失败，并且relocation也没有搞定，只能重新reset
11. 记录位姿信息，用于最后保存所有的轨迹。

### 2.1 当前帧初始位姿的计算：TrackReferenceKeyFrame() 和 TrackWithMotionModel()
#### 2.1.1 TrackReferenceKeyFrame:
1. 获取当前帧和参考关键帧之间匹配的关键点，当前帧关键点描述子转化为BoW向量，通过BoW加速当前帧与关键帧的特征匹配，记录存在匹配关系的MapPoints（vpMapPointMatches）。如果匹配数目小于15，认为跟踪失败。用到的函数：ComputeBoW() 和 SearchByBoW()
2. 将上一帧（mLastFrame）的位姿作为当前帧位姿的初始值，加速收敛
3. 优化3D-2D的重投影误差获得当前帧位姿:Optimizer::PoseOptimization()
4. 剔除优化后的匹配点中的外点。优化过程中会对外点进行标记
#### 2.1.2 TrackWithMotionModel:
1. 更新上一帧的位姿（UpdateLastFrame()）：
	1. 因为上一帧的参考关键帧可能由于BA导致位姿变化，所以上一帧的位姿需要同步更新。Frame中会保存参考关键帧以及相对参考关键帧的位姿，由此得到更新的位姿。
	2. 对于双目和RGBD，还会为上一帧生成新的临时地图点，只用来跟踪，不加入地图，跟踪完成后删除。1）得到上一帧中具有有效深度值的特征点（不一定是地图点），并按深度值从小到大排序，如果没有，就直接退出；2）找出不是地图点的部分，如果这个点不在上一帧中的地图点中，或者创建后没有被观测到，则生成一个临时的地图点。3）如果深度值大于35倍基线或者已经创建了100个地图点，则停止。
2. 根据之前估计的速度，用恒速模型得到当前帧的初始位姿。
3. 清空当前帧的地图点。
4. 设置特征匹配过程的搜索半径，单目是15，双目是7.
5. 用上一帧的地图点进行投影匹配，如果匹配点不够（少于20个），则扩大半径再来一次。
6. 优化3D-2D的重投影误差获得当前帧位姿:Optimizer::PoseOptimization()
7. 剔除优化后的匹配点中的外点。优化过程中会对外点进行标记
### 2.2 当前帧位姿的优化：TrackLocalMap()
当前帧的初始位姿是有跟踪一帧得到的，优化过程中会搜索局部关键帧、局部地图点和当前帧进行投影匹配，得到更多匹配的MapPoints后，进行Pose的优化
1. 更新局部地图（UpdateLocalMap()）：包括局部关键帧和局部地图点
	1. 更新局部关键帧（UpdateLocalKeyFrames()）：遍历当前帧的地图点，将观测到这些地图点的关键帧和相邻的关键帧及其父子关键帧，作为mvpLocalKeyFrames。
		1. 遍历当前帧的地图点，记录所有能观测到当前帧地图点的关键帧（称为**一级共视关键帧**），并记录每个共视关键帧所观测到的地图点keyframeCounter[it->first]++
		2. 更新局部关键帧，有三种类型
			1. 遍历keyframeCounter，忽略要删除的KF，添加进局部关键帧列表，并寻找具有最大观测数目的关键帧（共视程度最高），并记录当前关键帧的id，防止重复添加。（类型一：一级共视关键帧）
			2. 遍历一级共视关键帧，
				将一级共视关键帧的共视关键帧加入局部地图（类型二：二级共视关键帧）
				将一级共视关键帧的子关键帧和父关键帧加入局部地图（类型三）
		3. 更新当前帧的参考关键帧，与自己共视程度最高的关键帧作为参考关键帧。
	2. 更新局部关键点（UpdateLocalPoints()）：先把局部地图清空，然后将局部关键帧的有效地图点添加到局部地图中。步骤：
		1. 清空局部地图点
		2. 遍历局部关键帧，将其中的地图点添加到局部地图中（mvpLocalMapPoints），其中地图点的成员变量mnTrackReferenceForFrame记录当前帧的id，防止重复添加到地图中。
2. 筛选局部地图中新增的在视野范围内的地图点，投影到当前帧搜索匹配，得到更多匹配关系（SearchLocalPoints()）。其中局部地图点中已经是当前帧   地图点的不需要再投影，只需要将此外的并且在视野范围内的点和当前帧进行投影匹配
	1. 遍历当前帧的地图点，标记这些地图点不参与之后的投影搜索匹配。
	2. 判断所有局部地图点中除当前帧地图点外的点，是否在当前帧的视野中（isInFrustum()）:
		1. 获取地图点的世界坐标，根据当前帧的粗糙位姿转化为当前相机坐标系下的三维点Pc
		2. 关卡一：如果深度值为正，继续下一个关卡
		3. 关卡二：将地图点投影到当前帧的像素坐标，如果在图像有效范围内（图像边界内），继续下一个关卡
		4. 关卡三： 计算地图点到相机中心的距离，如果在有效距离范围内，继续下一个关卡
		5. 关卡四：计算当前相机指向地图点向量和地图点的平均观测方向夹角，小于60度，继续下一个关卡。
		6. 通过上述判断后，根据地图点到光心的距离预测一个尺度
		7. 记录计算得到的参数（是否要被投影；像素坐标；该地图点的尺度层级；方向夹角）
	3. 对符合上述条件的点进行投影匹配（matcher.SearchByProjection()）：
		1. 遍历有效的局部地图点，获取当前帧的金字塔层数
		2. 根据地图点的视角，设定搜索窗口的大小，当前视角和平均视角夹角较小时，取较小的值
		3. 通过投影点以及搜索窗口和预测的尺度进行搜索，找出搜索半径内的候选匹配点所索引（可能有多个）
		4. 遍历候选点，寻找候选匹配点中的最佳和次佳匹配点：
			1. 如果候选点在该frame中已经有其他对应的MapPoint了，就忽略这个候选点
			2. 计算地图点和候选点的描述子距离，找到描述子距离最小和次小的特征点和索引
		5. 筛选最佳匹配点，最佳匹配距离需要满足在设定的阈值内。
			1. 如果最佳和次佳在同一金字塔层级且最佳的距离与次佳距离的比值大于阈值，则表示该地图点匹配失败
			2. 否则匹配成功，为Frame中的特征点增加对应的MapPoint
3. 当前帧位姿优化：Optimizer::PoseOptimization(&mCurrentFrame)
4. 更新当前帧的地图点被观测程度，并统计跟踪局部地图点后匹配数目。
5. 根据跟踪匹配数目集重定位情况判断是否跟踪成功，如果最近发生了重定位，那么至少匹配50个点才认定跟踪成功，正常情况只需要跟踪的地图点大于30个。

### 2.3 关键帧生成策略（NeedNewKeyFrame()、CreateNewKeyFrame()）
#### 2.3.1 判断是否需要插入新关键帧：NeedNewKeyFrame()
1. 纯VO模式不插入关键帧
2. 如果局部地图线程倍闭环检测使用，则不插入关键帧。
3. 如果距离上一次重定位比较近，而且当前关键帧数目超出最大限制，不插入关键帧。
4. 得到参考关键帧跟踪到的地图点数量mpReferenceKF->TrackedMapPoints(nMinObs)
5. 查询局部地图线程是否繁忙，当前能否接受新的关键帧mpLocalMapper->AcceptKeyFrames()
6. 对于双目或者RGBD摄像头，统计成功跟踪的近点的数量，如果跟踪到的近点太少，没有跟踪到的近点较多，可以插入关键帧。
7. 决策是否需要插入关键帧
	1. 设定比例阈值，当前帧和参考关键帧跟踪到点的比例，比例越大，越倾向于增加关键帧。如果关键帧只有一帧，那么插入关键帧的阈值设置的低一点，插入频率较低，单目情况下，插入关键帧的频率设置的高一些。
	2. 长时间没有插入关键帧，可以插入
	3. 满足插入关键帧的最小间隔并且localMapper处于空闲状态，可以插入。
	4. 双目和RGBD下，当前帧跟踪到的点比参考关键帧的0.25倍还少，或满足bNeedToInsertClose，可以插入
	5. 和参考帧相比，当前跟踪到的点太少或者满足bNeedToInsertClose，同时跟踪到的内点还不能太少
	6. 当满足（（2 || 3 || 4||）&& 5）的情况下，如果localMapping线程空闲，可以直接插入关键帧；否则：（1）对于双目或RGBD，待插入的关键帧队列中的帧数少于三个，则localMapper可以边pop边插入，否则不能插入关键帧。（2）单目情况，直接返回不能插入。
#### 2.3.2 插入新的关键帧CreateNewKeyFrame()
1. 设置不允许局部建图线程关闭，否则无法插入新关键帧
2. 将当前帧构造成关键帧
3. 将上面构造的关键帧设置为当前帧的参考关键帧
4. 对于双目或rgbd摄像头，为当前帧生成新的地图点；单目无操作
	1. 根据Tcw计算mRcw、mtcw和mRwc、mOw：mCurrentFrame.UpdatePoseMatrices();
	2. 得到当前帧有深度值的特征点（不一定是地图点）
	3. 按照深度从小到大排序
	4. 对其中不是地图点的生成临时地图点
5. 插入关键帧到列表mlNewKeyFrames中，等待localMapping线程处理
6. 插入完成，允许局部地图停止
7. 当前帧称为新的关键帧，mnLastKeyFrameId = mCurrentFrame.mnId，mpLastKeyFrame = pKF


# LocalMapping
## 1. 成员函数和成员变量
**成员函数**
public：
LocalMapping(Map* pMap, const float bMonocular)
void SetLoopCloser(LoopClosing* pLoopCloser)：设置回环检测指针，用于线程之间的信息交互
void SetTracker(Tracking* pTracker)：设置追踪线程指针，用于线程之间的信息交互
void Run()：线程主函数
void InsertKeyFrame(KeyFrame* pKF)：外部线程调用，插入关键帧
void RequestStop()：外部线程调用，请求停止当前线程的工作
void RequestReset()：由外部线程调用，阻塞的，请求当前线程复位
int KeyframesInQueue()：查看mlNewKeyFrames队列中等待插入的关键帧数目
protected：
bool CheckNewKeyFrames()：查看列表中是否有等待被插入的关键帧
void ProcessNewKeyFrame()：处理列表中的关键帧
void CreateNewMapPoints()：相机运动过程中和共视程度比较高的关键帧通过三角化恢复出一些MapPoints
void MapPointCulling()：剔除ProcessNewKeyFrame和CreateNewMapPoints函数中引入的质量布好的MapPoints
void SearchInNeighbors()：检查并融合当前关键帧与相邻帧（两级相邻）重复的MapPoints
void KeyFrameCulling()：关键帧剔除，在Covisibility Graph中的关键帧，其90%以上的MapPoints能被其他关键帧（至少3个）观测到，则认为该关键帧为冗余关键帧。
cv::Mat ComputeF12(KeyFrame* &pKF1, KeyFrame* &pKF2)：根据两关键帧的姿态计算两个关键帧之间的基本矩阵
cv::Mat SkewSymmetricMatrix(const cv::Mat &v)：计算三维向量v的反对称矩阵

**成员变量**
Map* mpMap：局部地图
LoopClosing* mpLoopCloser：回环检测
Tracking* mpTracker：追踪线程
std::list<KeyFrame*> mlNewKeyFrames：等待处理的关键帧列表
KeyFrame* mpCurrentKeyFrame：当前正在处理的关键帧
std::list<MapPoint*> mlpRecentAddedMapPoints：存储当前关键帧生成的地图点，也是等待检测的地图点列表
bool mbAcceptKeyFrames：当前局部建图线程是否允许关键帧输入。

## 流程
线程主函数为Run()，函数中主要是一个无限循环流程，通过判断mlNewKeyFrames是否有数据，来进行建图或者接收新的关键帧。
1. 首先设置mbAcceptKeyFrames为false，不接收Tracking发来的新关键帧请求；
2. 如果mlNewKeyFrames不为空
	1. 处理mlNewKeyFrames列表中的关键帧，包括计算BoW、更新观测、描述子、共视图、插入到地图等：ProcessNewKeyFrame()
	2. 根据地图点的观测情况剔除质量布好的地图点：MapPointCulling()
	3. 当前关键帧与相邻关键帧通过三角化产生新的地图点，使跟踪更稳定：CreateNewMapPoints()
	4. 当处理完mlNewKeyFrames中的关键帧后，检查并融合当前关键帧与相邻关键帧（两级相邻）中重复的地图点：SearchInNeighbors()
	5. 把mbAbortBA设置为false，Tracking线程会进行修改。
	6. 当已经处理完mlNewKeyFrames中的关键帧以及闭环检测没有请求停止LocalMapping
		1. 当局部地图中的关键帧大于两个的时候，进行局部地图的BA：Optimizer::LocalBundleAdjustment(mpCurrentKeyFrame,&mbAbortBA, mpMap)
		2. 检测并剔除当前帧相邻的关键帧中冗余的关键帧，冗余判定：该关键帧的90%的地图点可以被其他关键帧观测到：KeyFrameCulling()
	7. 将当前帧加入到闭环检测队列中：mpLoopCloser->InsertKeyFrame(mpCurrentKeyFrame)
3. 如果mlNewKeyFrames为空，判断是否要终止当前线程Stop()，其中LoopClosing、System、Viewer线程会调用RequestFinish()来传入停止的信号，如果收到信号，就会跳出当前线程的主循环
4. 查看是否有复位线程的请求
5. 给Tracking线程发信号，表示可以接收新的keyFrame

## 主要函数
### 1. ProcessNewKeyFrame()步骤
1. 从缓冲队列mlNewKeyFrames取出一帧关键帧赋给mpCurrentKeyFrame，注意需要加锁
2. 对mpCurrentKeyFrame的特征点计算词袋向量
3. 处理关键帧中有效的地图点，更新normal、描述子等信息，对当前处理的关键帧中的所有地图点进行遍历：
	1. 如果地图点不是来自当前帧的观测，则为当前地图点添加观测 pMP->AddObservation(mpCurrentKeyFrame, i)
	2. 获得该点的平均观测方向和观测距离范围：pMP->UpdateNormalAndDepth()
	3. 更新地图点的最佳描述子：pMP->ComputeDistinctiveDescriptors()
	4. 如果当前帧中已经包含了这个地图点，但是这个地图点中没有包含这个关键帧的信息，那这些地图点可能来自双目或rgbd跟踪过程中新产生的地图点，或者CreateNewMapPoints中通过三角化产生，将这些地图点放入mlpRecentAddedMapPoints，等待后续MapPointCulling函数的检验
4. 更新关键帧间的链接关系（共视图）：mpCurrentKeyFrame->UpdateConnections()
5. 将该关键帧插入到地图中：mpMap->AddKeyFrame(mpCurrentKeyFrame)
### 2. MapPointCulling()：检查新增地图点，根据地图点的观测情况剔除质量不好的新增地图点，mlpRecentAddedMapPoints中存储新增地图点，删除其中不好的
1. 根据相机类型设置不同的观测阈值
2. 遍历检查新添加的地图点
	1. 如果已经是坏点，直接从队列中删除
	2. 如果跟踪到该地图点的帧数相比预计可观测到该地图点的帧数比例小于25%，从地图中删除。其中mnFound表示地图点被多少帧（包括普通帧）看到，次数越多越好；mnVisible：地图点应该被观测到的次数，该点在相机视角内。
	3. 从该点建立开始到现在，已经过了不少于2个关键帧，但是观测到该点的相机数却不超过阈值，从地图中删除。
	4. 从建立该点开始，已经过了3个关键帧而且没有被剔除，则认为是质量高的点。
### 3. CreateNewMapPoints()：用当前关键帧与相邻关键帧通过三角化产生新的地图点，使得跟踪更稳定
1. 在当前关键帧的共视关键帧中找到共视程度最高的nn帧相邻关键帧
2. 遍历相邻关键帧，搜索匹配并用极线约束剔除误匹配，最终三角化
	1. 判断当前关键帧与相邻关键帧的基线是否足够长，基线太短恢复的地图点不稳定：双目下，关键帧间距小于本身基线时，不产生3D点；单目下，计算相邻关键帧的场景深度中值，如果基线与景深的比值太小，说明基线太短，恢复3D点不稳定，跳过当前相邻的关键帧。
	2. 根据两个关键帧的位姿计算它们之间的基础矩阵
	3. 通过词袋对两关键帧的未匹配的特征点快速匹配，用极线约束抑制离群点，生成新的匹配点对：matcher.SearchForTriangulation(mpCurrentKeyFrame,pKF2,F12,vMatchedIndices,false)
	4. 对每对匹配通过三角化生成3D点：
		1. 取出匹配特征点
		2. 利用匹配点反投影得到视差角
		3. 对于双目，利用双目得到视差角
		4. 三角化恢复3D点
		5. 检测生成的3D点是否在相机的前方，不在的话，就放弃这个点
		6. 计算3D点在两个关键帧下的重投影误差，卡方检测
		7. 检查尺度连续性
		8. 三角化生成3D点成功，构造成MapPoint
		9. 为该MapPoint添加属性：观测到该MapPoint的关键帧；该MapPoint的描述子；平均观测方向和深度范围；
		10. 将该MapPoint放入检测队列mlpRecentAddedMapPoints，等待MapPointCulling函数的检验
### 4. SearchInNeighbors()：检查并融合当前关键帧与相邻关键帧（两级相邻）重复的地图点
一级相邻关键帧：当前关键帧的邻接关键帧
二级相邻关键帧：与一级相邻关键帧相邻的关键帧
1. 获得当前关键帧在共视图中权重排名前nn的邻接关键帧，单目检查20个邻接关键帧，双目或rgbd检查10个
2. 存储一级相邻关键帧和其二级相邻关键帧
	1. 遍历所有的相邻关键帧，如果没有和当前关键帧融合过，则加入相邻关键帧列表
	2. 筛选与一级相邻关键帧共视程度最好的5个相邻关键帧，如果没有与当前关键帧融合过，且不是当前关键帧，则加入相邻关键帧列表
3. 将当前帧的地图点分别投影到两级相邻关键帧，寻找匹配点对应的地图点进行融合，称为正向融合，融合策略：
	1. 将地图点投影到关键帧中进行匹配和融合
	2. 如果地图点能匹配关键帧的特征点，并且该点有对应的地图点，那么选择观测数目多的替换两个地图点
	3. 如果地图点能匹配关键帧的特征点，并且该点没有对应的地图点，那么为该点添加该投影地图点
此时对地图点的融合操作是立即生效的。
4. 将两级相邻关键帧地图点分别投影到当前关键帧，寻找匹配点对应的地图点进行融合，称为反向融合。
	1. 遍历每一个一级和二级邻接关键帧，收集它们的地图点存储到vpFuseCandidates
	2. 进行地图点投影融合，与正向融合完全相同
5. 更新当前帧地图点的描述子、深度、平均观测方向等属性。
6. 更新当前帧与其它帧的共视连接关系。
### 5. KeyFrameCulling()：检测当前关键帧在共视图中的关键帧，根据地图点在共视图中的冗余程度剔除该共视关键帧，冗余关键帧的判定条件：90%以上的地图点能被其他关键帧（至少3个）观测到。步骤：
1. 根据共视图提取当前关键帧的所有共视关键帧，遍历所有共视关键帧
2. 第一个关键帧不能删除。提取每个共视关键帧的地图点，
3. 遍历该共视关键帧的所有地图点，其中能倍其他至少三个关键帧观测到的地图点为冗余地图点
4. 如果该关键帧90%以上的有效地图点被判断为冗余的，则认为该关键帧是冗余的，需要删除该关键帧

# LoopClosing
## 成员函数和成员变量
public：
LoopClosing(Map* pMap, KeyFrameDatabase* pDB, ORBVocabulary* pVoc,const bool bFixScale)：bFixScale：表示sim3中的尺度是否要计算，对于双目和rgbd，尺度是固定的，s=1,bFixScale=true；单目情况下尺度是不确定的，此时bFixScale=false，sim3中的s需要计算
void SetTracker(Tracking* pTracker)：设置追踪线程指针
void SetLocalMapper(LocalMapping* pLocalMapper)：设置局部建图线程指针
void Run()：回环检测线程主函数
void InsertKeyFrame(KeyFrame *pKF)：外部调用，由局部建图线程调用
void RequestReset()：外部调用，请求复位当前线程，在回环检测复位完成前，该函数一直保持阻塞状态
void RunGlobalBundleAdjustment(unsigned long nLoopKF)：全局BA线程，这个函数是此线程的主函数。nLoopKF是当前关键帧的ID
bool isRunningGBA()：在回环纠正的时候调用，查看当前是否已经有一个全局优化的线程在进行
bool isFinishedGBA()
void RequestFinish()：外部调用，请求终止当前线程
bool isFinished()：外部调用，判断当前回环检测线程是否已经正确终止
protected：
bool CheckNewKeyFrames()：查看mlpLoopKeyFrameQueue列表中是否有等待被插入的关键帧
bool DetectLoop()：检测回环，如果有就返回true
bool ComputeSim3()：计算当前帧与闭环帧的sim3变换等
void SearchAndFuse(const KeyFrameAndPose &CorrectedPosesMap)：通过将鼻环是相连关键帧的MapPoints投影到这些关键帧中，进行MapPoints检查与替换
void CorrectLoop()：闭环纠正
void ResetIfRequested()：当前线程调用，检查是否有外部线程请求复位当前线程，如果有的话就复位回环检测线程。
bool CheckFinish()：当前线程调用，查看是否有外部线程请求当前线程
void SetFinish()：有当前线程调用，执行完成该函数之后线程主函数退出，线程销毁

Map* mpMap：全局地图的指针
Tracking* mpTracker：跟踪线程指针
KeyFrameDatabase* mpKeyFrameDB：关键帧数据库
ORBVocabulary* mpORBVocabulary：词袋模型中的大字典
LocalMapping *mpLocalMapper：局部建图线程句柄
std::list<KeyFrame*> mlpLoopKeyFrameQueue：队列，其中存储了参与到回环检测的关键帧
std::mutex mMutexLoopQueue：操作回环检测队列中的关键帧时，使用的互斥量
float mnCovisibilityConsistencyTh：连续性阈值，构造函数中将其设置成3
KeyFrame* mpCurrentKF：正在处理的关键帧
KeyFrame* mpMatchedKF：最终检测出来的，和当前关键帧行程闭环的闭环检测关键帧
std::vector<ConsistentGroup> mvConsistentGroups：上一次执行的时候产生的连续组s
std::vector<KeyFrame*> mvpEnoughConsistentCandidates：从上面的关键帧中进行筛选得到的具有足够的“连续性”的关键帧，相当于更高层级、更加优质的闭环候选帧
std::vector<KeyFrame*> mvpCurrentConnectedKFs：和当前关键帧相连的关键帧形成的“当前关键帧组”
std::vector<MapPoint*> mvpCurrentMatchedPoints：下面变量中存储的地图点在“当前关键帧”中成功找到匹配点的地图点的集合
std::vector<MapPoint*> mvpLoopMapPoints：闭环关键帧上的所有相连关键帧的地图点
cv::Mat mScw和g2o::Sim3 mg2oScw：得到当前关键帧的闭环关键帧以后，计算出来的从世界坐标系到当前帧的sim3变换；两种表示格式
long unsigned int mLastLoopKFid：上一次闭环帧的id
bool mbRunningGBA：全局BA线程是否在进行
bool mbFinishedGBA：全局BA线程在收到停止请求之后是否停止的标志
bool mbStopGBA：由当前线程调用，请求停止当前正在进行的全局BA
std::mutex mMutexGBA：在对和全局线程标志量有关的操作的时候使用的互斥量
std::thread* mpThreadGBA：全局BA线程句柄
bool mbFixScale：如果是在双目或者rgbd情况下，就要固定尺度，表示是否要固定尺度的标志
bool mnFullBAIdx：已经进行了的全局BA次数（包含中途被打断的）

## 主流程
回环线程主函数是一个循环流程，循环遍历闭环检测队列mlpLoopKeyFrameQueue，如果有关键帧，则进行闭环检测。其中mlpLoopKeyFrameQueue中的关键帧是有LocalMapping线程发送的。
1. 检查mlpLoopKeyFrameQueue中是否有关键帧：CheckNewKeyFrames()
2. 对mlpLoopKeyFrameQueue中的数据进行闭环检测：DetectLoop()
3. 检测到闭环的情况下，计算相应关键帧的变换：ComputeSim3()
4. 使用变换关系进行闭环矫正：CorrectLoop()
5. 闭环矫正后，查看是否有外部线程请求复位当前线程：ResetIfRequested()
6. 查看外部线程是否有终止当前线程的请求，如果有的话，就跳出这个线程的主函数的主循环
7. 终止当前线程。

## 主要函数
### 1. DetectLoop()
1. 从mlpLoopKeyFrameQueue中互斥的取出关键帧mpCurrentKF
2. 判断是否进行闭环检测：如果距离上次闭环少于10帧，或者map中关键帧数量少于10，不进行闭环
3. 遍历当前关键帧的所有连接（大于15个共视地图点）关键帧，计算当前关键帧与每个共视关键帧的bow相似度得分，得到最低的分minScore
4. 根据minScore在所有关键帧（mpKeyFrameDB）中找出闭环候选帧，候选帧与当前帧没有连接。minScore作用：认为和当前关键帧具有回环关系的关键帧，不应该低于当前关键帧的相邻关键帧的最低相似度minScore。得到的这些候选关键帧和当前关键帧具有较多的公共单词，并且相似度评分都较高。如果没找到候选关键帧就返回false
5. 在候选帧中检测具有连续性的候选帧
	~~~
	概念说明：
	组（group）：对于某个关键帧，其和其具有共视关系的关键帧组成一个“组”
	子候选组（CandidateGroup）：对于某个候选的回环关键帧，其和其具有共视关系的关键帧组成一个“组”
	连续（Consistent）：不同的组之间如果共同拥有至少一个关键帧，那么成这两个组之间具有连续关系
	连续性（Consistency）：连续长度，表示累计的连续的链的长度A--B--C--D为3。具体反映在数据类型ConsistentGroup.second上。
	连续组（Consistent Group)：mvConsistentGroups存储了上次执行回环检测时，新的被检测出来的具有连续性的多个组的集合。由于组之间的连续关系是网状结构，因此可能存在一个组因为和不同的连续组链都有连续关系，而被添加两次的情况（连续性度量不同）
	连续组链：自造的称呼,类似于菊花链A--B--C--D这样形成了一条连续组链.对于这个例子中,由于可能E,F都和D有连续关系,因此连续组链会产生分叉;为了简化计算,连续组中将只会保存最后形成连续关系的连续组们(见下面的连续组的更新)
	子连续组：上面连续组中的一个组
	连续组的初始值：在遍历某个候选帧的过程中，如果该子候选组没有能够和任何一个上次的子连续组产生连续关系，那么就将添加自己组为连续组，并且连续性为0（相当于新开了一个连续链）
	连续组的更新：当前次回环检测过程中，所有被检测到的和之前的连续组链有连续性关系的组，都将在对应的连续组链后面+1，这些子候选组（可能有重复）都将成为新的连续组，换而言之连续组mvConsistentGroups中只保存连续组链中末尾的组
	~~~
	1. 遍历每一个候选帧
	2. 将自己以及与自己相连的关键帧构成一个“子候选组”：spCandidateGroup
	3. 遍历前一次闭环检测得到的连续组链mvConsistentGroups。mvConsistentGroups[i].first表示每个连续组中的关键帧集合，mvConsistentGroups[i].second为每个连续组的连续长度
	4. 遍历每个子候选组，检测子候选组中每个关键帧在子连续组中是否存在，如果有一帧共同存在于”子候选组“和之前的”子连续组“，那么”子候选组“与该”子连续组“连续，跳出当前子候选组的遍历（找到一个就行）
	5. 如果判定连续，然后判断是否达到连续的条件，取出和当前的候选组发生连续关系的子连续组的”已连续次数“，将当前候选组连续长度在原子连续组的基础上+1，如果上述连续关系没有记录在vCurrentConsistentGroups，则记录一下。如果连续长度满足要求，那么当前的这个候选关键帧是足够靠谱的
	6. 如果该”子候选组“的所有关键帧都和上次闭环无关（不连续），vCurrentConsistentGroups 没有新添加连续关系，就把子候选组全部拷贝到vCurrentConsistentGroups ，用于更新mvConsistentGroups，连续性计数器设置为0.
	遍历完成
	7. 更新连续组mvConsistentGroups = vCurrentConsistentGroups
	8. 当前闭环检测的关键帧添加到关键帧数据库中mpKeyFrameDB->add(mpCurrentKF)
	9. mvpEnoughConsistentCandidates不为空，说明检测到闭环，否则没检测到。

### 2. ComputeSim3() 
1. 遍历闭环候选帧集mvpEnoughConsistentCandidates，初步筛选出与当前关键帧的匹配特征点数大于20的候选帧集合，并为每个候选帧构造一个Sim3Solver
	1. 从闭环候选帧集中取出一帧有效关键帧pKF，设置为不能剔除，避免LocalMapping中KeyFrameCulling函数的剔除，如果帧的质量不高，则pass
	2. 将当前帧mpCurrentKF与闭环候选帧pKF匹配，通过bow加速得到匹配特征点，过滤掉匹配太少的帧。
	3. 为保留的候选帧构造Sim3求解器，使用RANSAC方法
	4. 记录保留的关键帧数量
2. 对每一个保留下来的候选帧用Sim3Solver迭代匹配，知道有一个候选帧成功，或者全部失败
	1. 取出从1.3中为当前候选帧猴枣的Sim3Solver并开始迭代，最多迭代5次，返回的Scm是候选帧pKF到当前帧mpCurrentKF的sim3变换（T12）。如果达到迭代次数还没有求出合格的sim3变换，则剔除该关键帧。
	2. 如果求出了Sim3变换，则继续匹配出更多点进行优化，因为之前SearchByBoW匹配可能会有遗漏。通过Sim3变换，投影搜索pKF1的特征点在pKF2中的匹配，同样，投影搜索pKF2的特征点在pKF1中的匹配，只有互相都匹配成功的才认为是可靠的匹配：matcher.SearchBySim3()
	3. 用新的匹配来优化sim3，优化mpCurrentKF与pKF对应的MapPoints间的Sim3，得到优化后的量gScm，只要有一个候选帧通过Sim3的求解与优化，就跳出停止对其他候选帧的判断。pKF就是与当前关键帧形成闭环的关键帧。
3. 取出与当前关键帧闭环匹配上的关键帧及其共视关键帧，以及这些共视关键帧的地图点：mpMatchedKF：是与当前关键帧形成闭环的关键帧；vpLoopConnectedKFs：包含闭环关键帧及其共视关键帧；mvpLoopMapPoints：vpLoopConnectedKFs中的地图点
4. 将mvpLoopMapPoints中的地图点投影到当前关键帧进行投影匹配，根据Sim3变换，将每个mvpLoopMapPoints投影到mpCurrentKF上，搜索新的匹配对
5. 统计当前帧与闭环关键帧的匹配地图点数目，超过40个说明成功闭环，否则失败

### 3. CorrectLoop() 闭环矫正
首先结束局部地图线程、全局BA，为闭环矫正做准备，防止回环矫正时局部地图线程中InsertKeyFrame函数插入新的关键帧
1. 根据共视关系更新当前关键帧与其他关键帧之间的连接关系，因为之前闭环检测、计算sim3中改变了该关键帧的地图点，所以需要更新
2. 通过位姿传播，得到sim3优化后，与当前相连的关键帧的位姿，以及他们的地图点，当前帧与世界坐标系之间的sim变换在ComputeSim3函数中已经确定并优化，通过相对位姿关系，可以确定这些相连的关键帧与世界坐标系之间的Sim3变换
	1. 通过mg2oScw（认为是准的）来进行位姿传播，得到当前关键帧的共视关键帧的世界坐标系下Sim3位姿
	2. 得到矫正的当前关键帧的共视关键帧位姿后，修正这些共视关键帧的地图点
	3. 将共视关键帧的Sim3转换为SE3，根据更新的Sim3更新关键帧的位姿。其实现在已经有了更新后的关键帧组中关键帧的位姿，只是上面的操作存储到了KeyFrameAndPose类型的变量中，还没有写回到关键帧变量中
	4. 根据共视关系更新当前帧与其他关键帧之间的连接。地图点的位置改变了，可能会引起共视关系、权值的改变
3. 检查当前帧的地图点与经过闭环匹配后该帧的地图点是否存在冲突，对冲突的进行替换或者填补
4. 将闭环连接关键帧组mvpLoopMapPoints投影到当前关键帧组中，进行匹配、融合、新增或者替换当前关键帧组中KF的地图点SearchAndFuse(CorrectedSim3)
5. 更新当前关键帧组之间的两级共视相连关系，得到因闭环时地图点融合而得到的新的连接关系
	1. 遍历当前帧相连关键帧组（一级相连）
	2. 得到与当前相连关键帧的相连关键帧（二级相连）
	3. 更新一级相连关键帧的连接关系
	4. 取出该帧更新后的链接关系
	5. 从连接关系中去除闭环之前的二级连接关系，剩下的连接就是由闭环得到的连接关系
	6. 从连接关系中取出闭环之前的一级连接关系，剩下的连接是由闭环的到的连接
6. 进行本质图优化，优化本质图中所有的关键帧位姿和地图点Optimizer::OptimizeEssentialGraph
7. 添加当前帧与闭环匹配帧之间的边（这个连接关系不优化）
8. 新建一个线程用于全局BA优化：mpThreadGBA = new thread(&LoopClosing::RunGlobalBundleAdjustment,this,mpCurrentKF->mnId);
9. 释放掉Local Mapping线程。

### 4. SearchAndFuse()：将闭环相连关键帧组mvpLoopMapPoints投影到当前关键帧组中，进行匹配，新增或者替换当前关键帧组中KF的地图点。因为闭环相连关键帧组mvpLoopMapPoints在地图中时间比较久，经历了多次优化，认为是准的，而当前关键帧组中的关键帧的地图点是最近新计算的，可能有累计误差。
1. 遍历带矫正的当前KF的相连关键帧pKF
2. 将mvpLoopMapPoints（与当前关键帧闭环匹配上的关键帧及其共视关键帧组成的地图点）投影到pKF帧匹配，检查地图点冲突并融合。
3. 遍历闭环帧组的所有的地图点，替换掉需要替换的地图点。