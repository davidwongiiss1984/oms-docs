# 直播

直播与短视频编辑没有区别，除了导出时进行直播推流外，在时间线上有摄像头和音频采集设备。

基于时间线可以实时更改滤镜和特效，加入跨房直播流，以及建立本地或者云导播台

## 建立直播间

以下采用代码的方式建立直播间，也可以从已经导出的模板加载。

```C++
// 查询本地或者远端设备
DeviceManager dm;
dm.query();

// 假设有2个摄像头（移动端前后置，或者实现多机位）
Camera camera1 = dm.camera(0);
Camera camera2 = dm.camera(1);

// 构建多场景轨道，可以实现多场景画中画功能
// 如果不同场景建立在相同轨道上，按播控时间进行场景切换
// 场景切换时，可以加转场
Track scene1, scene2;
scene1.add(camera1);
scene2.add(camera2);
scene1.set_visible(true);
scene2.set_visible(true);
// 画中画，右上角，根据渲染尺寸，按画中画视频中心点，自动计算画中画位置和缩放比例
scene2.set_margins("top:10%,left:70%,right:10%");

// 加入绿幕抠图
camera2.add_filter(Filter("green_matte"));

// 加入背景视频轨
Track bg_video;
bg_video.add(Asset("/bgm_video.mp4"));
// 直播暂停，再次播放时，从暂停点继续播放
bg_video.child(0).set_resume_mode(Player::ResumeMode::Continue);
// 播放完自动重复
bg_video.child(0).set_loop_mode(Player::LoopMode::Repeat);

// 加入BGM轨
Track bgm;
bgm.add(Asset("/bgm.mp3"));
// 直播暂停或者静音BGM，再次播放时，从暂停点继续播放
bgm.child(0).set_resume_mode(Player::ResumeMode::Continue);
// 播放完自动重复
bgm.child(0).set_loop_mode(Player::LoopMode::Repeat);

Timeline timeline;
timeline.add(bgm);
timeline.add(scene1);
timeline.add(scene2);
```

## 直播预览

```C++
// 创建环境上下文
Context ctx = new Context();

// 准备播放器
Player player = new Player(ctx);

// 播放器绑定渲染窗口
RenderView rv = get_render_view();
player.add_render_view(rv);

// 连接时间线
player.connect(timeline);
player.play();
```

## 导出
```C++
// 推流
Exporter exporter = new Exporter(ctx);
// 连接时间线
exporter.connext(timeline);
// 设置导出参数
exportor.setProfile(...);
// 设置导出地址，本地或者推流
exporter.setUri("");
exporter.start();

// 录像
Exporter record = new Exporter(ctx);
// 连接视频采集流
record.connext(scene1.child(0));
// 设置导出参数
record.setProfile(...);
// 设置导出地址，本地或者推流
record.set_uri("/record.mkv");
record.start();

int n = ctx.countExporters()
EXPECt_EQ(n, 2);
```

# 跨房直播
请参考点播


# 本地/云导播台



# 更多请参考

[点播]()

[实时音视频]()

[播放器]()

