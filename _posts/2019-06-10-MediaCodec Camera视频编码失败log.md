
我们的Camera是使用OpenGL开发的滤镜相机，有滤镜、贴纸等功能，测试报了如下问题：

**问题现象**：使用滤镜相机录制视频，结束后视频播放显示全黑，没有画面。

查看mtklog，发现了如下log：
~~~text
05-31 14:43:35.819242 27778 28253 I MPEG4Writer: Received total/0-length ($\0/0{red}{red}$) buffers and encoded 0 frames. - Video
05-31 14:43:35.819443 27778 28252 I MPEG4Writer: Received total/0-length ($\7/0{red}{red}$) buffers and encoded 7 frames. - Audio
~~~

这两行log虽然没有Error标志，但是仍然值得我们注意，正常情况下的MPEG4Wrier的log输出如下：
~~~text
05-31 14:47:06.620068 27778 31527 I MPEG4Writer: Received total/0-length (==149/0==) buffers and encoded 149 frames. - Audio
05-31 14:47:06.620111 27778 31527 I MPEG4Writer: Audio track drift time: 0 us
05-31 14:47:06.620292 27778 31526 I MPEG4Writer: Received total/0-length (==62/0==) buffers and encoded 62 frames. - Video
~~~

第一种异常情况下，Video的编码帧只有0 frame，说明当时没有采集到视频帧数据。
第二种正常情况下，Video和Audio的帧数都比较正常。

项目Camera是使用OpenGL进行渲染的，视频录制部分采用MediaEncoder录制，封装的MediaCodec，参考https://github.com/google/grafika/blob/master/app/src/main/java/com/android/grafika/VideoEncoderCore.java

通过log分析，可以推测当时的复现路径：进入相机-->打开滤镜-->长按录制-->马上停止。
极短时间内，开始录像并停止，受限于机器性能，Camera Capture 数据还没有送到VideoEncoderCore，导致OpenGL没有编码数据，也就没有输出，导致最终录制出来的视频黑屏。

**解决方法**：

1. 添加判断，录像时间小于800ms不保存
1. 在VideoEncoderCore stop录像时，catch exception，停止异常时不保存录像。

**MARK**：
解决编码异常问题时，多查看MPEG4Writer输出的log，查看编码的帧是否有异常。
