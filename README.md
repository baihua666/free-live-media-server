# free-live-media-server
使用第三方平台的流媒体资源，实现自己的业务功能，作为学习演示使用。但是千万不要拿来大量使用，否则会成被告。

## 需求场景：
做学习、演示小项目开发，需要用到流媒体服务器，购买上行下行带宽比较大的云服务器，价格不菲，能省则省。自己用阿里云搭了一个流媒体服务器，使用开源的解决方案，比较简单，但是服务器配置很低，带宽不够。于是想着找些免费的资源，例如B站等直播平台。
思路：先在B站等平台注册一批直播帐号，把帐号里面的配置信息保存起来(其实就是需要推流地址和拉流地址)，作为自己业务服务器的流媒体服务器资源。然后在自己的业务里面，需要推流的时候，从服务器上获取当前可用的服务器资源，客户端推流或者拉流。假如有一批B站的直播室可以使用，那绝对可以满足我们学习或者演示的小需求了，只要并发数不超过服务器资源帐号数量就可以。

## 实现步骤：
### 1.获取服务器资源，如果有批量的帐号注册脚本，那是最好不过了，不过只能想想而已，估计是很难实现的，这里先人工注册一个B站直播帐号，B站注册需要邀请码，或者答题，于是经过了约10分钟的答题过程，勉强及格，注册成功。
注册地址： https://passport.bilibili.com/register/phone.html
#### 1.1获取注册帐号后的个人信息
进入开播设置页面：https://link.bilibili.com/p/center/index#/my-room/start-live

![image](https://raw.githubusercontent.com/abc19abc91/free-live-media-server/master/images/image0.png)

上面几条信息需要保存：
我的直播间地址http://live.bilibili.com/8167278
你的rtmp地址：rtmp://xl.live-send.acg.tv/live-xl/
你的直播码：?streamname=live_277053127_2361311&key=639240e03cbc5137b9856a733481c7c8
#### 1.2获取推流地址
通过以上信息，就可以得到推流地址:
rtmp://xl.live-send.acg.tv/live-xl/?streamname=xxx&key=xxx
将xxx替换后的结果
rtmp://xl.live-send.acg.tv/live-xl/?streamname=live_277053127_2361311&key=639240e03cbc5137b9856a733481c7c8
#### 1.3获取拉流地址
在浏览器上打开直播间地址，右键检查，就能抓到真实的拉流地址。这个办法比较笨，但是比较直接，后面可以改成软件自动抓包分析，或者用爬虫，分析协议的方式做的更智能一些。
![image](https://raw.githubusercontent.com/abc19abc91/free-live-media-server/master/images/image1.png)

图中有用的信息为
https%3A%2F%2Fxl.live-play.acgvideo.com%2Flive-xl%2F256070%2Flive_277053127_2361311.flv%3FwsSecret%3Da0c2634900258d5f59a29de3dcd20acd%26wsTime%3D1515998009
将以上信息放在URL在线解码中，解出来地址为：
https://xl.live-play.acgvideo.com/live-xl/256070/live_277053127_2361311.flv?wsSecret=a0c2634900258d5f59a29de3dcd20acd&wsTime=1515998009
于是就有了拉流地址
### 2.推流
开源的推流方案比较多，这里为了方便，安装ffmpeg命令行，做简单的演示，测试环境为MAC，用brew安装ffmpeg
准备一下测试的文件，例如一个MP4文件。命令行定位到文件目录，执行推流命令：
ffmpeg -re -i video1.mp4 -c:v libx264 -preset veryfast -maxrate 3000k \
-bufsize 6000k -pix_fmt yuv420p -g 50 -b:a 160k -ac 2 \
-ar 44100 -f flv "rtmp://xl.live-send.acg.tv/live-xl/?streamname=live_277053127_2361311&key=639240e03cbc5137b9856a733481c7c8”
推流前要确认点了开启直播按钮，就是说直播间是开的，否则会推流失败，如果成功了，在流星器里面打开直播间地址，就可以看到我们的测试MP4已经在直播出来了。
### 3.拉流
为方便测试，我直接用VLC客户端来做拉流测试。下载VLC客户端并运行
![image](https://raw.githubusercontent.com/abc19abc91/free-live-media-server/master/images/image2.png)

选择network,输入上面说的拉流地址

![image](https://raw.githubusercontent.com/abc19abc91/free-live-media-server/master/images/image3.png)

点OPEN
就可以看到拉取到的测试数据了。

## 优化方向：
1.帐号注册，个人信息获取，多媒体流的地址，都是手工获取的，需要有批量获取的方法。
2.第三方平台如果更改了策略，会导致业务出错，最好用多个第三方平台帐号资源，加上自己的备用服务器来保证稳定。
