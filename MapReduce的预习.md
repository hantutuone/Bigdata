# MapReduce的定义

用于在大型的商用硬件集群中(成千上万的结点)对海量数据（多个兆字节数据集）实施可靠的、高容错的分布式计算的框架，也是一种经典的并行计算模型。

# MapReduce的基本原理

将一个复杂的问题(数据集)分成若干个简单的子问题(数据块)进行解决(Map函数)；然后对子问题的结果进行合并（Reduce函数），得到原有问题的解（结果）。MapReduce模型适合于大文件的处理，对很多小文件的处理效率不是很高。

[]: https://img-blog.csdnimg.cn/20200205214451508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTQ0NTUw,size_16,color_FFFFFF,t_70	"**MapReduce的基本原理图**"

# MapReduce编程模型简介

MapReduce运行环境一般由两种不同类型的节点组成，分别是负责数据处理的Worker节点，负责任务调度以及各节点间数据共享管理的Master节点。其核心是 Map 和 Reduce 两个函数，这两个函数由用户负责实现，功能是按一定的映射规则将输入的 <key, value> 对转换成另一个或一批 <key, value> 对输出。 Map <k1, v1> List(<k2,v2>) 1. 将小数据集进一步解析成一批 <key,value> 对，输入 Map 函数中进行处理。每一个输入的 <k1,v1> 会输出一批 <k2,v2>。 <k2,v2> 是计算的中间结果。Reduce <k2,List(v2)> <k3,v3> 输入的中间结果 <k2,List(v2)> 中的 List(v2) 表示是一批属于同一个 k2 的 value

## MapReduce编程模型分类

**MapReduce简单模型**：模型只有Mapper过程，由Maper产生的数据直接写入HDFS。

**MapReduce复杂模型**：对于大部分的任务来说，都是需要Reduce过程的，并且由于任务繁重，会启动多个Reducer（默认为1，根据任务量可由用户自己设定合适的Reducer 数量》来进行汇。

## MapReduce编程实例（WordCount）

```
public class WordCount {
  public static class TokenizerMapper  extends Mapper<Object, Text, Text, IntWritable>{
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        //针对每个单词输出一个<word ,1>
        //MapReduce 计算框架会将这些<word ,1>收集起来，将相同的word放一起，形成
        //<word,<1,1,1,...>>这样的<key,value集合>，然后输入给reduce
        context.write(word, one);
      }
    }
  }
 
  public static class IntSumReducer    extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();
    public void reduce(Text key, Iterable<IntWritable> values, Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
          //reduce对每个word对应的所有1 进行求和，最终将<word,合计>输出
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
}
```

(**MapReduce适用于大问题分解而的小问题彼此之间没有依赖关系的情形**)

# MapReduce数据流 

[]: https://img-blog.csdn.net/2018071622185744?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5ndGlhbnB1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70

Split：交由MapReduce作业来处理的数据块，是MapReduce中最小的计算单元

 HDFS：blockSize 是HDFS中最小的存储单元默认为128M

 默认情况下：split和HDFS的Block他们两个是一一对应的关系，当然也可以手工设置（不建议）

 InputFormat:将我们的输入数据进行分片（split）

 TextInpuFormat：处理文本格式的数据

 OutputFormat：与上面InputFormat对应处理输出

Combiner： 本地的reducer， 减少Map Tasks输出的数据量及数据网络传输量

 Partitioner：决定MapTask输出的数据交由那个ReduceTask处理

[]: https://img-blog.csdn.net/20180716221916337?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5ndGlhbnB1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70
[]: https://img-blog.csdn.net/20180716221948790?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5ndGlhbnB1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70

# Yarn的基本组成

[]: https://img-blog.csdn.net/20180716221948790?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5ndGlhbnB1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70	"Yarn中MapReduce的任务运行流程"

1. client 向 ResourceManager提交任务。
2. ResourceManage分配该任务的第一个 container，并通知相应的NodeManager 自动MRAppMaster
3. NodeManager接收命令后，开辟一个 container资源空间，并在 container中启动相应的MRAppMaster
4. MRAppMaster启动之后，第一步会向 ResourceManager注册，这样用户可以直接通过MRAppMaster监控任务的运行状态；之后则直接由MRAppMaster调度任务运行，重复5)
   8）直到任务结束。
5. MRAppMaster以轮询的方式向ResourceManager申请任务运行所需的资源。
6. 一旦ResourceManager配给了资源，MRAppMaster便会与相应的NodeManager通信，让它划分Container并启动相应的任务(MapTask或ReduceTask)。
7. NodeManager准备好运行环境，启动任务。
8. 各任务运行，并定时通过RPC协议向MRAppMaster汇报自己的运行状态和进度。MRAppMaster也会实时地监控任务的运行，当发现某个Task假死或失败时，便杀死它重新启动任务。
9. 任务完成，MRAppMaster向ResourceManager通信，注销并关闭自己。

