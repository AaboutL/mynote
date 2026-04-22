---
title: 【C++】STL容器中拷贝构造函数调用
updated: 2022-10-30 02:49:42Z
created: 2022-10-30 02:37:38Z
latitude: 39.92317400
longitude: 116.44142800
altitude: 0.0000
---

# 有bug的代码
```
class FeatureChain{
public:
    FeatureChain(unsigned long chainId, int windowSize, int startId);
    FeatureChain(const FeatureChain& featureChain);

    void AddFeature(const Feature& feat);
    int GetChainLen() const;
    Eigen::Vector2d GetOffset() const;
    void EraseFront();
    void EraseBack();

public:
    unsigned long mChainId;
    int mWindowSize;
    std::vector<Feature> mvFeatures;
    int mStartIdx;
    Eigen::Vector3d mWorldPos; // position in world frame
    bool mbGood;
    bool mbPosSet;
};
FeatureChain::FeatureChain(const FeatureChain& featureChain):
mChainId(featureChain.mChainId), mWindowSize(featureChain.mWindowSize), 
mvFeatures(featureChain.mvFeatures), mStartIdx(featureChain.mStartIdx),
mWorldPos(featureChain.mWorldPos), mbGood(featureChain.mbGood), mbPosSet(featureChain.mbGood){
    std::cout << "call once" << std::endl;
}
```
上面是自定义的类，其中拷贝构造函数的实现中mbPosSet(featureChain.mbGood)这里写错了，本来应该是mbPosSet(featureChain.mbPosSet)。但是这个自定义的类在代码中并没有主动调用拷贝构造函数，所以一直以为拷贝构造函数一直没有被调用。但是在程序运行的时候，通过mbPosSet变量进行判断之后的代码的运行结果与预期不符。后来发现在上面错误的情况下，下面的代码执行的结果也是错的
```
const auto &mChains = mpFM->GetChains();
```
GetChains()的定义如下：
```
std::unordered_map<unsigned long, FeatureChain> FeatureManager::GetChains() const{
    return mmChains;
}
```
FeatureManager类的定义如下：
```
class FeatureManager{
public:
    FeatureManager(int windowSize);
    void Manage(const Frame& frame, unsigned long frameId, int startId);
    int GetChains(int chainLen, std::vector<FeatureChain>& vChains) const;
    std::unordered_map<unsigned long, FeatureChain> GetChains() const;

    void EraseFront();
    void EraseBack();

    int GetMatches(int pos1, int pos2, std::vector<std::pair<Eigen::Vector2d, Eigen::Vector2d>>& vMatches) const;
    int GetMatches(int pos1, int pos2, std::vector<Eigen::Vector2d>& vPts1, std::vector<Eigen::Vector2d>& vPts2, 
                   std::vector<unsigned long>& vChainIds) const;
    int GetMatches(int pos1, int pos2, std::vector<Eigen::Vector3d>& vPts3D, std::vector<Eigen::Vector2d>& vPts2D, 
                   std::vector<Eigen::Vector2d>& vPts1, std::vector<Eigen::Vector2d>& vPts2,
                   std::vector<unsigned long>& vChainIds) const;

    void SetWorldPos(unsigned long chainId, const Eigen::Vector3d& pos);
    void SetChainGood(unsigned long chainId, bool bGood);
    void UpdateWorldPos(unsigned long chainId, const Eigen::Vector3d& pos);

    bool IsChainGood(unsigned long chainId);
    bool IsChainPosSet(unsigned long chainId);

private:
    std::unordered_map<unsigned long, FeatureChain> mmChains;
    int mWindowSize;
};
```

上面的GetChains()函数目的是为了获取私有成员变量mmChains。mmChains是一个无序的哈希字典。所以GetChains()在返回mmChains的时候，调用了FeatureChain的拷贝构造函数。