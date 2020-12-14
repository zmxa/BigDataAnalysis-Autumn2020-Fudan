# 大数据分析技术 期末作业
### 作业选题
使用MapReduce框架做伴随车模型。
***
### 数据形式
提供的数据为(车牌，时间，地点)
其中的车牌是纯数字形式。
进行预处理，对每一地点的车辆在每一固定时间窗口进行输出。
得到预处理数据（车牌1，车牌2……）。
以此不定长数据进行以下运行。
***
### 算法介绍
使用Apriori算法，传统的关联规则算法，用于求解频繁k-项集。
其概念是：对于频繁k项集(a1, a2 ... , ak)，其所有子集均是频繁集。
频繁k-项集定义为，出现次数超过所有数据的百分比的具有k个元素的项的集合。
本项目采用MapReduce模型下的apriori算法，在此基础上进行简单优化以适应集群、数据。  

在本算法中，在得到频繁(k-1)-项集后，使用一次遍历来得到候选k-项集：
1. 在Mapper::setup函数中读入上一阶段的输出，由于所有(k-1)-项集是按照记录顺序排序的，因此对每两个相邻元素A、B，若其前(k-2)项是相同的，则添加(相同的k-2项，A的最后一项，B的最后一项)至候选k项集中。
2. 在候选k项集创建完成后，使用树状结构（trie树）存储候选k项集，随后进入Mapper::map函数。
3. map函数逐行地读入变长的预处理数据，对这一行根据k的大小进行深度优先搜索，得到所有的k-项的元素。
4. 随后在Mapper保存的trie树中逐个查找这些k-项元素，每个匹配的候选k-项集将被map函数写入context。
6. 随后的Reducer::reduce函数中，计算所有的出现次数，并最后输出所有不小于支持度阈值的项。

此外，本算法对频繁1、2-项集不由递归方式构建。
对于频繁1项集，原因显然。
对于频繁2项集，考虑本实验过程中的真实情况：
>定义阈值为10，则对于3948828行变长数据，平均每行数目在7-10之间。遍历得到871994个频繁一项集，优化计算得到的最终频繁二项集共150251对。而对于迭代算法，若基于频繁一项集计算，将产生380,187,204,015条候选二项集数据。即使不考虑任何数据结构，全部仅采用整数存储，JAVA将消耗1416 GB内存用于存储。这是无法接受的。
因此，只需采用不剪枝的无优化方式统计频繁2-项集即可。

详细算法内容请查看代码，注释展示了各个函数的用途。
***
### 文件结构
```
src
├── apriori
│   ├── Ap.java             // main
│   ├── Map1.java           // _ --> 1
│   ├── Map2.java           // 1 --> 2
│   ├── MapK.java           // k-1 --> k
│   └── Reduce.java         //
├── data
│   ├── ItemSet.java        // ArrayList<Integer> for data transit.
│   ├── Transaction.java    // ArrayList<Integer> for read from HDFS
│   ├── Trie.java           //
│   └── TrieNode.java       //
├── README.md
├── other
│   ├── Entry.java          // pre-process
│   └── EntryFormat.java    //
└── utils
    └── Funcs.java          // All static method.
```
