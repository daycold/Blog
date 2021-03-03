# MYSQL
## ddd
### q:
1. mysql 查询到叶后怎么查询片
二分查找法

2. 间隙锁

    在可重复读隔离级别下，会有读锁。如果是主键索引会直接锁索引，其他会锁查询条件相关的行，避免在读的时候被修改

3. 分区


    alter table user drop partition p4;
    alter table user add partition(partition p4 values less than MAXVALUE)；#新增range分区
    alter table list_part add partition(partition p4 values in(25,26,27));   #新增list分区
    alter table hash_part add partition partitions 4; # hash重新分区
    alter table key_part add partition partitions 4; #key 重新分区
    //子分区添加新分区，虽然我没有指定子分区，但是系统会给子分区命名的 
    alter table sub1_part add partition(partition p3 values less than MAXVALUE);
    //range重新分区  
    ALTER TABLE user REORGANIZE PARTITION p0,p1,p2,p3,p4 INTO (PARTITION p0 VALUES LESS THAN MAXVALUE);  
    //list重新分区  
    ALTER TABLE list_part REORGANIZE PARTITION p0,p1,p2,p3,p4 INTO (PARTITION p0 VALUES in (1,2,3,4,5));  
     #hash和key分区不能用REORGANIZE，官方网站说的很清楚
    
存在四种分区：range（连续范围），list（离散范围），hash（使用函数计算范围），key（使用mysql自带的hash函数）

需要注意，设置了分区后每个值都必须落在某一个分区上。null值落在最左边的分区上。使用分区会根据分区裂变 idb 文件。

b+树索引页字节点放的是分区。每个分区是 1m，64片，每片是16k。扫描到数据所在分区后会将分区添加到内存中再在内存中查找具体的列，使用二分查找法。索引是有序的双向链表。一般b+树索引都是2-4层，查询时每层进行一次IO操作。

如果使用分区，分区后b+树层树与原有的b+树层数一致，则不会有性能提升，反而在跨区查询时有性能损失。

索引失效的场景：
1. 索引列太多null值
2. 非聚簇索引查询到的数量太多，且在主键（磁盘）上非常分散。该种场景下固态硬盘可以强制使用索引。

Multi-Range read优化：将非聚簇索引根据id排序后再查询聚簇索引。避免不连续的查询相同页，减少缓冲池中页的被替换次数。

mysql优先从缓冲池中读取数据，数据的修改也会先落在缓冲池中，并写在日志文件。当条件触发时刷到物理内存中（如触发定时刷，缓存池内存达到一定阈值）

