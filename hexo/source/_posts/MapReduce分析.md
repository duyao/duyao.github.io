title: MapReduce分析
toc: true
date: 2017-03-14 15:14:57
tags: Hadoop
categories: note
---
input - map - combine - shuffle - reduce - output
1. input
如果输入为hdfs，先将块转化为InputSpilt，然后将InputSpilt作为map的输入，
InputSpilt只是逻辑上的数据分片，不会真的在磁盘上存储，只记录原节点信息，
默认会把InputSpilt设置为块大小，可以调整。
然后FileInputFormat会将InputSpilt取出键值对，key为行号，作为map的输入
2. map
执行用户写的map
每一个map对应一个环形缓冲区，用于存储map的输出结果，
缓冲区大小为100m，阈值为80
当超过这个大小，就会在将缓冲区内容溢写spill到磁盘
这个过程中，还会把缓冲的数据划分为若干干分区，，每个分区都会进行排序
如果有combine，这个时候会执行，时间点在合并之前
最后将这写spill文件系进行排序合并，作为map的输出
为了提高io，还可以把map进行压缩

3.shuffle=copy+sort
map的结果在节点的本地磁盘上，因此reduce不会等到所有的任务都会才进行。
只要有一个完成，reduce就会进行复制输出，这就是shuffle的copy阶段
默认reduce会有5个复制线程
数据复制到reduce节点上，也是先进缓冲区，超出阈值，spill到磁盘
复制结束后进入到sort阶段，进行归排序
默认的将归并因子设置为10
从shuffle可以看出，map处理的是一个inputspilt，而reduce是一个分区的所有map中间结果

4.reduce
这时候只需要对这些按键分区有序的结果调用reduce就可以了

mr过程一共有3次排序


