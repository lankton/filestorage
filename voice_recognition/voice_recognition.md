# 调研目标
了解当前各厂商提供的语音识别服务，并结合移动开发这一场景，对其提供的方案进行体验和分析。

# 调研内容

## 服务提供方
现在很多厂商都提供了语音识别服务。国外，Google Cloud Platform 提供了 Cloud Speech API，微软提供了Bing Speech API。国内，一线互联网大厂，也都不甘落后，提供了各自的语音识别服务。比如，微信提供的语音开放平台、阿里云的ASR等，同时，一些专攻语音识别领域的企业，如科大讯飞等，也活跃在行业的前线。  

## 服务提供方式
这些厂商的语音识别服务，归结起来，有2种服务提供方式：  
1. 提供REST API;  
2. 提供SDK方便各平台开发。
### REST API
这种服务的使用方式，是将本地的语音文件，通过POST请求的方式，上传至厂商的服务器。服务器分析语音文件，得出识别结果，返回给请求方。  
提供厂商：Bing Speech API 等。
#### 优点
1. REST API具有良好的平台无关性，任何平台都可以通过HTTP请求获取服务；
2. 厂商的职责定位更加清晰，仅对外提供计算或者识别服务，专注于服务本身的优化。

#### 缺点
1. 对开发者不够友好；
2. 不方便进行实时语音识别。

### 提供SDK
这种服务方式，由厂商提供封装好的SDK，供各平台开发者集成。SDK往往提供了语音采集、分析、结果返回的全套API，开发者可以方便地使用这些SDK进行相关的开发。  
提供厂商：微信、阿里云、科大讯飞、Bing Speech API等。
#### 优点
1. 对服务使用者足够友好，开发者只需要关心语音识别SDK返回的结果，整个流程对其保持透明；
2. 厂商可以在SDK中集成更多的功能，如本地语音识别等，可以使得服务的提供更加便捷和人性化。  

#### 缺点
1. 具有明显的平台相关性，不同平台的开发需要集成不同的SDK；
2. SDK中可能包含超出开发者实际需求的功能，导致SDK相对冗余。

## 实际体验
在繁复的语音识别服务中，分别选取了微软的Bing Speech API 和 国内的科大讯飞进行了实际体验。同时考虑到移动端开发以及实时语音识别的场景，体验集中在集成SDK的服务提供方式上。  
体验平台:Android

### Bing Speech API
#### 服务获取方式
云端
#### Android平台最低支持版本
4.1
#### 语言支持
Bing Speech API 支持多种语言。

<img src="https://github.com/lankton/filestorage/blob/master/voice_recognition/bing_table.png?raw=true" width="500px"/>

#### 使用方式
##### 1. 集成
在gradle文件中添加依赖  

```
compile 'com.microsoft.projectoxford:speechrecognition:2.1.0'
```
##### 2. 实现 ISpeechRecognitionServerEvents 接口
```java
public interface ISpeechRecognitionServerEvents {
    void onPartialResponseReceived(String var1); // 当前接收到语音的累加

    void onFinalResponseReceived(RecognitionResult var1); // 最终结果

    void onIntentReceived(String var1);

    void onError(int var1, String var2);

    void onAudioEvent(boolean var1); // 麦克风状态
}
```
##### 3. 构造 MicrophoneRecognitionClient
```java
this.micClient =
SpeechRecognitionServiceFactory.createMicrophoneClientWithIntent(
                                    this, // Activity
                                    this.getDefaultLocale(), // 语言，中文“zh-cn”
                                    this, // ISpeechRecognitionServerEvents
                                    this.getPrimaryKey(),
                                    this.getLuisAppId(),
                                    this.getLuisSubscriptionID());
```
##### 4. MicrophoneRecognitionClient开始工作并开启麦克风
```
this.micClient.startMicAndRecognition();
```
##### 5. MicrophoneRecognitionClient结束工作并关闭麦克风
```
this.micClient.endMicAndRecognition();
```
#### demo效果
<img src="https://github.com/lankton/filestorage/blob/master/voice_recognition/bing.png?raw=true" width="250px"/>  
### 讯飞听写SDK
#### 服务获取方式
云端 ＋ 本地， 其中使用本地服务，需要本地集成资源文件。
#### 语言支持
讯飞听写 支持设置不同的语言类型参数以及方言参数。  
***例：中文，普通话***

