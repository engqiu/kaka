# kaka
中文分词，经过严格筛选的分词词典，以及通过google_books生成的ngram。kaka分词是一个Java实现的分布式的中文分词组件，提供了多种基于词典的分词算法，并利用ngram模型来消除歧义。能准确识别英文、数字，以及日期、时间等数量词，能识别人名、地名、组织机构名等未登录词。能通过自定义配置文件来改变组件行为，能自定义用户词库、自动检测词库变化、支持大规模分布式环境，能灵活指定多种分词算法，能使用refine功能灵活控制分词结果，还能使用词频统计、词性标注、同义标注、反义标注、拼音标注等功能。提供了10种分词算法，还提供了10种文本相似度算法，同时还无缝和Lucene、Solr、ElasticSearch、Luke集成。注意：需要JDK1.8
	
### 分词使用方法：

#### 1、对文本进行分词

	移除停用词：List<Word> words = WordSegmenter.seg("澳大利亚是一个美丽的国家。");
	保留停用词：List<Word> words = WordSegmenter.segWithStopWords("澳大利亚是一个美丽的国家。");

#### 2、对文件进行分词

	String input = "d:/text.txt";
	String output = "d:/word.txt";
	移除停用词：WordSegmenter.seg(new File(input), new File(output));
	保留停用词：WordSegmenter.segWithStopWords(new File(input), new File(output));
	
#### 3、自定义配置文件

	自定义配置文件为类路径下的word.local.conf，需要用户自己提供
	如果自定义配置和默认配置相同，自定义配置会覆盖默认配置
	配置文件编码为UTF-8
		
#### 4、自定义用户词库

	自定义用户词库为一个或多个文件夹或文件，可以使用绝对路径或相对路径
	用户词库由多个词典文件组成，文件编码为UTF-8
	词典文件的格式为文本文件，一行代表一个词
	可以通过系统属性或配置文件的方式来指定路径，多个路径之间用逗号分隔开
	类路径下的词典文件，需要在相对路径前加入前缀classpath:
		
	指定方式有三种：
		指定方式一，编程指定（高优先级）：
			WordConfTools.set("dic.path", "classpath:dic.txt，d:/custom_dic");
			DictionaryFactory.reload();//更改词典路径之后，重新加载词典
		指定方式二，Java虚拟机启动参数（中优先级）：
			java -Ddic.path=classpath:dic.txt，d:/custom_dic
		指定方式三，配置文件指定（低优先级）：
			使用类路径下的文件word.local.conf来指定配置信息
			dic.path=classpath:dic.txt，d:/custom_dic
 	
	如未指定，则默认使用类路径下的dic.txt词典文件

	除此之外, 还可以在程序中用代码维护词库, 方法如下:

    // 单个操作
    // 添加一个自定义词
    DictionaryFactory.getDictionary().add("杨尚川");
    // 删除一个自定义词
    DictionaryFactory.getDictionary().remove("刘诗诗");
    // 批量操作
    List<String> words = new ArrayList<>();
    words.add("刘德华");
    words.add("景甜");
    words.add("赵丽颖");
    // 添加一批自定义词
    DictionaryFactory.getDictionary().addAll(words);
    // 删除一批自定义词
    DictionaryFactory.getDictionary().removeAll(words);
	
#### 5、自定义停用词词库

	使用方式和自定义用户词库类似，配置项为：
	stopwords.path=classpath:stopwords.txt，d:/custom_stopwords_dic
		
#### 6、自动检测词库变化

	可以自动检测自定义用户词库和自定义停用词词库的变化
	包含类路径下的文件和文件夹、非类路径下的绝对路径和相对路径
	如：
	classpath:dic.txt，classpath:custom_dic_dir,
	d:/dic_more.txt，d:/DIC_DIR，D:/DIC2_DIR，my_dic_dir，my_dic_file.txt
	
	classpath:stopwords.txt，classpath:custom_stopwords_dic_dir，
	d:/stopwords_more.txt，d:/STOPWORDS_DIR，d:/STOPWORDS2_DIR，stopwords_dir，remove.txt
	
#### 7、显式指定分词算法

	对文本进行分词时，可显式指定特定的分词算法，如：
	WordSegmenter.seg("澳大利亚是一个美丽的国家。", SegmentationAlgorithm.BidirectionalMaximumMatching);
	
	SegmentationAlgorithm的可选类型为：	 
	正向最大匹配算法：MaximumMatching
	逆向最大匹配算法：ReverseMaximumMatching
	正向最小匹配算法：MinimumMatching
	逆向最小匹配算法：ReverseMinimumMatching
	双向最大匹配算法：BidirectionalMaximumMatching
	双向最小匹配算法：BidirectionalMinimumMatching
	双向最大最小匹配算法：BidirectionalMaximumMinimumMatching
	全切分算法：FullSegmentation
	最少词数算法：MinimalWordCount
	最大Ngram分值算法：MaxNgramScore
	
	
