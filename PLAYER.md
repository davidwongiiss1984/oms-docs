# 播放器

## 立即播放


```C++
Asset movie("rtmp://a.mp4");

RenderView view = get_render_view();

// 创建播放上下文
Context ctx;

Player player(ctx);

// 播放缺省视频流和音频流
player.connect(movie);
player.add_render_view(view);

// 绑定事件回调
player.on(...);
player.play();

```

## 多轨支持

```C++
Multitrack mt;
mt.set_uri("rtmp://a.mp4");

// 构建媒体中包含的视频流，音频流，字幕流等轨道
mt.build();

player.connect(mt);
player.add_render_view(view);

// 绑定事件回调
player.on(...);
player.play();

```

## 自定义播放

```C++
Asset movie("rtmp://a.mp4");

// 计算流数
movie.count_streams();
Stream s0 = movie.stream(0);
Stream s1 = movie.stream(1);
// maybe a subtitle track
Stream s3 = movie.stream(2);
// 加字幕特效
s3.add_filter(Filter("text"));

// 建轨
Track t, t1, t2;
t.add(s0);
t1.add(s1);
t2.add(s2);

Multitrack mt;
mt.add(t);
mt.add(t1);
mt.add(t2);

//...
player.connect(mt);
//...
```