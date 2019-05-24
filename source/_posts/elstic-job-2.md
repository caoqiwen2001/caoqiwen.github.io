---
title: elastic-job 源码分析（分片策略）
date: 2019-03-21 15:16:50
tags:
  - elastic-job源码分析
categories:
    - elastic-job源码分析
type: tags
---

### elastic-job分片策略
elastic-job为我们提供了三种分片策略,一种是基于平均分配算法的分片策略，也是项目中默认的分片策略。下面分别说说这三种算法：
####    AverageAllocationJobShardingStrategy类
该类是这几种算法中最基础的，其他三种都是基于它实现的：其中有一个方然shardingAliquot，来看看它的具体实现：
```
   private Map<JobInstance, List<Integer>> shardingAliquot(final List<JobInstance> shardingUnits, final int shardingTotalCount) {
        Map<JobInstance, List<Integer>> result = new LinkedHashMap<>(shardingTotalCount, 1);
        int itemCountPerSharding = shardingTotalCount / shardingUnits.size();
        int count = 0;
        for (JobInstance each : shardingUnits) {
            List<Integer> shardingItems = new ArrayList<>(itemCountPerSharding + 1);
            for (int i = count * itemCountPerSharding; i < (count + 1) * itemCountPerSharding; i++) {
                shardingItems.add(i);
            }
            result.put(each, shardingItems);
            count++;
        }
        return result;
    }
```
从这里可以看出，它的算法还是比较简单的，通过采用这种算法算出分片的数量。
其中有一个方法：

```
  private void addAliquant(final List<JobInstance> shardingUnits, final int shardingTotalCount, final Map<JobInstance, List<Integer>> shardingResults) {
        int aliquant = shardingTotalCount % shardingUnits.size();
        int count = 0;
        for (Map.Entry<JobInstance, List<Integer>> entry : shardingResults.entrySet()) {
            if (count < aliquant) {
                entry.getValue().add(shardingTotalCount / shardingUnits.size() * shardingUnits.size() + count);
            }
            count++;
        }
    }
```
该方法是用来处理对于奇数个分片数目进行调整的算法，如果余数大于0则对当前分片数进行计算。

#### OdevitySortByNameJobShardingStrategy
从官方文档来看，它的作用主要是通过作业值的哈希值进行排序。

```
    public Map<JobInstance, List<Integer>> sharding(final List<JobInstance> jobInstances, final String jobName, final int shardingTotalCount) {
        long jobNameHash = jobName.hashCode();
        if (0 == jobNameHash % 2) {
            Collections.reverse(jobInstances);
        }
        return averageAllocationJobShardingStrategy.sharding(jobInstances, jobName, shardingTotalCount);
    }
```
如果哈希值为偶数，则对实例列表进行翻转。其他的调用算法不变。

#### RotateServerByNameJobShardingStrategy
该算法对job的哈希值对服务器列表进行轮询从而达到分片策略。具体算法如下：

```
 private List<JobInstance> rotateServerList(final List<JobInstance> shardingUnits, final String jobName) {
        int shardingUnitsSize = shardingUnits.size();
        int offset = Math.abs(jobName.hashCode()) % shardingUnitsSize;
        if (0 == offset) {
            return shardingUnits;
        }
        List<JobInstance> result = new ArrayList<>(shardingUnitsSize);
        for (int i = 0; i < shardingUnitsSize; i++) {
            int index = (i + offset) % shardingUnitsSize;
            result.add(shardingUnits.get(index));
        }
        return result;
    }
```
从代码层面来看，也是对求的hashcode值进行取偏移取模计算。
### 总结
以上就是elastic-job的几种策略。在实际项目中可以选择我们需要的策略对数据进行分片。

