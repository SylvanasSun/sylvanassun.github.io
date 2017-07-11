---
layout:     post
title:      "Spring Batch数据批处理"
subtitle:   "Spring Batch"
date:       2016-08-04 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - Spring Boot
tags:
    - Spring Boot
    - Spring
    - Java
---


### 概述


----------


&nbsp;&nbsp;Spring Batch是一个轻量级的处理大量数据操作的框架,主要用于读取大量数据,然后进行一定处理后输出成指定的格式.

&nbsp;&nbsp;Spring Batch 提供了大量可重用的组件,包括了日志、追踪、事务、任务作业统计、任务重启、跳过、重复、资源管理。对于大数据量和高性能的批处理任务,Spring Batch 同样提供了高级功能和特性来支持,比如分区功能、远程功能。总之,通过 Spring Batch 能够支持简单的、复杂的和大数据量的批处理作业.

&nbsp;&nbsp;Spring Batch 是一个批处理应用框架,不是调度框架,但需要和调度框架合作来构建完成的批处理任务。它只关注批处理任务相关的问题,如事务、并发、监控、执行等,并不ﰁ供相应的调度功能。如果需要使用调用框架,在商业软件和开源软件中已经有很多优秀的企业级调度框架(如 Quartz、Tivoli、Control-M、Cron 等)可以使用.

### 应用场景


----------


 - 周期性的提交处理.
 - 并行处理任务.
 - 消息驱动应用分级处理.
 - 手工或调度使任务失败之后重新启动.
 - 有依赖步骤的顺序执行(使用工作流驱动扩展).
 - 处理时跳过部分记录.
 - 成批事务：为小批量的或有的存储过程/脚本的场景使用.

### 组成结构


----------


![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6navcbgzhj20ki0860tg.jpg)

| Name          | Description                                     |
| ------------- | ----------------------------------------------- |
| Job           | 实际要执行的任务,包含一个或多个Step             |
| JobRepository | 用于注册Job的容器                               |
| JobLauncher   | 用于启动Job的接口                               |
| Step          | Step步骤包含ItemReader,ItemProcessor,ItemWriter |
| ItemReader    | 用于读取数据的接口                              |
| ItemProcessor | 用于处理数据的接口                              |
| ItemWriter    | 用于输出数据的接口                              |

**&nbsp;&nbsp;以上组件只需要注册成Spring的Bean即可,并在配置类上使用`@EnableBatchProcessing`开启批处理的支持.**

### Job Listener


----------


&nbsp;&nbsp;如果想要监听Job的执行情况,则需要定义一个实现了**JobExecutionListener**的类,并在定义Job的Bean上绑定该监听器.

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6ncme1ztdj20kk0e0n09.jpg)

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6ncmeoja5j20c4054wey.jpg)

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6ncmeln1gj20he07tjsq.jpg)

### 数据处理和校验


----------


&nbsp;&nbsp;数据的处理和校验都要通过ItemProcessor接口实现来完成的.

#### 数据处理


----------


&nbsp;&nbsp;实现ItemProcessor接口,并重写process方法,即可对数据进行处理.输入参数是从ItemReader读取到的数据,返回给ItemWriter.

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6nd7a60nrj20kv06adh4.jpg)

#### 数据校验


----------


&nbsp;&nbsp;数据校验可以使用JSR-303的注解来校验ItemReader读取的数据是否符合要求.

