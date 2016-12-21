---
layout: post
title: MongoDB 地理计算
date: 2016-12-19
tags: Groovy   
---

Mongo 计算地球上两点之间的距离


### 概要

- [完美的参考文章](https://docs.mongodb.com/v3.2/reference/command/geoNear/)
- 步骤
* `db.store.createIndex({loc: "2dsphere"})` 

### entity 设计，注此代码为groovy,NamedNativeQueries 中括号表示问题
```
@Document
class Store {

    /**
     * 默认id
     */
    @Id
    String id;

    /**
     * 门店所在城市
     */
    int city;

    /**
     * 门店类型
     */
    StoreType storeType;

    /**
     * 门店名称
     */
    String name;

    /**
     * 门店地址
     */

    String address;

    /**
     * 经度
     */
    double lng;

    /**
     * 纬度
     */
    double lat;

    /**
     * 经纬度坐标 
     */
    @GeoSpatialIndexed
    Point loc
}
```
### 执行

```
@CompileStatic
@Service("storePoiService")
class StorePoiService implements StorePoiFacade {

    private static final Logger logger = LoggerFactory.getLogger(StorePoiService.class);

    @PersistenceContext
    private EntityManager entityManager

    @Autowired
    StoreConverte storeConverte

    @Autowired
    ObjectMapper objectMapper

     /**
     *  根据给定经纬度,获取给定范围的门店信息列表
     * @param lng 经度
     * @param lat 纬度
     * @param radius 范围
     * @return List < StorePoiDTO >  范围内门店信息列表
     */
    @Override
    List<StorePoiDTO> findNearestStoresByLocation(double lng, double lat, double radius) {

        Point point = new Point(lng, lat);
        NearQuery nearQuery = NearQuery.near(point, Metrics.KILOMETERS);
        // 范围radius 的points 单位KM
        if (radius) {
            nearQuery.maxDistance(new Distance(radius, Metrics.KILOMETERS))
        } else {
            nearQuery.maxDistance(new Distance(distanceMultiplier, Metrics.KILOMETERS))
        }
        // 是否计算距离
        nearQuery.spherical(true)
        nearQuery.inKilometers();
        GeoResults<Store> geoResult = mongoTemplate.geoNear(nearQuery, Store.class) as GeoResults<Store>;
        def storePoiDTOList = new ArrayList();
        for (result in geoResult.content) {
            def store = result.getContent()
            def distance = result.distance.getValue()
            def storePoiDTO = new StorePoiDTO();
            // 转化storePoiDTO
            storePoiDTO.storeDTO = storeConverte.converteStore(store)
            storePoiDTO.distance = storeUtil.decFormat(distance)
            storePoiDTOList.add(storePoiDTO)
        }
        logger.info(objectMapper.writeValueAsString(storePoiDTOList))
        return storePoiDTOList
    }

```

### 问题

`no geo indices for geoNear` 需要建立索引