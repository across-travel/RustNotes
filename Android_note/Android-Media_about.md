---
title: Android 多媒体相关
date: 2017-12-20 20:59:14
category: Android_note
---


**注意，如果手动创建MediaFormat对象的话，一定要记得设置"csd-0"和"csd-1"这两个参数：**

"csd-0"和"csd-1"这两个参数一定要和接收到的帧对应上。
```java
    public static final String MIME_TYPE = "video/avc";
    public static final String CSD0 = "csd-0";
    public static final String CSD1 = "csd-1";
    private static final byte[] SPS = { (byte)0x00, (byte)0x00, (byte)0x01, (byte)0x67 /* 基于协议 */};
    private static final byte[] PPS = { (byte)0x00, (byte)0x00, (byte)0x01, (byte)0x68 /* 基于协议 */};
    private MediaCodec mediaCodec;

    /**
    * 实例化MediaCodec
    */
    public void initMediaCodec() {
        mediaCodec = MediaCodec.createDecoderByType(MIME_TYPE);
        MediaFormat mediaFormat = MediaFormat.createVideoFormat(
                MIME_TYPE, width, height);
        mediaFormat.setByteBuffer(CSD0, ByteBuffer.wrap(SPS));
        mediaFormat.setByteBuffer(CSD1, ByteBuffer.wrap(PPS));
        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT,
                MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Flexible);
        mediaCodec.configure(mediaFormat, null, null, 0);
        mediaCodec.start();
    }
```
"csd-0"和"csd-1"是什么，对于H264视频的话，它对应的是sps和pps，对于AAC音频的话，对应的是ADTS，做音视频开发的人应该都知道，它一般存在于编码器生成的IDR帧之中。

得到的mediaFormat
```
mediaFormat in:{height=720, width=1280, csd-1=java.nio.ByteArrayBuffer[position=0,limit=7,capacity=7], mime=video/avc, csd-0=java.nio.ByteArrayBuffer[position=0,limit=13,capacity=13], color-format=2135033992}
```

```java
    private static void dumpFile(String fileName, byte[] data) {
        FileOutputStream outStream;
        try {
            outStream = new FileOutputStream(fileName);
        } catch (IOException ioe) {
            throw new RuntimeException("Unable to create output file " + fileName, ioe);
        }
        try {
            outStream.write(data);
            outStream.close();
        } catch (IOException ioe) {
            throw new RuntimeException("failed writing data to file " + fileName, ioe);
        }
    }

    private void compressToJpeg(String fileName, Image image) {
        FileOutputStream outStream;
        try {
            outStream = new FileOutputStream(fileName);
        } catch (IOException ioe) {
            throw new RuntimeException("Unable to create output file " + fileName, ioe);
        }
        Rect rect = image.getCropRect();
        YuvImage yuvImage = new YuvImage(getDataFromImage(image, COLOR_FormatNV21), ImageFormat.NV21, rect.width(), rect.height(), null);
        yuvImage.compressToJpeg(rect, 100, outStream);
    }
```

## 参考资料

* [Android: MediaCodec视频文件硬件解码,高效率得到YUV格式帧,快速保存JPEG图片(不使用OpenGL)(附Demo)](https://www.polarxiong.com/archives/Android-MediaCodec%E8%A7%86%E9%A2%91%E6%96%87%E4%BB%B6%E7%A1%AC%E4%BB%B6%E8%A7%A3%E7%A0%81-%E9%AB%98%E6%95%88%E7%8E%87%E5%BE%97%E5%88%B0YUV%E6%A0%BC%E5%BC%8F%E5%B8%A7-%E5%BF%AB%E9%80%9F%E4%BF%9D%E5%AD%98JPEG%E5%9B%BE%E7%89%87-%E4%B8%8D%E4%BD%BF%E7%94%A8OpenGL.html)
* [Android MediaCodec stuff](http://bigflake.com/mediacodec/)
* [雷霄骅(leixiaohua1020)的专栏](http://blog.csdn.net/leixiaohua1020/)
* [[总结]FFMPEG视音频编解码零基础学习方法 - 雷霄骅](http://blog.csdn.net/leixiaohua1020/article/details/15811977)
* [Bilibili/ijkplayer - Github](https://github.com/Bilibili/ijkplayer)