#### 8、词性标注

	将分词结果作为输入参数，调用PartOfSpeechTagging类的process方法，词性保存在Word类的partOfSpeech字段中
	如下所示：
	List<Word> words = WordSegmenter.segWithStopWords("我爱中国");
	System.out.println("未标注词性："+words);
	//词性标注
	PartOfSpeechTagging.process(words);
	System.out.println("标注词性："+words);
	输出内容：
	未标注词性：[我, 爱, 中国]
    标注词性：[我/r, 爱/v, 中国/ns]
    
#### 9、refine

    我们看一个切分例子：
    List<Word> words = WordSegmenter.segWithStopWords("我国工人阶级和广大劳动群众要更加紧密地团结在党中央周围");
    System.out.println(words);
    结果如下：
    [我国, 工人阶级, 和, 广大, 劳动群众, 要, 更加, 紧密, 地, 团结, 在, 党中央, 周围]
    假如我们想要的切分结果是：
	[我国, 工人, 阶级, 和, 广大, 劳动, 群众, 要, 更加, 紧密, 地, 团结, 在, 党中央, 周围]
    也就是要把“工人阶级”细分为“工人 阶级”，把“劳动群众”细分为“劳动 群众”，那么我们该怎么办呢？
    我们可以通过在word.refine.path配置项指定的文件classpath:word_refine.txt中增加以下内容：
    工人阶级=工人 阶级
    劳动群众=劳动 群众
	然后，我们对分词结果进行refine：
	words = WordRefiner.refine(words);
	System.out.println(words);
	这样，就能达到我们想要的效果：
	[我国, 工人, 阶级, 和, 广大, 劳动, 群众, 要, 更加, 紧密, 地, 团结, 在, 党中央, 周围]
	
	我们再看一个切分例子：
	List<Word> words = WordSegmenter.segWithStopWords("在实现“两个一百年”奋斗目标的伟大征程上再创新的业绩");
	System.out.println(words);
	结果如下：
	[在, 实现, 两个, 一百年, 奋斗目标, 的, 伟大, 征程, 上, 再创, 新的, 业绩]
	假如我们想要的切分结果是：
	[在, 实现, 两个一百年, 奋斗目标, 的, 伟大征程, 上, 再创, 新的, 业绩]
	也就是要把“两个 一百年”合并为“两个一百年”，把“伟大, 征程”合并为“伟大征程”，那么我们该怎么办呢？
	我们可以通过在word.refine.path配置项指定的文件classpath:word_refine.txt中增加以下内容：
	两个 一百年=两个一百年
	伟大 征程=伟大征程
	然后，我们对分词结果进行refine：
	words = WordRefiner.refine(words);
	System.out.println(words);
	这样，就能达到我们想要的效果：
	[在, 实现, 两个一百年, 奋斗目标, 的, 伟大征程, 上, 再创, 新的, 业绩]
	
#### 10、同义标注

    List<Word> words = WordSegmenter.segWithStopWords("楚离陌千方百计为无情找回记忆");
    System.out.println(words);
	结果如下：
	[楚离陌, 千方百计, 为, 无情, 找回, 记忆]
	做同义标注：
	SynonymTagging.process(words);
	System.out.println(words);
	结果如下：
	[楚离陌, 千方百计[久有存心, 化尽心血, 想方设法, 费尽心机], 为, 无情, 找回, 记忆[影象]]
	如果启用间接同义词：
	SynonymTagging.process(words, false);
	System.out.println(words);
	结果如下：
	[楚离陌, 千方百计[久有存心, 化尽心血, 想方设法, 费尽心机], 为, 无情, 找回, 记忆[影像, 影象]]
    
    List<Word> words = WordSegmenter.segWithStopWords("手劲大的老人往往更长寿");
	System.out.println(words);
	结果如下：
	[手劲, 大, 的, 老人, 往往, 更, 长寿]
	做同义标注：
	SynonymTagging.process(words);
	System.out.println(words);
	结果如下：
	[手劲, 大, 的, 老人[白叟], 往往[常常, 每每, 经常], 更, 长寿[长命, 龟龄]]
	如果启用间接同义词：
	SynonymTagging.process(words, false);
	System.out.println(words);
	结果如下：
	[手劲, 大, 的, 老人[白叟], 往往[一样平常, 一般, 凡是, 寻常, 常常, 常日, 平凡, 平居, 平常, 平日, 平时, 往常, 日常, 日常平凡, 时常, 普通, 每每, 泛泛, 素日, 经常, 通俗, 通常], 更, 长寿[长命, 龟龄]]

	以词“千方百计”为例：
	可以通过Word的getSynonym()方法获取同义词如：
	System.out.println(word.getSynonym());
	结果如下：
	[久有存心, 化尽心血, 想方设法, 费尽心机]
	注意：如果没有同义词，则getSynonym()返回空集合：Collections.emptyList()
	
	间接同义词和直接同义词的区别如下：
	假设：
	A和B是同义词，A和C是同义词，B和D是同义词，C和E是同义词
	则：
	对于A来说，A B C是直接同义词
	对于B来说，A B D是直接同义词
	对于C来说，A C E是直接同义词
	对于A B C来说，A B C D E是间接同义词
	
