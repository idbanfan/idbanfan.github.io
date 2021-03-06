

## 1.引用依赖

```xml
<dependency>
    <groupId>com.idaoben</groupId>
    <artifactId>multiple-table-processor-starter</artifactId>
    <version>0.0.14</version>
</dependency>
```

## 2.编写配置
```java
public class Test{
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

## 3.生成代码

### 生成的文件

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

### 参数

```java
@Data
@NoArgsConstructor
public class UserQuery {
    private User user;
    private UserRole userRole;
    private Role role;
    @Data
    public static class User   {
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
    public static class UserRole   {
        private Long id;
        private ZonedDateTime createTime;
        private ZonedDateTime updateTime;
        private String value;
        private Long userId;
        private Long roleId;
    }

    @Data
    public static class Role   {
        private Long id;
        private ZonedDateTime createTime;
        private ZonedDateTime updateTime;
        private Integer isEnable;
        private String notes;
        private String rolename;
    }
}
```

### 控制器
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
        List<Map> objects = multipleTableService.get(command);
        Long count = multipleTableService.count(command);
        return Res.success(objects,count);
    }

}
```

## 4.接口使用举例

例子：根据用户查出所有的用户信息
### 入参
```json
{
  "page": {
    "current": 0,
    "size": 3,
    "total": 0
  },
  "parameter": {
    "user": {
      "id|>": 0
    },
    "userInfo": {
      "id|nonNull": 0
    }
  },
  "result": {
    "user": {},
    "userInfo": {
      "createTime": "",
      "id": 0,
      "isEnable": 0,
      "notes": "",
      "updateTime": "",
      "userId": 0
    }
  }
}
```
## 出参
```json
{
  "data": [
    {
      "userInfo": {
        "notes": "info1",
        "createTime": "2022-04-11T16:27:28",
        "updateTime": "2022-04-22T16:27:32",
        "id": 1,
        "userId": 1,
        "isEnable": 1
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
      }
    },
    {
      "userInfo": {
        "notes": "info3",
        "createTime": "2022-04-11T16:27:28",
        "updateTime": "2022-04-22T16:27:32",
        "id": 3,
        "userId": 1,
        "isEnable": 1
      }
    }
  ],
  "code": 200,
  "massage": "OK",
  "success": true,
  "total": 19
}
```

## 执行的SQL

```sql
SELECT
    `user_info`.id AS `userInfo.id`,
    `user_info`.create_time AS `userInfo.createTime`,
    `user_info`.update_time AS `userInfo.updateTime`,
    `user_info`.is_enable AS `userInfo.isEnable`,
    `user_info`.notes AS `userInfo.notes`,
    `user_info`.user_id AS `userInfo.userId`
FROM
    `user`
        LEFT JOIN `user_info` ON `user_info`.user_id = `user`.id
        AND `user_info`.is_delete = '0'
WHERE
    1 = 1
  AND `user`.is_delete = '0'
  AND `user`.id > '0'
  AND `user_info`.id IS NOT NULL
LIMIT 0,3
```