---
title: elastic-job 源码分析（一）整理启动流程
date: 2019-01-31 18:12:26
tags:
  - elastic-job源码分析
categories:
    - elastic-job源码分析
type: tags
---
 ### elastic-job 源码分析
分布式任务调度框架值的好好研究，之前项目中都是直接关联在项目中的用Spring直接调用，这有个坏处在大数据量的时候如果是分布式部署的时候 做了大量重复的劳动，而当当的elastic-job 这套能很好的解决该问题。   
### 实例演示
只需要实现SimpleJob接口，该接口有execute方法，来看看他的接口实现：

```
public class SpringSimpleJob implements SimpleJob {

    private static final Logger LOGGER = LoggerFactory.getLogger(SpringSimpleJob.class);

    @Resource
    private FooRepository fooRepository;

    @Override
    public void execute(final ShardingContext shardingContext) {
        LOGGER.info(String.format("Item: %s | Time: %s | Thread: %s | %s",
                shardingContext.getShardingItem(), new SimpleDateFormat("HH:mm:ss").format(new Date()), Thread.currentThread().getId(), "SIMPLE"));
//        System.out.println(String.format("Item: %s | Time: %s | Thread: %s | %s",
//                shardingContext.getShardingItem(), new SimpleDateFormat("HH:mm:ss").format(new Date()), Thread.currentThread().getId(), "SIMPLE"));
        List<Foo> data = fooRepository.findTodoData(shardingContext.getShardingParameter(), 10);
        for (Foo each : data) {
            fooRepository.setCompleted(each.getId());
        }
    }
}
```
先来看看他的启动流程是怎么样的？笔者在git上下载了SpringBoot的demo，然后启动之，它的启动代码是这样的：

```
   @Bean(initMethod = "init")
    public JobScheduler simpleJobScheduler(final SimpleJob simpleJob, @Value("${simpleJob.cron}") final String cron, @Value("${simpleJob.shardingTotalCount}") final int shardingTotalCount,
                                           @Value("${simpleJob.shardingItemParameters}") final String shardingItemParameters) {
        return new SpringJobScheduler(simpleJob, regCenter, getLiteJobConfiguration(simpleJob.getClass(), cron, shardingTotalCount, shardingItemParameters), jobEventConfiguration);
    }
    
    private LiteJobConfiguration getLiteJobConfiguration(final Class<? extends SimpleJob> jobClass, final String cron, final int shardingTotalCount, final String shardingItemParameters) {
        return LiteJobConfiguration.newBuilder(new SimpleJobConfiguration(JobCoreConfiguration.newBuilder(
                jobClass.getName(), cron, shardingTotalCount).shardingItemParameters(shardingItemParameters).build(), jobClass.getCanonicalName())).overwrite(true).build();
    }
```
发现它是new一个SpringJobScheduler出来的，这个玩意是啥？发现它是继承自JobScheduler的，具体来看看obScheduler这个类，首先看看它的构造函数：

```
   private JobScheduler(final CoordinatorRegistryCenter regCenter, final LiteJobConfiguration liteJobConfig, final JobEventBus jobEventBus, final ElasticJobListener... elasticJobListeners) {
        JobRegistry.getInstance().addJobInstance(liteJobConfig.getJobName(), new JobInstance());
        this.liteJobConfig = liteJobConfig;
        this.regCenter = regCenter;
        List<ElasticJobListener> elasticJobListenerList = Arrays.asList(elasticJobListeners);
        setGuaranteeServiceForElasticJobListeners(regCenter, elasticJobListenerList);
        schedulerFacade = new SchedulerFacade(regCenter, liteJobConfig.getJobName(), elasticJobListenerList);
        jobFacade = new LiteJobFacade(regCenter, liteJobConfig.getJobName(), Arrays.asList(elasticJobListeners), jobEventBus);
    }
```
可以发现构造里面多了SchedulerFacade类和LiteJobFacade这两个类以及JobRegistry这个类，通过查看JobRegistry这个类，发现它实现了单例模式。  
接着看看init方法，为什么要看这个方法呢？因为在springboot方法上是调用该方法：