#### 11、反义标注

    List<Word> words = WordSegmenter.segWithStopWords("5月初有哪些电影值得观看");
    System.out.println(words);
	结果如下：
	[5, 月初, 有, 哪些, 电影, 值得, 观看]
	做反义标注：
	AntonymTagging.process(words);
	System.out.println(words);
	结果如下：
	[5, 月初[月底, 月末, 月终], 有, 哪些, 电影, 值得, 观看]
    
    List<Word> words = WordSegmenter.segWithStopWords("由于工作不到位、服务不完善导致顾客在用餐时发生不愉快的事情,餐厅方面应该向顾客作出真诚的道歉,而不是敷衍了事。");
	System.out.println(words);
	结果如下：
	[由于, 工作, 不到位, 服务, 不完善, 导致, 顾客, 在, 用餐, 时, 发生, 不愉快, 的, 事情, 餐厅, 方面, 应该, 向, 顾客, 作出, 真诚, 的, 道歉, 而不是, 敷衍了事]
	做反义标注：
	AntonymTagging.process(words);
	System.out.println(words);
	结果如下：
	[由于, 工作, 不到位, 服务, 不完善, 导致, 顾客, 在, 用餐, 时, 发生, 不愉快, 的, 事情, 餐厅, 方面, 应该, 向, 顾客, 作出, 真诚[糊弄, 虚伪, 虚假, 险诈], 的, 道歉, 而不是, 敷衍了事[一丝不苟, 兢兢业业, 尽心竭力, 竭尽全力, 精益求精, 诚心诚意]]

	以词“月初”为例：
	可以通过Word的getAntonym()方法获取反义词如：
	System.out.println(word.getAntonym());
	结果如下：
	[月底, 月末, 月终]
	注意：如果没有反义词，getAntonym()返回空集合：Collections.emptyList()
	
#### 12、拼音标注

	List<Word> words = WordSegmenter.segWithStopWords("《速度与激情7》的中国内地票房自4月12日上映以来，在短短两周内突破20亿人民币");
	System.out.println(words);
	结果如下：
	[速度, 与, 激情, 7, 的, 中国, 内地, 票房, 自, 4月, 12日, 上映, 以来, 在, 短短, 两周, 内, 突破, 20亿, 人民币]
	执行拼音标注：
	PinyinTagging.process(words);
	System.out.println(words);
	结果如下：
    [速度 sd sudu, 与 y yu, 激情 jq jiqing, 7, 的 d de, 中国 zg zhongguo, 内地 nd neidi, 票房 pf piaofang, 自 z zi, 4月, 12日, 上映 sy shangying, 以来 yl yilai, 在 z zai, 短短 dd duanduan, 两周 lz liangzhou, 内 n nei, 突破 tp tupo, 20亿, 人民币 rmb renminbi]
	
	以词“速度”为例：
	可以通过Word的getFullPinYin()方法获取完整拼音如：sudu
	可以通过Word的getAcronymPinYin()方法获取首字母缩略拼音如：sd
	


#### 13、判定句子是有意义的人话的可能性：

	通过如下命令:
	unix-like:
		chmod +x sentence-identify.sh & ./sentence-identify.sh
	windows:
		./sentence-identify.bat
	运行 org.apdplat.word.analysis.SentenceIdentify 类的结果如下所示:

	1. 句子: 我是一个男人你是一个女人, 概率: 0.71428573
    2. 句子: 我是一个人, 概率: 0.6666667
    3. 句子: 我爱读书, 概率: 0.5
    4. 句子: 我爱学习, 概率: 0.5
    5. 句子: 法蒂小室汝辈武学大师改个入门处, 概率: 0.2857143
    6. 句子: 显气孔率高压线塔总监督室波洛奈兹王毅陈刘玉荣, 概率: 0.2857143
    7. 句子: 王捷俊汇报演出干草加韦拉一杠地垄墙未尝不可, 概率: 0.25
    8. 句子: 八九点钟山光水色饱经世变普留申科淮河镇乐不极盘模拟飞行, 概率: 0.22222222
    9. 句子: 物位任务区亡灵书巴纳尔没脑子揪人心肺复习功课林友力避风塘, 概率: 0.2
    10. 句子: 参与方植物学报白善烨暗影狂奔骑白马痦子山城堡犹豫不定岳阳机场, 概率: 0.2

	接着可根据命令行提示输入句子并回车来获得句子的评分

	例如输入句子并回车:为中国崛起而努力奋斗
	程序返回结果如下:
	随机单词: [为, 中国, 崛起, 而, 努力, 奋斗]
	生成句子: 为中国崛起而努力奋斗
	句子概率: 1.0

	例如输入句子并回车:人脑的记忆是保存在生物电上还是在细胞里？
	程序返回结果如下:
	随机单词: [人脑, 的, 记忆, 是, 保存, 在, 生物, 电, 上, 还是, 在, 细胞, 里]
    生成句子: 人脑的记忆是保存在生物电上还是在细胞里？
    句子概率: 0.8333333

