## 1.介绍

1.1 查询

1.1.1 支持一对多，多对多自动关联查询
1.1.2 支持按需返回查询结果
1.1.3 支持选择查询符号

1.2 更新

1.2.1 支持自动关联更新

1.3 代码生成

1.3.1 支持按配置生成代码，能解决80%的多表关联查询和更新接口。

## 2.引用依赖

```xml
<dependency>
    <groupId>com.idaoben</groupId>
    <artifactId>multiple-table-processor-starter</artifactId>
    <version>0.0.14</version>
</dependency>
```

## 3.编写配置

```java
public class Test {
    public static void main(String[] args) {
        YieldConfig config = YieldConfig.builder()
                .projectPath("C:/project/e360/web")
                .navicatNdm2Path("yield/default_table_model.ndm2")
                .packagePath("/yield/controller")
                .apiConfigList(
                        List.of(
                                ApiConfig.builder()
                                        .key("user")
                                        .name("用户")
                                        .query("user", "userRole", "role")
                                        .update("user", "userRole", "role"),
                                ApiConfig.builder()
                                        .key("permission")
                                        .name("权限")
                                        .query("role", "rolePermission", "permission")
                                        .update("role", "rolePermission", "permission")
                        )
                ).build();
        Yield.yield(config);
    }
}
```

## 4 代码生成

### 4.1 生成的文件

```
  ├──controller
    ├──parameter
      └──PermissionQuery.java
      └──PermissionUpdate.java
      └──UserQuery.java
      └──UserUpdate.java
    └──PermissionController.java
    └──UserController.java
```

### 4.2 控制器

```java

@RestController
@RequestMapping("/api/user")
public class UserController {

    @Resource
    private IMultipleTableService multipleTableService;

    @ApiModelProperty("保存用户")
    @PostMapping("/save")
    public Res<Object> saveUser(@RequestBody @Validated UpdateReq<UserUpdate> command) {
        List<Object> save = multipleTableService.saveWithReturn(command);
        return Res.success(save);
    }

    @ApiModelProperty("获取用户")
    @PostMapping("/get")
    public Res<Object> getUser(@RequestBody @Validated QueryReq<UserQuery> command) {
        DataWithCount andCount = multipleTableService.getAndCount(command);
        return Res.success(andCount.getData(), andCount.getCount());
    }

}
```

### 4.3 入参

```java

@Data
@NoArgsConstructor
public class UserQuery {
    private User user;
    private UserRole userRole;
    private Role role;

    @Data
    public static class User {
        private Long id;
        private ZonedDateTime createTime;
        private ZonedDateTime updateTime;
        private Integer isEnable;
        private String notes;
        private String username;
        private String nickname;
        private String password;
    }

    @Data
    public static class UserRole {
        private Long id;
        private ZonedDateTime createTime;
        private ZonedDateTime updateTime;
        private String value;
        private Long userId;
        private Long roleId;
    }

    @Data
    public static class Role {
        private Long id;
        private ZonedDateTime createTime;
        private ZonedDateTime updateTime;
        private Integer isEnable;
        private String notes;
        private String rolename;
    }
}
```

## 5.接口使用举例

例子：根据用户查出所有的用户信息

### 5.1 入参

```json
{
  "condition": [
    {
      "field": "user.username",
      "sign": "like"
    },
    {
      "field": "user.id",
      "order": "asc",
      "sign": ">"
    }
  ],
  "page": {
    "current": 0,
    "size": 3,
    "total": 0
  },
  "parameter": {
    "user": {
      "isEnable": 1
    }
  },
  "result": {
    "user": {
      "id": 0
    },
    "userInfo": {
      "notes": "",
      "createTime": "",
      "updateTime": "",
      "id": 0,
      "userId": 0,
      "isEnable": 0
    }
  }
}
```

### 5.2 出参

```json
{
  "data": [
    {
      "userInfo": {
        "notes": "info3",
        "createTime": "2022-04-11 16:27:28",
        "updateTime": "2022-04-22 16:27:32",
        "id": 3,
        "userId": 1,
        "isEnable": 1
      },
      "user": {
        "id": 1
      }
    },
    {
      "userInfo": {
        "notes": "info1",
        "createTime": "2022-04-11 16:27:28",
        "updateTime": "2022-04-22 16:27:32",
        "id": 1,
        "userId": 1,
        "isEnable": 1
      },
      "user": {
        "id": 1
      }
    },
    {
      "userInfo": {
        "notes": "info2",
        "createTime": "2022-04-11T16:27:28",
        "updateTime": "2022-04-22T16:27:32",
        "id": 2,
        "userId": 2,
        "isEnable": 1
      },
      "user": {
        "id": 2
      }
    }
  ],
  "code": 200,
  "massage": "OK",
  "success": true,
  "total": 12
}
```

### 5.3 执行的SQL

```sql
SELECT `user`.id               AS `user.id`,
       `user_info`.id          AS `userInfo.id`,
       `user_info`.create_time AS `userInfo.createTime`,
       `user_info`.update_time AS `userInfo.updateTime`,
       `user_info`.is_enable   AS `userInfo.isEnable`,
       `user_info`.notes       AS `userInfo.notes`,
       `user_info`.user_id     AS `userInfo.userId`
FROM `user`
         LEFT JOIN `user_info` ON `user_info`.user_id = `user`.id
WHERE 1 = 1
  AND `user`.is_enable = '1'
ORDER BY `user`.id ASC
LIMIT 0,3
```

## 6 详细说明

前端控制

### 6.1 查询条件，支持一对多，多对多关联查询 (自动关联)

查询用户id=1的所有角色。

```json
{
  "condition": [],
  "page": {
    "current": 0,
    "size": 2,
    "total": 0
  },
  "parameter": {
    "user": {
      "id": 1
    }
  },
  "result": {
    "role": {
      "rolename": ""
    },
    "userRole": {}
  }
}
```

结果

```json
{
  "data": [
    {
      "role": {
        "rolename": "超级角色"
      }
    },
    {
      "role": {
        "rolename": "角色2"
      }
    }
  ],
  "code": 200,
  "massage": "OK",
  "success": true,
  "total": 3
}
```

### 6.2 查询结果，按需返回

输入result，表示需要返回那些结果。返回的结果为数组，元素结构和result一致。

```json
{
  "result": {
    "user": {
      "id": 0
    },
    "userInfo": {
      "notes": "",
      "createTime": "",
      "id": 0
    }
  }
}
```

### 6.2 查询条件，支持选择查询符号和排序

```json
{
  "condition": [
    {
      "field": "user.username",
      "sign": "likeLeft"
    },
    {
      "field": "user.id",
      "order": "asc",
      "sign": ">"
    }
  ]
}
```

### 6.4 查询条件，分页

```shell
{
  "page": {
    "current": 0,
    "size": 3,
    "total": 0
  }
}
```

后端控制

### 6.6 查询

### 6.7 查询

### 6.3 查询条件，支持一层嵌套