```java
mIat.setParameter(SpeechConstant.LANGUAGE, "zh_cn");    
mIat.setParameter(SpeechConstant.ACCENT, "mandarin "); 
```

#### 使用方式
##### 1. 集成
从官网下载相应的jar和so，手动配置进工程
##### 2. 实现RecognizerListener接口
```java
public interface RecognizerListener {
    void onVolumeChanged(int var1, byte[] var2);

    void onBeginOfSpeech();

    void onEndOfSpeech();

    void onResult(RecognizerResult var1, boolean var2); // 第一个参数为输入结果， 第二个参数为true时会画结束

    void onError(SpeechError var1); // 错误回调

    void onEvent(int var1, int var2, int var3, Bundle var4);
}

```
***与bing speech不同， onResult每次返回当次输入的结果，而非累加。***
##### 3. 初始化
```java
SpeechUtility.createUtility(this, SpeechConstant.APPID+"=" + APPID);
```
##### 4. 创建SpeechRecognizer对象
```java
final SpeechRecognizer mIat = SpeechRecognizer.createRecognizer(this, null);
        mIat.setParameter(SpeechConstant.DOMAIN, "iat");
        mIat.setParameter(SpeechConstant.LANGUAGE, "zh_cn");
        mIat.setParameter(SpeechConstant.ACCENT, "mandarin "); // 口音 普通话
```
##### 5. 开始监听
```java
mIat.startListening(mRecordListener);
```
##### 6. 停止监听
```java
mIat.startListening(mRecordListener)
```
#### demo效果
<img src="https://github.com/lankton/filestorage/blob/master/voice_recognition/xunfei1.png?raw=true" width="250px"/>  <img src="https://github.com/lankton/filestorage/blob/master/voice_recognition/xunfei2.png?raw=true" width="250px"/>  

### 对比
#### 准确性
在两个语音识别服务的体验中，使用了相同的语音内容："昨天我没来，所以今天我来了。"可以看到，在语音转换为文本这一点上，二者都比较准确的完成了功能。
#### 返回数据
Bing Speech API 直接将字符串作为数据返回。 而讯飞听写，则是返回JSON格式的数据，这样使得返回的数据有了更多的内容。同时，讯飞听写分段返回，每段之间自动添加上了标点符号，对每段的内容，也进行了合适的分词处理。  
由返回数据的不同，可以看出2个服务各自适合的使用场景：    

1. Bing Speech API 返回数据格式较简单，但数据获取方便，适用于短语句或单语句的分析处理，比如语音搜索场景；
2. 讯飞听写返回的结果比较复杂，需要进行JSON解析，但提供的信息更多。适用于大段文字的输入，同时其自带的分词处理，可以方便使用者进行数据分析或者语义分析。         
***附：讯飞听写返回结果各字段含义***
<img src="https://github.com/lankton/filestorage/blob/master/voice_recognition/xunfei_table.png?raw=true" width="800px" />


#### 拓展性
1. Bing Speech API 虽然返回的数据较简单，但其仍然支持功能的拓展。其除了提供基本的语音识别API(Speech Recognition), 也提供了语音识别意图分析API(Speech Recognition Intent API)，在返回识别后的文本的基础上，也提供结构化的信息来表明语音发出者的意图，可以为app的功能拓展提供驱动。Speech Recognition Intent API的使用条件是，使用Language Understanding Intelligent Service (LUIS)创建模型，并在app中申明相应的LUIS ID，有一定的学习和使用成本;
2. 讯飞听写的功能拓展比较简单，支持上传联系人(每个用户终端设备对应一个联系人列表)，上传用户词表(每个用户终端设备对应一个词表)等。这种方式虽然不够智能，但在一般的使用场景中已经足够，比如：汽车行业的应用，上传常见行业术语、车系名称作为词表，即可以对用户的日常语音输入产生极大的便利。

# 小结
现在国内外厂商提供的语音识别服务种类繁多，在服务的使用方式、返回结果等方面各有异同。针对不同的业务需求，这些厂商提供的服务各有优势，在实际技术选型时，要根据业务的具体需求，作出合适的选择。