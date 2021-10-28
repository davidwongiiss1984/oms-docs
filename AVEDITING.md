# 短视频编辑

## 创建时间线

```C++
Multitrack multitrack;

// 加入视频轨
Track video_track = multitrack.add_track(Track.Type.Video, "V0");

// 加入10s视频片段1
Asset video("rtmp://xxxxx");	// 拉10秒流
Clip video_clip(video, AVTime::Zero, AVTime::from_seconds(10, 600));
video_track.add(video_clip);

// 加入10s视频片段2
Asset video1("/video/demo.mp4");	// 播放10视频
Clip video_clip1(video1, AVTime::Zero, AVTime::from_seconds(10, 600));
video_track.add(video_clip1);
EXPECT_EQ(video_clip1.start_time_on_timeline(), AVTime::from_seconds(10, 600));

EXPECT_EQ(video_track.count_children(), 2);

// 加入音频
Track audio_track = multitrack.add_track(Track.Type.Audio, "A0");

Asset audio("/sdcard/demo.mp3");
Clip audio_clip(audio, AVTime::Zero, video_track.duration());
audio_track.add(audio_clip);
```

## 时间线分组嵌套
---------------

为了支持以下常见场景

- 片头片尾
- 剪同款
- 直播、实时音视频场景切换
- 实时音视频会话管理

时间线支持以下分组嵌套：

- Track可作为Clip加入到Track中
- Multitrack可作为Clip加入到Track中
- Multitrack可作为Track加入到Multitrack中

```C++
// ...构建scene1, scene2，不同场景可以加入不同采集设备或者机位...
Multitrack scene1, secne2;

Multitrack mt,
mt.add(scene1);
mt.add(scene2);
EXPECT_EQ(mt.is_root());
EXPECT_TRUE(scene1.parent(), mt);
EXPECT_TRUE(scene2.parent(), mt);

// ...构建直播轨，包含音视频采集轨，实时连麦轨，切房轨...
Multitrack live;
mt.add(live);
EXPECT_TRUE(mt.count_children(), 3);

scene1.set_visible(true);
scene2.set_visible(false);

// 场景切换，淡入淡出5秒
Transition ts("fade", AVTime::from_seeonds(5, 600));
ts.connect(scene1, scene2);
scene1.set_visible(false);
scene2.set_visible(true);

// 多机位分屏同时显示
scene1.set_visible(true);
scene1.set_rect(...);
scene2.set_rect(...);
```

## 滤镜和特效

```C++
Filter f("blur");

EXPECT_EQ(f.type(), Filter.Type.Builtin); // 内置滤镜
EXPECT_EQ(f.target(), Filter.Target.Video);  // 视频滤镜

// 视频片段加滤镜
video_clip.add_filter(f);
f.set_float("vert", 2.0);

EXPECT_EQ(video_clip.count_filters(), 1);

Filter clone_f2 = video_track.add_filter(f);  // 轨道加滤镜
Filter clone_f3 = multitrack.add_filter(f); // 多轨加滤镜

// 音频滤镜
Filter f("volume");
EXPECT_EQ(f.target(), Filter.Target.Audio);
f.set_float("value", 2.0);

// 调速，控制滤镜
Filter f("speed");
EXPECT_EQ(f.target(), Filter.Target.Control);
f.set_data(...);
video_clip.add_filter(f);
```

## 转场

```C++

// 时间线时长20s
AVTime t = video_track.duration();
EXPECT_EQ(t, AVTime.from_seconds(20, 600));

// 加5s转场
Transition ts("cube", AVTime::from_seconds(5, 600));
ts.connect(video_clip, video_clip1);

// 转场覆盖模式
if (ts.overlapped_mode() == Transition.Mode.Overlapped) {
  // 吃时长
  EXPECT_EQ(video_track.duration(), t - ts.duration());
}
else (t.overlapped_mode() == Transition.Mode.None) {
  // 不吃时长(运镜转场)
  EXPECT_EQ(video_track.duration(), t);
}

// 转场是否作为可视化clip，默认为false
EXPECT_FALSE(video_track.transition_as_visual_clip());

// 设置转场为可视化clip
video_track.set_transition_as_visual_clip(true);
EXPECT_EQ(video_track.count_children(), 3);

// 删除转场
video_track.remove(1);
EXPECT_EQ(video_track.count_children(), 2);

// 使用clip索引重新加入转场
video_track.insert(1, ts);
EXPECT_EQ(ts.outgoing(), video_track.child(0));
EXPECT_EQ(ts.incoming(), video_track.child(2));
EXPECT_EQ(video_track.count_children(), 3);

// 转场加滤镜
Filter f2 = ts.add_filter(new Filter("brightness"));
f2.set_float("value", 0.25);

// 转场连接轨道
Transition track_ts("cube");
track_ts.connect(video_track, video_track1);
```

## 预览
```C++
// 运行上下文
Context ctx;
Player pl(ctx);

// 绑定平台窗口
RenderView view = get_render_vidw(...);
pl.add_render_view(view);

// 绑定时间线
pl.connect(multitrack);
// 绑定事件监听
pl.on(...)
pl.play();
```

## 导出

```C++
// 推流
Exporter live(ctx);

live.set_uri("rtmp://xxxx");
live.set_profile(...); // 设置导出参数
live.connect(multitrack); // 绑定时间线
live.on(...); // 绑定事件监听
live.start();

// 本地导出
Exporter record(ctx);

record.set_uri("/local.mp4");
record.set_profile(...); // 设置导出参数
record.connect(multitrack); // 绑定时间线
record.on(...); // 绑定事件监听
record.start();

// 仅录音
Exporter record1(ctx);
record.connect(audio_track); // 仅绑定时间线上的音轨
record1.on(...); // 绑定事件监听
record1.start();

// 运行环境中存在三个导出会话
EXPECT_EQ(ctx.countExporters(), 3);

// 可以分别进行操作
live.pause();
```

## 模板

时间线支持模板导出，可以制作剪同款或者场景复用

```C++
// 默认不可替换
EXPECT_EQ(video_track.child(0).replace_holder(), "");

video_track.child(0).set_replace_holder("{0}");
video_track.child(1).set_replace_holder("{1}");

// 导出模板
Template templ(uri); // 远程或者本地uri
templ.connect(multitrack);
templ.save();
templ.close();

// 加载模板
templ.load();
EXPECT_EQ(templ.count_placeholders(), 2);
// 使用新素材替换0
templ.replace("{0}", "rtmp://xxx");
// 使用新轨道替换1
Mulitrack mt2;
templ.replace("{1}", mt2);

// 构建时间线
Multitrack mt = templ.build();
```

## 操作指令
---------------

操作指令目的是解决


