title: HDFS分析
toc: true
date: 2017-03-14 15:11:36
tags: Hadoop
categories: note
---

# Namenode
<ul>
<li><strong>重点掌握</strong>：</li>
</ul>
<ol>
<li><code>NameNode</code> 的作用；</li>
<li><code>NameNode</code> 元数据的底层结构；</li>
<li><code>SecondaryNameNode</code> 的作用以及工作流程，以及为什么需要SecondaryNameNode。</li>
</ol>
<a id="more"></a>
<h2 id="NameNode简介"><a href="#NameNode简介" class="headerlink" title="NameNode简介"></a>NameNode简介</h2><ul>
<li><strong>管理节点</strong></li>
</ul>
<ol>
<li><strong>核心元数据</strong>，包括文件(夹)的目录结构和属性信息，还有文件与其所在位置的映射信息。</li>
<li><strong>一切读写的操作必须经过NameNode，但是传输数据本身不经过NameNode</strong>（好好理解这句话）。</li>
</ol>
<h3 id="NameNode"><a href="#NameNode" class="headerlink" title="NameNode"></a>NameNode</h3><ul>
<li><p><strong>NameNode</strong> 包括以下文件：（<strong>保存在linux的文件系统中</strong>）</p>
<blockquote>
<ol>
<li><code>fsimage</code>：元数据镜像文件，存储某一时段NameNode内存元数据信息即保存了最新的元数据checkpoint。</li>
<li><code>edits</code>：操作日志文件。</li>
<li><code>fstime</code>：保存最近一次checkpoint的时间<br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160412193118.png" alt="NameNode元数据信息"></li>
</ol>
</blockquote>
</li>
<li><p><strong>NameNode</strong> 元数据</p>
</li>
</ul>
<ol>
<li>NameNode 为了保证交互速度，会在内存中保存这些元数据信息，但同时也会将这些信息保存到硬盘上进行持久化存储；</li>
<li>fsimage文件是内存中的元数据在硬盘上的checkpoint，它是一种序列化的格式，不能直接修改。</li>
<li>Hadoop在重启时就是通过<code>fsimage+edits</code>来状态恢复，fsimage相当于一个checkpoint，首先将最新的checkpoint的元数据信息从fsimage中加载到内存，然后逐一执行edits修改日志文件中的操作以恢复到重启之前的最终状态。</li>
<li>Hadoop的持久化过程是将上一次checkpoint以后最近一段时间的操作保存到修改日志文件edits中。</li>
</ol>
<h3 id="SecondaryNameNode"><a href="#SecondaryNameNode" class="headerlink" title="SecondaryNameNode"></a>SecondaryNameNode</h3><p>上面出现的一个问题是：edits会随着时间增加而越来越大，导致以后重启时需要花费很长的时间来按照edits中记录的操作进行恢复，于是Hadoop就专门弄了一个进程<strong>SecondaryNameNode</strong>。</p>
<p><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160626120418.png" alt="SecondaryNameNode工作流程"></p>
<ul>
<li><strong>SecondaryNameNode节点</strong> 的主要功能是周期性将元数据节点的命名空间镜像文件（<em>fsimage</em>）和修改日志（<em>edits</em>）进行合并，以防edits日志文件过大。下面来看一看<strong>合并的流程</strong>：</li>
</ul>
<ol>
<li>SecondaryNameNode节点 需要合并时，首先通知<code>NameNode节点</code>生成新的日志文件，以后的日志都写到新的日志文件中。</li>
<li>SecondaryNameNode节点 用<code>http get</code>从<code>NameNode节点</code>获得<code>fsimage文件</code>及<code>旧的edits日志文件</code>。</li>
<li>SecondaryNameNode节点 将 <strong>fsimage 文件加载到内存中，并执行日志文件中的操作，然后生成新的fsimage文件</strong>。</li>
<li>SecondaryNameNode 节点将新的fsimage文件用http post<strong>传回</strong>NameNode节点上。</li>
<li>NameNode 节点可以将旧的fsimage文件及旧的日志文件，换为新的fsimage文件和新的日志文件(第一步生成的)，然后更新fstime文件，写入此次checkpoint的时间。</li>
<li>这样NameNode 节点中的fsimage文件保存了最新的checkpoint的元数据信息，日志文件也重新开始，不会变的很大了</li>
</ol>
<ul>
<li><code>注意</code>：</li>
</ul>
<ol>
<li>这种机制有个问题：因edits存放在NameNode中，当NameNode挂掉，edits也会丢失，导致<strong>利用Secondary NameNode恢复Namenode时，会有部分数据丢失</strong>。</li>
<li>HDFS设置了两种机制进行条件合并（hdfs-site.xml）：<br>第一种：当时间间隔大于或者等于dfs.namenode.checkpoint.period配置的时间是做合并（默认一小时）<br>第二种：当最后一次往journalNode写入的TxId（这个可以在namenode日志或者50070界面可以看到）和最近一次做Checkpoint的TxId的差值大于或者等于dfs.namenode.checkpoint.txns配置的数量（默认1000000）时做一次合并<br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160626195254.png" alt="HDFS设置了两种条件合并"></li>
</ol>
<h2 id="底层文件查看"><a href="#底层文件查看" class="headerlink" title="底层文件查看"></a>底层文件查看</h2><ul>
<li><p>保存的<code>NameNode元数据</code>信息，在<code>HADOOP_HOME/etc/hadoop/hdfs-site.xml 的 dfs.namenode.name.dir</code>配置，如下图：</p>
<figure class="highlight xml"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line"><span class="tag">&lt;<span class="name">property</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">name</span>&gt;</span>dfs.namenode.name.dir<span class="tag">&lt;/<span class="name">name</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">value</span>&gt;</span>file:///usr/local/hadoop_repo/name<span class="tag">&lt;/<span class="name">value</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">property</span>&gt;</span></span><br><span class="line"><span class="comment">&lt;!-- 我切换到/usr/local/hadoop_repo/name/current就能查看元数据信息了 --&gt;</span></span><br><span class="line"><span class="comment">&lt;!-- value可以配置多个，用逗号隔开，这样也算是一种备份方式吧--&gt;</span></span><br></pre></td></tr></table></figure>
</li>
<li><p><strong>第一步：开启服务</strong>，查看某个fsimage文件，开启服务命令：<code>bin/hdfs oiv -i fsimage文件</code>，不然提示：ls: Connection refused</p>
</li>
<li><p><strong>第二步：查看内容</strong>，命令：<code>bin/hdfs dfs -ls -R webhdfs://127.0.0.1:5978/</code>。我的hdfs里面有hbase和hive的数据，所以元数据也比较多，这里来个部分截图（这个看上去就和<code>web</code>以及<code>hdfs dfs -ls</code>查看的结果差不多）：<br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160626143146.png" alt="NameNode元数据信息"></p>
</li>
<li><p><strong>第三步：导出结果</strong>，这是导出fsimage文件内容，命令：<code>hdfs oiv -p XML -i /usr/local/hadoop_repo/name/current/fsimage_0000000000000003292 -o fsimage.xml</code><br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160626174605.png" alt="导出的fsimage.xml文件"></p>
</li>
</ul>
<ol>
<li>xml节点在上面大家也能看见，主要看inode节点，下面就是文件以及文件夹的基本信息；</li>
<li><code>id</code>、<code>name</code>、<code>type</code>、<code>mtime</code>、<code>permission</code>、<code>nsquota</code>、<code>dsquota</code>；</li>
<li>如果是文件还有更多属性，比如<code>atime</code>、<code>perferredBlockSize</code>、<code>blocks</code>等</li>
<li>fsimage 保存有如下信息：<figure class="highlight c"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br></pre></td><td class="code"><pre><span class="line">一、加载Img头信息，如下：</span><br><span class="line">  <span class="number">1</span>、imgVersion(<span class="keyword">int</span>)：当前image的版本信息</span><br><span class="line">  <span class="number">2</span>、namespaceID(<span class="keyword">int</span>)：用来确保别的HDFS instance中的datanode不会误连上当前NN。</span><br><span class="line">  <span class="number">3</span>、numFiles(<span class="keyword">long</span>)：整个文件系统中包含有多少文件和目录</span><br><span class="line">  <span class="number">4</span>、genStamp(<span class="keyword">long</span>)：生成该image时的时间戳信息。</span><br><span class="line">二 、如果加载目录，包含以下信息：</span><br><span class="line">  <span class="number">1</span>、path(String)：该目录的路径，如”/user/build/build-index”</span><br><span class="line">  <span class="number">2</span>、replications(<span class="keyword">short</span>)：副本数（目录虽然没有副本，但这里记录的目录副本数也为<span class="number">3</span>）</span><br><span class="line">  <span class="number">3</span>、mtime(<span class="keyword">long</span>)：该目录的修改时间的时间戳信息</span><br><span class="line">  <span class="number">4</span>、atime(<span class="keyword">long</span>)：该目录的访问时间的时间戳信息</span><br><span class="line">  <span class="number">5</span>、blocksize(<span class="keyword">long</span>)：目录的blocksize都为<span class="number">0</span></span><br><span class="line">  <span class="number">6</span>、numBlocks(<span class="keyword">int</span>)：文件块数,<span class="number">-1</span>表示目录,大于<span class="number">1</span>时，表示该文件对应有多个block信息</span><br><span class="line">  <span class="number">7</span>、nsQuota(<span class="keyword">long</span>)：<span class="keyword">namespace</span> Quota值，若没加Quota限制则为<span class="number">-1</span></span><br><span class="line">  <span class="number">8</span>、dsQuota(<span class="keyword">long</span>)：disk Quota值，若没加限制则也为<span class="number">-1</span></span><br><span class="line">  <span class="number">9</span>、username(String)：该目录的所属用户名</span><br><span class="line">  <span class="number">10</span>、group(String)：该目录的所属组</span><br><span class="line">  <span class="number">11</span>、permission(<span class="keyword">short</span>)：该目录的permission信息，如<span class="number">644</span>等，有一个<span class="keyword">short</span>来记录。</span><br><span class="line">三、如果加载文件，则还会额外包含如下信息：</span><br><span class="line">  <span class="number">1</span>、blockid(<span class="keyword">long</span>)：属于该文件的block的blockid，</span><br><span class="line">  <span class="number">2</span>、numBytes(<span class="keyword">long</span>)：该block的大小</span><br><span class="line">  <span class="number">3</span>、genStamp(<span class="keyword">long</span>)：该block的时间戳</span><br></pre></td></tr></table></figure>
</li>
</ol>
<p>在namenode启动时，就需要对fsimage按照如下格式进行顺序的加载，以将fsimage中记录的HDFS元数据信息加载到内存中。</p>
<ul>
<li><strong>第三步：继续导出</strong>，这是导出edits文件内容，命令：<code>bin/hdfs oev -i /usr/local/hadoop_repo/name/current/edits_0000000000000003295-0000000000000003296 -o edits.xml</code><br><img src="http://img.xiaoxiaomo.com/blog%2Fimg%2F20160626180932.png" alt="导出的edits.xml文件"></li>
</ul>
<ol>
<li>最大的节点是<code>EDITS</code>，下面就是版本信息<code>EDITS_VERSION</code>和很多的<code>RECORD</code>节点;</li>
<li>每个<strong>edits</strong>文件第一个<code>RECORD</code>的<code>RECORD</code>都是以<code>OP_START_LOG_SEGMENT</code>开头;</li>
<li><code>RECORD</code>类型有很多，比如 OP_ADD、OP_TIMES、OP_DELETE、OP_ALLOCATE_BLOCK_ID、OP_ADD_BLOCK、OP_RENAME_OLD、OP_CLOSE等</li>
</ol>
<ul>
<li><strong>总结</strong>：</li>
</ul>
<ol>
<li>从上面我们就可以看出，edits文件的信息特别详细，记录了每一步操作，所以文件大小增长也特别的快；</li>
<li>fsimage文件内容就是edits文件详细步骤的浓缩。</li>
</ol>

## DateNode
一、Block块（数据存储单元），二、文件备份数，掌握Block块信息，副本数的设置。
http://blog.csdn.net/androidlushangderen/article/details/47945597

http://blog.xiaoxiaomo.com/2016/06/26/Hadoop-HDFS%E4%B9%8BDataNode/
