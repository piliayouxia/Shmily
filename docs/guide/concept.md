# 理念

### why

现有的聊天记录导出归档项目主要分为以下几步, 以 Wechat 为例

```

导出数据库 --> 解密数据库 --> 解密资源(图片等) --> 生成 HTML (多为模板直出)

```

在此流程中有很多弊端

-   使用上
    -   大都为 HTML 模板直出，无分页，几百兆的 HTML 不可用
    -   导出的归档数据无法查询, 筛选、评论等. 失去了归档数据的意义
    -   没有图表统计，数据未体现额外价值
    -   显示数据单一， 无法多项目如 QQ Wechat 按时间线合并查看
    -   多次导出的数据 (如 2020 年导出的数据和 2021 年导出的数据) 不能累计查看并分析
-   程序上
    -   获取数据不完整，只能拿到当前 HTML 所需导出的数据, 无法拿到数据库中完整的额外字段
    -   全流程耦合，如果对产物不满意, 只能从头开始造轮子, 难以从中间(如解密后的数据库)开始

```
  基于以上问题，Shimly 的亮点在于
```

-   使用上

    -   Show 基于 Vue 组件化开发，纯前端实现，240M 数据文件浏览搜索[无压力](https://github.com/lqzhgood/Shmily-Show/blob/main/docs/Q_A.md#性能如何)
    -   支持筛选、搜索（支持正则）、评论（支持双向引用链接且显示）
    -   支持我能想到所有维度的数据图表统计
    -   支持多种类、多批次数据合并显示与统计

-   程序上
    - 完整的数据导出、不管是否 Show 有使用，能导尽导，万一别人需要呢
    - 架构 （见下）
<hr />

### 架构

基于以上问题, 我提出以下架构进行 "前后端" "功能化" 拆分, 减少后人的 **轮子**

```
|---------|----------------------- Get -----------------------|------------------------ Show -------------------------|
|---------|------- ExportDB ------|----------- ToMsg ---------|------- ModifyServer ------|---------- Web ------------|

Wechat:   导出数据库 -> 解密数据库 -> 解密资源(图片等)                       添加评论                  消息/评论 筛选查询
                                              \                        /            \            /
SMS:      导出数据   ->                       导出为标准数据 -> 合并处理->  -修改错误-  -> 存档显示 ->  时间线合并显示
                                              /                        \            /            \
MobileQQ: 导出数据库 ->              解密资源(图片等)                       数据预处理                 大数据图表

更多来源...
```

将过程分为 `Get`(后端采集) 和 `Show`(前端展示), 并细化为 4 个部分(库), 中间使用 [Shimly-Msg](./use/msg/schema.md) 标准格式进行融合

-   Get
    -   ExportDB 从设备导出原始数据(库)并解密
    -   ToMsg 从解密数据库导出为标准数据 `${Shmily-Msg}`, 并解密资源(图片等)
        -   导出的数据为 json 格式, 并尽可能导出所有数据, 不管用没用，历史不嫌多
-   Show
    -   ModifyServer 用于手动修改 _数据文件_ 和 _添加评论_
    -   Web 用于最终显示 _时间线_ 和 _大数据图表_

让过程变为 4 个部分后

-   完全解耦, 便于各种二开造轮子, 便于各个阶段变为轮子
-   使用现代 Web 框架组件化开发，展示上有了更多可能性. 更多 `Show` 的问答参考 [Shmily-Show/docs/Q_A.md](https://github.com/lqzhgood/Shmily-Show/blob/main/docs/Q_A.md)
-   多种类、多批次的 Get 都可以融合在一个 Show 中时间线展示

### 问题

拆分后势必增加运行成本, 我希望通过批处理等脚本方式粘合各个库之间的运行, 尽量做到开箱即用. 也会完善文档让非程序员也能阅读明白, 如有不懂之处, 欢迎 Github 提 [issues](https://github.com/lqzhgood/Shmily/issues/new).

### 感想

花了近 3 年用 JS 写了一大堆的 [Get](./use/get.md), 我发现还是原生语言开发最方便, 如 MobileQQ 中 `Java Serializable` 的[问题](https://github.com/ZhangJun2017/QQChatHistoryExporter/issues/4) 用 `JS` 太难了, 所以以后我还是希望由开源社区贡献与维护 [Get](./use/get.md), 我会尽力维护 [Show](https://github.com/lqzhgood/Shmily-Show) 部分.

> 数据本身就归属用户, 不提供数据导出的都是耍流氓.
