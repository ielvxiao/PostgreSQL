#Postgresql使用扩展库进行模糊查询

> 前后模糊：gin, gist, rum

> 正则表达： gin, gist

###需要插件 
插件：pg_trgm。pg_trgm是用来做相似度匹配的，在一些情况下也可以拿来代替全文检索做字符匹配。
###创建测试表
    create table test(id int , info text);   
###测试数据
插入100万测试数据

  `  insert into test select id, md5(random()::text) from generate_series(1,1000000) t(id);  `

###不添加索引进行模糊查询
    explain (analyze,buffers)  select * from test where info ~ '2e9a2c'
    
    
    Seq Scan on test  (cost=0.00..20834.00 rows=100 width=37) (actual time=189.992..980.877 rows=1 loops=1)
    
      Filter: (info ~ '2e9a2c'::text)
    
      Rows Removed by Filter: 999999
    
    
    Buffers: shared hit=8334
    
    Planning time: 0.321 ms
    
    Execution time: 980.906 ms
###创建gin索引
        CREATE INDEX trgm_idx1 ON test USING GIN (info gin_trgm_ops);  
    
        耗时 19秒  
    
        占用空间 102MB  
###查询
    explain (analyze,buffers)  select * from test where info ~ '2e9a2c'
    
    Bitmap Heap Scan on test  (cost=56.77..425.16 rows=100 width=37) (actual time=3.178..3.179 rows=1 loops=1)
    
    Recheck Cond: (info ~ '2e9a2c'::text)
    
      Heap Blocks: exact=1
    
    Buffers: shared hit=25
    
    ->  Bitmap Index Scan on trgm_idx1  (cost=0.00..56.75 rows=100 width=0) (actual time=3.155..3.155 rows=1 loops=1)
    
    Index Cond: (info ~ '2e9a2c'::text)
    
    Buffers: shared hit=24
    
    Planning time: 0.538 ms
    
    Execution time: 3.220 ms
###创建gist索引
    CREATE INDEX trgm_idx2 ON test USING GiST (info gist_trgm_ops);
    时间: 22.645s
    占用空间 177MB 

###查询
    explain (analyze,buffers)  select * from test where info ~ '2e9a2c'
    
    Bitmap Heap Scan on test  (cost=13.19..381.57 rows=100 width=37) (actual time=223.692..223.693 rows=1 loops=1)
    
    Recheck Cond: (info ~ '2e9a2c'::text)
    
    Heap Blocks: exact=1
    
    Buffers: shared hit=15638
    
      ->  Bitmap Index Scan on trgm_idx2  (cost=0.00..13.16 rows=100 width=0) (actual time=223.669..223.669 rows=1 loops=1)
    
    Index Cond: (info ~ '2e9a2c'::text)
    
    Buffers: shared hit=15637
    
    Planning time: 0.330 ms
    
    Execution time: 223.749 ms

##pg_trgm做索引的注意事项

1. pg_trgm的工作原理是把字符串切成N个3元组，然后对这些3元组做匹配，所以如果作为查询条件的字符串小于3个字符它就罢工了。
对pg_trgm的gist索引，走的还是索引扫描。但是悲剧的是，索引没有起到任何筛选的作用，702476条记录一个不落全给提出来了，这样还不如直接全表扫描呢。
1. 数据库的LC_CTYPE需要设置为中文区域
2. gin 索引比gist索引占用空间大, 但是gin查询速度快.
