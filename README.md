# mongo
mongo使用
1.插入多个文档,使用批量插入会明显提高速度。一次批量插入只是单个的TCP请求,
避免许多零碎的请求带来的开销。无需处理大连搞的消息头,可以减少插入的时间。

2.当前版本mongo消息的最大长度是16MB

3.传统的注入式攻击对Mongo来说是无效的。

4.查询
/*查询指定的key:value*/
db.apiReqLogDO.find({"key":"value"})
/*多个字段,相当于and*/
db.apiReqLogDO.find({"key1":"value1","key2":"value2"})

5.查询返回指定的键
返回resultCode不是9999999的数据,并且返回字段只有_id和resultCode
db.apiReqLogDO.find({},{"resultCode":"99999999"})
需要返回的字段,指定值为1
db.apiReqLogDO.find({},{"resultCode":1,"apiUrl":1})

查询字段xx >= n的数据
db.apiReqLogDO.find({"costTime" : {"$gte" : 2500}})
db.集合名称.find({"字段" : {"符号" : "值"}})
符号可以为:$lt $lte $gt $gte
区间查询:
大于1000小于2000
db.apiReqLogDO.find({"costTime" : {"$gte" : 1000,"$lte" : 2000}})
db.集合名称.find({"字段" : {"符号1" : "值1","符号2" : "值2"}})


日期区间查询
start = new Date("2018/03/01")
end = new Date("2018/03/10")
db.gatherAppUserInfoDO.find({"createTime" : {"$gt" : start,"$lt" : end}})

不等于:
db.gatherAppUserInfoDO.find({"deviceType" : {"$ne" : "OPPO R9m"}})
deviceType 不等于 OPPO R9m

OR查询:
$in可以用来查询一个键的多个值
查询consId是44,85的记录
db.gatherAppUserInfoDO.find({"consId" : {"$in" : [44,85]}})
$in可以指定不同类型的条件和值,v1,v2..类型可以不同
db.集合名.find(){"字段" : {"$in" : [v1,v2...]}}

not in 用$nin

多条件查询consId = 44 deviceType = "oppo" or
db.gatherAppUserInfoDO.find({"$or" : [{"consId" : 44},{"deviceType" : "Mi-4c"}]})


$not
$mod会将查询的值除以v1,若余数 = v2则返回该结果
db.集合名.find({"字段" : {"$mod" : [v1,v2]}})
获取id是1,6,11,16的用户
db.user.find({"userId" : {"$mod" : [5,1]}})
查询id是2,3,4,5,的用户
$not 结合 $mod
db.user.find({"userId" : {"$not" : {"$mod" : [5,1]}}})

注意:一个键不能有多个修改器

6.特定于类型的查询
(1) null
 db.集合名.find({"字段" : null})
 不仅会筛选出来字段为null的,缺少这个字段的文档也会返回
 如果仅仅想要为null的文档:
 db.集合名.find({"字段" : {"$in" : [null],"$exists" : true}})

(2)正则表达式
 查找某个字段忽略字段值的大小写:
 db.集合名.find({"字段" : /字段值?/i})

(3)查询数组

 db.gatherAppUserInfoDO.find({"deviceType" : {$all : ["OPPO R9m","Mi-4c"]}}) 

