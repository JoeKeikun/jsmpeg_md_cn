# JSMpeg – JavaScript的MPEG1视频和MP2音频解码器

JSMpeg是一个采用JavaScript语言编写的视频播放器。它由MPEG-TS分路器、MPEG1视频解码器、MP2音频解码器、WebGL渲染器、Canvas2D渲染器和WebAudio音频输出组成。JSMpeg可以通过Ajax（异步方式）加载静态视频文件，同时它也允许通过WebSockets方式进行低延迟（50ms左右）流式传输视频数据。

JSMpeg可以在iPhone 5S上以30fps速率解码720p的视频，其适用于任何现代浏览器（Chrome，Firefox，Safari，Edge），且gzip压缩后只需20kb。

你可以像这样简单地使用它：

```html
<script src="jsmpeg.min.js"></script>
<div class="jsmpeg" data-url="video.ts"></div>
```

更多信息和演示: [jsmpeg.com](http://jsmpeg.com/)


## 用法

可以通过在HTML Element上指明 `jsmpeg` CSS类名，JSMpeg将会自动在其子Element上创建视频播放器：

```html
<div class="jsmpeg" data-url="<url>"></div>
```

或者通过在JavaScript中通过直接调用 `JSMpeg.Player()` 构造函数来创建Player实例：

```javascript
var player = new JSMpeg.Player(url [, options]);
```

请注意，在使用HTML Element（该Element对应属性： `JSMpeg.VideoElement`）来创建播放器时，会比使用 `JSMpeg.Player()` 的创建方式多一些特色。（即一个用SVG生成的按钮控制暂停/播放，以及在iOS设备上能“解锁”音频的功能组件。）

`url` 参数接受一个MPEG.TS静态文件或WebSocket服务器（WS://..）的URL地址。

`options` 参数支持以下属性：

- `canvas` - 用于视频渲染的HTML画布。如果没有设定，渲染器将自动创建一个Canvas Element。
- `loop` - 是否循环视频（仅对静态文件有效）。默认为 `true`。
- `autoplay` - 是否立即开始播放（仅对静态文件有效）。默认为 `false`。
- `audio` - 是否解码音频。默认为 `true`。
- `video` - 是否解码视频。默认为 `true`。
- `poster` - 指向一张图片的URL地址，用于视频播放之前当作封面显示。
- `pauseWhenHidden` - 当前tab不活动时，是否暂停播放。默认为 `true`。请注意，浏览器通常会在非活动状态的tab中节制JS的执行。
- `disableGl` - 是否禁用WebGL，总是使用Canvas2D渲染器。默认为 `false`。
- `preserveDrawingBuffer` - WebGL context是否由 `preserveDrawingBuffer` 创建 - 在通过类似 `canvas.toDataURL()` 方法进行“截图”时必须设置为 `true`。默认为 `false`。
- `progressive` - 是否分块加载数据（仅对静态文件有效）。当设为 `true` 时，可以在资源未被完全加载之前就开始播放。默认为 `true`。
- `throttled` - 当 `progressive` 为 `true` 时，是否在不需要播放时推迟数据加载。默认为 `true`。
- `chunkSize` - 当 `progressive` 为 `true` 时，一次加载的块大小（以字节为单位）。默认为 `1024*1024` （1mb）。
- `decodeFirstFrame` - 是否解码并显示视频的第一帧。用于设置画布大小并将该帧当作“封面”图像。当 `autoplay` 为 `true` 或使用WebSockets流式传输时，将不会产生任何效果。默认为 `true`。
- `maxAudioLag` - 采用流式传输时，最大音频队列长度（以秒为单位）。
- `videoBufferSize` - 采用流式传输时，视频解码缓冲区的大小（以字节为单位）。默认512 * 1024（512kb）。当视频比特率很高的时候，你可能需要增大此值。
- `audioBufferSize` - 采用流式传输时，音频解码缓冲区的大小（以字节为单位）。默认128 * 1024（128kb）。当音频比特率很高的时候，你可能需要增大此值。

除了 `canvas` 之外的所有设置项也可以通过 `data-` 属性与HTML Element一起使用。

例如，设置循环和自动播放功能，在JavaScript中可以这样写：

```javascript
var player = new JSMpeg.Player('video.ts' {loop: true, autoplay: true});
```

而在HTML中则可以这样写：

```html
<div class="jsmpeg" data-url="video.ts" data-loop="true" data-autoplay="true"></div>
```

注意，在HTML中驼峰格式的参数项必须转换成使用连字符表示的 `data-` 属性才能生效。例如， `decodeFirstFrame: true` 对应HTML Element中的 `data-decode-first-frame="true"`。


## JSMpeg.Player API

`JSMpeg.Player` 实例支持以下方法和属性：

- `.play()` – 开始播放。
- `.pause()` – 暂停播放。
- `.stop()` – 停止播放并回到开头。
- `.destroy()` – 停止播放，中断源的连接并清除WebGL和WebAudio的状态。播放器对象将不能再被调用。
- `.volume` – 获取或设置音量（0-1）
- `.currentTime` – 获取或设置当前播放位置（以秒为单位）


## 为JSMpeg转码Video/Audio

JSMpeg只能播放使用MPEG1视频编码和MP2音频编码的MPEG-TS文件。JSMpeg的视频解码器无法处理B-Frames（尽管在现代进阶编码器中似乎都没使用过该模式），而且视频的长度必须是2的倍数。

你可以使用 [ffmpeg](https://ffmpeg.org/) 来转码一个符合要求的视频，代码如下：

```sh
ffmpeg -i in.mp4 -f mpegts -codec:v mpeg1video -codec:a mp2 -b 0 out.ts
```

你还可以控制视频大小（`-s`）、帧速率（`-r`）、视频比特率（`-b：v`）、音频比特率（`-b：a`）、音频通道数（`-ac`）、采样率（`-ar`）等等。更多相关详细信息，请参阅ffmpeg文档。

综合示例：

```sh
ffmpeg -i in.mp4 -f mpegts \
	-codec:v mpeg1video -s 960x540 -b:v 1500k -r 30 -bf 0 \
	-codec:a mp2 -ar 44100 -ac 1 -b:a 128k \
	out.ts
```


## 性能注意事项

尽管JSMpeg可以在iPhone 5S上以30fps处理720p视频，但请记住MPEG1不如现代其他编解码器的效率高。使用MPEG1编码的HD视频文件会占用更多的带宽。720p的视频需要在高于2Mbits/s（即250kb/s）码率下才看起来不模糊。此外，比特率越高，给JavaScript带来的解码工作量就越大（影响其他JavaScript代码执行的流畅度）。

对静态文件（或者在同一WiFi环境中使用流式传输）来说带宽应该不是问题。如果你不需要支持移动设备的话，1080p的视频在10Mbit/s码率上效果比较好（如果你的编码器可以跟上的话）。对于其他的视频，我建议你尽量使用最大2Mbit/s码率的540p（960x540）视频。


## WebSockets流式传输

JSMpeg可以与发送二进制MPEG-TS数据的WebSocket服务器建立连接。在流式传输时，JSMpeg将尽可能降低播放的延迟：它会立即对已下载的内容进行解码，并忽略视频和音频的时间戳。为了确保音频和视频内容的同步（以及低延迟），在转码时音频数据应当交织在每帧视频数据之间（可以通过ffmpeg设置 `-muxdelay` 参数实现）。

我们可以预想一种独立且自带缓冲的流传输模式，在这种模式下JSMpeg会预加载几秒钟的数据，并以精确的时序和音/视频同步的方式呈现所有内容。（但目前尚未实现）

视频和音频的内部缓冲区很小（分别只有512kb和128kb），所以JSMpeg将会丢弃旧的（甚至还未播放的）数据，以便为新下载的数据腾出空间从而不让视频出现过多的模糊。如此一来可能会在网络拥塞时造成解码残像，但却能将延迟控制在最低限度。在必要的情况下，你也可以通过将 `videoBufferSize` 和 `audioBufferSize` 参数调大以减少上诉问题的发生。

JSMpeg带有一个用Node.js编写的小型 WebSocket “中转服务器（relay）”。该服务器通过HTTP接收MPEG-TS数据源，并通过WebSocket将其共享给所有与服务器相连的浏览器。其中传入的HTTP流数据可以通过 [ffmpeg](https://ffmpeg.org/) ， [gstreamer](https://gstreamer.freedesktop.org/) 或其他方式生成。

因为ffmpeg不懂WebSocket协议，所以将源文件和WebSocket中转服务器分拆开来是必要的。但是，这种分拆模式可以允许你在公共服务上安装WebSocket中转服务器，并在Internet上共享你的数据流（注意：路由器中的NAT会阻止公共网络 _接入_ 本地网络）。

简而言之，它的工作原理如下：

1. 运行websocket-relay.js
2. 运行ffmpeg，将输出数据发送到中转服务器的HTTP端口
3. 将浏览器中的JSMpeg连接到中转服务器的Websocket端口


## 流式传输安装示例：树莓派（Raspberry Pi） Live网络摄像头

在这个示例中，ffmpeg和WebSocket中转服务器都运行在同一系统上。如此一来你就可以在本地查看数据流，而不用连到公共网络上去查看数据。

这个示例默认你的网络摄像头兼容Video4Linux2，并将数据输出到 `/dev/video0` 这个文件路径。大多数USB网络摄像头都支持UVC标准，应该可以正常运行。通过加载内核模块命令： `sudo modprobe bcm2835-v4l2` ，可以让板载树莓摄像头（Raspberry Camera）作为V4L2设备来使用。

1) 安装ffmpeg（参见 [How to install ffmpeg on Debian / Raspbian](http://superuser.com/questions/286675/how-to-install-ffmpeg-on-debian)）。通过ffmpeg，我们可以捕获网络摄像头的视频和音频数据，并将其编码为MPEG1/MP2的格式。

2) 安装Node.js和npm（参见最新版的 [Installing Node.js on Debian and Ubuntu based Linux distributions](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)）。其中Websocket中转服务器是用Node.js编写的。

3) 安装http-server：
`sudo npm -g install http-server`
我们将利用它来提供静态文件（view-stream.html, jsmpeg.min.js）的访问服务，以便我们可以通过浏览器查看站点。同理其他同类的网络服务器程序也能做到（例如：nginx, apache等等）。

4) 安装git并克隆此库（或者只需将其下载为ZIP并解压缩）：
```
sudo apt-get install git
git clone https://github.com/phoboslab/jsmpeg.git
```

