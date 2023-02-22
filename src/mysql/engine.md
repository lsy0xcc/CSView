### MySQL的执行引擎有哪些？<Badge text="重要" type="danger" />

主要有MyISAM、InnoDB、Memery等引擎：

- InnoDB引擎提供了对事务ACID的支持，还提供了行级锁和外键的约束。
- MyISAM引擎不支持事务，也不支持行级锁和外键约束。
- Memery就是将数据放在内存中，数据处理速度很快，但是安全性不高。



### MyISAM和InnoDB存储引擎的区别？<Badge text="重要" type="danger" />

- **锁的细粒度不同**：InnoDB比MyISAM更好的支持并发，因为InnoDB的支持行锁，而MyISAM支持表锁，行锁对每一条记录上锁，所以开销更大，但是可以解决脏读和不可重复读的问题，相对来说也更容易发生死锁。
- **可恢复性**：InnoDB有事务日志，数据库崩溃后可以通过日志进行恢复，MyISAM没有日志支持。
- **查询性能**：MyISAM要好于InnoDB，因为InnoDB在查询过程中是在维护数据缓存。并且先要定位到所在数据块，然后从数据块定位到数据内存地址来查找数据。
- **表结构文件**：MyISAM 的表结构文件包括 .frm(表结构定义)，.MYI(索引)、.MYD（数据）；而InnDB的表数据文件为 .ibd(数据和索引集中存储)和.frm(表结构定义)。
- **记录存储顺序**：MyISAM按照记录插入顺序，InnoDB按照主键大小顺序有序插入。
- **外键和事务**：MyISAM均不支持，InnoDB支持。对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务。对一个包含外键的InnoDB表转为MYISAM会失败。
- **操作速度**：对于SELECT前者更优，INSERT、UPDATE、DELETE后者更优。select count(*)使用MyISAM更块，因为内部维护了一个计数器，可以直接调度。
- **存储空间**：MyISAM可被压缩，存储空间较小，InnoDB的表需要更多的内存和存储，会在主内存中建立专用的缓冲池用于高速缓存数据和索引。
- **索引方式**：二者都是B+树索引，前者是堆表，后者是索引组织表。

::: tip 为什么InnoDB没有计数器变量？

因为InnoDB的事务特性，同一时刻表中的行数对于不同事务而言是不同的，因此计数器统计的是当前事务对应的行数，而不是总行数。

:::



### 存储引擎如何选择？<Badge text="了解" type="info" />

如果没有特别的需求，使用默认的InnoDB即可。

要支持事务选择InnoDB，如果不需要可以考虑MyISAM；如果表中绝大多数都只是读查询考虑MyISAM，如果既有读也有写使用InnoDB存储引擎。

系统奔溃后，MyISAM恢复起来更困难，能否接受系统崩溃的程度；MySQL5.5版本开始Innodb已经成为MySQL的默认引擎(之前是MyISAM)，说明其优势是有目共睹的。



### MyISAM索引与InnoDB索引的区别？<Badge text="了解" type="info" />

- InnDB是聚簇索引，MyISAM是非聚簇索引。
- InnoDB的主键索引的叶子节点存储着行数据，因此主键索引非常高效。MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。
- InnoDB非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效。