![定义实体类的检验规则](http://ww1.sinaimg.cn/mw690/63503acbjw1f6ndeuabtvj20gt06ijs8.jpg)

![定义校验器](http://ww4.sinaimg.cn/mw690/63503acbjw1f6ndevin2gj20p30ku44i.jpg)

![ItemProcessor](http://ww4.sinaimg.cn/mw690/63503acbjw1f6ndeuf8dkj20kc0c8acs.jpg)

![注册校验器](http://ww4.sinaimg.cn/mw690/63503acbjw1f6ndev1nz8j20dk04s0t9.jpg)

![注册ItemProcessor](http://ww3.sinaimg.cn/mw690/63503acbjw1f6ndeupgvxj20g407lgmy.jpg)

### 参数后置绑定


----------


&nbsp;&nbsp;实现参数后置绑定可以在JobParameters中绑定参数,在Bean定义的时候使用一个特殊的Bean生命周期注解`@StepScope`,然后通过`@Value`注入此参数.

![绑定参数](http://ww4.sinaimg.cn/mw690/63503acbjw1f6nejvlqcuj20hs0b3jtx.jpg)

![定义ItemReader](http://ww3.sinaimg.cn/mw690/63503acbjw1f6nejv24v2j20u70e1n1d.jpg)

### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot自动初始化了Spring Batch存储批处理的数据库,并当程序启动时,会自动执行Job.

&nbsp;&nbsp;Spring Boot使用以`spring.batch`为前缀的属性进行相关配置.

```
spring.batch.job.name=job1,job2 #启动时要执行的job,默认执行全部job.
spring.batch.job.enabled=true #是否自动执行定义的job,默认为true
spring.batch.initializer.enabled=true #是否初始化Spring Batch的数据库,默认为true.
spring.batch.schema=
spring.batch.table-prefix= #设置Spring Batch的数据库表的前缀.
```

&nbsp;&nbsp;Spring Batch默认自动加载hsqldb驱动,如要使用其他数据库驱动则需要手动去除.

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6neue2xagj20lw0i3n25.jpg)

#### Config Example




![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6s3o0w6n1j20u60dtdjx.jpg)

```java

    /**
     * 定义ItemProcessor
     */
    @Bean
    public ItemProcessor<Person, Person> processor() {
        // 使用自定义的ItemProcessor
        CsvItemProcessor processor = new CsvItemProcessor();
        // 指定校验器
        processor.setValidator(csvBeanValidator());
        return processor;
    }

    /**
     * 定义Validator
     */
    @Bean
    public Validator<Person> csvBeanValidator() {
        return new CsvBeanValidator<Person>();
    }

    /**
     * 定义ItemWriter
     */
    @Bean
    public ItemWriter<Person> writer(DataSource dataSource) {
        JdbcBatchItemWriter<Person> writer = new JdbcBatchItemWriter<>();
        writer.setItemSqlParameterSourceProvider
                (new BeanPropertyItemSqlParameterSourceProvider<Person>());
        // 设置要执行批处理的sql语句
        String sql = "insert into person " + "(id,name,age,nation,address)"
                + "values(hibernate_sequence.nextval,:name,:age,:nation,:address)";
        writer.setSql(sql);
        writer.setDataSource(dataSource);
        return writer;
    }

    /**
     * 定义JobRepository
     */
    @Bean
    public JobRepository jobRepository(DataSource dataSource,
                                       PlatformTransactionManager transactionManager)
            throws Exception {
        JobRepositoryFactoryBean jobRepositoryFactoryBean = new JobRepositoryFactoryBean();
        jobRepositoryFactoryBean.setDataSource(dataSource);
        jobRepositoryFactoryBean.setTransactionManager(transactionManager);
        jobRepositoryFactoryBean.setDatabaseType("mysql");
        return jobRepositoryFactoryBean.getObject();
    }

    /**
     * 定义JobLauncher
     */
    @Bean
    public JobLauncher jobLauncher(DataSource dataSource,
                                   PlatformTransactionManager transactionManager)
            throws Exception {
        SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
        jobLauncher.setJobRepository(jobRepository(dataSource, transactionManager));
        return jobLauncher;
    }

    /**
     * 定义Job
     */
    @Bean
    public Job importJob(JobBuilderFactory jobs, Step s1) {
        return jobs.get("importJob")
                .incrementer(new RunIdIncrementer())
                .flow(s1) // 为job指定step
                .end()
                .listener(csvJobListener()) // 绑定监听器
                .build();
    }

    /**
     * 定义Step
     */
    @Bean
    public Step step1(StepBuilderFactory stepBuilderFactory, ItemReader<Person> reader,
                      ItemWriter<Person> writer,
                      ItemProcessor<Person, Person> processor) {
        return stepBuilderFactory
                .get("step1")
                .<Person, Person>chunk(65000) // 批处理每次提交65000条数据.
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .build();
    }

    /**
     * 定义自定义的Job监听器
     */
    @Bean
    public CsvJobListener csvJobListener() {
        return new CsvJobListener();
    }
	
```

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
