# 点播

介绍直播场景下的点播，实时连麦以及多房间并发观看

## 点播

```C++
// 准备运行上下文
LiveStreamingContext ctx;

// 登录秀场或者房间
LiveStreamingProject project(ctx);
project.set_uri(roomId);
project.login(...);
User usr = project.me();
EXCEPT_EQ(usr.role(), User::Role::Audience);

// 时间线绑定project
Timeline timeline;
timeline.connect(project);

// 构建player....
Player player(ctx);
RenderView view = get_render_view();

player.add_render_view(view);
player.connect(timeline);
player.on(...);
player.play();

// 获取连接用户数
int n = project.count_users();

// 退出房间
project.logout();

// player自动暂停，时间线自动清空
EXPECT_EQ(player.stopped());
EXPECT_EQ(timeline.empty());
```

## 实时连麦

```C++
// 获取观众轨，用于观众连麦推流
User usr = project.me();
EXCEPT_EQ(usr.role(), User::Role::Audience);
Multitrack me_mt = timeline.child(user.id());

// 视频采集轨
Track vt = me_mt.child(0);
// 请求视频连接
vt.set_visible(true);

// 音频采集轨
Track at = me_mt.child(1);
// 请求连麦连接
at.set_visible(true);
at.set_mute(false);

// 主播轨
Multitrack anchor_mt = timeline.child(project.uri());
```

## 多房间并发观看

### 共享时间线模式

```C++
// 准备运行上下文
LiveStreamingContext ctx;

// 登录房间1
LiveStreamingProject project(ctx);

project.set_uri(room_id);
project.login(...);

// 登录房间2
LiveStreamingProject project1(ctx);
project1.set_uri(room_id1);
project1.login(...);

// 连接多个房间
Timeline timeline;
timeline.connect(project);
timeline.connect(project1);

Player player(ctx);
player.connect(timeline);

// 切换房间1
Multitrack mt1 = timeline.child(project.uri());
Multitrack mt2 = timeline.child(project1.uri());

mt1.set_visible(true);
// 不可见时，自动静音并且停止拉流
mt2.set_visible(false);

// 切换房间2
mt1.set_visible(false);
mt2.set_visible(true);

// 获取切换之前的时间戳继续进行拉流
AVTime tm = mt2.paused_time();
player.seek(tm);
player.play();

// 本地录制或者再次推流
Exporter exporter(ctx);
exporter.connect(timeline);
exporter.set_uri("/video.mkv");
exporter.start();
```
> 以前后台隐藏的方式切换房间时，连麦前，必须首先至流结尾
```C++
// 连麦前，跳至当前同步时间戳进行拉流
player.seek(-1);
EXPECT_EQ(player.current_time(), ctx.current_time());

// 获取观众轨，用于观众连麦推流
User usr = project.me();
EXCEPT_EQ(usr.role(), User::Role::Audience);
Multitrack me_mt = timeline.child(user.id());
```

> 可以多房间以画中画方式同时显示
```C++
// 设置布局
mt1.set_rect(...);
mt2.set_rect(...);

// 设置静音，但是依旧拉流(语音通信时停止拉流，参见实时音视频)
mt1.set_mute(false);
mt1.set_mute(true);
```
> 布局，请参考渲染引擎位置滤镜

### 多上下文模式

Context支持多实例，每个实例管理并运行独立的播放器，内部渲染和编解码引擎。

该模式适合批量观看或者监控，并且调用者可以独立管理各对象和窗口布局。

> 该模式无法进行合流，更多请参看直播"导播台"

```C++
LiveStreamingContext* ctx;
LiveStreamingProject* project;
Timeline* timeline;
Player* player;

for (int i = 0; i < N; ++i) {
  // 准备运行上下文
  ctx = new LiveStreamingContext();

  // 登录房间
  project = new LiveStreamingProject(ctx);

  project->set_uri(roomId);
  project->login(...);

  // 自动构建时间线
  timeline->connect(project);

  // 构建播放器
  player = Player(ctx);
  player->add_render_view(get_render_view());
  player->connect(timeline);
  player->play();
}

int n = LiveStreamingContext::count();

// 切换当前context，其余context自动静音，但是依旧拉流
LiveStreamingContext::instance(0).activate();

```
