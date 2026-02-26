# 接口设计规范

本文档定义 Controller、Service、Mapper 三层接口的设计规范和示例。

## 1. 三层架构说明

```
Controller 层 (控制器层)
    ↓ 调用
Service 层 (业务逻辑层)
    ↓ 调用
Mapper 层 (数据访问层)
```

- **Controller**: 处理 HTTP 请求，参数校验，权限控制
- **Service**: 实现业务逻辑，事务控制
- **Mapper**: 数据库操作，SQL 执行

## 2. Controller 接口设计

### 2.1 基本规范

```java
@RestController
@RequestMapping("/api/module")
@Tag(name = "模块名称")
public class ModuleController {

    @GetMapping("/page")
    @Operation(summary = "分页查询")
    @PreAuthorize("@ss.hasPermission('module:query')")
    public CommonResult<PageResult<ModuleRespVO>> getPage(
        @Valid ModulePageReqVO pageReqVO) {
        // 实现
    }
}
```

### 2.2 接口命名规范

| 操作类型 | HTTP 方法 | 路径 | 方法名 |
|----------|-----------|------|--------|
| 分页查询 | GET | /page | getPage |
| 列表查询 | GET | /list | getList |
| 详情查询 | GET | /get | get{Entity} |
| 新增 | POST | /create | create{Entity} |
| 修改 | PUT | /update | update{Entity} |
| 删除 | DELETE | /delete | delete{Entity} |

### 2.3 完整示例

#### 分页查询接口

```java
@GetMapping("/page")
@Operation(summary = "获取公海池线索分页")
@PreAuthorize("@ss.hasPermission('crm:lead-pool:query')")
public CommonResult<PageResult<LeadPoolRespVO>> getLeadPoolPage(
    @Valid LeadPoolPageReqVO pageReqVO) {

    PageResult<LeadPoolDO> pageResult = leadPoolService.getLeadPoolPage(pageReqVO);
    return success(LeadPoolConvert.INSTANCE.convertPage(pageResult));
}
```

**权限码**: `crm:lead-pool:query`

**请求参数** (LeadPoolPageReqVO):
- `brandName`: 品牌名（模糊搜索）
- `brandCountry`: 品牌国家/地区（多选）
- `contactName`: 联系人姓名（模糊搜索）
- `mobile`: 手机号（模糊搜索）
- `leadSourceIds`: 线索来源ID列表（多选）
- `leadStatus`: 线索状态（多选，默认UNASSIGNED）
- `referrerId`: 介绍人ID（多选）
- `createTime`: 创建时间范围
- `pageNo`: 页码
- `pageSize`: 每页大小

**响应数据**: 分页结果，包含线索列表

#### 详情查询接口

```java
@GetMapping("/get")
@Operation(summary = "获取公海池线索详情")
@PreAuthorize("@ss.hasPermission('crm:lead-pool:query')")
public CommonResult<LeadPoolRespVO> getLeadPool(
    @RequestParam("id") Long id) {

    LeadPoolDO leadPool = leadPoolService.getLeadPool(id);
    return success(LeadPoolConvert.INSTANCE.convert(leadPool));
}
```

**权限码**: `crm:lead-pool:query`

**请求参数**:
- `id`: 线索ID

**响应数据**: 线索详情对象

#### 保存接口（新增/修改）

```java
@PostMapping("/save")
@Operation(summary = "保存线索")
@PreAuthorize("@ss.hasPermission('crm:lead-pool:create')")
public CommonResult<Long> saveLeadPool(
    @Valid @RequestBody LeadPoolSaveReqVO saveReqVO) {

    Long id = leadPoolService.saveLeadPool(saveReqVO);
    return success(id);
}
```

**权限码**: `crm:lead-pool:create`

**请求参数** (LeadPoolSaveReqVO):
- `id`: 线索ID（修改时必填，新增时为空）
- `brandName`: 品牌名称
- `contactName`: 联系人姓名
- `mobile`: 手机号
- `leadSourceId`: 线索来源ID
- ...其他业务字段

**响应数据**: 保存后的线索ID

#### 删除接口

```java
@DeleteMapping("/delete")
@Operation(summary = "删除线索")
@PreAuthorize("@ss.hasPermission('crm:lead-pool:delete')")
public CommonResult<Boolean> deleteLeadPool(
    @RequestParam("id") Long id) {

    leadPoolService.deleteLeadPool(id);
    return success(true);
}
```

**权限码**: `crm:lead-pool:delete`

**请求参数**:
- `id`: 线索ID

**响应数据**: 删除成功返回 true

### 2.4 接口文档格式

每个接口需要说明以下内容：

```markdown
#### [接口名称]

​```java
[接口代码]
​```

**权限码**: `module:action`

**请求参数**:
- `param1`: 参数说明
- `param2`: 参数说明

**响应数据**: 响应说明
```

## 3. Service 接口设计

### 3.1 基本规范

```java
public interface ModuleService {

    /**
     * 分页查询
     */
    PageResult<ModuleDO> getPage(ModulePageReqVO pageReqVO);

    /**
     * 详情查询
     */
    ModuleDO get(Long id);

    /**
     * 保存（新增/修改）
     */
    Long save(ModuleSaveReqVO saveReqVO);

    /**
     * 删除
     */
    void delete(Long id);
}
```

### 3.2 方法命名规范

| 操作类型 | 方法名 | 返回值 |
|----------|--------|--------|
| 分页查询 | getPage | PageResult<DO> |
| 列表查询 | getList | List<DO> |
| 详情查询 | get | DO |
| 统计查询 | count | Long/Integer |
| 保存 | save | Long (返回ID) |
| 删除 | delete | void |
| 批量操作 | batch{Action} | void/List |

