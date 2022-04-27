# mapstruct 属性拷贝工具使用

## 添加依赖

```xml
<!--bean属性转换工具-->
      <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-jdk8</artifactId>
        <version>1.2.0.Final</version>
      </dependency>
      <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-processor</artifactId>
        <version>1.2.0.Final</version>
      </dependency>
```

## 写接口，配合spring使用

```java
package com.ruqimobility.taxi.copier;

import com.ruqimobility.taxi.dao.model.TaxiOfflineOrderInfo;
import com.ruqimobility.taxi.request.TaxiOfflineOrderEndRequest;
import com.ruqimobility.taxi.request.TaxiOfflineOrderStartRequest;
import com.ruqimobility.taxi.request.TaxiOfflineSyncOrderRequest;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

import java.util.Date;
import java.util.Optional;

/**
 * 扬招单数据库实体类和请求的转换
 * @author miiarms
 * @date 2022/4/26 9:55
 **/
@Mapper(componentModel = "spring")
public interface TaxiOrderCopier {

    /**
     * 扬招单开始服务同步请求入参转po
     * @author miiarms
     * @date 2022/4/26 9:58
     **/
    @Mapping(source = "timestamp",target = "syncRequestTime")
    TaxiOfflineOrderInfo syncStart2Model(TaxiOfflineOrderStartRequest request);

   /**
    * 扬招单结束服务同步请求入参转po
    * @author miiarms
    * @date 2022/4/26 10:00
    **/
    @Mapping(source = "timestamp",target = "syncRequestTime")
    TaxiOfflineOrderInfo syncEnd2Model(TaxiOfflineOrderEndRequest request);

    /**
     * 扬招单全量信息同步请求 转 数据库实体
     * @author miiarms
     * @date 2022/4/26 10:00
     **/
    @Mapping(source = "timestamp",target = "syncRequestTime")
    TaxiOfflineOrderInfo syncFull2Model(TaxiOfflineSyncOrderRequest request);


    /**
     * Long 转 date
     * @author miiarms
     * @date 2021/10/27 15:22
     **/
    default Date timestamp2Date(Long value) {
        return Optional.ofNullable(value).map(Date::new).orElse(null);
    }
    /**
     * Long 转 date
     * @author miiarms
     * @date 2021/10/27 15:22
     **/
    default Long date2Timestamp(Date date) {
        return Optional.ofNullable(date).map(Date::getTime).orElse(null);
    }

    default Long str2Long(String str){
        return Optional.ofNullable(str).map(Long::parseLong).orElse(null);
    }

}
```

