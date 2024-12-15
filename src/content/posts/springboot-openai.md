---
title: 使用springboot整合OpenAI的Chatgpt api和Whisper api
published: 2023-09-17
description: '本文详细介绍如何在Spring Boot项目中集成OpenAI的ChatGPT和Whisper API，包含环境搭建、依赖配置、接口调用等完整实现步骤，帮助开发者快速实现AI对话和语音转文字功能。'
image: 'https://esay.579878700.xyz/i/2023/09/17/m8u5iu.png'
tags: [openai, springboot, chatgpt, whisper]
category: 'Guides'
draft: false 
lang: ''
---
# Springboot+Chatgpt+Whisper api

![](https://www.qbitai.com/wp-content/uploads/2020/02/openai-cover-e1582087905568.png)

本文通过springboot将两个OpenAI API的其中两个，ChatGPT和Whisper API集成在一起，方便后续的开发。

## OpenAI API

OpenAI是人工智能和机器学习工具及API的领先提供商之一。它训练了尖端的语言模型，擅长理解和生成文本。OpenAI API提供对这些模型的访问，并可用于解决几乎涉及处理语言的任何任务。它可以执行包括但不限于以下任务：

- Content generation 内容生成

- Summarization 总结

-   Classification, categorization, and sentiment analysis  
    分类、归类和情感分析
    
- Data extraction 数据提取

- Translation 翻译

- Transcription 转录

- Image generation 图像生成

我们可以通过任何语言发出HTTP请求与API进行交互。在本篇文章中，我们将使用Java构建一个Spring Boot微服务来集成Chatgpt API和Whisper API语音转文字功能。

## 集成chatgpt-whisper-spring-boot

1. 首先我们打开IDEA新建一个springboot项目

   ![](https://thumbsnap.com/i/3Y6zzgX5.png)
   
2. 然后往其中的pom.xml文件添加依赖

   ```java
   <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
               <version>4.0.1</version>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-configuration-processor</artifactId>
               <optional>true</optional>
   </dependency>
   ```

## Spring Cloud OpenFeign

对于第三方REST API集成，有许多可以用的工具，这里我们选择Spring Cloud OpenFeign，因为这个对于Spring Boot 3支持方面比较好。

接下来我们打开 `application.yml`，往其中添加下面的内容：

```java
openai-service:
  api-key: xxxxxxxxxx
  gpt-model: gpt-3.5-turbo
  audio-model: whisper-1
  http-client:
    read-timeout: 60000
    connect-timeout: 60000
  urls:
    base-url: https://chat.579878700.xyz/v1
    chat-url: /chat/completions
    create-transcription-url: /audio/transcriptions
```

其中 ` api-key`请填写自己的api key，如果您还没有API密钥，请到[https://platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys) 以生成API密钥。实际开发中这种配置肯定要放在配置中心里面，但是我们今天先简单的写个demo。

`gpt-model` ： `gpt-3.5-turbo` ，功能最强大的GPT-3.5模型，针对聊天进行了优化，成本仅为 `text-davinci-003` 的1/10，它可以执行任何语言任务，质量更好，输出更长，指令一致。

`audio-model` ： `whisper-1` ，通用语音识别模型。它是在一个大型的多样化音频数据集上进行训练的，也是一个多任务模型，可以执行多语言语音识别以及语音翻译和来自世界各地大约100种不同语言的音频文件的语言识别。

` read-time`和` connect-time`可以根据实际需求更改，这里我使用中文来回答，因此我设置的大一些，避免回答返回时间太长导致超时问题。

` base-url`这里我填写的是我使用CF进行反代的地址，因为一些原因，openai api在中国不能直接访问，所以使用这个方法。如果不需要，请改成` https://api.openai.com/v1`

`urls` ：根据OpenAI API文档，所有API URL均以 `[https://api.openai.com/v1](https://api.openai.com/v1)` 开头。我们将其定义为 `base-url` 。对于聊天功能， `chat-url` 被定义为 `/chat/completions` 。对于音频转录， `create-transcription-url` 被定义为 `/audio/transcriptions` 。

我们使用 `/chat/completions` 继续使用 `gpt-3.5-turbo` 模型。

## Feign Client

现在配置已经完成，让我们继续编写声明式Feign客户端，我们需要两个方法： `chat` 和 `createTranscription` ，调用它们各自的OpenAI API端点URL。你可以随意命名这些方法，为了简单起见，让我们继续使用 `chat` 和 `createTranscription` 。

```java
@FeignClient(
        name = "openai-service",
        url = "${openai-service.urls.base-url}",
        configuration = OpenAIClientConfig.class
)
public interface OpenAIClient {
    @PostMapping(value = "${openai-service.urls.chat-url}", headers = {"Content-Type=application/json"})
    ChatGPTResponse chat(@RequestBody ChatGPTRequest chatGPTRequest);

    @PostMapping(value = "${openai-service.urls.create-transcription-url}", headers = {"Content-Type=multipart/form-data"})
    WhisperTranscriptionResponse createTranscription(@ModelAttribute WhisperTranscriptionRequest whisperTranscriptionRequest);
}
```

## Models

通过严格遵循API文档，我们先定义请求/响应类。请参见下面使用 `gpt-3.5-turbo` 模型的示例请求和响应类。注意，使用 `text-davinci-003` 完成的请求/响应略有不同，有关详细信息，请参阅API文档。

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ChatGPTRequest implements Serializable {
    private String model;
    private List<Message> messages;
}
```

```java
@Data
public class ChatGPTResponse implements Serializable {
    private String id;
    private String object;
    private String model;
    private LocalDate created;
    private List<Choice> choices;
    private Usage usage;
}
```

使用 Whisper 模型的音频转录请求/响应示例：

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class TranscriptionRequest implements Serializable {
    private MultipartFile file;
}
```

```java
@Data
public class WhisperTranscriptionResponse implements Serializable {
    private String text;
}
```

## Service

下面是服务层类 `OpenAIClientService` ，构造模型并调用Feignclient以触发对OpenAI API的调用。

```java
@Service
@RequiredArgsConstructor
public class OpenAIClientService {

    private final OpenAIClient openAIClient;
    private final OpenAIClientConfig openAIClientConfig;

    private final static String ROLE_USER = "user";

    public ChatGPTResponse chat(ChatRequest chatRequest){
        Message message = Message.builder()
                .role(ROLE_USER)
                .content(chatRequest.getQuestion())
                .build();
        ChatGPTRequest chatGPTRequest = ChatGPTRequest.builder()
                .model(openAIClientConfig.getModel())
                .messages(Collections.singletonList(message))
                .build();
        return openAIClient.chat(chatGPTRequest);
    }

    public WhisperTranscriptionResponse createTranscription(TranscriptionRequest transcriptionRequest){
        WhisperTranscriptionRequest whisperTranscriptionRequest = WhisperTranscriptionRequest.builder()
                .model(openAIClientConfig.getAudioModel())
                .file(transcriptionRequest.getFile())
                .build();
        return openAIClient.createTranscription(whisperTranscriptionRequest);
    }
}
```

请注意，除了必需参数之外，我们还可以在请求正文中传递多个可选参数来指示 OpenAI API。例如，对于`/chat/completions`，我们可以传入`temperature`参数，它告诉 OpenAI API 使用什么采样温度，介于 0 和 2 之间。较高的值将使输出更具随机性和风险性，而较低的值会使它更集中和确定性。`temperature`如果我们不指定，默认值为 1。请参阅[OpenAI API 文档，](https://platform.openai.com/docs/api-reference)了解我们可以在请求正文中传递的所有可能的参数。

## REST Controller REST

最后，让我们看看REST Controller层的 `OpenAIClientController` ，里面的两个客户端：

- `chat` （ `/api/v1/chat` ）：POST调用，使用JSON格式的消息，接受一个问题，并输出响应内容。

- `createTranscription` （ `/api/v1/transcription` ）：POST调用，使用multipart/form-data，因为我们要上传音频文件，如 `m4a`和` mp3`等音频文件 ，API将把音频文件转录为文本。

```java
@RestController
@RequiredArgsConstructor
@RequestMapping(value = "/api/v1")
public class OpenAIClientController {

    private final OpenAIClientService openAIClientService;

    @PostMapping(value = "/chat", consumes = MediaType.APPLICATION_JSON_VALUE)
    public ChatGPTResponse chat(@RequestBody ChatRequest chatRequest){
        return openAIClientService.chat(chatRequest);
    }

    @PostMapping(value = "/transcription", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public WhisperTranscriptionResponse createTranscription(@ModelAttribute TranscriptionRequest transcriptionRequest){
        return openAIClientService.createTranscription(transcriptionRequest);
    }
}
```

## Authentication 认证

OpenAI API调用需要传入一个带有API密钥的请求头“ `Authorization` “。让我们声明一个 `RequestInterceptor `bean来处理它，参见 `OpenAIClientConfig `类的示例代码片段：

```java
@Bean
    public RequestInterceptor apiKeyInterceptor() {
        return request -> request.header("Authorization", "Bearer " + apiKey);
    }
```

## 最后运行项目

用Postman测试一下

下面是 `/chat` 客户端的示例请求/响应截图：

![](https://thumbsnap.com/i/vymQU5KY.png)

下面是 `/transcription` 的请求/响应：

![](https://thumbsnap.com/i/NbWLi3MG.png)

感觉对中文的识别准确度还是可以的。

使用Whisper模型的音频转录或翻译，文件上传目前限制为25 MB，并支持以下输入文件类型：`mp3`, `mp4`, `mpeg`, `mpga`, `m4a`, `wav`, `webm`

Whisper型号的定价目前设定为0.006美元/分钟。

## 使用Whisper API进行音频翻译

除了音频转录，Whisper API还提供音频翻译端点 `/audio/translations` 。我也测试了这个特性。然而，我发现，尽管它说可以把音频翻译成英语，但我录了一个中文短语，并在应用程序中调用了 `/audio/translations` api，结果发现回应仍然是中文。我也用日语测试了这个api，它完整的翻译成了英语。这应该是一个bug。

## 总结

与任何第三方REST API一样，将OpenAI API集成到Springboot中非常简单。

因此使用api开发自己的人工智能程序会变的非常便捷。
