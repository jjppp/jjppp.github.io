---
title: Jane Street 挂经
date: 2026-05-19
permalink: /posts/2026/05/Jane-Street-挂经/
tags:
  - Eureka Moments
---

没想过能给onsite，也没想过挂得这么莫名其妙

## recruiter call（第一轮）
投递一个月之后收到 recruiter 邮件约视频会议。简单问了问毕业时间、目标岗位就马上约了下一轮

## technical（第二轮）
一个面试官看着你写代码。题目大概是这样的：
维护一个list of items，每个item有一个长度。要求支持下面两个操作：
在某个 index 插入一个 item
查询某个 index 处 items 的前缀和
写完之后问了一个follow-up：
要求支持历史版本的查询

## onsite（第三轮）
第二轮后马上就约了 onsite，还包了机酒，四舍五入送一次HK旅游
面试在 js 的大楼里，一共三次，每轮两个面试官看着你写代码，风格仍然类似第二轮。每个人分配一个小隔间，所有的面试都在小隔间里

### 第一次：
给一个 list[dict[int, float]]，要求支持如下操作：

1. 对于给定的下标 index，如果存在某个dict把index 映射到了某个float，就返回那个float
2. 如果存在多个，取list中的第一个符合条件的map
3. 如果不存在，令index = index - 1继续找

follow-up：
之前如果index不存在就向前找（如果想象成在坐标轴上画图，就是向左画横线），现在希望根据前一个和后一个的值做插值

面试官要求查询尽可能快（常数时间）。corner case比较多，而且不断会被challenge代码中哪一行有问题

### 第二次：
给两个类，实现第三个类中类似直播视频播放的功能
```python
class VideoPlayer:
  def add_to_buffer(self, seg: Segment)

class HttpClient:
  def get_manifest(self) -> dict[int, list[str]]
  def start_download(self, url: str) -> Download
  def is_completed(self, download: Download) -> bool
  def to_segment(self, download: Download) -> Segment

class StreamPlayer:
  def tick(self, ...)
```
1. tick函数会被定时调用
2. 需要在最开始的时候获取manifest，找到最新的segment url
3. 如果有就开始下载
4. 下载完了就丢给video_player

我写了一个类似状态机的东西

follow-up：
manifest里包含不同quality的资源。假如给了几个可以估计剩余下载时间和buffer里还能播多久的API，怎么优化视频质量和播放质量

这部分感觉很无语。我问estimate准不准，得到的答复是可能会突变。我心想那还estimate个啥？口糊一通分类讨论贪心就时间结束了

最后吃了个午餐。我找了个座位坐下才被告知candidate只能回自己的小房间里吃，好吧。
吃完有个人过来说我挂了，就让我走了。没有面试反馈。
