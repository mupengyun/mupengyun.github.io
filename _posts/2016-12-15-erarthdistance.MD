---
layout: post
title: PostgreSQL 地理计算
date: 2016-12-15
tags: Groovy   
---

PostgreSQL 计算地球上两点之间的距离


### 概要

- [完美的参考文章](http://johanndutoit.net/searching-in-a-radius-using-postgres/)
- 步骤
* `CREATE EXTENSION cube;` 
* `CREATE EXTENSION earthdistance;`

### entity 设计，注此代码为groovy,NamedNativeQueries 中括号表示问题
```
@CompileStatic
@NamedNativeQueries(
        [
                @NamedNativeQuery(
                        name = "findEarthDistanceWithRadius",
                        query = " SELECT id,city,store_type,name,address,lng,lat,earth_distance(ll_to_earth(:lng,:lat),ll_to_earth(lng,lat)) as distance FROM store where earth_box(ll_to_earth(:lng,:lat),:radius) @> ll_to_earth(lng,lat) ORDER BY distance ASC ",
                        resultSetMapping = "returnPoiEntity"
                ),
                @NamedNativeQuery(
                        name = "findEarthDistanceWithoutRadius",
                        query = " SELECT id,city,store_type,name,address,lng,lat,earth_distance(ll_to_earth(:lng,:lat),ll_to_earth(lng,lat)) as distance FROM store  ORDER BY distance ASC ",
                        resultSetMapping = "returnPoiEntity"
                )
        ]

)
@SqlResultSetMappings(
        [
                @SqlResultSetMapping(
                        name = "returnPoiEntity",
                        entities = [
                                @EntityResult(
                                        entityClass = Store.class
                                )
                        ],
                        columns = [
                                @ColumnResult(name = "distance")
                        ]
                )
        ]
)
@Entity
@Table(name = "store")
class Store implements Serializable {

    /**
     * 默认id
     */
    @Id
    @GeneratedValue(generator = "system_uuid")
    @GenericGenerator(name = "system_uuid", strategy = "uuid2")
    @NotNull
    @Column(unique = true, nullable = false)
    String id;

    /**
     * 门店所在城市
     */
    @Column(nullable = false, length = 32)
    @NotNull
    int city;

    /**
     * 门店类型
     */
    @Column(nullable = false, name = "store_type")
    @NotNull
    @Enumerated(EnumType.STRING)
    StoreType storeType;

    /**
     * 门店名称
     */
    @Column(nullable = false, length = 256)
    @NotNull
    String name;

    /**
     * 门店地址
     */
    @Column(nullable = false, length = 256)
    @NotNull
    String address;

    /**
     * 经度
     */
    @Column(nullable = false)
    @NotNull
    double lng;

    /**
     * 纬度
     */
    @Column(nullable = false)
    @NotNull
    double lat;
}
```
**执行**

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
     *  根据给定经纬度,获取给定半径的门店信息列表
     * @param lng 经度
     * @param lat 纬度
     * @param radius 范围
     * @return List < StorePoiDTO >  范围内门店信息列表
     */
    @Override
    List<StorePoiDTO> findNearestStoresByLocation(double lng, double lat, double radius) {

        Query query;
        logger.info("开始本地查询")
        if(radius){
            query = entityManager.createNamedQuery("findEarthDistanceWithRadius");
            query.setParameter("lng", lng)
            query.setParameter("lat", lat)
            query.setParameter("radius", radius * 1000)
        }else{
            query = entityManager.createNamedQuery("findEarthDistanceWithoutRadius");
            query.setParameter("lng", lng)
            query.setParameter("lat", lat)
        }

        List<StorePoiDTO> storePoiDTOList = new LinkedList<StorePoiDTO>();
        List resultList = query.getResultList()
        logger.info(objectMapper.writeValueAsString(resultList))
        for (it in resultList) {
            List<Object> list = (List) it
            def storePoiDTO = new StorePoiDTO();
            storePoiDTO.storeDTO = storeConverte.converteStore((Store) list.get(0))
            // 默认距离为公里
            storePoiDTO.distance = storeUtil.decFormat((double) list.get(1) / 1000.toDouble())
            storePoiDTOList.add(storePoiDTO)
        }
        logger.info(objectMapper.writeValueAsString(storePoiDTOList))
        return storePoiDTOList
    }

```

### 性能提升

* 创建索引

```
CREATE INDEX distance on store USING gist(ll_to_earth(lat, lng));  
```