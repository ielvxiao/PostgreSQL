
###group by是对检索结果的保留行进行单纯分组，一般总爱和聚合函数一块用例如AVG（），COUNT（），max（），main（）等一块用。 partition by虽然也具有分组功能，但同时也具有其他的功能。 它属于oracle的分析用函数。 
借用一个勤快人的数据说明一下： 

***sum()   over   (PARTITION   BY   ...)   是一个分析函数。   他执行的效果跟普通的sum   ...group   by   ...不一样，它计算组中表达式的累积和，而不是简单的和。***
<pre><code>
表a，内容如下：   
B C D   
02 02 1   
02 03 2   
02 04 3   
02 05 4   
02 01 5   
02 06 6   
02 07 7   
02 03 5   
02 02 12   
02 01 2   
02 01 23   

select   b,c,sum(d)   e   from   a   group   by   b,c   
得到：   
B C E   
02 01 30   
02 02 13   
02 03 7   
02 04 3   
02 05 4   
02 06 6   
02 07 7   

而使用分析函数得到的结果是：   
SELECT   b,   c,   d,   SUM(d)   OVER(PARTITION   BY   b,c   ORDER   BY   d)   e   FROM   a   
B C E   
02 01 2   
02 01 7   
02 01 30   
02 02 1   
02 02 13   
02 03 2   
02 03 7   
02 04 3   
02 05 4   
02 06 6   
02 07 7   
结果不一样，这样看还不是很清楚，我们把d的内容也显示出来就更清楚了：   
SELECT   b,   c,   d,SUM(d)   OVER(PARTITION   BY   b,c   ORDER   BY   d)   e   FROM   a   
B C D E   
02 01 2 2 d=2,sum(d)=2   
02 01 5 7 d=5,sum(d)=7   
02 01 23 30   d=23,sum(d)=30   
02 02 1 1 c值不同，重新累计   
02 02 12 13   
02 03 2 2   
02 03 5 7   
02 04 3 3   
02 05 4 4   
02 06 6 6   
02 07 7 7 
</code></pre>