5) 转到 jsmpeg/ 目录：
`cd jsmpeg/`

6) 安装Node.js Websocket库：
`npm install ws`

7) 启动Websocket中转服务器，为传入的HTTP视频流设置一个密码和端口（方便在浏览器上连接该Websocket端口）：
`node websocket-relay.js supersecret 8081 8082`

8) 在一个新的终端窗口中执行指令：（需在 `jsmpeg/` 目录中，执行 `http-server` 。这样我们就可以将view-stream.html提供给浏览器）
`http-server`

9) 在浏览器中打开流媒体网站：
`http://192.168.[...]:8080/view-stream.html`
`http-server`会告诉你它正在运行的ip地址（通常是 `192.168.[...]` ）和端口（通常是 `8080` ）

10) 在第三个终端窗口中，启动ffmpeg以捕获网络摄像头视频并将其发送到Websocket中转服务器。为目标URL设置密码和端口（参照步骤7）：
```
ffmpeg \
	-f v4l2 \
		-framerate 25 -video_size 640x480 -i /dev/video0 \
	-f mpegts \
		-codec:v mpeg1video -s 640x480 -b:v 1000k -bf 0 \
	http://localhost:8081/supersecret
```

你现在应该可以在浏览器中看到试试网络摄像头的图像了。

如果ffmpeg无法打开输入视频，有可能是你的网络摄像头不支持给定的分辨率、格式或帧速率。想要获取可运行的兼容列表，请执行指令：
`ffmpeg -f v4l2 -list_formats all -i /dev/video0`

