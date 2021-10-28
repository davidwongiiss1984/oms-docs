![olio](/images/olio_log.png "logo")

# OMS: Open Media Streaming

OMS面向直播、点播、短视频编辑、实时音视频、音视频特效等音视频及通讯领域相关产品，提供开放的一站式前后端整合开发框架。

使用此框架，开发人员可以专注于音视频前置和后置处理，灵活的组织场景，定制新玩法。并针对特定场景的网络传输和编解码进行优化。

OMS又称为Olio(什锦菜) Media Streaming，基于时间线的抽象设计，实现了相关产品中绝大部分场景的功能和应用。您可以在此基础上进行二次封装，以实现直播、点播、实时音视频，播放器等业务化SDK接口。

她开源而且免费，她的口号是-"SHARE MORE, WASTE LESS!"

## 服务对象

QMS主要面向中小企业和初创团队，QMS开发团队可以在以下方面提供技术支持和服务

- 视音频特效定制化开发，以及第三方相关SDK接入
- 针对特殊场景的编解码优化
- 针对特殊场景的网络传输层优化
- 视频云主机前后端部署和接入

## 相关概念

### 时间线相关

![类图](/images/uml_class_assets.png "类图")

|概念 | 含义    |
|-----|---------| 
|Asset| 媒体资源，本地音视频文件，远程音视频流，摄像头麦克风等采集设备，白板，文档以及素材和特效      |
|Stream | Asset中的一个媒体流 |
|Clip | Asset或者Stream的一个时间片段 |
|Track| 容纳Clip的实体，分为音视频轨道，字幕轨，贴图轨等，可以加入Multitrack，也可以作为Clip加入另一个Track   |
|Multitrack| 容纳Track的实体, 包含多个音视频轨道，可以作为Clip加入一个Track，也可以作为Track加入另一个Multitrack    |
|Timeline| Multitrack的根节点    |
|Filter| 音视频滤镜和特效        |
|Transition| 音视频转场，片段间切换时，从一个音视频到另一个音视频的播放衔接过渡效果 |
|Template| 时间线模板，模板可以内嵌多个场景 |

### 时间线管理相关

|概念 | 含义    |
|-----|---------| 
|Scene| 子Timeline，直播，点播，实时音视频中的一个场景，可用于背景，多机位切换等 |
|Project| 直播，点播，实时音视频，短视频编辑生产的项目类，表示一个秀场，一个会议，一个短视频编辑工程，Project管理多个场景/模板 |
|Workspace| 直播，点播，实时音视频，短视频编辑生产的项目管理类 |

### 运行时相关

![序列图](/images/uml_seq_context.png "序列图")

|概念 | 含义    |
|-----|---------| 
|Context| 运行上下文，管理远程或者本地会话        |
|RenderView| 渲染窗口，绑定跨平台窗口句柄        |
|Player| 播放器，绑定时间线和渲染窗口，进行播控渲染     |
|Exporter| 导出器，绑定时间线或时间线上不同的媒体，进行推流或者本地导出，实现直播，录音，录像等功能  |

## 使用文档

[短视频编辑][AVEDITING]

[直播][LIVESTREAMING]

[点播][VOD]

[实时音视频][RTC]

[播放器][PLAYER]

## 服务端文档

[云剪辑][1]

[合流][2]

[云导播台][3]

## 技术文档

[渲染引擎][1]

[编解码引擎][2]

## 技术支持


[AVEDITING]: (AVEDITING.md)
[LIVESTREAMING]: (LIVESTREAMING.md)
[VOD]: (VOD.md)
[RTC]: (RTC.md)
[PLAYER]: (PLAYER.md)
[UML_CLASS_ASSETS]: (/images/uml_class_assets.png)
[UML_SEQ_CONTEXT]: (/images/uml_seq_context.png)

