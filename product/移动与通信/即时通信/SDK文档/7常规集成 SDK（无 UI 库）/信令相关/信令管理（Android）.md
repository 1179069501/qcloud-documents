## 概述
信令接口是基于 IM 消息提供的一套邀请流程控制的接口，可以实现多种实时场景，例如：
- 直播聊天室中进行上麦、下麦管理。
- 聊天场景中实现类似微信中的音视频通话功能。
- 教育场景中老师邀请同学们举手、发言的流程控制。

## 功能 
信令接口支持以下功能：
### 单聊邀请
在使用 [简单收发消息接口](https://cloud.tencent.com/document/product/269/44489#.E6.94.B6.E5.8F.91.E7.AE.80.E5.8D.95.E6.B6.88.E6.81.AF) 或 [富媒体消息接口](https://cloud.tencent.com/document/product/269/44489#.E6.94.B6.E5.8F.91.E5.AF.8C.E5.AA.92.E4.BD.93.E6.B6.88.E6.81.AF) 进行单聊的同时，可以使用 [invite](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a61e283c64f1a303b0a9f0e13a48de0a9) 信令接口进行点对点呼叫，对方收到邀请通知 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 后可以选择接受、拒绝或等待超时。
### 群聊邀请
首先需通过 [建群](https://cloud.tencent.com/document/product/269/44494#.E5.88.9B.E5.BB.BA.E7.BE.A4.E7.BB.84)、[加群](https://cloud.tencent.com/document/product/269/44494#.E5.8A.A0.E5.85.A5.E7.BE.A4.E7.BB.84)、[退群](https://cloud.tencent.com/document/product/269/44494#.E9.80.80.E5.87.BA.E7.BE.A4.E7.BB.84)、[解散群](https://cloud.tencent.com/document/product/269/44494#.E8.A7.A3.E6.95.A3.E7.BE.A4.E7.BB.84)以及[群资料](https://cloud.tencent.com/document/product/269/44494#.E7.BE.A4.E8.B5.84.E6.96.99.E5.92.8C.E7.BE.A4.E8.AE.BE.E7.BD.AE) 和 [群成员](https://cloud.tencent.com/document/product/269/44494#.E7.BE.A4.E6.88.90.E5.91.98.E7.AE.A1.E7.90.86) 相关接口完成对群组的管理，并监听群内的相关事件回调 [V2TIMGroupListener](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMGroupListener.html)。然后群成员可以在群内发起群呼叫邀请 [inviteInGroup](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#ad51059e9f430650da09bcae01f0bb3b8)，被邀请的群成员会收到邀请通知 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 后可以选择接受、拒绝或等待超时。
### 取消邀请
邀请者可以在超时前且被邀请者未处理前取消邀请 [cancel](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a9d69707620f038d6e47356cdaa3ab9bd)。被邀请者会收到取消通知 [onInvitationCancelled](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#adbdb9fe903e032b94a82330649484642)，该邀请流程结束。

![取消邀请](https://sdk-im-1252463788.cos.ap-hongkong.myqcloud.com/tools/resource/signaling/01_signaling_c2c_cancel.png)

### 接受邀请
被邀请者收到邀请通知 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 后可以在超时前且邀请者取消前接受邀请 [accept](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a4cd3629a0952db7c59186e0c222e17a0)，邀请者会收到接受邀请通知 [onInviteeAccepted](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#af4896215b6bf6febda701c100566b04c)，所有被邀请者处理完后（包括接受、拒绝、超时）该邀请流程结束。

![接受/拒绝邀请](https://sdk-im-1252463788.cos.ap-hongkong.myqcloud.com/tools/resource/signaling/02_signaling_c2c_accept_reject.png)

### 拒绝邀请
被邀请者收到邀请通知 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014)后可以在超时前且邀请者取消前拒绝邀请 [reject](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#ad9510bf8a333189fd1a0c1eafbde2266)，邀请者会收到拒绝邀请通知 [onInviteeRejected](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#ad351469b5f2f3b36833ae9832ed80d27)，所有被邀请者处理完后（包括接受、拒绝、超时）该邀请流程结束。
### 邀请超时
若邀请接口的超时时间大于0，且被邀请者未在超时时间之内响应则邀请超时，邀请者和被邀请者都会收到超时通知 [onInvitationTimeout](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#ac5ee85faf06f5deb359afdf6d88d43f5)，所有被邀请者处理完后（包括接受、拒绝、超时）该邀请流程结束。若邀请接口的超时时间等于0，则不会有超时通知。

![邀请超时](https://sdk-im-1252463788.cos.ap-hongkong.myqcloud.com/tools/resource/signaling/03_signaling_c2c_timeout.png)

## 应用场景案例
### 音视频通话
在开源项目 [TUIKit Demo](https://github.com/tencentyun/TIMSDK) 中，我们基于 [TRTC 组件](https://cloud.tencent.com/document/product/647/42045) 并对其稍作修改提供了一个适合聊天场景的1v1和多人音视频通话的方案，您可以直接基于我们提供的 Demo 进行修改适配。我们以1v1视频通话为例介绍下信令接口跟 TRTC SDK 的结合使用。

**1v1视频通话的流程：**
1. 邀请者根据业务层生成的 roomID 进入该 TRTC 房间，同时调用信令邀请接口 [invite](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a61e283c64f1a303b0a9f0e13a48de0a9) 发起音视频通话请求，并把 roomID 放到邀请接口的自定义字段中。
2. 被邀请者收到信令邀请通知 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014)，并通过自定义数据拿到 roomID，界面开始响铃。
3. 被邀请者处理邀请通知：
 - 接受邀请需调用信令 [accept](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a4cd3629a0952db7c59186e0c222e17a0) 接口，并根据 roomID 进入到 TRTC 房间，并同时调用 openCamera() 函数打开自己本地的摄像头，双方收到 TRTC SDK 的 `onRemoteUserEnterRoom` 回调后记录本次通话的开始时间。
 - 拒绝邀请需调用信令 [reject](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#ad9510bf8a333189fd1a0c1eafbde2266) 接口结束本次通话。
 - 如果被邀请者正在跟其他人通话，则调用信令 [reject](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#ad9510bf8a333189fd1a0c1eafbde2266) 接口拒绝本次邀请，并在自定义数据中告诉对方是由于本地线路忙而拒绝。
4. 接听并当双方的音视频通道建立完成后，通话的双方都会接收到 TRTC SDK 的 `onUserVideoAvailable` 的事件通知，表示对方的视频画面已经拿到。此时双方用户均可以调用 TRTC SDK 接口 `startRemoteView` 展示远端的视频画面。远端的声音默认是自动播放的。
5. 通话结束即某一方挂断电话，该用户退出 TRTC 房间。对方收到 TRTC SDK 的 `onRemoteUserLeaveRoom` 回调后计算通话总时长并再次发起一次邀请，此邀请的自定义数据中标明是结束通话并附带通话时长，方便 UI 界面做展示。

**时序图**
![](https://sdk-im-1252463788.cos.ap-hongkong.myqcloud.com/tools/resource/signaling/04_time_squence.png)

### 教育场景中老师邀请学生举手发言
该场景为老师先让同学们举手，再从举手的同学中选一个同学进行发言。详细流程如下：
1. 老师调用 [inviteInGroup](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#ad51059e9f430650da09bcae01f0bb3b8) 接口邀请同学们举手，自定义 `data` 中填入“举手操作”，同学们收到 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 回调。
2. 同学们根据 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 中的 `inviteeList` 和 `data` 字段判断被邀请者里有自己且是举手操作，那么调用  [accept](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a4cd3629a0952db7c59186e0c222e17a0) 接口举手。
3. 如果有学生举手，所有人都可以收到 [onInviteeAccepted](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#af4896215b6bf6febda701c100566b04c) 回调，判断 `data` 中的字段为“举手操作”，展示举手学生列表。
4. 老师从举手成员列表中邀请某个同学进行发言，调用  [inviteInGroup](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#ad51059e9f430650da09bcae01f0bb3b8) 接口，此时自定义 `data` 中填入“发言操作”，学生们都收到 [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 回调。
5. 学生根据  [onReceiveNewInvitation](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#aecc2341ca87eb58be37fdadf7a58c014) 回调中的 `inviteeList` 和 `data` 字段判断被邀请者里有自己且是发言操作，则调用 [accept](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html#a4cd3629a0952db7c59186e0c222e17a0) 接口发言。
6. 如果有学生发言，所有人都可以收到  [onInviteeAccepted](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingListener.html#af4896215b6bf6febda701c100566b04c) 回调，判断 `data` 中的字段为“发言操作”，展示发言成员列表。

## 交流&反馈

欢迎加入 QQ 群进行技术交流和反馈问题，QQ 群：**592465424**。

![img](https://qcloudimg.tencent-cloud.cn/raw/ca5f8724cd5a9002abc454f80bf3df12.png)



## 常见问题

### 1. 用户 A 邀请用户 B 时，用户 C 可以邀请用户 B 吗？
SDK 提供的信令接口（ [iOS](https://im.sdk.qcloud.com/doc/zh-cn/categoryV2TIMManager_07Signaling_08.html) | [Android](https://im.sdk.qcloud.com/doc/zh-cn/classcom_1_1tencent_1_1imsdk_1_1v2_1_1V2TIMSignalingManager.html) ）本身不限制邀请的逻辑，一个用户可以同时收到多个邀请信令。对于音视频通话场景，我们的 [TUICalling](https://cloud.tencent.com/document/product/647/70641) 组件做了“忙线”提醒。

### 2. 发送信令邀请时，是否可以同时发送多个 Invite 请求？
可以，上层根据实际业务需求加以区分。

### 3. IM SDK 的信令接口只有邀请、同意/拒绝、取消，如何实现挂断操作？
* 邀请操作，上层语义可以理解成**请求建连**。
* 挂断操作，上层语义可以理解成**请求挂断**。

可以使用 IM SDK 的**邀请**接口，结合自定义 data 来表示当前的邀请是**请求建连**还是**请求挂断**，由 IM 透传给对端处理。可以参见 TUICalling （ [iOS](https://github.com/TencentCloud/TIMSDK/blob/master/iOS/TUIKit/TUICalling/Source/Model/Impl/TRTCCalling%2BSignal.m) | [Android](https://github.com/TencentCloud/TIMSDK/blob/master/Android/TUIKit/TUICalling/tuicalling/src/main/java/com/tencent/liteav/trtccalling/model/TRTCCalling.java) ）组件的挂断逻辑。

### 4. 发送信令邀请时，对于信令邀请超时的处理逻辑是怎么样的？
* 当邀请发送方和接收方都在线时，超时信令由接收方触发，且发送方和接收方都会收到 `onInvitationTimeout` 回调。
* 当接收方不在线时，超时信令由发送发触发，发送方会收到 `onInvitationTimeout` 回调。
* 超时信令均由 IM SDK 发出。

### 5. 离线再上线，会收到未超时的信令消息吗？
App 冷启动（杀进程后再次点击 App 图标启动）时，根据聊天类型，有两种情况：

* 如果是单聊，IM SDK 会自动同步所有信令消息。如果信令未超时，则会回调 `onReceiveNewInvitation`。
* 如果是群聊，IMSDK 会自动同步最近 30 秒的消息，如果包含了未超时的信令消息，则会回调 `onReceiveNewInvitation`。
App 热启动（App 在后台，点击 App 图标启动）时，不管是单聊还是群聊，都会同步所有的未超时信令消息，并回调 `onReceiveNewInvitation`。

### 6. 信令回调中 inviteID 是不是唯一的？
是唯一的。inviteID 唯一标识了一组信令消息（包括邀请、同意/拒绝、超时）。