为了添加网络摄像头音频，只需在调用ffmpeg的输入中新增两个独立的参数。
```
ffmpeg \
	-f v4l2 \
		-framerate 25 -video_size 640x480 -i /dev/video0 \
	-f alsa \
		-ar 44100 -c 2 -i hw:0 \
	-f mpegts \
		-codec:v mpeg1video -s 640x480 -b:v 1000k -bf 0 \
		-codec:a mp2 -b:a 128k \
		-muxdelay 0.001 \
	http://localhost:8081/supersecret
```
注意其中的 `muxdelay` 参数。这可以减少延迟，但对流式传输视频和音频而言并不是一定有效的 - 请参阅下面的备注。


## 一些关于ffmpeg复用和延迟的备注

在MPEG-TS中加入一个音频流会引入相当大的延迟。我发现在linux上使用ALSA和V4L2时延迟尤为严重（但在macOS上使用AVFoundation却工作得很好）。但我找到一个简单的解决方法：只需同时运行两个ffmpeg实例，一个用于音频，一个用于视频。将两个输出同时发送到同一个Websocket中转服务器上。由于MPEG-TS格式的简单构造，在中转服务器上两个数据流会自动相互“适配”。

```
ffmpeg \
	-f v4l2 \
		-framerate 25 -video_size 640x480 -i /dev/video0 \
	-f mpegts \
		-codec:v mpeg1video -s 640x480 -b:v 1000k -bf 0 \
		-muxdelay 0.001 \
	http://localhost:8081/supersecret

# 在另一个terminal终端上
ffmpeg \
	-f alsa \
		-ar 44100 -c 2 -i hw:0 \
	-f mpegts \
		-codec:a mp2 -b:a 128k \
		-muxdelay 0.001 \
	http://localhost:8081/supersecret
```

