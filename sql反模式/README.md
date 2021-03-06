### 目录

---

**1** 引言

#### 逻辑性数据库设计模式

**2** 乱穿马路
		
		解决多对多关系
		
		一个字段存储用逗号分隔的数据(违反第一范式)

 		uid     pid
 		1       1,2,3
		
		交叉表(效率高)

 		uid     pid
 		1       1
 		1       2
 		1       3
 		2		2
	
**3** 单纯的树

		邻接表: 表中记录父id来表示层级关系
		递归查询: mysql不支持
		枚举路径: 专门有一个字段记录层级路径 1/2/3
		嵌套集: 
			nsleft: 小于该节点所有后代ID
			nsright: 大于该节点所有后代的ID
		闭包表: 单独一张表记录层级关系

**4** 需要ID

		1.理解主键和伪主键
		2.用自然键作主键
		3.用复合键作主键
		
		不要盲目的使用id,分析清楚"主键"和"伪主键",不要为了主键而主键.
		但是具体业务场景要具体分析,我再笔记内也记录了在使用mysql.innodb引擎的一些关于主键的考量.

**5** 不用钥匙的入口

		外键的重要性:
		
		1.使用外键保证数据的一致性.
		2.使用外键减少代码的量(用额外的代码来维护数据的完整性),越少的代码意味着越少的错误

**6** 实体 - 属性 - 值

		EAV 设计:
		每个属性都是一行数据,即:
		
		id,
		attr_name,
		attr_value
		
		不要使用EAV设计,可以使用如下方法:
		
		1.单表继承
		所有的字段都放在一张表中,用单独一个type字段来区分每行数据的归属.会有冗余数据.
		
		一张表
		
		order表:
		//order公共属性
		type 字段,区分属于 order_car 还是 order_computer
		//order_car 汽车订单的字段
		//order_computer 电脑订单的字段
		
		2.实体表继承
		每个实体都存放在单独的表中,相同的属性在每张表中都会重复出现.
		
		二张表
		
		order_car表:
		//order 属性
		//order_car 汽车订单的字段
		
		order_computer表:
		//order 属性
		//order_computer 电脑订单的字段

		
		3.类表继承
		模拟面向对象的继承方法,把公共的属性抽离成一张表.其他自己私有的属性一张表.列入:
		order,//包含订单的通用属性,状态,支付时间等等...
		order_car//汽车订单中独有的字段,表内的 order_id 和 order 表的 id 为对应关系
		order_computer//电脑订单中独有的字段,表内的 order_id 和 order 表的 id 为对应关系
		
		三张表
		
		order表:
		//order公共属性
		
		order_car表:
		//order_car 汽车订单的字段
		
		order_computer表:
		//order_computer 电脑订单的字段
		
		4.半结构化数据
		类似单表继承,但是把每个种类的私有的字段统一以序列化的形式存放在一个text字段中.用的时候通过程序反序列化取出.
		
		order表:
		//order公共属性
		type 字段,区分属于 order_car 还是 order_computer
		//attributes,序列化存储.如果 type == order_car 则该字段为序列化的 order_car 汽车订单私有字段.如果 type == order_computer 则该字段为序列化的 order_computer 电脑订单私有字段,

**7** 多态关联

		多态关联是comments中有一个字段type,来标记是来源于bugs表还是features表. 这样不能使用外键来约束. 必须使用程序来维护关系.
		
		解决方案:
		1. 交叉表 建立一个bugsComments表和featuresComments表. 只记录之间的关系, 可以使用外键约束
		2. 创建公用的超级表(基表), 让bugs和features都继承一张issues表. 让comments和issues来建立外键约束
		

