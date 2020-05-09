# mybatis使用经验

## in关键字
```
  <if test="cardSampleIdList != null and cardSampleIdList.size()>0">
        and cardSampleId in
            <foreach item="item" index="index" collection="cardSampleIdList" open="(" separator="," close=")">
                #{item}
            </foreach>
        </if>
```

## update 
```

```

## insert
```
```