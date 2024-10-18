---
title: Exploring High-Performance Data Transfer in Microservices using gRPC, WebSockets, and REST
date: 2024-10-17 12:00:00 -500
categories: [web-development]
tags: [server,spring-boot-3,java]
---


# Project Overview  

We set out to tackle a common scenario in distributed systems: transferring large volumes of data between microservices. 
Specifically, our goal was to efficiently move **10,000** entries in batches of **500** between two microservices, using **Spring Boot 3** and **Java 17**. 
This task served as a benchmark to evaluate the performance, scalability, and developer experience of each protocol.<br>
[Github repository](https://github.com/Source-Code-Wizard/Data-transfer-project/tree/develop)<br>

### Key features of our implementation include:
- Utilizing spring batch for data loading and processing.
- RESTful API endpoints with both synchronous and asynchronous implementations.
- gRPC streaming for efficient, bidirectional communication.
- WebSocket connections for real-time data transfer.
- Data transfer results for the three mentioned protocols. 


## Efficient Data Retrieval Strategy

Our primary objective is to transfer 10,000 entries from the database of Microservice A to the database of Microservice B. This task necessitates an efficient data retrieval strategy that minimizes database calls while avoiding excessive memory consumption.

### Batch Processing Approach

To achieve optimal performance, we've implemented a batch processing approach. This method strikes a balance between reducing the number of database queries and maintaining manageable memory usage within our application.

Key considerations:

1. **Avoiding Bulk Retrieval**: Executing a single `findAll()` query to fetch 10,000 entries simultaneously is suboptimal. Such an approach would lead to excessive memory allocation, potentially degrading application performance.

2. **Batch Size Optimization**: After careful consideration, we've determined that a batch size of 500 entries provides an ideal balance. This configuration results in a total of 20 database calls to retrieve the entire dataset, effectively distributing the memory load across multiple operations.

3. **Leveraging Spring Boot 3**: To implement this batch processing strategy, we've utilized the Spring Batch library, a powerful feature of Spring Boot 3. This library provides robust support for batch processing operations, significantly simplifying our implementation.

By adopting this approach, we ensure efficient data retrieval while maintaining optimal application performance throughout the transfer process.


## Key Points of Spring Batch Configuration

```java
package source.code.wizard.senderapp.configuration;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.batch.item.ItemWriter;
import org.springframework.batch.item.data.RepositoryItemReader;
import org.springframework.batch.item.data.builder.RepositoryItemReaderBuilder;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;
import org.springframework.data.domain.Sort;
import org.springframework.transaction.PlatformTransactionManager;
import source.code.wizard.senderapp.batch.DataTransferItemWriter;
import source.code.wizard.senderapp.model.DataEntity;
import source.code.wizard.senderapp.service.DataTransferService;
import source.code.wizard.senderapp.repository.DataEntityRepository;

import java.util.Map;

@Configuration
@DependsOn("RandomDataGenerator")
public class SpringBatchConfiguration {

    private final DataEntityRepository dataEntityRepository;

    @Qualifier("RestDataTransferService")
    private final DataTransferService dataTransferService;

    private final JobRepository jobRepository;

    private static final int CHUNK_SIZE = 500;
    
  public SpringBatchConfiguration(@Qualifier("DataTransferService") DataTransferService dataTransferService,
                                    JobRepository jobRepository,
                                    DataEntityRepository dataEntityRepository) {
        this.dataTransferService = dataTransferService;
        this.jobRepository = jobRepository;
        this.dataEntityRepository = dataEntityRepository;
    }
    
    @Bean
    public Job dataTransferJob(Step dataTransferStep) {
        return new JobBuilder("dataTransferJob", jobRepository)
                .start(dataTransferStep)
                .build();
    }

    @Bean
    public Step dataTransferStep(JobRepository jobRepository,
                                 PlatformTransactionManager platformTransactionManager,
                                 RepositoryItemReader<DataEntity> reader,
                                 ItemWriter<DataEntity> writer) {
        return new StepBuilder("dataTransferStep", jobRepository)
                .<DataEntity, DataEntity>chunk(500, platformTransactionManager)
                .reader(reader)
                .processor(processor())
                .writer(writer)
                .build();

    }

    @Bean
    public ItemWriter<DataEntity> writer() {
        return new DataTransferItemWriter(dataTransferService);
    }

    @Bean
    public ItemProcessor<DataEntity, DataEntity> processor() {
        // Simple pass-through processor
        return item -> item;
    }

    @Bean
    public RepositoryItemReader<DataEntity> reader() {
        return new RepositoryItemReaderBuilder<DataEntity>()
                .name("dataEntityReader")
                .repository(dataEntityRepository)
                .methodName("findAll")
                .arguments()
                .sorts(Map.of("id", Sort.Direction.ASC))
                .pageSize(CHUNK_SIZE)
                .build();
    }
}
```

### 1. Spring Batch Configuration Overview
This configuration sets up a Spring Batch job for transferring data using a repository-based item reader and a custom item writer. It uses the newer `JobBuilder` and `StepBuilder` introduced in Spring Batch 5, which allows for better configuration flexibility.

### 2. Annotations Used
- `@Configuration`: Marks the class as a Spring configuration class.
- `@DependsOn("RandomDataGenerator")`: Ensures that this configuration is loaded after the specified bean (`RandomDataGenerator`) has been initialized. This bean is responsible for creating and inserting 10.000 dummy entries in our database.

### 3. Bean Definitions
The configuration defines several beans required for the batch job setup:
- **Job (`dataTransferJob`)**: Defines a batch job named `dataTransferJob` that executes the specified step (`dataTransferStep`).
- **Step (`dataTransferStep`)**: Configures a step within the job using chunk-based processing, where the size of each chunk is set to 500 items.
- **RepositoryItemReader (`reader`)**: Reads data from the `DataEntityRepository` using pagination and sorting.
- **ItemProcessor (`processor`)**: Processes each item in the batch; in this case, it's a simple pass-through processor.
- **ItemWriter (`writer`)**: Uses a custom writer (`DataTransferItemWriter`) that sends data in batches using a REST-based service.

### 4. Writer Setup
- **Custom ItemWriter**: The`DataTransferItemWriter` class, implements the ItemWriter interface and delegates the task of sending the data to a DataTransferService service. This approach helps in decoupling data processing from data transmission.

```java
package source.code.wizard.senderapp.batch;

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.item.Chunk;
import org.springframework.batch.item.ItemWriter;
import org.springframework.beans.factory.annotation.Qualifier;
import source.code.wizard.senderapp.model.DataEntity;
import source.code.wizard.senderapp.service.DataTransferService;

import java.util.List;


@Slf4j
public class DataTransferItemWriter implements ItemWriter<DataEntity> {

    @Qualifier("DataTransferService")
    private final DataTransferService dataTransferService;

    public DataTransferItemWriter(@Qualifier("DataTransferService") DataTransferService dataTransferService) {
        this.dataTransferService = dataTransferService;
    }

    @Override
    public void write(Chunk<? extends DataEntity> chunk) {
        final List<? extends DataEntity> itemsToBeTransfered = chunk.getItems();
        dataTransferService.sendDataInBatches((List<DataEntity>) itemsToBeTransfered);
    }

}
```
### 5. Dependencies
```xml

<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-core</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-infrastructure</artifactId>
</dependency>
```