```
/**
     * 初始化作业.
     */
    public void init() {
        LiteJobConfiguration liteJobConfigFromRegCenter = schedulerFacade.updateJobConfiguration(liteJobConfig);
        JobRegistry.getInstance().setCurrentShardingTotalCount(liteJobConfigFromRegCenter.getJobName(), liteJobConfigFromRegCenter.getTypeConfig().getCoreConfig().getShardingTotalCount());
        JobScheduleController jobScheduleController = new JobScheduleController(
                createScheduler(), createJobDetail(liteJobConfigFromRegCenter.getTypeConfig().getJobClass()), liteJobConfigFromRegCenter.getJobName());
        JobRegistry.getInstance().registerJob(liteJobConfigFromRegCenter.getJobName(), jobScheduleController, regCenter);
        schedulerFacade.registerStartUpInfo(!liteJobConfigFromRegCenter.isDisabled());
        jobScheduleController.scheduleJob(liteJobConfigFromRegCenter.getTypeConfig().getCoreConfig().getCron());
    }
```
可以发现，所有的操作都是围绕SchedulerFacade和JobFacade这几个类来操作的。  
在schedulerFacade中有一个updateJobConfiguration方法，该方法主要用来更新作业配置

```
/**
     * 更新作业配置.
     *
     * @param liteJobConfig 作业配置
     * @return 更新后的作业配置
     */
    public LiteJobConfiguration updateJobConfiguration(final LiteJobConfiguration liteJobConfig) {
        configService.persist(liteJobConfig);
        return configService.load(false);
    }
```
它首先通过configService类去持久化liteJobConfig任务配置信息。

```
  /**
     * 持久化分布式作业配置信息.
     * 
     * @param liteJobConfig 作业配置
     */
    public void persist(final LiteJobConfiguration liteJobConfig) {
        checkConflictJob(liteJobConfig);
        if (!jobNodeStorage.isJobNodeExisted(ConfigurationNode.ROOT) || liteJobConfig.isOverwrite()) {
            jobNodeStorage.replaceJobNode(ConfigurationNode.ROOT, LiteJobConfigurationGsonFactory.toJson(liteJobConfig));
        }
    }
    
```
首先检查zk下面是否存在config的节点，如果存在则抛出异常，否则则将该节点持久化到zk下面。节点名字也是config。数据通过Gson持久化。以下是我电脑上序列化出来Json字符串。

```
{"jobName":"com.dangdang.ddframe.job.example.job.dataflow.SpringDataflowJob","jobClass":"com.dangdang.ddframe.job.example.job.dataflow.SpringDataflowJob","jobType":"DATAFLOW","cron":"0/5 * * * * ?","shardingTotalCount":8,"shardingItemParameters":"0\u003dBeijing,1\u003dShanghai,2\u003dGuangzhou","jobParameter":"","failover":false,"misfire":true,"description":"","jobProperties":{"job_exception_handler":"com.dangdang.ddframe.job.executor.handler.impl.DefaultJobExceptionHandler","executor_service_handler":"com.dangdang.ddframe.job.executor.handler.impl.DefaultExecutorServiceHandler"},"streamingProcess":true,"monitorExecution":true,"maxTimeDiffSeconds":-1,"monitorPort":-1,"jobShardingStrategyClass":"","reconcileIntervalMinutes":10,"disabled":false,"overwrite":true}
```
持久化完成后还需要从zk中拿出具体的配置：

```
  public LiteJobConfiguration load(boolean fromCache) {
        String result;
        if (fromCache) {
            result = this.jobNodeStorage.getJobNodeData("config");
            if (null == result) {
                result = this.jobNodeStorage.getJobNodeDataDirectly("config");
            }
        } else {
            result = this.jobNodeStorage.getJobNodeDataDirectly("config");
        }

        return LiteJobConfigurationGsonFactory.fromJson(result);
    }
```
这里传入的默认为false，则没有走缓存拿，直接从zk中拿取的数据，拿到数据之后通过反序列化将数据转换成LiteJobConfiguration对象。
### 总结
本文简单的简单的介绍了elastic-job 局部启动流程，对elastic-job有了一个大概的了解，为接下来详细分析每个操作步骤奠定了基础。




