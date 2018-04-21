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
     接着在返回值中找最大值的标签finalize附带一个函数,在每组结果传递到客户端之前被调用一次,可以用其修改结果的返回
     db.runCommand({"group" : {
        "ns" : "集合名",
        "key" : {"tags" : true},分组字段
        "initial" : {"tags" : {}},
        "$reduce" : {

        },
        "finalize" : function(prev){
            var mostPopular = 0;
            for(i in prev.tags){
              if(prev.tags[i] > mostPopular){
                prev.tags = i;
                mostPopular = prev.tags[i];
              }
            }
            delete prev.tags
        }
     }});
     (5)将函数作为键使用
     db.集合名.group({
        "ns" : "集合名",
        "$keyf" : function(x) {return x.category.toLowerCase();},
        ...
     });
12.MapReduce(很慢,不要用在实时的任务,要作为后台任务来运行)
     (1)它是一个轻松并行化到多个服务器的聚合方法,拆分问题 -> 将各个部分发送到不同的机器上
      -> 让每台机器完成一部分。-> 当所有机器都完成的时候,再把结果汇集起来形成最终完整的结果.
     (2)实例:找出集合中的所有键
     this当前映射文档的引用
      map = function(){
        for(var key iin this){
          emit(key,{count : 1});
        }
      }
      reduce函数有两个值:一个是key,另一个是emit返回的第一个值
      reduce = function(key,emits){
        total = 0;
        for(var i in emits){
          total += emits[i].count;
        }
        return {"count" : total};
      }

      mr = db.runCommand({
        "mapreduce" : "集合",
        "map" :  map,
        "reduce": reduce
      });

      (3)网页分类
       给链接打标签,找出哪个链接最热门
       map = function(){
          for(var i in this.tags){
              var recency = 1(new Date() - this.date);
              var score = recency * this.score;
              emit(this.tags[i],{"urls" : [this.url], "score" : score)});
          }
       }

       简化同一个标签的所有值,形成这个标签的分数
       reduce = function(key,emits){
          var total = {urls : [],score : 0}
          for(var i in emits){
              emits[i].urls.forEach(function(url){
                  total.urls.push(url);
              })
              this.score += emits[i].score;
          }
          return total;
       }


13数据库命令
  (1)getLastError查询一个更新对多少个文档起作用
  (2)命令工作原理:
  (3)命令:
  db.listCommands()获取所有命令的最新列表
  (4)数据库引用:
  dbref类似url,唯一确定一个到文档的引用