(4)$slice
 返回博客评论的前10条
 db.blog.posts.findOne(criteria,{"comments" : {"$slice" : 10}})
 后10条
 db.blog.posts.findOne(criteria,{"comments" : {"$slice" : -10}})
 跳过前n1个元素,取n2条
 db.blog.posts.findOne(criteria,{"comments" : {"$slice" : [n1,n2]}})

 (5)内嵌文档查询
 {
 	"name" : {
 		"first" : "Joe",
 		"last" : "hao"
 	},
 	"age" : 45
 }

 寻找姓名为Joe hao的人可以
 db.集合名.find({"name" : {"first" : "Joe","last" : "hao"}})

 也可以:
 db.集合名.find({"name.first" : "Joe","name.last" : "hao"})

 {
 	"content" : "...",
 	"comments" : [
 		{
 			"author" : "joe",
 			"score" : 3,
 			"comment" : "nice..."
 		},
 		{
 			"author" : "hao",
 			"score" : 6,
 			"comment" : "good job"
 		}
 	]
 }

 文档类型如上,找到由joe发表的5分以上的评论
 db.blog.find({"comments" : {"$elemMatch" : {"author" : "joe","score" : {"$gte" : 5}}}})

 $elemMatch将限定条件进行分组,仅当需要对一个内嵌文档的多个键操作时才会用到

 7.$where查询
 (1)好处
  $where子句,可以执行任意的js作为查询的一部分
  db.foo.insert({"apple" : 1,"banana" : 6,"peach" : 3})
  db.foo.insert({"apple" : 8,"spinach" : 4,"watermelon" : 4})

  查询,如果某个记录中有两个字段值相等,则返回该文档
  可以用$where实现
  db.foo.find("$where" : function(){
  		for(var current in this){
  			for(var other in this){
  				if(current != other && this[current] == this[other]){
  					return true;
  				}
  			}
  		}
  		return false;
  });
  如果返回true,文档就作为结果的一部分被返回,如果是false,则不然

  也可以:
  db.foo.find({"$where" : "this.x + this.y == 10"})
  和
  db.foo.find({"$where" : "function() {return this.x + this.y == 10;}"})
  等价
  $where查询比常规查询慢

  8游标
  (1)作用
   可以用游标接受find执行的结果
   var cursor = db.集合.find();
   遍历集合
   while(cursor.hasNext()){
   		obj = cursor.next();
   }
   还可以:
   var cursor = db.集合.find()
   cursor.forEach(function(x) {
   		print(x.name);
   })

 (2)limit
  限制返回结果的数量:
  db.集合名.find().limit(n)
 (3)skip跳过
  db.集合名.find().skip(n)
  越过前n个匹配的文档,返回剩下的
 (4)sort用一个对象作为参数,排序方向1(升序),-1(降序)
  db.集合名.find().sort({"排序字段1" : 1,"排序字段"})

 (5)注意
   不用skip对结果分页,数据量大的话会很慢
9.索引
  当查询中仅使用一个键时,可以对该键建立一个索引.
  对某个键加索引会加速对该键的查询,对于其他查询可能没有帮助

  实践证明:一定要创建查询中用到的所有键的索引

  每个集合默认的最大索引个数是64个

  为排序创建索引:没有索引的键调用sort,mongo需要将所有数据提取到内存进行排序,
  一旦数据量很大的时候,就会出错.
  按照排序来索引以便让mongo按照顺序提取数据

  唯一索引:确保集合的每一个文档的指定键都有唯一值
  例如:保证username的值都不一样
  db.集合名.ensureIndex({"username" : 1},{"unique" : true})

  消除重复:
  当为已有的集合创建索引,可能有些值已经有重复了。如果发生这种情况,那么索引常见就是失败。
  将包含重复值的文档都删掉:保留发现的第一个文档,而删除接下来的有重复值的文档
  db.集合名.ensureIndex({"username" : 1},{"unique" : true,"dropDups" : true})

  复合唯一索引:
  复合唯一索引,单个建的值可以相同.


  使用explain和hint
  db.集合名.find().explain()会返回查询使用的索引情况,耗时以及扫描文档数的统计信息
  hint强制使用某个索引
  db.集合名.find().hint(索引)

  索引管理:
  索引的原信息保存在每个数据库的system.indexs集合中,不能对其插入或者删除文档

  修改索引:
  db.集合名.ensureIndex({"username" : 1},{"background" : true})
  建立索引比较耗时,好资源,最好后台完成,同事正常处理请求.{"background" : true}
  如果不后台处理的话,数据库会阻塞建立索引期间的所有请求


