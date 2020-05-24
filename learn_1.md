FSdirectrory open 用来打开索引文件夹，用于存放后面生成的索引文件
调用Open(FSLockFactory.getDefault())

1.1 FSLockFactrory默认是NativeFsLockFactory->文件锁NativeFsLock

1.2 假设FsDirectory创建的是一个NIOFSDirectory


SimpleAnalzer

SimpleAnalyzer->indexWriteConfig

Indexwriter ----  infostream   fsdirectroy  obtainLock  文件锁

IndexWriter 经过一系列的创建和赋值操作   -》 SegmentInfos readLastestCommit 读取段信息

SegmentInfos-> FindSegmentFile  -> run   

GETlASTcOMMITgfENERATION -> 最后一个 

readCommit -》 checksumindexunput  -》 返回一个segementInfos ->  segementInfos里面保存了段信息


indexWriter  -》 SegmentInfos -》   Segementinfos.creteBakecupSegmentInfos函数  备份信息   ，用于rollback

indexWriter 调用 changed方法  使段信息发生了变化

indexwriter pendingNumDocs  记录了索引文档的总数 
             globalNumberMap 记录了该段中欧给你Field的相关信息
            getFlushPolicy  返回在 LiveIndexWriteConfig中创建的FlushByRamOrCountsPolicy



创建索引 
FileReaderAll  从文件中欧给你读取字符串， 创建indexWriter后 -》  遍历索引文件，构建Document和Field

IndexWritter:: addDocument 函数开始索引

IndexWriter  addDocument  


updateDocument函数完成索引 添加

preupdate  从DocumentWriterFLushControl 中取出 DoucmentWriterPerThread ，通过doFlush将 dwpt中欧给你的索引数据写入硬盘里

DocumentWriterFlushContro的obtainAndLock获取一个dwpt        
obtainAndLock  getAndLock  freeList 不为空 从里面取出一个不为空的dwpt的ThreadState，否则创建一个ThreadState（通过newThreadState）

uodateDocument-> dwpt的updateDocument函数来索引文档发
dwpt首先调用 reserveOneDoc查看索引中文档是否超过限制（DocState） ， consumer-》processDocument    
    然后调用consumer的processDocument        

ProcessDocument
初始化 FreqProxTermsWriter TermVectorsConsumer 分别保存了词和词向量的信息，

DefaultIndexChain :: pro cessDocument -> fijllStoreFiedls 

storeFieldWriterr


CompressingStoreFiedldsFormat CokppressingStoreWrittrer

.fdt fdx   - > 字段数据 字段索引


DefaultIndexChain->startStoreFiedls -> startDocument -》 finishStoreFields      

DefaultIndexChain  processDocument  遍历Field  processField 逐个阈

processField  getOrAddField  Field name 创建一个PerField
dieldHash   hash桶   用来存储 field的名字和name等信息   fieldHash                   

getOrAddFiled 扩充hash桶      


ProcessField  invert 

TYPE_STORED store true 表示需要对该域的值的进行村住     

CompressingStoreFieldWriter::  writeField

DefaultIndexingChain  processDocument   遍历field  取出processField创建的PerField  调用finish函数     最后调用finishSt oreFields函数  flush

DocumentWriteer  uodateDocument   numbnerDocInRam保存了有多少文档被索引了，然后调用 DocumentyWriterFlushControl 的  doAgterDocument 函数继续处理       
release 释放开始创建的ThreadState创建的，放到freelist给后面的程序调用，  最后通过postUpdate函数 选择DoucmentWritePerThread  调用DoFLush 写入硬盘中 

IndexWriteer udpateDocuments  processEvents



倒排表 
倒排表的存储函数是DefaultIndexingCHain的preocessField函数中欧给你的invert
传入参数Field 假设 TextField  第一次 reset invertState 吃初始化

接下来 tokenstream,,   tokenstream 是一个分词器 ，  

Analzer 默认的Strategy 为ReuserStrategy

TokenStreamComponents 缓存到ReuserStrategy中， 最后设置Field的阈值为Reader，并返回TokenStream

PerField invert     TokenStream 的reset 初始化，  

invert 函数通过 setAttributeSource 设置 invertState的各个属性， 然后依调用TermsHashPerField的各个Start函数进行一些初始化