14管理
  (1)启动:
  mongod --help查看帮助
  mongod --dbpath 设置数据目录,默认/data/db,每个mongod进程都需要独立的数据目录.
  如果有3个实例,必须要有3个独立的数据目录.mongod启动时,会在数据目录创建mongod.lock文件.
  这个文件用于防止其他mongod进程使用该数据目录.
  --port 指定监听的端口号,默认27017,运行多个的话,需要指定不同的端口号
  --fork 以守护进程的方式运行mongodb,创建服务器进程
  --logpath 指定日志输出路径,默认会覆盖,追加用--logappend
  --config 指定配置文件,加载命令行未指定的各种选项

  例如:启动mongo,作为守护进程监听8888,并输出日志到mongo.log
  ./mongod --port 8888 --fork --logpath mongo.log
  (2)配置文件
  shutdownServer()
  (3)监控
  启动mongod时,还会启动一个http server,端口比mongo端口大1000,访问
  localhost:port可以查看mongo信息
  利用管理接口需要在启动时:
  ./mongod --rest来开启rest支持
  --nohttpinterface关闭管理接口
  localhost:port/_status查看服务器状态
  (4)状态信息统计
  globalLock:全局所占用服务器的时间
  mem:包含服务器内存映射了多少数据,服务器进程的虚拟内存和常驻内存的占用情况(Mb)
  indexCounters:表示B树在磁盘检索(misses),内存检索(hits)的次数,比值上升就要
  考虑添加内存了.
  backgroundFlushing:后台做了多少次fsync以及用了多少时间
  opcounters:包含每种主要操作的次数
  asserts:统计断言的次数

  (5)安全认证
  db.addUser("用户名","密码",true(只读权限))
  建议将mongodb的服务器布置在防火墙后或者布置在只有应用服务器能访问的网络中.
  如果需要被外面访问的话,建议使用--bindid选项,指定绑定到的本地ip地址

  例如:只能从本机应用服务器访问,
  mongod --bindip localhost
  禁止服务端运行js
  --noscripting

  (6)备份(不实时)
   mongodump运行时备份
   从备份中恢复
   mongorestore

   mongorestore获取mongodump的结果,并将备份数据插入到运行的mongodb实例

   例子:从数据库test到backup目录的热备,然后调用mongorestore
   ./mongodump -d(指定要备份的数据库) test -o(指定输出) backup
   
   恢复:./mongorestore -d foo --drop backup/test/
   --drop代表在恢复前删除集合(若存在),否则,数据库就会与现有的集合数据合并
  (7)fsync和锁
   fsync能在mongo运行时复制数据目录还不会损毁数据
   fsync命令会强制服务器简化所有缓冲区写入磁盘,还可以上锁阻止对数据库的进一步写入，
   直到释放锁。

   例子:强制执行fsync并获得写入锁
   use db
   db.runCommand({
      "fsync" : 1,
      "lock" : 1
   });
   备份好了,解锁
   db.$cmd.sys.unlock.findOne();
   返回:{"ok" :1,"info" : "unlock requested"}
   db.currentOp()
   返回:{"inprog" : []}
   运行currentOp是为了确保已经解锁了

   利用fsync可以灵活备份,代价是:一些写入操作暂时被阻塞了.

   注意:唯一不耽误写还能保证实时快照的备份方式就是通过 -> 从服务器备份
  
   (8)从属备份:推荐

   (9)修复
    i:未能正常停止mongo后应该修复数据库.要是未正常停止,下次启动服务器备份时mongo会提示:
    old lock file: ....
    recommand removing file and running --repair
    see:xxx
    ii:修复所有数据库最简单的方式:mongod --repair来穹顶服务起

    iii:修复数据库能起到压缩数据的作用,闲置的空间在修复后被重新回收.
    iiii:修复运行中的数据库
    use db
    db.repairDatabase()
    15复制(应对故障,数据集成,多扩展,热备,离线批处理的数据源)
    (1)主从复制
     i:场景:备份,故障恢复,读扩展
    ii:类型
       一主一从:
       一主多从:
   iii:描述:
       最基本的设置是建立一个主节点和一个或者多个从节点,每个从节点要知道主节点的地址。
       运行 -> mongod --master就启动了主服务器.
       运行 -> mongod --slave --source master_ip 则启动从服务器

   iiii:示例
       首先,给主节点建立数据目录,并绑定端口8888
       mkdir -p ~/dbs/master
       ./mongod --dbpath ~/dbs/master --port 8888 --master

       设置从节点,需要不同的目录和端口
       mkdir -p ~/dbs/slave
       ./mongod --dbpath ~/dbs/slave --port 9999 --slave --source localhost:8888

       一般从节点不超过12个
       不能丛丛复制

    (2)主从复制的选项
      --only 在从节点上指定复制特定的某个数据库(不指定默认复制所有)
      --slavedelay 用在从节点上,当应用主节点的操作时增加延时(秒),这样就能轻松设置延时从节点,
      作用:防护无意删除,插入垃圾数据

      --fastsync 以主节点的数据快照为基础启动从节点。如果数据目录一开始是主节点的数据快照,从
      节点用这个选项启动要比做完整的同步快的多

      --autoresync 如果从节点与主节点不同步了,则自动重新同步

      --oplogSize 主节点oplog大小(Mb)

    (3)添加及删除源
      启动从节点时可以用 --source指定主节点
      假设主节点绑定localhost:27017。启动从节点时可以不添加源,而是随后向source集合添加主节点信息.
      先启动从节点:
      ./mongod --slave --dbpath ~/dbs/slave --port 端口

       在shell中添加master
       use local
       db.sources.insert({"host" : "localhost:27017"})
       查看原:
       db.sources.find()

       假设在生产环境下,想更改从节点的配置,改用prod.example.com为源,则可以用insert和remove来完成
       db.sources.insert({"host" : "prod.example.com:27017"})
       db.sources.remove({"host" : "localhost:27017"})

       如果切换的两个主节点有相同的集合,mongo会尝试合并,但不能保证正确合并.
       如果一个从节点对应多个不同的主节点,最好在主节点上使用不同的命名空间

    (4)副本集
       i:什么是副本集?
         有自动故障恢复功能的主从集群。主从集群和副本集的区别在于,副本集没有固定的主节点
         整个集群会选举出一个主节点,当其不能工作时则变更到其它节点.

         副本集总会有一个活跃节点和一个或者多个备份节点

       ii:优点?
          副本集最美妙的地方就是所有的东西都是自动化的。自动提升备份节点成为活跃节点.

      iii:初始化副本集?
           示例:两个服务器,不能用localhost地址作为成员,得设置主机名
            cat /etc/hostname
            morton
           ->首先,为每一个服务器创建数据目录,选择端口
           mkdir -p ~/dbs/node1 ~/dbs/node2

           ->启动之前,给副本集起个名字,blort

           ->启动服务器,--replSet让服务器知晓在这个blort副本集中还有别的同伴
             位置在morton:10002还没启动
             ./mongod --dbpath ~/dbs/node1 --port 10001 --replSet blort/morton:10002

           ->以同样的方式启动另一台
             ./mongod --dbpath ~/dbs/node2 --port 10002 --replSet blort/morton:10001

           ->如果想添加第三台,可以:
             ./mongod --dbpath ~/dbs/node3 --port 10003 --replSet blort/morton:10001
             或者
             ./mongod --dbpath ~/dbs/node3 --port 10003 --replSet blort/morton:10001,morton:10002
    
           副本集的亮点自动检测功能,在指定单台服务器后,mongo会自动搜索并连接其余的节点
           ->在shell中初始化副本集
           打开shell连接其中一个服务器,例morton:10001,初始化命令只能执行一次
           ./mongod morton:10001/admin

           ->db.runCommand({
              "replSetInitiate" : {
                "_id" : "blort", 副本集名字
                "members" : [ 副本集服务器列表
                    {
                      "_id" : 1,
                      "host" : "morton:10001"
                    },
                    {
                      "_id" : 2,
                      "host" : "morton:10002"
                    }
                ]
              }
            })

        iiii:副本集中的节点
           任何时间,集群只有一个活跃及诶单,其它都是备份节点。
           节点类型:
               standard:常规节点,存储一份完整的数据副本,参与投票选举,有可能成为活跃节点
               passive:存储了完整的数据副本,参与投票,不能成为活跃节点
               arbiter:仲裁者只参与投票,不接收复制的数据,也不能成为活跃节点

            节点区别:
               标准节点和被动节点之间的区别仅仅是数量的差别;参与节点(非仲裁)有个优先权。
               优先权为0则是被动的,不能成为活跃节点。优先值不为0，则按照由大到小选出活跃节点
               优先值一样的话则看谁的数据比较新。
               如果有两个优先值为1和一个优先值为0.5的节点,最后一个节点只有在前两个节点都不可用
               的时候才能成为活跃节点

            修改节点priority键,配置成标准节点或者被动节点
            members.push({
                "_id" : 3,
                "host" : "morton:10003",
                "priority" : 40
              });
            默认优先级是1，可以是0-1000(包含1000)

            指定仲裁节点:arbiterOnly
            members.push({
                "_id" : 4,
                "host" : "morton:10004",
                "arbiterOnly" : true
              });

        (5)副本集故障切换和活跃节点选举

        (6)主节点的操作日志oplog
           主节点的操作记录称为oplog.oplog存储在一个特殊的数据库中,叫做local.
           oplog就在其中的oplog.$main集合里面
           oplog中的每个文档都代表主节点上执行的一个操作.
           ts:操作时间戳,跟踪操作执行的时间,由4字节的时间戳,4字节的递增计数器构成
           op:操作类型,只有1字节代码("i")代表插入
           ns:执行操作的命名空间(集合名)
           o:进一步制定要执行的操作的文档,对插入来说,就是要插入的文档
