# HDFS Data Processing with MapReduce
## Introduction

This project demonstrates the use of Hadoop MapReduce for processing large datasets. It includes two exercises:

1. Sales Data Processing
2. Web Log Analysis

## Project Structure

```
src
 ├───main
 │   ├───java
 │   │   └───com
 │   │       └───mouakkal
 │   │           ├───ex1
 │   │           │   ├───job1
 │   │           │   │       Driver.java
 │   │           │   │       JobMapper.java
 │   │           │   │       JobReducer.java
 │   │           │   └───job2
 │   │           │           Driver.java
 │   │           │           JobMapper.java
 │   │           │           JobReducer.java
 │   │           ├───ex2
 │   │           │       Driver.java
 │   │           │       JobMapper.java
 │   │           │       JobReducer.java
 │   │           └───word_counting
 │   │                   WordCountDriver.java
 │   │                   WordCountMapper.java
 │   │                   WordCountReducer.java
 │   └───resources
 └───test
     └───java
```

## Exercise 1: Sales Data Processing

### Objective

Analyze sales data stored in a text file to compute the total sales by city. Each line in the file contains information about a sale, including the date, city, product, and price in the following format:

```
date city product price
```

### Task 1: Calculate Total Sales by City

#### Driver Code

```java
public class Driver {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(Driver.class);
        job.setMapperClass(JobMapper.class);
        job.setReducerClass(JobReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(DoubleWritable.class);
        job.setInputFormatClass(TextInputFormat.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

#### Mapper Code

```java
public class JobMapper extends Mapper<LongWritable, Text, Text, DoubleWritable> {
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, DoubleWritable>.Context context) throws IOException, InterruptedException {
        String[] tokens = value.toString().split(" ");
        if (tokens.length == 4) {
            String city = tokens[1];
            double price = Double.parseDouble(tokens[3]);
            context.write(new Text(city), new DoubleWritable(price));
        }
    }
}
```

#### Reducer Code

```java
public class JobReducer extends Reducer<Text, DoubleWritable, Text, DoubleWritable> {
    @Override
    protected void reduce(Text key, Iterable<DoubleWritable> values, Reducer<Text, DoubleWritable, Text, DoubleWritable>.Context context) throws IOException, InterruptedException {
        double totalSales = 0.0;
        for (DoubleWritable value : values) {
            totalSales += value.get();
        }
        context.write(key, new DoubleWritable(totalSales));
    }
}
```

#### Execution

```bash
hadoop jar tp2_hdfs-1.0-SNAPSHOT.jar com.mouakkal.ex1.job1.Driver /TP2-HADOOP/ventes.txt /TP2-HADOOP/ex1_job1_output
```

### Task 2: Calculate Total Sales by Product and City for a Given Year

#### Driver Code

```java
public class Driver {
   public static void main(String[] args) throws Exception {
      Configuration conf = new Configuration();
      Job job = Job.getInstance(conf);
      job.setJarByClass(Driver.class);
      job.setMapperClass(JobMapper.class);
      job.setReducerClass(JobReducer.class);
      job.setOutputKeyClass(Text.class);
      job.setOutputValueClass(DoubleWritable.class);
      job.setInputFormatClass(TextInputFormat.class);
      FileInputFormat.addInputPath(job, new Path(args[0]));
      FileOutputFormat.setOutputPath(job, new Path(args[1]));
      job.waitForCompletion(true);
   }
}
```

#### Mapper Code

```java
public class JobMapper extends Mapper<LongWritable, Text, Text, DoubleWritable> {
   @Override
   protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, DoubleWritable>.Context context) throws IOException, InterruptedException {
      String[] tokens = value.toString().split(" ");
      if (tokens.length == 4) {
         String city = tokens[1];
         double price = Double.parseDouble(tokens[3]);
         context.write(new Text(city), new DoubleWritable(price));
      }
   }
}
```

#### Reducer Code

```java
public class JobReducer extends Reducer<Text, DoubleWritable, Text, DoubleWritable> {
   @Override
   protected void reduce(Text key, Iterable<DoubleWritable> values, Reducer<Text, DoubleWritable, Text, DoubleWritable>.Context context) throws IOException, InterruptedException {
      double totalSales = 0.0;
      for (DoubleWritable value : values) {
         totalSales += value.get();
      }
      context.write(key, new DoubleWritable(totalSales));
   }
}
```

#### Execution (With Year 2024 as Parameter)

```bash
hadoop jar tp2_hdfs-1.0-SNAPSHOT.jar com.mouakkal.ex1.job2.Driver /TP2-HADOOP/ventes.txt /TP2-HADOOP/ex1_job2_output 2024
```

## Exercise 2: Web Log Analysis

### Objective

Analyze a web log file to count the total number of requests and the number of successful requests (HTTP response code 200) per IP address. Each line in the log file contains information about a request, including the IP address, date, HTTP method, URL, response code, and other details.

### Driver Code

```java
public class Driver {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(Driver.class);
        job.setMapperClass(JobMapper.class);
        job.setReducerClass(JobReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setInputFormatClass(TextInputFormat.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

### Mapper Code

```java
public class JobMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        String word = value.toString();
        String[] informations = word.split(" ");
        String ipAdd = informations[0].split(" -- ")[0];
        context.write(new Text(ipAdd), new IntWritable(1));
    }
}
```

### Reducer Code

```java
public class JobReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        int counter = 0;
        Iterator<IntWritable> iterator = values.iterator();
        while (iterator.hasNext()) {
            counter += iterator.next().get();
        }
        context.write(key, new IntWritable(counter));
    }
}
```

### Execution

```bash
hadoop jar tp2_hdfs-1.0-SNAPSHOT.jar com.mouakkal.ex2.Driver /TP2-HADOOP/log.txt /TP2-HADOOP/ex2_output
```

## Conclusion

This project demonstrates the efficient use of Hadoop MapReduce to process large datasets in various contexts, including sales data analysis and web log analysis. The exercises illustrate the capability of MapReduce to distribute computational tasks, enabling fast and scalable data processing essential for big data applications.