10.地理空间索引
  场景:找到离当前位置最近的n个场所.此时可以用到mongo的地理空间索引
  创建一个地理空间索引:
  db.map.ensureIndex({"gps" : "2d"})
  默认,地理空间索引假设值的范围是-180 - 180
  如果要自己设定:
  db.集合.ensureIndex({"字段" : "2d"},{"min" : 最小值,"max" : 最大值})


  地理空间查询的两种方式:
  方式1:
  db.map.find({"gps" : {"$near" : [n1,n2]}})
  会按照离(n1,n2)由近及远的方式将查询出来的文档全部返回,没有指定limit,默认100个文档
  方式2:
  db.runCommand({geoNear : "集合名",near : [n1,n2],num:返回文档数});
  goNear会返回每个文档到查询点的距离,这个距离是以你插入的数据为单位.

  指定形状内的文档:将$near替换成$within,$within获取数量不断早呢更加的形状作为参数
  db.map.find({"gps" : {"$within" : {"$box" : [[x1,y1],[x2,y2]]}}})
  $box矩形,x1,y1左下角坐标,x2,y2右下角坐标
   
  圆形:
  db.map.find({"gps" : {"$within" : {"$center" : [[x,y],r]}}})
  x,y圆心,r半径

  复合地理空间索引:
  场景:用户要找出周围所有的咖啡店或者披萨店。不是一个地点
  解决:地理空间索引 + 普通索引
  db.ensureIndex({"location" : "2d" ,"desc" : 1})
  desc指代店铺名称
  找最近的咖啡馆：
  db.map.find({"location" : {"$near" : [n1,n2]},"desc" : "coffeeshop"}).limit(1)

11.聚合
   (1)count
    返回集合中的文档数量
    db.集合名.count()

    (2)distinct
     用来找出给定键的所有不同的值,使用时必须指定键和集合
     db.runCommand({"distinct" : "集合","key" : "字段"})
     db.runCommand({"distinct" : "people","key" : "age"})
     获取people中age字段的值
    (3)group
     group过程:先选定分组依据的键,mongo将集合依据选定的键分成若干组,然后聚合每一组内的文档

     场景:跟踪股票价格,从上午10点到下午4点每隔几分钟就更新一下某只股票的价格,并保存到mongo,
     现在报表程序要或得近30天的收盘价

     解决:先把集合按天分组->在每一组里取包含最新时间戳的文档->将其放置到结果中
     db.runCommand({
     	"group" : {
     		"ns" : "stocks",
     		"key" : "day",
     		"initial" : {"time" : 0},
     		"$reduce" : function(doc,prev){
     			if(doc.time > prev.time){
     				prev.price = doc.price;
     				prev.time = doc.time;
     			}
     		},
     		"condition" : {"day" : {"$gt" : "日期"}}
     	}
     });
     "ns" : 集合
     "key" : 分组依据的键
     "initial" : {"time":0}每一组reduce函数调用的初始时间,会作为初始文档传递
     给后续过程。每一组的所有成员都会使用这个累加器.
     "$reduce":function(doc,prev){}
     每个文档都对应一次这个调用,系统会传递两个参数,当前文档和累加文档(本组当前的结果)
     "condition" :筛选条件
     执行结果:
     {
     	"retval" : [
     		{
     			"day":
     			"time":
     			"price":
     		},
     		...
     	],
     	"count":
     	"keys":
     	"ok":1
     }
    (4)使用完成器
     完成器(finalizer)精简从数据库传到用户的数据,这个步骤非常的重要,因为group命令的输出一定要
     能放在单个数据库响应中.

     场景:博客,其中每篇文章都有很多标签(tag).现在要找出每天最热点的标签。
     解决:按天分组 -> 为每一个标签计数
     db.posts.group({
     	"key":{"tags" : true},
     	"initial":{"tags" : {}},
     	"$reduce":function(doc,prev){
     		for(i in doc.tags){
     			if(doc.tags[i] in prev.tags){
     				prev.tags[doc.tags[i]]++;
     			}else{
     				prev.tags[doc.tags[i]] = 1;
     			}
     		}
     	}
     });
     