PreField::invert->FreqProxyTermsWriterPerField::add
bytepool 存放词频和位置信息  intpool 存在每个词在bytepool中的偏移量， 当空间不足时都会通过 nextBuffer函数分配新的空间。 
Field 数据存储到 FreqProxyPostintArray的freqProxPosingsArrays 中欧给你， 以及TermVertorsPostingArray中



Flush函数
DocumentsWriter的doflush函数分析 
ticketQueue被定义为DocumentsWriterFlushQueueu ，用来同步多个flush线程

addFlushTicket首先通过incTickets增加计量，prepareFlush 操作在flush开始前旧爱那个一些被标记的文档删除。  创建一个SegmentFlushTicket并添加进内部队列queueu中

DocumentWriter的doflush 函数中， getNumDocsInRam获取在内存中的文档数， 然后调用DoucmentsWriterPerThread的flush函数进行

flush的pendingUpdates保存在等待删除或更新的文档发ID。假设待删除或更新的文档数大于0,就标记这些文档。


DefaultIndexingChain flush 
参数state中segmentInfo 是 Dwpt构造函数中创建的segmentInfo，保存了相应的段信息，maxDoc函数返回目前在内存中的文档树。DefaultIndexingChain的flush函数接下来测试通过writeNrms函数
将norm信息写入.nvm和.nvd中文档和字段的length和boost系数的编码


writeDocValue,和writeNorms函数类似，遍历得到perField，根据Field不同的值域类型被定义为numericDocVauesWriter、BinaryDocValuesWriter、SortedDocValuesWriter SortedNumericDocValuesWriter和 SortedSetDocValuesWriter


DefaultIndexingChain::flush->indexDocValue

writeDocValues  ->docValuesFormat函数返回一个PerFieldDocValuesFormat,并通过PerFieldDocValuesFormat的fieldsConsumer获得一个DocValuesConsumer

getDocValuesFormatForField Lucenne54DocValuesFormat， 
dvd,dvm 额外的得分系数或者每个文档的值信息编码


DefaultIndexChain的writerPoints函数
writePoints函数中的pointsFormat最终返回Lucene60PointsFormat，然后通过fieldsWriter函数获得一个Lucene60PointsWriter

DefaultIndexingChain:writePoints->PointValuesWriter::flush

PointValuesWriter flush函数中的PointsReader定义，writeField函数最终会调用BKDWriter的add函数，BKD是一种数据结构，add
offlinePointWriter   heapPointWriter被写入内存，当内存中数据超过maxPointsSortLnHeap时，就调用spillToOffline函数进行切换
offlinePointWriter的构造函数会创建类似段名bkd_spill临时文件数量.tmp的文件名对应的输出流，然后通过append函数复制HeapPointWriter中的数据
finish函数最终写入.dim


defaultIndexingchain flush
initStoredFieldsWriter函数初始化一个StoreFieldsWriter，storeFieldsFormat函数Lucenne50St oreFieldsFormat,其发ieldsWriter函数会接着调用ComporessingStoredFieldsFormat
的fieldsWriter函数，最终欧冠你返回CompressingStoredFieldsWriter


defaultIndexingChain flush   fieldsToFlush封装了fieldHash函数中的域信息，termsHash flush函数，termsHash在DefualtIndexingChain的构造有函数中被定义为FreqProxTermsWriter
.txv->词向量索引
.tvd->词向量文件
BlockTreeTerms 的 temsWriter被定义为TermsWriter
TermsWriter的writer将数据信息写入.doc .pos .pay    .doc->频率信息，包含那些含有一个词的频率的文档列表   .pos 位置信息，存储词在索引中出现的位置信息   .pay 存储每个位置的元数据信息，如字符偏移量和用户复负载
TermsWriter的finish函数会通过内部的writeBlocks的函数旧爱那个索引信息写入.tim .tip ，tim->词典 tip-》指向词典的索引




defaultIndexingChain flush
.fnm文件，将field域相关信息写入该文档中   .fnm->存储字段信息

.cfs .cfe 符合文件，一个可选的虚拟文件，包括所有其他索引文件系统频繁用完的文件句柄

.si 段信息