**8** 多列属性

		解决一对多关系 (和第2章的区别是 一对多 和 多对多 关系)

		CREATE TABLE Bugs (
			bug_id		SERIAL	PRIMARY KEY,
			description	CARCHAR(1000),
			tag1		VARCHAR(20),
			tag2		VARCHAR(20),
			tag3		VARCHAR(20),
		);
		
		bug_id	description		tag1	tag2	tag3
		1		xxxx			xx		xx		xx
		
		修改为添加一个'从属表':
		
		CREATE TABLE Tags (
			bug_id		BIGINT	UNSIGNED NOT NULL,
			tag			VARCHAR(20),
			PRIMARY KEY (bug_id, tag),
			FOREIGN KET (bug_id) REFERENCES Bugs(bug_id)
		);	
	
**9** 元数据分裂

		分表,比如按照每年一张表, 也可能按照列来分(每年添加一个新的列,这种场景我遇见的不多)
		
		缺点:
		1. 不断产生的新表
		2. 没有任何办法自动地对数据和相关表名做限制(除非添加CHECK约束)
		3. 如果上面的表中的数据要修改日期, 就需要先从旧表移除, 在添加到新表
		4. 每张分的表都有自己的主键, 无法确保全部数据的唯一性, 除非专门定义一张记录逐渐的表
		5. 跨表查询, 使用 UNION 来联合其他表的查询结果
		6. 如果一张表的列修改了, 还要手动的同步其他表
		7. 无法使用外键(一个外键必须指定单个表）
		
		解决方案:
		1. 水平分区 PARTITION
		2. 垂直分区 把 BLOB 和 TEXT 单独存储到一张表
		3. 创建关联表来统计查询.

#### 物理数据库设计模式

**10** 取值错误
	
		FLOAT 遵循IEEE754标准,表示十进制数有误差, 是非精确值.
		
		使用 NUMERIC 或 DECIMAL 来代替 FLOAT 使用

**11** 每日新花样

		不要使用MySQL的ENUM, 使用单独的一张表来维护所有配置项, 其实就是数据字典.
		然后使用外键来约束这些对于配置的引用.

**12** 幽灵文件

		讨论文件是以二进制`blob`(mysql)形式一起存到数据库, 还是独立存储到另外一个地方.
		
		存储到数据库, 就可以享受数据库的遍历(事务, 回滚, 备份)，但是随着而来的缺点: 备份较大, 操作文件不便捷.
		
		我本人还是倾向于文件存储在数据外边. 数据库备份一次, 文件额外备份. 而且现在更多的网上文件存储服务, 这样也不用担心备份问题了.

**13** 乱用索引

		正确的使用索引: (无索引, 过多索引)
		
		复合索引的规则: 最左前缀
		
		不能使用索引的情况(具体看连接的笔记)
		
		索引分离率: 所有不重复的值得数量和总记录条数之比. 分离率越低,索引的效率越低
		
		使用Mysql的 EXPLAIN 分析建立的索引
		
		定期重建索引 ALALYZE TABLE, OPTIMIZE TABLE

#### 查询反模式

**14** 对未知的恐惧 

		NULL + 12345 结果为 NULL
		NULL || "string" 结果为 NULL
		
		NULL = 0 结果为 NULL
		NULL <> 12345 结果为 NULL
		NULL = NULL 结果为 NULL
		NULL <> NULL 结果为 NULL
		
		NULL AND TRUE 结果为 NULL
		NULL AND FALSE 结果为 FALSE
		NULL OR FALSE 结果为 FALSE
		NULL OR TRUE 结果为 TRUE
		NOT(NULL) 结果为 NULL
	
**15** 模棱两可的分组

		单值规则: 对于每个分组来说都必须返回且仅返回一个值
		
		SELECT product_id, MAX(date_reported) AS latest, bug_id
		FROM Bugs JOIN BugsProducts USING (bug_id)
		GROUP BY product_id;
		一个给定的product_id有很多不同的bug_id.
		
		由于对这些"额外"的列没有办法保证它们满足单值规则, 数据库就假设它们违反了单值规则,
		MySQL中返回的值是这一组结果的第一条, 其排序规则是按照实际的物理存储顺序来定义的,
		
		问题解决思路:
		
		1. 只查询功能依赖的列
		2. 使用关联子查询
		3. 使用衍生表
		4. 使用JOIN
		5. 对额外的列使用聚合函数
		6. 连接同组所有值(GROUP_CONTACT()函数)

**16** 随机选择

		使用`rand()`意味着整个排序过程无法利用索引.
		
		随机选择的解决方案:
		
		1. 从1到最大值之间随机选择
		2. 选择下一个最大值
		3. 获取所有的键值, 随机选择一个
		4. 使用偏移量选择随机行

**17** 可怜人的搜索引擎

		 SQL执行搜索字符串效率较低. 可以使用内置的MySQL全文检索, 但是只支持MyISAM引擎, 可以使用第三方的搜索引擎工具Sphinx,Lucence,ES(我自己目前正在研究使用的).
		
		文中还提出一个反向索引技术, 但是个人感觉应用场景太窄.
		
		* 关键词表
		* 关键词文章关系表
		* 文章表
		
		有两个缺点:
		
		* 关系表需要自己每次主动更新, 如果存在关键词可以查询很快, 如果不存在还需要每次主动更新.
		* 如果新添加一片文章, 需要查出文章对已有关键词的所有关系, 并更新到关系表中.

**18** 意大利面条式查询

		平衡一条sql语句和多条sql语句在效率和解决问题之间找到平衡点. 
		
		一个怪兽般的SQL查询的开销可能成指数级别增长, 而使用多个简单的查询却有更好的效果.

**19** 隐式的列

		使用通配符*(SELECT *...) 和 未命名列(INSERT INTO xxx VALUES ) 的缺点
		
		1. SELECT情况无法预测返回多少行
		2. INSERT输入必须严格按照定义表时的那些列的顺序.有可能把数据写到错误的列.
		3. 隐藏的开销(*会查询到其他不想要的列), 网络带宽负载更多.
		
		避免使用`*`和`未命名列`, 复合`尽早出错原则`.

##### 应用程序开发反模式

**20** 明文密码

		不要使用明文密码, 使用 哈希+盐
		
		不要恢复密码, 而是重置密码
		1. 使用临时密码让用户登录
		2. 邮件发送的链接带一个临时的令牌用于标记身份

**21** SQL注入

		解决sql注入
		
		1. 过滤输入内容.(如果你需要一个整数, 那就只是用输入中的整数部分)
		2. 参数动态化内容
		3. 给动态输入的值加引号, 因为一些动态参数会影响到索引使用, 可以直接过滤参入放入SQL语句
		4. 将用户与代码隔离
		5. 找个可靠的人来帮你审查代码

**22** 伪键洁癖

		* 主键 和 行号 不同
		* 不要太担心不连贯的伪键(数据事务回滚都会造成连续的伪键断档)
		* 可以使用随机生成的GUID来代替伪键
		
		主键(来唯一确定一行记录):
		
		* 唯一
		* 非空
		
		不一定非得是连续值才能用来标记行(只是看起来像是行号).

**23** 非礼勿视

		不要忽视MySQL的API返回值, 需要判断可能出错的地方. 
		可以把语句中生成的SQL输出到日志, 方便调试.

**24** 外交豁免权

		要像对待程序代码那样对待数据库的文件
		1. 版本控制
		2. 自动化单元测试 功能测试
		3. 编写文档(清单)
			3.1 ER
			3.2 表,列以及视图
			3.3 关系
			3.4 触发器
			3.5 存储过程
			3.6 SQL 安全
			3.7 数据库基础设置
		
		最好开发时每个开发人员数据库实例独立起来, 不要使用一个, 这样可以不互相影响.

**25** 魔豆

		将模型和表解耦.
		
		活动记录(orm的模型)只是单纯的映射了一张数据库的表, 里面也仅仅有了CRUD方式, 没有呈现出业务逻辑. 需要		使用领域模型来封装业务逻辑.