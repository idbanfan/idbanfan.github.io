



## 1.使用方式

```java
    @LogRecord(express = "操作人：${getCurrentUser()}，门店代理人：${getAgent(store?.id)}")
    public void updateStore(Store store) {
        //...
    }
```

## 2.日志效果

```shell
操作人：admin，门店ID：1 ，门店代理人：storeAgent
```

## 3.扩展方式

实现LogFunction接口即可

```java
@Component
public class GetStoreAgent implements LogFunction<Integer> {
    @Override
    public String getName() {
        return  "getStoreAgent";
    }

    @Override
    public String apply(Integer storeId) {
        //storeIdToAgent
        return "storeAgent";
    }
}

```