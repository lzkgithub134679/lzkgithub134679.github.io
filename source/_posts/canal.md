---
title: canal的快速搭建，和redis异步更新
date: 2019-06-26 13:47:22
tags:
- Show All
- canal
header-img: "Demo.png"
---

canal基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了mysql 。canal的原理是基于mysql binlog技术，所以这里一定需要开启mysql的binlog写入功能，并且配置binlog模式为row. 

一、下载安装并运行canal服务端

1.下载

<https://github.com/alibaba/canal/releases>   

例:[canal.deployer-1.1.3.tar.gz](https://github.com/alibaba/canal/releases/download/canal-1.1.3/canal.deployer-1.1.3.tar.gz)

2.解压并打开conf/example/instance.properties进行配置

```
#################################################
## mysql serverId , v1.0.26+ will autoGen
# canal.instance.mysql.slaveId=0

# enable gtid use true/false
canal.instance.gtidon=false

# position info
# 这里填写自己数据库地址
canal.instance.master.address=127.0.0.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
# 这里填写数据库的账号和密码
canal.instance.dbUsername=root
canal.instance.dbPassword=root
canal.instance.connectionCharset = UTF-8
# 指定监听哪个数据库
canal.instance.defaultDatabaseName = ceshi  
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=

# mq config
canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#################################################

```

3.配置完后，运行bin/startup.bat (linux运行startup.sh)

二、编写客户端进行连接，并监控

1.连接服务端，监控类

```
public class SimpleCanalClientExample {


public static void main(String args[]) {
    // 创建链接
    CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress("39.105.223.218",
                                                                                        11111), "example", "", "");
    int batchSize = 1000;
    try {
        connector.connect();
        connector.subscribe(".*\\..*");
        connector.rollback();
      //  int totalEmptyCount = 120;
        while (true) {
            Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据
            long batchId = message.getId();
            int size = message.getEntries().size();
            if (batchId == -1 || size == 0) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace(); 
                }
            } else {
                printEntry(message.getEntries());
            }

            connector.ack(batchId); // 提交确认
            // connector.rollback(batchId); // 处理失败, 回滚数据
        }
    } finally {
        connector.disconnect();
    }
}

private static void printEntry(List<Entry> entrys) {
    for (Entry entry : entrys) {
        if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {
            continue;
        }

        RowChange rowChage = null;
        try {
            rowChage = RowChange.parseFrom(entry.getStoreValue());
        } catch (Exception e) {
            throw new RuntimeException("ERROR ## parser of eromanga-event has an error , data:" + entry.toString(),
                                       e);
        }

        EventType eventType = rowChage.getEventType();
        System.out.println(String.format("binlog[%s:%s] , name[%s,%s] , eventType : %s",
                                         entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                                         entry.getHeader().getSchemaName(), entry.getHeader().getTableName(),
                                         eventType));
        //String schemaName = entry.getHeader().getSchemaName();//库名
        String tableName = entry.getHeader().getTableName();//表名
        System.out.println(tableName);
        for (RowData rowData : rowChage.getRowDatasList()) {
            if (eventType == EventType.DELETE) {
            	//String currentTime = DateUtils.getCurrentTimeDefultString();
                printColumn(rowData.getBeforeColumnsList());
                redisDelete(rowData.getBeforeColumnsList());
            } else if (eventType == EventType.INSERT) {
            	//String currentTime = DateUtils.getCurrentTimeDefultString();
                printColumn(rowData.getAfterColumnsList());
                redisInsert(rowData.getAfterColumnsList()); 
            } else {
            	//String currentTime = DateUtils.getCurrentTimeDefultString();
                System.out.println("-------&gt; before");
                printColumn(rowData.getBeforeColumnsList());
                System.out.println("-------&gt; after");
                printColumn(rowData.getAfterColumnsList());
               // redisUpdate(rowData.getAfterColumnsList());
            }
        }
    }
}

private static void printColumn(List<Column> columns) {
    for (Column column : columns) {
       System.out.println(column.getName() + " : " + column.getValue() + "  update=" + column.getUpdated());
    }
}

private static void redisInsert( List<Column> columns){
	  String currentTime = DateUtils.getCurrentTimeDefultString();
	  JSONObject json=new JSONObject();
	  for (Column column : columns) {  
		  json.put(column.getName(), column.getValue());  
      }  
	  if(columns.size()>0){
		  System.out.println("===============开始=============="+"\r\n"+"时间: "+currentTime);
		  RedisUtil.hashSet("ebag:dict", json.getString("dict_id"), json.getString("dict_val"));
		  System.out.println("redis新增:  "+"hashKey: ebag:dict"+"\r\n"+"key: "+ json.getString("dict_id")+
					"  value: "+json.getString("dict_val"));
		  System.out.println("===============结束==============");
	  }
 }

 private static  void redisDelete( List<Column> columns){
	   String currentTime = DateUtils.getCurrentTimeDefultString();
	   JSONObject json=new JSONObject();
		  for (Column column : columns) {  
			  json.put(column.getName(), column.getValue());  
	       }  
		  if(columns.size()>0){
			System.out.println("==============开始==============="+"\r\n"+"时间: "+currentTime);
			RedisUtil.delHashKey("ebag:dict",json.getString("dict_id"));
			System.out.println("redis删除:  "+"hashKey: ebag:dict"+"\r\n"+"key: "+ json.getString("dict_id")+
					"  value: "+json.getString("dict_val"));
			System.out.println("==============结束===============");
		  }
 }

}
```

2.redis连接工具

```
public class RedisUtil {
 
	// Redis服务器IP
	private static String ADDR = "39.105.223.218";
 
	// Redis的端口号
	private static int PORT = 6379;
 
	// 访问密码
	private static String AUTH = "123456";
 
	// 可用连接实例的最大数目，默认值为8；
	// 如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
	private static int MAX_ACTIVE = 1024;
 
	// 控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8。
	private static int MAX_IDLE = 200;
 
	// 等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException；
	private static int MAX_WAIT = 10000;
 
	// 过期时间
	protected static int  expireTime = 60 * 60 *24;
	
	// 连接池
	protected static JedisPool pool;
 
	/**
	 * 静态代码，只在初次调用一次
	 */
	static {
		JedisPoolConfig config = new JedisPoolConfig();
		//最大连接数
		config.setMaxTotal(MAX_ACTIVE);
		//最多空闲实例
		config.setMaxIdle(MAX_IDLE);
		//超时时间
		config.setMaxWaitMillis(MAX_WAIT);
		//
		config.setTestOnBorrow(false);
		pool = new JedisPool(config, ADDR, PORT, 1000,AUTH);
	}
 
	/**
	 * 获取jedis实例
	 */
	protected static synchronized Jedis getJedis() {
		Jedis jedis = null;
		try {
			jedis = pool.getResource();
		} catch (Exception e) {
			e.printStackTrace();
			if (jedis != null) {
				pool.returnBrokenResource(jedis);
			}
		}
		return jedis;
	}
 
	/**
	 * 释放jedis资源
	 * 
	 * @param jedis
	 * @param isBroken
	 */
	protected static void closeResource(Jedis jedis, boolean isBroken) {
		try {
			if (isBroken) {
				pool.returnBrokenResource(jedis);
			} else {
				pool.returnResource(jedis);
			}
		} catch (Exception e) {
 
		}
	}
 
	/**
	 *  是否存在key
	 * 
	 * @param key
	 */
	public static boolean existKey(String key) {
		Jedis jedis = null;
		boolean isBroken = false;
		try {
			jedis = getJedis();
			jedis.select(3);
			return jedis.exists(key);
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		return false;
	}
 
	/**
	 *  删除key
	 * 
	 * @param key
	 */
	public static void delKey(String key) {
		Jedis jedis = null;
		boolean isBroken = false;
		try {
			jedis = getJedis();
			jedis.select(3);
			jedis.del(key);
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
	}
	/**
	 * 删除hash里的某个key
	 */
	public static void delHashKey(String hashKey,String key) {
		boolean isBroken = false;
		Jedis jedis = null;
		try {
			jedis = getJedis();
			if (jedis != null) {
				jedis.select(3);
				jedis.hdel(hashKey, key);
			}
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		
	}
	
	/**
	 *  取得key的值
	 * 
	 * @param key
	 */
	public static String stringGet(String key) {
		Jedis jedis = null;
		boolean isBroken = false;
		String lastVal = null;
		try {
			jedis = getJedis();
			jedis.select(3);
			lastVal = jedis.get(key);
			jedis.expire(key, expireTime);
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		return lastVal;
	}
 
	/**
	 *  添加string数据
	 * 
	 * @param key
	 * @param value
	 */
	public static String stringSet(String key, String value) {
		Jedis jedis = null;
		boolean isBroken = false;
		String lastVal = null;
		try {
			jedis = getJedis();
			jedis.select(3);
			lastVal = jedis.set(key, value);
			jedis.expire(key, expireTime);
		} catch (Exception e) {
			e.printStackTrace();
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
		return lastVal;
	}
 
	/**
	 *  添加hash数据
	 * 
	 * @param key
	 * @param field
	 * @param value
	 */
	public static void hashSet(String key, String field, String value) {
		boolean isBroken = false;
		Jedis jedis = null;
		try {
			jedis = getJedis();
			if (jedis != null) {
				jedis.select(3);
				jedis.hset(key, field, value);
				jedis.expire(key, expireTime);
			}
		} catch (Exception e) {
			isBroken = true;
		} finally {
			closeResource(jedis, isBroken);
		}
	}
 
}
```

