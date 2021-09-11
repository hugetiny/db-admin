### 说明
**简洁明了的的数据库在线增删改查功能**

本插件默认为uniCloud admin基础框架的插件，也可用于提供给用户直接操作某个非系统的collection的场景。

新增的逻辑并不是和默认模块一样跳转到一个新页面添加。因为在实际操作中，新添加的数据需要参考其他数据的格式所以采用在当前页面新添一行的行为。

#### 组件相关
unicloud-db正在完善中，本插件会同步unicloud-db更新完善,unicloud-db目前后台返回数据一次性最多500条，所以每页/每次加载最多500条

大数据表性能优化中...

#### 显示相关
MongoDB是灵活的文档数据库，每条数据格式类型可能不尽相同。

目前null显示为空，修改为空值保存为null。

带双引号的是字符串类型，双引号会显示在界面上。

如果是原本不存在的键值，会有灰色的"不存在"的placeholder提示。

### 使用指南
0.下载安装后目录结构为:

├── uni_modules
│   │── db-admin               # 数据库管理所在文件夹

在菜单管理中添加数据库管理即可

1.把插件根目录config.js中的表名更改成你自己的表名collection即可，一般填写一个。

支持输入多个表名，多个表名就是联标查询，第一个collection是主表，后面的都是副表
```
多个collection字符串拼接

用于联表查询，注意主表副表之间需要在schema内以foreignKey关联（副表支持多个）。
```
[unicloud-db联表查询](https://uniapp.dcloud.io/uniCloud/unicloud-db?id=%e8%81%94%e8%a1%a8%e6%9f%a5%e8%af%a2)

查询结果如果存在主副表关系，会出现_value的键值，并且该单元格不可以被直接修改


2.修改你的你自己的collection的schema权限的properties，确保每条数据的键值都有描述。

[unicloud schema文档](https://uniapp.dcloud.net.cn/uniCloud/schema)


3.如果控制台提示schema不存在，右键uniCloud目录下database下载所有DB Schema及扩展校验函数，提示已经存在跳过即可。DB Schema会在你创建表也就是collection的时候自动创建的


### 更新的动力少不了你的支持
建议或者意见请在评论区留言
插件好用的话，拜托点赞收藏五星好评哦~