![mr过程](http://img.xiaoxiaomo.com/blog%2Fimg%2Fmapreduce00.jpg)

主要有快排和归并

<h1 id="MapReduce理解"><a href="#MapReduce理解" class="headerlink" title="MapReduce理解"></a>MapReduce理解</h1><ul>
<li><strong>MapRedeuce</strong>，我们可以把它分开来理解：</li>
</ul>
<ol>
<li><strong>映射（Mapping）</strong> ：<strong>对集合里的每个目标应用同一个操作</strong>。即，<em>如果你想把表单里每个单元格乘以二，那么把这个函数单独地应用在每个单元格上的操作就属于mapping</em>（这里体现了<strong>移动计算</strong>而不是移动数据）；</li>
<li><strong>化简（Reducing）</strong>：<strong>遍历集合中的元素来返回一个综合的结果</strong>。即，输出表单里一列数字的和这个任务属于reducing。</li>
</ol>
<a id="more"></a>
<ul>
<li><p><strong>计算框架</strong><br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160412199999.png" alt="一个简单的MapReduce执行流程"><br>简单理解，<strong>MapReduce计算框架</strong>：</p>
<blockquote>
<p><strong>把需要计算的东西放入到MapReduce中进行计算，然后返回一个我们期望的结果</strong>。所以<strong>首先</strong>我们需要一个来源（需要计算的东西）即输入（input），<strong>然后</strong>MapReduce操作这个输入（input），通过定义好的计算模型，<strong>最后</strong>得到一个（期望的结果）输出（output）。</p>
</blockquote>
</li>
<li><p><strong>计算模型</strong><br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160703142402.png" alt="Map和Reduce"><br>在这里我们主要讨论的是<strong>MapReduce计算模型</strong>：</p>
<blockquote>
<p>在运行一个mapreduce计算任务时候，任务过程被分为两个阶段：map阶段和reduce阶段，每个阶段都是用键值对（key/value）作为输入（input）和输出（output）。而程序员要做的就是定义好这两个阶段的函数：map函数和reduce函数。</p>
</blockquote>
</li>
</ul>
<h1 id="实例代码"><a href="#实例代码" class="headerlink" title="实例代码"></a>实例代码</h1><ul>
<li>以MapReduce统计单词次数为例（伪代码），主要<strong>四个模块</strong>来讲解，如上图计算框架：</li>
</ul>
<ol>
<li><p><strong>Input，数据读入</strong></p>
<figure class="highlight java"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line"><span class="comment">// 设置数据输入来源</span></span><br><span class="line">FileInputFormat.setInputPaths(job, args[<span class="number">0</span>]);</span><br><span class="line">FileInputFormat.setInputDirRecursive(job, <span class="keyword">true</span>); <span class="comment">//递归</span></span><br><span class="line">job.setInputFormatClass(TextInputFormat.class);	<span class="comment">//设置输入格式</span></span><br><span class="line"></span><br><span class="line"><span class="comment">//TextInputFormat，一种默认的文本输入格式，Mapper一次读取文本中的一行数据。</span></span><br></pre></td></tr></table></figure>
</li>
<li><p><strong>使用Mapper计算</strong></p>
<figure class="highlight java"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br><span class="line">23</span><br><span class="line">24</span><br><span class="line">25</span><br><span class="line">26</span><br><span class="line">27</span><br><span class="line">28</span><br><span class="line">29</span><br><span class="line">30</span><br><span class="line">31</span><br><span class="line">32</span><br><span class="line">33</span><br><span class="line">34</span><br><span class="line">35</span><br></pre></td><td class="code"><pre><span class="line"><span class="comment">//设置Job的Mapper计算类和K2、V2类型</span></span><br><span class="line">job.setMapperClass(WordCountMapper.class);	<span class="comment">//1.设置Mapper类</span></span><br><span class="line">job.setMapOutputKeyClass(Text.class);	<span class="comment">//设置Mapper输出Key的类型</span></span><br><span class="line">job.setMapOutputValueClass(LongWritable.class);<span class="comment">//设置Mapper输出Value的类型</span></span><br><span class="line"></span><br><span class="line"><span class="comment">//WordCountMapper类</span></span><br><span class="line"><span class="comment">/**</span><br><span class="line"> * 自定义的Map 需要继承Mapper</span><br><span class="line"> * K1 : 行序号</span><br><span class="line"> * V1 : 行信息</span><br><span class="line"> * K2 : 单词</span><br><span class="line"> * V2 : 次数</span><br><span class="line"> */</span></span><br><span class="line"><span class="keyword">public</span> <span class="keyword">static</span> <span class="class"><span class="keyword">class</span> <span class="title">WordCountMapper</span> <span class="keyword">extends</span> <span class="title">Mapper</span>&lt;<span class="title">LongWritable</span>,<span class="title">Text</span>,<span class="title">Text</span>,<span class="title">LongWritable</span>&gt; </span>&#123;</span><br><span class="line"></span><br><span class="line">    Text k2 = <span class="keyword">new</span> Text() ;</span><br><span class="line">    LongWritable v2 = <span class="keyword">new</span> LongWritable();</span><br><span class="line"></span><br><span class="line">    <span class="meta">@Override</span></span><br><span class="line">    <span class="function"><span class="keyword">protected</span> <span class="keyword">void</span> <span class="title">map</span><span class="params">(LongWritable key, Text value, Context context)</span> </span><br><span class="line">            <span class="keyword">throws</span> IOException, InterruptedException </span>&#123;</span><br><span class="line"></span><br><span class="line">        <span class="comment">//1. 获取行信息</span></span><br><span class="line">        String line = value.toString();</span><br><span class="line"></span><br><span class="line">        <span class="comment">//2. 获取行的所用单词</span></span><br><span class="line">        String[] words = line.split(<span class="string">"\t"</span>);<span class="comment">//这里假设一行文本单词分隔符为"\t"</span></span><br><span class="line">        <span class="keyword">for</span> (String word : words) &#123;</span><br><span class="line">            k2.set(word.getBytes()) ; <span class="comment">//设置键</span></span><br><span class="line">            v2.set(<span class="number">1</span>);                <span class="comment">//设置值</span></span><br><span class="line">            context.write(k2,v2);</span><br><span class="line">        &#125;</span><br><span class="line">        </span><br><span class="line">    &#125;</span><br><span class="line">&#125;</span><br></pre></td></tr></table></figure>
</li>
<li><p><strong>使用Reducer合并计算</strong></p>
<figure class="highlight java"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br><span class="line">23</span><br><span class="line">24</span><br><span class="line">25</span><br><span class="line">26</span><br><span class="line">27</span><br><span class="line">28</span><br></pre></td><td class="code"><pre><span class="line"><span class="comment">//设置Job的Reducer计算类和K3、V3类型</span></span><br><span class="line">job.setReducerClass(WordCountReducer.class);	<span class="comment">//自定义的Reducer类</span></span><br><span class="line">job.setOutputKeyClass(Text.class);		<span class="comment">//输出Key类型</span></span><br><span class="line">job.setOutputValueClass(LongWritable.class);	<span class="comment">//输出Value类型</span></span><br><span class="line"></span><br><span class="line"><span class="comment">//WordCountReducer 类</span></span><br><span class="line"><span class="comment">/**</span><br><span class="line"> * 自定义的Reduce 需要继承Reducer</span><br><span class="line"> * K2 : 字符串</span><br><span class="line"> * V3 : 次数（分组）</span><br><span class="line"> * K3 : 字符串</span><br><span class="line"> * V3 : 次数（统计总的）</span><br><span class="line"> */</span></span><br><span class="line"><span class="keyword">public</span> <span class="keyword">static</span> <span class="class"><span class="keyword">class</span> <span class="title">WordCountReducer</span> <span class="keyword">extends</span> <span class="title">Reducer</span>&lt;<span class="title">Text</span>,<span class="title">LongWritable</span>,<span class="title">Text</span>,<span class="title">LongWritable</span>&gt;</span>&#123;</span><br><span class="line"></span><br><span class="line">    LongWritable v3 = <span class="keyword">new</span> LongWritable() ;</span><br><span class="line">    <span class="keyword">int</span> sum  = <span class="number">0</span> ;</span><br><span class="line">    <span class="meta">@Override</span></span><br><span class="line">    <span class="function"><span class="keyword">protected</span> <span class="keyword">void</span> <span class="title">reduce</span><span class="params">(Text key, Iterable&lt;LongWritable&gt; values, Context context)</span> </span><br><span class="line">            <span class="keyword">throws</span> IOException, InterruptedException </span>&#123;</span><br><span class="line">        sum = <span class="number">0</span> ;</span><br><span class="line">        <span class="keyword">for</span> (LongWritable value : values) &#123;</span><br><span class="line">            sum +=value.get() ;</span><br><span class="line">        &#125;</span><br><span class="line">        v3.set(sum);</span><br><span class="line">        context.write( key , v3 );</span><br><span class="line">    &#125;</span><br><span class="line">&#125;</span><br></pre></td></tr></table></figure>
</li>
<li><p><strong>Output，数据写出</strong></p>
<figure class="highlight java"><table><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">FileOutputFormat.setOutputPath(job, <span class="keyword">new</span> Path(args[<span class="number">1</span>]));</span><br></pre></td></tr></table></figure>
</li>
</ol>
<h1 id="运行机制"><a href="#运行机制" class="headerlink" title="运行机制"></a>运行机制</h1><ul>
<li>下面从<strong>两个角度</strong>来讲解MapReduce的运行机制：</li>
</ul>
<ol>
<li><strong>各个角色实体</strong>;</li>
<li><strong>运行的时间先后顺序</strong>。<br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160419199999.jpg" alt="MapReduce计算模型的运行机制"></li>
</ol>
<h2 id="各个角色实体"><a href="#各个角色实体" class="headerlink" title="各个角色实体"></a>各个角色实体</h2><ol>
<li><p><strong>程序运行时过程设计到的一个角色实体</strong><br>1.1. <strong>Client</strong>：编写mapreduce程序，配置作业，提交作业的客户端 ；<br>1.2. <strong>ResourceManager</strong>：集群中的资源分配管理 ；<br>1.3. <strong>NodeManager</strong>：启动和监管各自节点上的计算资源 ；<br>1.4. <strong>ApplicationMaster</strong>：每个程序对应一个AM，负责程序的任务调度，本身也是运行在NM的Container中 ；<br>1.5. <strong>HDFS</strong>：分布式文件系统，保存作业的数据、配置信息等等。</p>
</li>
<li><p><strong>客户端提交Job</strong><br>2.1. 客户端编写好Job后，调用Job实例的Submit()或者waitForCompletion()方法提交作业；<br>2.2. 客户端向ResourceManager请求分配一个Application ID，客户端会对程序的输出、输入路径进行检查，如果没有问题，进行作业输入分片的计算。</p>
</li>
<li><p><strong>Job提交到ResourceManager</strong><br>3.1. 将作业运行所需要的资源拷贝到HDFS中（jar包、配置文件和计算出来的输入分片信息等）；<br>3.2. 调用ResourceManager的submitApplication方法将作业提交到ResourceManager。</p>
</li>
<li><p><strong>给作业分配ApplicationMaster</strong><br>4.1. ResourceManager收到submitApplication方法的调用之后会命令一个NodeManager启动一个Container ；<br>4.2. 在该NodeManager的Container上启动管理该作业的ApplicationMaster进程。</p>
</li>
<li><p><strong>ApplicationMaster初始化作业</strong><br>5.1. ApplicationMaster对作业进行初始化操作；<br>5.2. ApplicationMaster从HDFS中获得输入分片信息(map、reduce任务数)</p>
</li>
<li><p><strong>任务分配</strong><br>6.1. ApplicationMaster为其每个map和reduce任务向RM请求计算资源；<br>6.2. map任务优先于reduce任，map数据优先考虑本地化的数据。</p>
</li>
<li><p><strong>任务执行</strong>，在 Container 上启动任务（通过YarnChild进程来运行），执行map/reduce任务。</p>
</li>
</ol>
<h2 id="时间先后顺序"><a href="#时间先后顺序" class="headerlink" title="时间先后顺序"></a>时间先后顺序</h2><ol>
<li><p><strong>输入分片（input split）</strong><br>每个输入分片会让一个map任务来处理，默认情况下，以HDFS的一个块的大小（默认为128M，可以设置）为一个分片。map输出的结果会暂且放在一个环形内存缓冲区中（<code>默认mapreduce.task.io.sort.mb=100M</code>）,当该缓冲区快要溢出时（<code>默认mapreduce.map.sort.spill.percent=0.8</code>）,会在本地文件系统中创建一个溢出文件，将该缓冲区中的数据写入这个文件；</p>
</li>
<li><p><strong>map阶段</strong>：由我们自己编写，最后调用 context.write(…)；</p>
</li>
<li><p><strong>partition分区阶段</strong><br>3.1. 在map中调用 context.write(k2,v2)方法输出<k2,v2>，该方法会立刻调用 Partitioner类对数据进行分区，一个分区对应一个 reduce task。<br>3.2. 默认的分区实现类是 <strong>HashPartitioner</strong> ，根据<code>k2的哈希值 % numReduceTasks</code>，可能出现“数据倾斜”现象。<br>3.3. 可以自定义 partition ，调用 job.setPartitioner(…)自己定义分区函数。</k2,v2></p>
</li>
<li><p><strong>combiner合并阶段</strong>：将属于同一个reduce处理的输出结果进行合并操作<br>4.1. 是可选的；<br>4.2. 目的有三个：1.减少Key-Value对；2.减少网络传输；3.减少Reduce的处理。</p>
</li>
<li><p><strong>shuffle阶段</strong>：即Map和Reduce中间的这个过程<br>5.1. 首先 <strong>map</strong> 在做输出时候会在内存里开启一个环形内存缓冲区，专门用来做输出，同时map还会启动一个守护线程；<br>5.2. 如缓冲区的内存达到了阈值的80%，守护线程就会把内容写到磁盘上，这个过程叫<strong>spill</strong>，另外的20%内存可以继续写入要写进磁盘的数据；<br>5.3. 写入磁盘和写入内存操作是互不干扰的，如果缓存区被撑满了，那么map就会阻塞写入内存的操作，让写入磁盘操作完成后再继续执行写入内存操作;<br>5.4. 写入磁盘时会有个排序操作，如果定义了combiner函数，那么排序前还会执行combiner操作；<br>5.5. 每次spill操作也就是写入磁盘操作时候就会写一个溢出文件，也就是说在做map输出有几次spill就会产生多少个溢出文件，等map输出全部做完后，map会合并这些输出文件，这个过程里还会有一个Partitioner操作（如上）<br>5.6. 最后 <strong>reduce</strong> 就是合并map输出文件，Partitioner会找到对应的map输出文件，然后进行复制操作，复制操作时reduce会开启几个复制线程，这些线程默认个数是5个（可修改），这个复制过程和map写入磁盘过程类似，也有阈值和内存大小，阈值一样可以在配置文件里配置，而内存大小是直接使用reduce的tasktracker的内存大小，复制时候reduce还会进行排序操作和合并文件操作，这些操作完了就会进行reduce计算了。</p>
</li>
<li><p><strong>reduce阶段</strong>：由我们自己编写，最终结果存储在hdfs上的。</p>
</li>
</ol>



http://blog.xiaoxiaomo.com/2016/07/03/Hadoop-MapReduce%E8%AF%A6%E8%A7%A3/
http://blog.xiaoxiaomo.com/2016/07/08/Hadoop-MapReduce%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/
