## MapReduce

### 1. Word Count 

#### 1.1 java

##### 1.1.1 pom.xml

```xml
<!-- Apache Hadoop Client Aggregator -->
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>3.2.1</version>
</dependency>
```

##### 1.1.2 map

```java

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WordCountMapper extends Mapper<LongWritable, Text, Text, LongWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        String[] words = line.split("");
        for (String word : words) {
            context.write(new Text(word), new LongWritable(1));
        }

    }
}

```

##### 1.1.3 reduce

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReducer extends Reducer<Text, LongWritable, Text, LongWritable> {

    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for (LongWritable value : values) {
            count += value.get();
        }
        context.write(key, new LongWritable(count));
    }
}
```

##### 1.1.4 runner

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class WordCountRunner {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);
        // Hadoop会自动根据驱动程序的类路径来扫描该作业的Jar包。
        job.setJarByClass(WordCountRunner.class);

        // 指定mapper
        job.setMapperClass(WordCountMapper.class);
        // 指定reducer
        job.setReducerClass(WordCountReducer.class);

        // map程序的输出键-值对类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        // 输出键-值对类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        // 输入文件的路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        // 输入文件路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean res = job.waitForCompletion(true);
        System.exit(res ? 0 : 1);
    }
}
```

#### 1.2 打包上传至服务器

```shell
mvn clean package
--> hadoop-case-1.0-SNAPSHOT.jar
```

```shell
scp target/hadoop-case-1.0-SNAPSHOT.jar root@aliyun:/usr/apps/mine
```

#### 1.3 执行

##### 1.3.1 准备待分析文件

```shell
root@Spectred:/usr/apps/mine# hdfs dfs -cat /home/wc/test.txt
2020-01-06 10:46:17,010 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
Hello World 

Hello Hadoop
```

1.3.2 执行MapReduce

```shell
hadoop jar hadoop-case-1.0-SNAPSHOT.jar org.example.hadoop.mapreduce.WordCountRunner /home/wc/ /home/wc/out/

2020-01-06 10:38:18,603 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2020-01-06 10:38:19,522 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2020-01-06 10:38:19,551 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/root/.staging/job_1578277346863_0005
2020-01-06 10:38:19,737 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2020-01-06 10:38:19,954 INFO input.FileInputFormat: Total input files to process : 1
2020-01-06 10:38:20,013 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2020-01-06 10:38:20,035 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2020-01-06 10:38:20,042 INFO mapreduce.JobSubmitter: number of splits:1
2020-01-06 10:38:20,222 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2020-01-06 10:38:20,266 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1578277346863_0005
2020-01-06 10:38:20,266 INFO mapreduce.JobSubmitter: Executing with tokens: []
2020-01-06 10:38:20,591 INFO conf.Configuration: resource-types.xml not found
2020-01-06 10:38:20,593 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2020-01-06 10:38:20,666 INFO impl.YarnClientImpl: Submitted application application_1578277346863_0005
2020-01-06 10:38:20,721 INFO mapreduce.Job: The url to track the job: http://Spectred:8088/proxy/application_1578277346863_0005/
2020-01-06 10:38:20,721 INFO mapreduce.Job: Running job: job_1578277346863_0005
2020-01-06 10:38:29,995 INFO mapreduce.Job: Job job_1578277346863_0005 running in uber mode : false
2020-01-06 10:38:29,997 INFO mapreduce.Job:  map 0% reduce 0%
2020-01-06 10:38:36,090 INFO mapreduce.Job:  map 100% reduce 0%
2020-01-06 10:38:43,139 INFO mapreduce.Job:  map 100% reduce 100%
2020-01-06 10:38:44,159 INFO mapreduce.Job: Job job_1578277346863_0005 completed successfully
2020-01-06 10:38:44,302 INFO mapreduce.Job: Counters: 54
        File System Counters
                FILE: Number of bytes read=305
                FILE: Number of bytes written=452055
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=130
                HDFS: Number of bytes written=43
                HDFS: Number of read operations=8
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters 
                Launched map tasks=1
                Launched reduce tasks=1
                Data-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=3908
                Total time spent by all reduces in occupied slots (ms)=4090
                Total time spent by all map tasks (ms)=3908
                Total time spent by all reduce tasks (ms)=4090
                Total vcore-milliseconds taken by all map tasks=3908
                Total vcore-milliseconds taken by all reduce tasks=4090
                Total megabyte-milliseconds taken by all map tasks=4001792
                Total megabyte-milliseconds taken by all reduce tasks=4188160
        Map-Reduce Framework
                Map input records=3
                Map output records=25
                Map output bytes=249
                Map output materialized bytes=305
                Input split bytes=103
                Combine input records=0
                Combine output records=0
                Reduce input groups=11
                Reduce shuffle bytes=305
                Reduce input records=25
                Reduce output records=11
                Spilled Records=50
                Shuffled Maps =1
                Failed Shuffles=0
                Merged Map outputs=1
                GC time elapsed (ms)=182
                CPU time spent (ms)=1120
                Physical memory (bytes) snapshot=326651904
                Virtual memory (bytes) snapshot=5068230656
                Total committed heap usage (bytes)=170004480
                Peak Map Physical memory (bytes)=213000192
                Peak Map Virtual memory (bytes)=2530734080
                Peak Reduce Physical memory (bytes)=113651712
                Peak Reduce Virtual memory (bytes)=2537496576
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters 
                Bytes Read=27
        File Output Format Counters 
                Bytes Written=43
```

1.3.3 查看执行结果

```shell
root@Spectred:/usr/apps/mine# hdfs dfs -ls /home/wc/out
Found 2 items
-rw-r--r--   1 root supergroup          0 2020-01-06 10:38 /home/wc/out/_SUCCESS
-rw-r--r--   1 root supergroup         43 2020-01-06 10:38 /home/wc/out/part-r-00000
```

```shell
root@Spectred:/usr/apps/mine# hdfs dfs -cat /home/wc/out/part-r-00000
2020-01-06 10:45:15,250 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
        1
        3
H       3
W       1
a       1
d       2
e       2
l       5
o       5
p       1
r       1
```

