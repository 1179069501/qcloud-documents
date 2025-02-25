## 适用场景
CDN 直播观看，也叫 “CDN 旁路直播”，由于 TRTC 采用 UDP 协议进行传输音视频数据，而标准直播 CDN 则采用的 RTMP\HLS\FLV 等协议进行数据传输，所以需要将 TRTC 中的音视频数据**旁路**到直播 CDN 中，才能在让观众通过直播 CDN 进行观看。

给 TRTC 对接 CDN 观看，一般被用于解决如下两类问题：
- **问题一：超高并发观看**
TRTC 的低延时观看能力，单房间支持的最大人数上限为10万人。CDN 观看虽然延迟要高一些，但支持10万人以上的并发观看，且 CDN 的计费价格更加便宜。
- **问题二：移动端网页播放**
TRTC 虽然支持 WebRTC 协议接入，但主要用于 Chrome 桌面版浏览器，移动端浏览器的兼容性非常不理想，尤其是 Android 手机浏览器对 WebRTC 的支持普遍都很差。所以如果希望通过 Web 页面在移动端分享直播内容，还是推荐使用 HLS(m3u8) 播放协议，这也就需要借助直播 CDN 的能力来支持 HLS 协议。


## 原理解析
腾讯云会使用一批旁路转码集群，将 TRTC 中的音视频数据旁路到直播 CDN 系统中，该集群负责将 TRTC 所使用的 UDP 协议转换为标准的直播 RTMP 协议。
[](id:directCDN)
**单路画面的旁路直播**
当 TRTC 房间中只有一个主播时，TRTC 的旁路推流跟标准的 RTMP 协议直推功能相同，不过 TRTC 的 UDP 相比于 RTMP 有更强大的弱网络抗性。
![](https://main.qcloudimg.com/raw/b682b1493a81bc8a53aea6a07f375ba2.gif)

[](id:mixCDN)
**混合画面的旁路直播**
TRTC 最擅长的领域就是音视频互动连麦，如果一个房间里同时有多个主播，而 CDN 观看端只希望拉取一路音视频画面，就需要使用 [云端混流服务](https://cloud.tencent.com/document/product/647/16827) 将多路画面合并成一路，其原理如下图所示：
![](https://main.qcloudimg.com/raw/f2feaaaac176bb4fe7dd1c318490f9e1.gif)

>? **为什么不直接播放多路 CDN 画面？**
>播放多路 CDN 画面很难解决多路画面的延迟对齐问题，同时拉取多路画面所消耗的下载流量也比单独画面要多，所以业内普遍采用云端混流方案。

## 前提条件
已开通腾讯 [云直播](https://console.cloud.tencent.com/live) 服务。应国家相关部门的要求，直播播放必须配置播放域名，具体操作请参考 [添加自有域名](https://cloud.tencent.com/document/product/267/20381)。

## 使用步骤

[](id:step1)
### 步骤1：开启旁路推流功能

1. 登录 [实时音视频控制台](https://console.cloud.tencent.com/trtc)。
2. 在左侧导航栏选择**应用管理**，单击目标应用所在行的**功能配置**。
3. 在**旁路推流配置**中，单击**启用旁路推流**右侧的![](https://main.qcloudimg.com/raw/5f58afe211aa033037e5c0b793023b49.png)，在弹出的**开启旁路推流功能**对话框中，单击**开启旁路推流功能**即可开通。


[](id:step2)
### 步骤2：配置播放域名并完成 CNAME
1. 登录 [云直播控制台](https://console.cloud.tencent.com/live/)。
2. 在左侧导航栏选择**域名管理**，您会看到在您的域名列表新增了一个推流域名，格式为 `xxxxx.livepush.myqcloud.com`，其中 xxxxx 是一个数字，叫做 bizid，您可以在**实时音视频控制台** > **[应用管理](https://console.cloud.tencent.com/trtc/app)** > **应用信息**中查找到 bizid 信息。
3. 单击**添加域名**，输入您已经备案过的播放域名，选择域名类型为**播放域名**，选择加速区域（默认为**中国大陆**），单击**确定**即可。
4. 域名添加成功后，系统会为您自动分配一个 CNAME 域名（以`.liveplay.myqcloud.com`为后缀）。CNAME 域名不能直接访问，您需要在域名服务提供商处完成 CNAME 配置，配置生效后，即可享受云直播服务。具体操作请参见 [CNAME 配置](https://cloud.tencent.com/document/product/267/19908)。

![](https://main.qcloudimg.com/raw/97b24ee2a758b311ba8dea23db04c3ae.png)

>! **不需要添加推流域名**，在 [步骤1](#step1) 中开启旁路直播功能后，腾讯云会默认在您的云直播控制台中增加一个格式为  `xxxxx.livepush.myqcloud.com` 的推流域名，该域名为腾讯云直播服务和 TRTC 服务之间约定的一个默认推流域名，暂时不支持修改。

[](id:step3)
### 步骤3：关联 TRTC 的音视频流到直播 streamId
开启旁路推流功能后， TRTC 房间里的每一路画面都配备一路对应的播放地址，该地址的格式如下：
```
http://播放域名/live/[streamId].flv
```
地址中的 streamId 可以在直播中唯一标识一条直播流，您可以自己指定 streamId，也可以使用系统默认生成的。

#### 方式一：自定义指定 streamId
您可以在调用 `TRTCCloud` 的 `enterRoom` 函数时，通过其参数 `TRTCParams` 中的 `streamId` 参数指定直播流 ID。
以 iOS 端的 Objective-C 代码为例：

```ObjectiveC
TRTCCloud *trtcCloud = [TRTCCloud sharedInstance];
TRTCParams *param = [[TRTCParams alloc] init];
param.sdkAppId = 1400000123;     // TRTC 的 SDKAppID，创建应用后可获得
param.roomId   = 1001;           // 房间号
param.userId   = @"rexchang";    // 用户名
param.userSig  = @"xxxxxxxx";    // 登录签名
param.role     = TRTCRoleAnchor; // 角色：主播
param.streamId = @"stream1001";  // 流 ID
[trtcCloud enterRoom:params appScene:TRTCAppSceneLIVE]; // 请使用 LIVE 模式
```
userSig 的计算方法请参见 [如何计算及使用 UserSig](https://cloud.tencent.com/document/product/647/17275)。

#### 方式二：系统指定 streamId
开启自动旁路推流后，如果您没有自定义指定 streamId，系统会默认为您生成一个缺省的 streamId，生成规则如下：

- **拼装 streamId 用到的字段**
  - SDKAppID：您可以在 [控制台](https://console.cloud.tencent.com/trtc/app) > **应用管理** > **应用信息**中查找到。
  - bizid：您可以在 [控制台](https://console.cloud.tencent.com/trtc/app) > **应用管理** > **应用信息**中查找到。
  - roomId：由您在 `enterRoom` 函数的参数 `TRTCParams` 中指定。
  - userId：由您在 `enterRoom` 函数的参数 `TRTCParams` 中指定。
  - streamType： 摄像头画面为 main，屏幕分享为 aux （WebRTC 由于同时只支持一路上行，因此 WebRTC 上屏幕分享的流类型是 main）。

- **拼装 streamId 的计算规则**
 <table>
<tr>
<th>拼装</th>
<th>2020年01月09日及此后新建的应用</th>
<th>2020年01月09日前创建且使用过的应用</th>
</tr>
<tr>
<td>拼装规则</td>
<td>streamId = urlencode(sdkAppId_roomId_userId_streamType)</td>
<td>StreamId = bizid_MD5(roomId_userId_streamType)</td>
</tr>
<tr>
<td>计算样例</td>
<td>例如：sdkAppId = 12345678，roomId = 12345，userId = userA，用户当前使用了摄像头。<br>那么：streamId = 12345678_12345_userA_main</td>
<td>例如：bizid = 1234，roomId = 12345，userId = userA，用户当前使用了摄像头。<br>那么：streamId = 1234_MD5(12345_userA_main) = 1234_8D0261436C375BB0DEA901D86D7D70E8</td>
</tr>
</table>


[](id:step4)
### 步骤4：控制多路画面的混合方案

如果您想要获得混合后的直播画面，需要调用 TRTCCloud 的 `setMixTranscodingConfig` 接口启动云端混流转码，该接口的参数 `TRTCTranscodingConfig` 可用于配置：
 - 各个子画面的摆放位置和大小。
 - 混合画面的画面质量和编码参数。

画面布局的详细配置方法请参考 [云端混流转码](https://cloud.tencent.com/document/product/647/16827)，整个流程所涉及的各模块关系可以参考 [原理解析](#mixCDN)。

>! `setMixTranscodingConfig` 并不是在终端进行混流，而是将混流配置发送到云端，并在云端服务器进行混流和转码。由于混流和转码都需要对原来的音视频数据进行解码和二次编码，所以需要更长的处理时间。因此，混合画面的实际观看时延要比独立画面的多出1s - 2s。


[](id:step5)
### 步骤5：给 SDK 配置 License 授权
1. 获取 License 授权：
	- 若您已获得相关 License 授权，需在 [云直播控制台](https://console.cloud.tencent.com/live/license) 获取 License URL 和 License Key。
	![](https://qcloudimg.tencent-cloud.cn/raw/7053ac66fd06b9f178bf416d9d52ea21.png)
	- 若您暂未获得 License 授权，需先参考 [新增与续期 License](https://cloud.tencent.com/document/product/454/34750) 进行申请。
2. 在您的 App 调用 SDK 相关功能之前（建议在 `Application` / `- [AppDelegate application:didFinishLaunchingWithOptions:]`中）进行如下设置：
<dx-codeblock>
::: Android java
public class MApplication extends Application {

@Override
public void onCreate() {
    super.onCreate();
    String licenceURL = ""; // 获取到的 licence url
    String licenceKey = ""; // 获取到的 licence key
    V2TXLivePremier.setLicence(this, licenceURL, licenceKey);
    V2TXLivePremier.setObserver(new V2TXLivePremierObserver() {
            @Override
            public void onLicenceLoaded(int result, String reason) {
                Log.i(TAG, "onLicenceLoaded: result:" + result + ", reason:" + reason);
            }
        });
}
:::
::: iOS objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    NSString * const licenceURL = @"<获取到的licenseUrl>";
    NSString * const licenceKey = @"<获取到的key>";
        
    // V2TXLivePremier 位于 "V2TXLivePremier.h" 头文件中
    [V2TXLivePremier setLicence:licenceURL key:licenceKey];
    [V2TXLivePremier setObserver:self];
    NSLog(@"SDK Version = %@", [V2TXLivePremier getSDKVersionStr]);
    return YES;
}

#pragma mark - V2TXLivePremierObserver
- (void)onLicenceLoaded:(int)result Reason:(NSString *)reason {
    NSLog(@"onLicenceLoaded: result:%d reason:%@", result, reason);
}
@end
:::
</dx-codeblock>

>! **License 中配置的 packageName/BundleId 必须和应用本身一致，否则会播放失败。**

[](id:step6)
### 步骤6：获取播放地址并对接播放
当您通过 [步骤2](#step2) 配置完播放域名和 [步骤3](#step3) 完成 streamId 的映射后，即可得到直播的播放地址。播放地址的标准格式为：
```
http://播放域名/live/[streamId].flv
```

例如，您的播放域名为`live.myhost.com`，您将房间（1001）中的用户 userA 的直播流 ID 通过进房参数指定为 streamId = "streamd1001"。
则您可以得到三路播放地址：
```
 rtmp 协议的播放地址：rtmp://live.myhost.com/live/streamd1001
 flv 协议的播放地址：http://live.myhost.com/live/streamd1001.flv
 hls 协议的播放地址：http://live.myhost.com/live/streamd1001.m3u8
```

我们推荐以 `http` 为前缀且以 `.flv` 为后缀的 **http - flv** 地址，该地址的播放具有时延低、秒开效果好且稳定可靠的特点。
播放器选择方面推荐参考如下表格中的指引的方案：

| 所属平台 | 对接文档 | API 概览 | 支持的格式|
|:-------:|:-------:|:-------:|-------|
| iOS App| [接入指引](https://cloud.tencent.com/document/product/454/56597) | [V2TXLivePlayer(iOS)](https://cloud.tencent.com/document/product/454/56044)  | 推荐 FLV |
| Android App | [接入指引](https://cloud.tencent.com/document/product/454/56598) | [V2TXLivePlayer(Android)](https://cloud.tencent.com/document/product/454/56045) | 推荐 FLV |
| Web 浏览器 | [接入指引](https://cloud.tencent.com/document/product/454/7503) | - |  <li/>桌面端 Chrome 浏览器支持 FLV <li/>Mac 端 Safari 和移动端手机浏览器仅支持 HLS |
|微信小程序| [接入指引](https://cloud.tencent.com/document/product/454/34931) | [&lt;live-player&gt; 标签](https://developers.weixin.qq.com/miniprogram/dev/component/live-player.html)| 推荐 FLV |


[](id:step7)
### 步骤7：优化播放延时

开启旁路直播后的 http - flv 地址，由于经过了直播 CDN 的扩散和分发，观看时延肯定要比直接在 TRTC 直播间里的通话时延要高。
按照目前腾讯云的直播 CDN 技术，如果配合 V2TXLivePlayer 播放器，可以达到下表中的延时标准：

| 旁路流类型 | V2TXLivePlayer 的播放模式 |  平均延时 |  实测效果 |
|:-------:|:-------:|:--------:|:---------:|
| 独立画面 | 极速模式（推荐） | **2s - 3s** | 下图中左侧对比图（橙色）|
| 混合画面 | 极速模式（推荐） | **4s - 5s** | 下图中右侧对比图（蓝色）|

下图中的实测效果，采用了同样的一组手机，左侧 iPhone 6s 使用了 TRTC SDK 进行直播，右侧的小米6 使用 V2TXLivePlayer 播放器播放 FLV 协议的直播流。
![](https://main.qcloudimg.com/raw/98cf3ebc48875d831ef0bd138f7a3cb5.jpg)

如果您在实测中延时比上表中的更大，可以按照如下指引优化延时：

- **使用 TRTC SDK 自带的 V2TXLivePlayer**
普通的 ijkplayer 或者 ffmpeg 基于 ffmpeg 的内核包装出的播放器，缺乏延时调控的能力，如果使用该类播放器播放上述直播流地址，时延一般不可控。V2TXLivePlayer 有一个自研的播放引擎，具备延时调控的能力。

- **设置 V2TXLivePlayer 的播放模式为极速模式**
可以通过设置 V2TXLivePlayer 的参数来实现极速模式，以 [iOS](https://cloud.tencent.com/document/product/454/56597#Delay) 为例：
```ObjectiveC
//自动模式
[_txLivePlayer setCacheParams:1 maxTime:5];
//极速模式
[_txLivePlayer setCacheParams:1 maxTime:1];
//流畅模式
[_txLivePlayer setCacheParams:5 maxTime:5];

//设置完成之后再启动播放
```

[](id:expense)
## 相关费用
使用 CDN 直播观看需要云直播服务资源和终端 SDK 直播播放能力的配合，可能会产生以下费用。

### 云服务消耗

CDN 直播观看需要使用**云直播**的资源进行直播分发。云直播的费用主要包括**基础服务费用和增值服务费用**：基础服务主要是直播推流/播放产生的消耗；增值服务是直播过程中，使用增值服务产生的消耗。

>! 本文中的价格为示例，仅供参考。最终价格与计费策略请以 [云直播](https://cloud.tencent.com/document/product/267/2818) 的计费说明为准。

- **基础服务费用：**
将 TRTC 的内容旁路到并云直播 CDN 观看时，**云直播**会收取观众观看产生的下行流量/带宽费用，可以根据实际需要选择适合自己的计费方式，默认采用流量计费，详情请参见 [云直播 > 标准直播 > 流量带宽](https://cloud.tencent.com/document/product/267/34175#.E6.B5.81.E9.87.8F.E5.B8.A6.E5.AE.BD) 计费说明。

- **增值服务费用：**
如果您使用了云直播的转码、录制、云导播等功能，会产生对应额外的增值服务费用。增值服务可按需使用进行付费。

### SDK 播放授权
音视频通话（TRTC）SDK 提供了功能全面性能强大的直播播放能力，可轻松配合云直播实现 CDN 直播观看功能。移动端 SDK 在10.1及以上的版本可通过获取指定 License 以解锁直播播放能力。
>! TRTC 的播放能力无需 License 授权。
>
您可直接 [购买视频播放 License](https://cloud.tencent.com/document/product/881/74588)，或通过 [购买的云点播流量包](https://buy.cloud.tencent.com/vod) 免费获赠视频播放 License 或 短视频 License，两种 License 均可用于解锁 SDK 的视频播放功能。并且点播资源包可以抵扣云点播的播放产生的日结流量，详细说明请参见 [云点播预付费资源包](https://cloud.tencent.com/document/product/266/14667)。
 License 计费说明参见 [腾讯云视立方License](https://cloud.tencent.com/document/product/1449/56972#.E8.85.BE.E8.AE.AF.E4.BA.91.E8.A7.86.E7.AB.8B.E6.96.B9-license)，License 购买完成后可参考[License操作指引](https://cloud.tencent.com/document/product/1449/56981)进行新增和续期等操作。


## 常见问题
**为什么房间里只有一个人时画面又卡又模糊?**
请将 `enterRoom` 中 TRTCAppScene 参数指定为 **TRTCAppSceneLIVE**。
VideoCall 模式针对视频通话做了优化，所以在房间中只有一个用户时，画面会保持较低的码率和帧率以节省用户的网络流量，看起来会感觉又卡又模糊。
