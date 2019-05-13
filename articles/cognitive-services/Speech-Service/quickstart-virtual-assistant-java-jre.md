---
title: 快速入门：自定义语音优先虚拟助手（预览版），Java（Windows、Linux）- 语音服务
titleSuffix: Azure Cognitive Services
description: 本快速入门介绍如何在 Java 控制台应用程序中使用认知服务语音软件开发工具包 (SDK)。 其中将会介绍将客户端应用程序连接到事先创建的某个 Bot Framework 机器人。该机器人配置为使用 Direct Line 语音通道，可以实现语音优先虚拟助手体验。
services: cognitive-services
author: bidishac
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 05/02/2019
ms.author: bidishac
ms.openlocfilehash: 83149a8422db25106a97b1711c0ae9ce3c6603eb
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65465667"
---
# <a name="quickstart-create-a-voice-first-virtual-assistant-with-the-speech-sdk-java"></a>快速入门：使用语音 SDK 创建语音优先虚拟助手 (Java)

针对[语音转文本](quickstart-java-jre.md)和[语音翻译](quickstart-translate-speech-java-jre.md)也提供了快速入门。

在本文中，你将使用[认知服务语音 SDK](speech-sdk.md) 创建一个 Java 控制台应用程序。 该应用程序将连接到事先创作的某个机器人。该机器人配置为使用 Direct Line 语音通道，发送语音请求，并返回语音响应活动（如果已配置）。 该应用程序是使用语音 SDK Maven 包和 Eclipse Java IDE 在 Windows、Ubuntu Linux 或 macOS 上生成的。 它在 64 位 Java 8 运行时环境 (JRE) 中运行。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* 操作系统：Windows（64 位）、Ubuntu Linux 16.04/18.04（64 位）或 MacOS 10.13 或更高版本
* [Eclipse Java IDE](https://www.eclipse.org/downloads/)
* [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) 或 [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* 语音服务的 Azure 订阅密钥。 [免费获得一个](get-started.md)。
* 使用 Bot Framework 4.2 或更高版本创建的且预先配置的机器人。 该机器人需要订阅新的“Direct Line 语音”通道，以接收语音输入。

    > [!NOTE]
    > 在预览版中，Direct Line 语音通道目前仅支持 **westus2** 区域。

    > [!NOTE]
    > [免费试用语音服务](get-started.md)中所述的适用于标准定价层的 30 天试用版仅限 **westus**（不是 **westus2**）区域，因此与 Direct Line 语音不兼容。 免费和标准层 **westus2** 订阅与此兼容。

如果运行 Ubuntu 16.04/18.04，请确保在启动 Eclipse 之前安装这些依赖项：

```console
sudo apt-get update
sudo apt-get install build-essential libssl1.0.0 libasound2 wget
```

如果运行 Windows（64 位），请确保已安装适用于平台的 Microsoft Visual C++ Redistributable：
* [下载 Microsoft Visual C++ Redistributable for Visual Studio 2017](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)

## <a name="optional-get-started-fast"></a>可选：快速入门

本快速入门会逐步介绍如何将一个简单的客户端应用程序连接到支持语音的机器人。 [语音 SDK 示例](https://aka.ms/csspeech/samples)中的 `quickstart` 文件夹下提供了本快速入门使用的完整可编译源代码，供你直接学习。

## <a name="create-and-configure-project"></a>创建并配置项目

[!INCLUDE [](../../../includes/cognitive-services-speech-service-quickstart-java-create-proj.md)]

此外，若要启用日志记录，请更新 **pom.xml** 文件以包含以下依赖项。

   ```xml
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.5</version>
    </dependency>
   ```

## <a name="add-sample-code"></a>添加示例代码

1. 若要向 Java 项目添加新的空类，请选择“文件” > “新建” > “类”。

1. 在“新建 Java 类”窗口中，在“包”字段内输入 **speechsdk.quickstart**，在“名称”字段内输入 **Main**。

   ![“新建 Java 类”窗口的屏幕截图](media/sdk/qs-java-jre-06-create-main-java.png)

1. 打开新建的 **Main** 类，将 `Main.java` 文件的内容替换为以下起始代码。

    ```java
    package speechsdk.quickstart;

    import com.microsoft.cognitiveservices.speech.audio.AudioConfig;
    import com.microsoft.cognitiveservices.speech.audio.PullAudioOutputStream;
    import com.microsoft.cognitiveservices.speech.dialog.BotConnectorConfig;
    import com.microsoft.cognitiveservices.speech.dialog.SpeechBotConnector;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    import javax.sound.sampled.AudioFormat;
    import javax.sound.sampled.AudioSystem;
    import javax.sound.sampled.DataLine;
    import javax.sound.sampled.SourceDataLine;
    import java.io.InputStream;

    public class Main {
        final Logger log = LoggerFactory.getLogger(Main.class);

        public static void main(String[] args) {
            // New code will go here
        }

        private void playAudioStream(PullAudioOutputStream audio) {
            ActivityAudioStream stream = new ActivityAudioStream(audio);
            final ActivityAudioStream.ActivityAudioFormat audioFormat = stream.getActivityAudioFormat();
            final AudioFormat format = new AudioFormat(
                    AudioFormat.Encoding.PCM_SIGNED,
                    audioFormat.getSamplesPerSecond(),
                    audioFormat.getBitsPerSample(),
                    audioFormat.getChannels(),
                    audioFormat.getFrameSize(),
                    audioFormat.getSamplesPerSecond(),
                    false);
            try {
                int bufferSize = format.getFrameSize();
                final byte[] data = new byte[bufferSize];

                SourceDataLine.Info info = new DataLine.Info(SourceDataLine.class, format);
                SourceDataLine line = (SourceDataLine) AudioSystem.getLine(info);
                line.open(format);

                if (line != null) {
                    line.start();
                    int nBytesRead = 0;
                    while (nBytesRead != -1) {
                        nBytesRead = stream.read(data);
                        if (nBytesRead != -1) {
                            line.write(data, 0, nBytesRead);
                        }
                    }
                    line.drain();
                    line.stop();
                    line.close();
                }
                stream.close();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
    ```

1. 在 **main** 方法中，首先配置 `BotConnectorConfig` 并使用它来创建 `SpeechBotConnector` 实例。 这会连接到 Direct Line 语音通道以便与机器人交互。 `AudioConfig` 实例还用于指定音频输入的源。 在此示例中，对 `AudioConfig.fromDefaultMicrophoneInput()` 使用了默认麦克风。

    * 将字符串 `YourSubscriptionKey` 替换为可从[此处](get-started.md)获取的订阅密钥。
    * 将字符串 `YourServiceRegion` 替换为与订阅关联的[区域](regions.md)。
    * 将字符串 `YourChannelSecret` 替换为 Direct Line 语音通道机密。

    > [!NOTE]
    > 在预览版中，Direct Line 语音通道目前仅支持 **westus2** 区域。

    > [!NOTE]
    > [免费试用语音服务](get-started.md)中所述的适用于标准定价层的 30 天试用版仅限 **westus**（不是 **westus2**）区域，因此与 Direct Line 语音不兼容。 免费和标准层 **westus2** 订阅与此兼容。

    ```java
    final String channelSecret = "YourChannelSecret"; // Your channel secret
    final String subscriptionKey = "YourSubscriptionKey"; // your subscription key
    final String region = "YourServiceRegion"; // Your service region. Currently assumed to be westus2
    final BotConnectorConfig botConnectorConfig = BotConnectorConfig.fromSecretKey(channelSecret, subscriptionKey, region);

    // Configure audio input from microphone.
    final AudioConfig audioConfig = AudioConfig.fromDefaultMicrophoneInput();

    // Create a SpeechjBotConnector instance
    final SpeechBotConnector botConnector = new SpeechBotConnector(botConnectorConfig, audioConfig);
    ```

1. `SpeechBotConnector` 依赖于多个事件来传达其机器人活动、语音识别结果和其他信息。 接下来请添加这些事件侦听器。

    ```java
    // Recognizing will provide the intermediate recognized text while an audio stream is being processed
    botConnector.recognizing.addEventListener((o, speechRecognitionResultEventArgs) -> {
        log.info("Recognizing speech event text: {}", speechRecognitionResultEventArgs.getResult().getText());
    });

    // Recognized will provide the final recognized text once audio capture is completed
    botConnector.recognized.addEventListener((o, speechRecognitionResultEventArgs) -> {
        log.info("Recognized speech event reason text: {}", speechRecognitionResultEventArgs.getResult().getText());
    });

    // SessionStarted will notify when audio begins flowing to the service for a turn
    botConnector.sessionStarted.addEventListener((o, sessionEventArgs) -> {
        log.info("Session Started event id: {} ", sessionEventArgs.getSessionId());
    });

    // SessionStopped will notify when a turn is complete and it's safe to begin listening again
    botConnector.sessionStopped.addEventListener((o, sessionEventArgs) -> {
        log.info("Session stopped event id: {}", sessionEventArgs.getSessionId());
    });

    // Canceled will be signaled when a turn is aborted or experiences an error condition
    botConnector.canceled.addEventListener((o, canceledEventArgs) -> {
        log.info("Canceled event details: {}", canceledEventArgs.getErrorDetails());
        botConnector.disconnectAsync();
    });

    // ActivityReceived is the main way your bot will communicate with the client and uses bot framework activities.
    botConnector.activityReceived.addEventListener((o, activityEventArgs) -> {
        final String act = activityEventArgs.getActivity().serialize();
            log.info("Received activity {} audio", activityEventArgs.hasAudio() ? "with" : "without");
            if (activityEventArgs.hasAudio()) {
                playAudioStream(activityEventArgs.getAudio());
            }
        });
    ```

1. 调用 `connectAsync()` 方法将 `SpeechBotConnector` 连接到 Direct Line 语音。 若要测试机器人，可以调用 `listenOnceAsync` 方法以从麦克风发送音频输入。 此外，还可以使用 `sendActivityAsync` 方法以序列化字符串的形式发送自定义活动。 这些自定义活动可以提供机器人在聊天中使用的其他数据。

    ```java
    botConnector.connectAsync();
    // Start listening.
    System.out.println("Say something ...");
    botConnector.listenOnceAsync();

    // botConnector.sendActivityAsync(...)
    ```

1. 保存对 `Main` 文件的更改。

1. 为支持响应播放，你将添加一个额外的类，该类用于将从 getAudio() API 返回的 PullAudioOutputStream 对象转换为一个 java InputStream 以方便处理。 此 ActivityAudioStream 是一个专用类，用于处理来自“Direct Line 语音通道”的音频响应。 它将提供访问器来提取处理播放所需的音频格式信息：为此，请选择“文件” > “新建” > “类”。

1. 在“新建 Java 类”窗口中，在“包”字段中输入 **speechsdk.quickstart**，在“名称”字段中输入 **ActivityAudioStream**。

1. 打开新建的 **ActivityAudioStream** 类并替换为下面提供的代码。

    ```java
    package com.speechsdk.quickstart;

    import com.microsoft.cognitiveservices.speech.audio.PullAudioOutputStream;

    import java.io.IOException;
    import java.io.InputStream;


    public final class ActivityAudioStream extends InputStream {
        /**
         * The number of samples played per second. (16 kHz)
         */
        public static final long SAMPLE_RATE = 16000;
        /**
         * The number of bits in each sample of a sound that has this format. (16 bits)
         */
        public static final int BITS_PER_SECOND = 16;
        /**
         * The number of audio channels in this format (1 for mono).
         */
        public static final int CHANNELS = 1;
        /**
         * The number of bytes in each frame of a sound that has this format (2).
         */
        public static final int FRAME_SIZE = 2;

        /**
         * Reads up to a specified maximum number of bytes of data from the audio
         * stream, putting them into the given byte array.
         *
         * @param b   the buffer into which the data is read
         * @param off the offset, from the beginning of array <code>b</code>, at which
         *            the data will be written
         * @param len the maximum number of bytes to read
         * @return the total number of bytes read into the buffer, or -1 if there
         * is no more data because the end of the stream has been reached
         */
        @Override
        public int read(byte[] b, int off, int len) {
            byte[] tempBuffer = new byte[len];
            int n = (int) this.pullStreamImpl.read(tempBuffer);
            for (int i = 0; i < n; i++) {
                if (off + i > b.length) {
                    throw new ArrayIndexOutOfBoundsException(b.length);
                }
                b[off + i] = tempBuffer[i];
            }
            if (n == 0) {
                return -1;
            }
            return n;
        }

        /**
         * Reads the next byte of data from the activity audio stream if available.
         *
         * @return the next byte of data, or -1 if the end of the stream is reached
         * @see #read(byte[], int, int)
         * @see #read(byte[])
         * @see #available
         * <p>
         */
        @Override
        public int read() {
            byte[] data = new byte[1];
            int temp = read(data);
            if (temp <= 0) {
                // we have a weird situation if read(byte[]) returns 0!
                return -1;
            }
            return data[0] & 0xFF;
        }

        /**
         * Reads up to a specified maximum number of bytes of data from the activity audio stream
         * putting them into the given byte array.
         *
         * @param b the buffer into which the data is read
         * @return the total number of bytes read into the buffer, or -1 if there
         * is no more data because the end of the stream has been reached
         */
        @Override
        public int read(byte[] b) {
            int n = (int) pullStreamImpl.read(b);
            if (n == 0) {
                return -1;
            }
            return n;
        }

        /**
         * Skips over and discards a specified number of bytes from this
         * audio input stream.
         *
         * @param n the requested number of bytes to be skipped
         * @return the actual number of bytes skipped
         * @throws IOException if an input or output error occurs
         * @see #read
         * @see #available
         */
        @Override
        public long skip(long n) {
            if (n <= 0) {
                return 0;
            }
            if (n <= Integer.MAX_VALUE) {
                byte[] tempBuffer = new byte[(int) n];
                return read(tempBuffer);
            }
            long count = 0;
            for (long i = n; i > 0; i -= Integer.MAX_VALUE) {
                int size = (int) Math.min(Integer.MAX_VALUE, i);
                byte[] tempBuffer = new byte[size];
                count += read(tempBuffer);
            }
            return count;
        }

        /**
         * Closes this audio input stream and releases any system resources associated
         * with the stream.
         */
        @Override
        public void close() {
            this.pullStreamImpl.close();
        }

        /**
         * Fetch the audio format for the ActivityAudioStream. The ActivityAudioFormat defines the sample rate, bits per sample and the # channels
         *
         * @return instance of the ActivityAudioFormat associated with the stream
         */
        public ActivityAudioStream.ActivityAudioFormat getActivityAudioFormat() {
            return activityAudioFormat;
        }

        /**
         * Returns the maximum number of bytes that can be read (or skipped over) from this
         * audio input stream without blocking.
         *
         * @return the number of bytes that can be read from this audio input stream without blocking.
         * As this implementation does not buffer this will be defaulted to 0
         */
        @Override
        public int available() {
            return 0;
        }

        public ActivityAudioStream(final PullAudioOutputStream stream) {
            pullStreamImpl = stream;
            this.activityAudioFormat = new ActivityAudioStream.ActivityAudioFormat(SAMPLE_RATE, BITS_PER_SECOND, CHANNELS, FRAME_SIZE, AudioEncoding.PCM_SIGNED);
        }

        private PullAudioOutputStream pullStreamImpl;

        private ActivityAudioFormat activityAudioFormat;

        /**
         * ActivityAudioFormat is an internal format which contains metadata regarding the type of arrangement of
         * audio bits in this activity audio stream.
         */
        static class ActivityAudioFormat {

            private long samplesPerSecond;
            private int bitsPerSample;
            private int channels;
            private int frameSize;
            private AudioEncoding encoding;

            public ActivityAudioFormat(long samplesPerSecond, int bitsPerSample, int channels, int frameSize, AudioEncoding encoding) {
                this.samplesPerSecond = samplesPerSecond;
                this.bitsPerSample = bitsPerSample;
                this.channels = channels;
                this.encoding = encoding;
                this.frameSize = frameSize;
            }

            /**
             * Fetch the number of samples played per second for the associated audio stream format.
             *
             * @return the number of samples played per second
             */
            public long getSamplesPerSecond() {
                return samplesPerSecond;
            }

            /**
             * Fetch the number of bits in each sample of a sound that has this audio stream format.
             *
             * @return the number of bits per sample
             */
            public int getBitsPerSample() {
                return bitsPerSample;
            }

            /**
             * Fetch the number of audio channels used by this audio stream format.
             *
             * @return the number of channels
             */
            public int getChannels() {
                return channels;
            }

            /**
             * Fetch the default number of bytes in a frame required by this audio stream format.
             *
             * @return the number of bytes
             */
            public int getFrameSize() {
                return frameSize;
            }

            /**
             * Fetch the audio encoding type associated with this audio stream format.
             *
             * @return the encoding associated
             */
            public AudioEncoding getEncoding() {
                return encoding;
            }
        }

        /**
         * Enum defining the types of audio encoding supported by this stream
         */
        public enum AudioEncoding {
            PCM_SIGNED("PCM_SIGNED");

            String value;

            AudioEncoding(String value) {
                this.value = value;
            }
        }
    }

    ```

1. 保存对 `ActivityAudioStream` 文件的更改。

## <a name="build-and-run-the-app"></a>生成并运行应用

按 F11，或选择“运行” > “调试”。
控制台会显示消息“请讲话”。此时，请讲出机器人理解的英语短语或句子。 你的语音将通过 Direct Line 语音通道传送到机器人，机器人将识别并处理你的语言，然后返回活动形式的响应。 如果机器人返回了语音作为响应，则会使用 `AudioPlayer` 类播放音频。

![成功识别后的控制台输出的屏幕截图](media/sdk/qs-java-jre-08-console-output.png)

## <a name="next-steps"></a>后续步骤

GitHub 上提供了其他示例，例如如何从音频文件中读取语音。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 Java 示例](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>另请参阅

- [快速入门：翻译语音，Java（Windows、Linux）](quickstart-translate-speech-java-jre.md)
- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)