经过我的测试，USB网络摄像头会有大约180ms的延迟，对此我们无能为力。然而，树莓派（Raspberry Pi）有一个 [camera module](https://www.raspberrypi.org/products/camera-module-v2/) 可以提供低延迟的视频录制。

如何在Windows或macOS上使用ffmpeg来捕获网络摄像头输入，请参阅 [ffmpeg Capture/Webcam Wiki](https://trac.ffmpeg.org/wiki/Capture/Webcam) 。


## JSMpeg架构和内部组件

为了将代码执行开销保持在最低限度，本库采用相同模块化的方式构建。在不修改任何其他模块的情况下，实现了新的解复用器、解码器、输出（渲染器，音频设备）和源。但是，你仍需通过子属性 `JSMpeg.Player` 来使用这些新模块。

查看 [jsmpeg.js source](https://github.com/phoboslab/jsmpeg/blob/master/src/jsmpeg.js) 的源码，可以让你进一步了解模块如何互连以及它们应提供哪些具体API。此外，我还写了一篇关于JSMpeg内部构造的博文可供参考： [Decode It Like It's 1999](http://phoboslab.org/log/2017/02/decode-it-like-its-1999)。

若你只需要JSMpeg库提供的一部分功能，而不想创建一个完整的播放器也很简单。譬如，你可以创建一个独立的 `JSMpeg.Decoder.MPEG1Video` 类实例、 `.connect()` 一个渲染器、 `.write()` 一些数据给实例、以及 `.decode()` 一帧画面，通过上诉方法你可以不用接触JSMpeg的其他部分。


## 早期版本

目前存在于此repo中的JSMpeg版本是对原始jsmpeg库的完全重写，该库只能解码原始mpeg1编码的视频。如果你正在寻找旧版本，请参阅 [v0.2 tag](https://github.com/phoboslab/jsmpeg/releases/tag/v0.2)。
