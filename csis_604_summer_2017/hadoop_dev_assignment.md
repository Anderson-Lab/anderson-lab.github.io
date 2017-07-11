---
permalink: /csis_604_summer_2017/hadoop_dev_assignment/
title: "Distributed Computing"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

In this assignment, we will install and configure IntelliJ and a Hadoop development environment. We will make a change to one of the Hadoop demos and deploy our new code.

## Download and install IntelliJ
On master
<code>
wget https://download.jetbrains.com/idea/ideaIC-2017.1.5.tar.gz
tar xfvz ideaIC-2017.1.5.tar.gz
</code>

Now make sure it works and your X11 forwarding is enabled (some documentation: https://www.seas.upenn.edu/cets/answers/x11-forwarding.html). To start:
<code>
idea-IC-171.4694.70/bin/idea.sh
</code>
I just kept the defaults for everything.

## Creating a Hadoop App Project
1. Create a Java project in IntelliJ.
2. Set up the libraries you need. I added basically everything which is overkill, but still... 
<img src="csis_604_summer_2017/screenshots/Libraries.PNG">
3. Now I created a source file:
<code>
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import java.io.IOException;
import java.util.StringTokenizer;

public class TestWordCount {

    public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer
            extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                           Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

   public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 3) {
            System.err.println("Usage: TestWordCount <in> <out>");
            System.exit(2);
        }
        System.out.println("Called with: "+otherArgs[0]+" "+otherArgs[1]+" "+otherArgs[2]);
        Job job = Job.getInstance(conf);

        job.setJarByClass(TestWordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[1]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[2]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
</code>
4. After that we need to setup the artifacts, so it produces a jar we can run on Hadoop. Click File -&gt; Project Structure, select artifacts on the left. Click Add button -&gt; Jar -&gt; From modules with dependencies. Choose the module you create. If you specify the Main class here, you don’t need to add its class name in the following command. Click OK to save the settings.
5. Then build the artifiacts. I had one more jar I needed to add, so I downloaded it using wget http://mirror.metrocast.net/apache//commons/cli/binaries/commons-cli-1.4-bin.tar.gz. Then extracted it with tar xfvz commons-cli-1.4-bin.tar.gz. Then my jar artifact was built successfully. 
6. We can try it from the command line, and assuming your input directory still exists, you can do so by running your jar on your cluster. My jar ended up in /home/lab/IdeaProjects/HadoopExample/out/artifacts/HadoopExample_jar. Your jar might be somewhere else depending on how you named things. Then you can try to run something like: hadoop jar HadoopExample.jar TestWordCount /input output

## Modify Code
For the final part, modify the Java program so that it is case insensitive. Show how the two programs now give different answers.
