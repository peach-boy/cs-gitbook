## 运用策略模式替换switch case


### 一、业务场景
一个接口需要根据前段传入的资产类型（assetType），调用不同的业务类中的资产明细方法（dealList），最初的做法就是最常见的switch case或者if/else调用不同的service。相信每个程序员看到代码中大段的判断逻辑代码都会很头疼，刚好之前看过《重构既有代码》这本Java届的经典书，里面就明确指出switch case违反开闭原则，可以用策略模式改写。

<!--more-->

### 二、实现逻辑

1.定义统一资产接口，每种资产实现此接口，代码如下：
```java
public interface AssetService {
    Integer assetType();

    DealListResponse queryDealList(Integer assetId, Integer status, Integer sinceId, Integer count);
}
```
2.资产service实现接口，并重写assetType()和queryDealList()方法，代码如下：

```java
@Service
public class CashService implements AssetService {
    @Autowired
    private CashDetailService cashDetailService;

    @Override
    public Integer assetType() {
        return AssetType.CASH.getId();
    }
	 @Override
    public DealListResponse queryDealList(Integer assetId, Integer status, Integer sinceId, Integer count) {
		   
	 }
    
```
3.统一分发路由类，根据assetType调用对应的service,代码如下：
```java
@Service
public class AssetServiceRoute {
    @Resource
    private AssetService[] assetServices;


    public AssetService route(Integer assetType) {
        for (AssetService assetService : assetServices) {
            if (Objects.equals(assetType, assetService.assetType())) {
                return assetService;
            }
        }
        throw new BusinessRuntimeException(GiveErrorCode.CHECK_PARAM_FAILED);
    }

}
```
4.调用，代码如下；
```java
 public DealListResponse dealList(Integer assetType, Integer assetId, Integer status, Integer sinceId, Integer count) {
        AssetService assetService = assetServiceRoute.route(assetType);
        if (assetService == null) {
            throw new BusinessRuntimeException(GiveErrorCode.INVALID_PARAMS);
        }
        return assetService.queryDealList(assetId, status, sinceId, count);
    }
```
### 三、代码分析
意图：定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。
主要解决：在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护（违反开闭原则）。
何时使用：一个系统有许多许多类，而区分它们的只是他们直接的行为。
如何解决：将这些算法封装成一个一个的类，任意地替换。
关键代码：实现同一个接口。
