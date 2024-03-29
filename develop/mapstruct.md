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

import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

import java.util.Date;
import java.util.Optional;

@Mapper(componentModel = "spring")
public interface OrderCopier {

    @Mapping(source = "timestamp",target = "syncRequestTime")
    OrderInfo sync2Model(OrderStartRequest request);

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