### 3.3 完整示例

```java
public interface LeadPoolService {

    /**
     * 分页查询公海池线索
     */
    PageResult<LeadPoolDO> getLeadPoolPage(LeadPoolPageReqVO pageReqVO);

    /**
     * 获取公海池统计卡片
     */
    LeadPoolStatisticsRespVO getStatistics();

    /**
     * 获取线索详情
     */
    LeadPoolDO getLeadPool(Long id);

    /**
     * 保存线索（新增/修改）
     */
    Long saveLeadPool(LeadPoolSaveReqVO saveReqVO);

    /**
     * 删除线索
     */
    void deleteLeadPool(Long id);

    /**
     * 批量分配线索
     */
    void batchAssignLeads(List<Long> leadIds, Long ownerId);
}
```

## 4. Mapper 接口设计

### 4.1 基本规范

```java
@Mapper
public interface ModuleMapper extends BaseMapperX<ModuleDO> {

    default PageResult<ModuleDO> selectPage(ModulePageReqVO reqVO) {
        return selectPage(reqVO, new LambdaQueryWrapperX<ModuleDO>()
            .likeIfPresent(ModuleDO::getName, reqVO.getName())
            .eqIfPresent(ModuleDO::getStatus, reqVO.getStatus())
            .betweenIfPresent(ModuleDO::getCreateTime, reqVO.getCreateTime())
            .orderByDesc(ModuleDO::getId));
    }
}
```

### 4.2 方法命名规范

| 操作类型 | 方法名 | 说明 |
|----------|--------|------|
| 分页查询 | selectPage | 返回分页结果 |
| 列表查询 | selectList | 返回列表 |
| 单条查询 | selectOne | 返回单条记录 |
| 统计查询 | selectCount | 返回数量 |
| 插入 | insert | 插入记录 |
| 更新 | update | 更新记录 |
| 删除 | delete | 删除记录 |

### 4.3 完整示例

```java
@Mapper
public interface LeadPoolMapper extends BaseMapperX<LeadPoolDO> {

    /**
     * 分页查询公海池线索
     */
    default PageResult<LeadPoolDO> selectPage(LeadPoolPageReqVO reqVO) {
        return selectPage(reqVO, new LambdaQueryWrapperX<LeadPoolDO>()
            .likeIfPresent(LeadPoolDO::getBrandName, reqVO.getBrandName())
            .inIfPresent(LeadPoolDO::getBrandCountry, reqVO.getBrandCountry())
            .likeIfPresent(LeadPoolDO::getContactName, reqVO.getContactName())
            .likeIfPresent(LeadPoolDO::getMobile, reqVO.getMobile())
            .inIfPresent(LeadPoolDO::getLeadSourceId, reqVO.getLeadSourceIds())
            .inIfPresent(LeadPoolDO::getLeadStatus, reqVO.getLeadStatus())
            .inIfPresent(LeadPoolDO::getReferrerId, reqVO.getReferrerId())
            .betweenIfPresent(LeadPoolDO::getCreateTime, reqVO.getCreateTime())
            .orderByDesc(LeadPoolDO::getId));
    }

    /**
     * 统计今日新增线索数
     */
    default Long selectTodayNewCount() {
        LocalDateTime startOfDay = LocalDateTime.now().with(LocalTime.MIN);
        return selectCount(new LambdaQueryWrapperX<LeadPoolDO>()
            .ge(LeadPoolDO::getCreateTime, startOfDay));
    }

    /**
     * 统计未分配线索数
     */
    default Long selectUnassignedCount() {
        return selectCount(new LambdaQueryWrapperX<LeadPoolDO>()
            .eq(LeadPoolDO::getLeadStatus, LeadStatusEnum.UNASSIGNED));
    }
}
```

## 5. VO 对象设计

### 5.1 请求对象（ReqVO）

```java
@Data
public class ModulePageReqVO extends PageParam {

    @Schema(description = "名称", example = "张三")
    private String name;

    @Schema(description = "状态", example = "1")
    private Integer status;

    @Schema(description = "创建时间")
    @DateTimeFormat(pattern = FORMAT_YEAR_MONTH_DAY_HOUR_MINUTE_SECOND)
    private LocalDateTime[] createTime;
}
```

### 5.2 响应对象（RespVO）

```java
@Data
public class ModuleRespVO {

    @Schema(description = "ID", requiredMode = Schema.RequiredMode.REQUIRED, example = "1")
    private Long id;

    @Schema(description = "名称", requiredMode = Schema.RequiredMode.REQUIRED, example = "张三")
    private String name;

    @Schema(description = "状态", requiredMode = Schema.RequiredMode.REQUIRED, example = "1")
    private Integer status;

    @Schema(description = "创建时间", requiredMode = Schema.RequiredMode.REQUIRED)
    private LocalDateTime createTime;
}
```

## 6. 权限码规范

权限码格式：`模块:操作`

常用操作：
- `query`: 查询
- `create`: 新增
- `update`: 修改
- `delete`: 删除
- `export`: 导出
- `import`: 导入

示例：
- `crm:lead-pool:query`
- `crm:lead-pool:create`
- `crm:lead-pool:update`
- `crm:lead-pool:delete`

## 7. 接口文档模板

```markdown
### 4.1 Controller 接口

#### 4.1.1 分页查询接口

​```java
[接口代码]
​```

**权限码**: `module:query`

**请求参数**:
- `param1`: 参数说明
- `param2`: 参数说明

**响应数据**: 响应说明

### 4.2 Service 接口

​```java
[Service 接口定义]
​```

### 4.3 Mapper 接口

​```java
[Mapper 接口定义]
​```
```
