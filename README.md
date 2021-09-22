# 商品实时推荐系统 flink-recommend-system

## 1. 系统架构

* 1.1 系统架构

* 1.2 模块说明

  在日志数据模块中，主要分为6个Flink任务：

  * 用户-产品浏览历史 --> 实现基于系统过滤的推荐逻辑

    通过Flink去记录用户浏览过这个类目下的那些产品，为后面的基于协同过滤做准备，实时记录用户的评分到Hbase中，为后续离线处理做准备。

    数据存储在Hbase的p_history表

  * 用户-兴趣 --> 实现基于上下文的推荐逻辑

    根据用户对同一个商品操作计算兴趣度，计算规则通过操作时间的间隔（如：购物 - 浏览  < 100s）则判定为一定兴趣事件，通过Flink的valueState实现，如果用户的操作action=3(收藏)，则清除这个产品的state,如果超过100s没有出现action=3的事件，也会清除这个state。

    数据存储在Hbase的u_interest表

  * 用户画像计算 --> 实现基于标签的推荐逻辑

    按照三个维度去计算用户用户画像，分别是用户的颜色兴趣、用户的产地兴趣、用户的风格兴趣。根据日志不断的修改用户画像的数据，记录在Hbase表。

    数据存储在Hbase的user表

  * 产品画像记录 --> 实现基于标签的推荐逻辑

    用两个维度记录产品画像，一个是喜爱该产品的年龄段、一个是性别

    数据存储在Hbase的prod表

  * 事实热度榜 --> 实现基于热度推荐逻辑

    通过Flink的时间窗口机制，统计当前时间的实时热度，并将数据缓存到Redis中，通过Flink的窗口机制计算实时热度，使用ListState保存热度榜

    数据存储到redis中，按照时间戳list

  * 日志导入 

    从kafka接收到数据，直接导入Hbase事实表，保存完整的日志log,日志包含了用户id,用户操作的产品id,操作时间，行为（购买、点击、推荐等）

    数据按照窗口统计数据大屏需要的数据，返回展示

    数据存储到Hbase的con表。

## 2.推荐逻辑说明

* 2.1 基于热度的推荐逻辑

* 2.2 基于产品画像的产品相似度计算方法

  



