# MapReduce (WordCount Example)

Consider the `MapReduce` program for the `WordCount` program.

Here are some things to consider first- 
- Input file - `poem.text`

Here is some part of the input file `poem.txt`

```txt
What I won't tell you is how I became a flute
and brushed against lips but there was no music.
When the blows came furious as juniper.
```

So input-value type is `Text`, and now we have to decide for the key value, since key value has to be unique we can choose the line no to be its key, and so input-key type is `LongWritable`. A example of key-value pair in this case is `<0, "What I won't tell you is how I became a flute">`.

In output of this program we want the total occurences of a particular word, i.e. output key-value pair will look like `<"the", 5>`.

So here is summary of what will happen:-

* input file
    - `poem.txt`

* driver class will recieve this input file
    - driver class will then call the mapper function

* mapper class will also recieve the same input file but in serialized form
    - key-value pair generated by the mapper class
      - `{ <0, "What I won't tell you is how I became a flute">,`
      - `<1, "and brushed against lips but there was no music.">, ... }`
    
* reducer class will recieve the key-value pair generated by the mapper class
    - reducer class will reduce the mapper key-value pairs to the output key-value pairs
    - output key-value pair generated by the reducer class
      - `{ <"What", 1>, <"I", 1>, <"won't", 1>, <"tell", 1>, ...}`
    
* output file
    - `part1000`


## Driver Class

``` java
public class WordCountDriver {
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Job j = new Job(); // is creates a new job object.
        j.setJobName("My First Job"); // sets the job name
        j.setJarByClass(WordCountDriver.class); // sets the name of the driver class
        j.setMapperClass(WordCountMapper.class); // sets the name of the mappper class
        j.setReducerClass(WordCountReducer.class); // sets the name of the reducer class
        j.setOutputKeyClass(Text.class); // sets the type of the input key
        j.setOutputValueClass(IntWritable.class); // sets the type of the output value
        FileInputFormat.addInputPath(j, new Path(args[0])); // sets the input path, input file
        FileOutputFormat.setOutputPath(j, new Path(args[1])); // sets the output path
        System.exit(job.waitForCompletion(true) ? 0 : 1); // return the return value of the job execution
    }
}
```

## Mapper Class

Here are some thing we need to make a mapper class:
- input key type - `LongWritable` in this case
- input value type - `Text` in this case
- output key type - `Text` in this case
- output value type - `IntWritable` in this case

- `public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{}` 
  - the declaration of the mapper class
    - `Mapper<[key-type1], [value-type-1], [key-type-2], [value-type-2]>`
- `public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException{}`
  - overriding the map function

```java 
public void map(LongWritable key, Text value, Context context) {

    /* 
        sample input
        key - 0, value - "What I won't tell you is how I became a flute"
        key - 1, value - "and brushed against lips but there was no music."
     */

    /* convering the Text type to native String type so we can perform operations on it */
	String inputstring = value.toString(); 
    /* splitting the sentence into words */
    /* "What I won't tell you is how I became a flute" will split into - "What", "I", "won't", ... */
	for (String x : inputstring.split(" ")) {
        /* writing key value pairs to the context object */
		context.write(new Text(x), new IntWritable(1));
	}

    /* 
        sample output, key value pairs written to the context object
        <"What", 1>, <"I", 1>, <"won't", 1>, ...
    */

}
```

## Reducer Class

Here are some things we need to consider while making reducer class
- input key type - `Text` (same as mapper output key type)
- input value type - `IntWritable` (same as mapper output value type)
- output key type - `Text` in this case
- output value type - `IntWritable` in this case


- `public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {}`
  - the declaration of the reducer class
    - `Reducer<[key-type-1], [value-type-1], [key-type-2], [value-type-2]>`
- `public void reduce(Text key, Iterable<IntWritable> values, Context context)throws IOException, InterruptedException {}`
  - overriding the reduce function

```java

/* 
    suppose output of the reduce function is 
        <"the", 1>, <"a", 1>, <"the", 1>, <"hello", 1>, <"a", 1>, <"the", 1>
    then reduce function will receive 
        <"the", [1,1,1]>, <"a", [1,1]>, <"hello", [1]>
    i.e. a key and a iterable object contining the values
    and then the ouptut we need in this case is
        <"the", 3>, <"a", 2>, <"hello", 1>
*/

public void reduce(Text key, Iterable<IntWritable> values, Context context)
	int y = 0;
    /* looping over the values of the iterable object */
	for (IntWritable x : values) {
        /* summing them up */
		y++;
	}

    /* writing the output key value pair to the contexet object */
	context.write(key, new IntWritable(y));
}
```

An then the final context object is passed back to the `Job` object in the driver class which then produces the output file.

## Execution

```sh
H_CLASSPATH=$(hadoop classpath)
javac *.java
jar -xcf wordcount.jar *.class
hadoop -fs -put poem.txt
hadoop jar wordcount.jar poem.txt wordcountout
hadoop fs -ls wordcountout
```