---
title: Android 编解码 MediaCodec, Image
date: 2017-12-20 20:59:14
category: Android_note
---


**注意，如果手动创建MediaFormat对象的话，一定要记得设置"csd-0"和"csd-1"这两个参数：**

"csd-0"和"csd-1"这两个参数一定要和接收到的帧对应上。
```java
import android.media.MediaCodec;
import android.media.MediaCodecInfo;
import android.media.MediaFormat;

    private static final int COLOR_FormatI420 = 1; // 自定义的格式类型
    private static final int COLOR_FormatNV21 = 2;

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

    private int TIME_INTERNAL = 30;

    /**
     * 将数据添加到mediaCodec中，有一帧数据就放到识别方法中识别
     *
     * @param datas 添加到mediaCodec中
     */
    private void addDataToMediaC(byte[] datas) {
        ByteBuffer[] inputBuffers = mediaCodec.getInputBuffers();
        int inputBufferIndex = mediaCodec.dequeueInputBuffer(50);
        if (inputBufferIndex >= 0) {
            ByteBuffer inputBuffer = inputBuffers[inputBufferIndex];
            inputBuffer.clear();
            inputBuffer.put(datas, 0, datas.length);
            mediaCodec.queueInputBuffer(inputBufferIndex, 0, datas.length, mCount
                    * TIME_INTERNAL, 0);
            mCount++;
        }
    }

    private void findFrame() {
        MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
        int outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 50);
        while (outputBufferIndex >= 0) {
            Image image = mediaCodec.getOutputImage(outputBufferIndex);
            if (null != image) {
                Log.d(TAG, "image.getFormat:" + image.getFormat());
                final byte arr[] = getDataFromImage(image);
                // 进行处理图片的操作
            }

            mediaCodec.releaseOutputBuffer(outputBufferIndex, true);
            outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
        }
    }

    private byte[] getDataFromImage(Image image) {
        return getDataFromImage(image, COLOR_FormatNV21);
    }

    /**
     * 将Image根据colorFormat类型的byte数据
     */
    private byte[] getDataFromImage(Image image, int colorFormat) {
        if (colorFormat != COLOR_FormatI420 && colorFormat != COLOR_FormatNV21) {
            throw new IllegalArgumentException("only support COLOR_FormatI420 " + "and COLOR_FormatNV21");
        }
        if (!isImageFormatSupported(image)) {
            throw new RuntimeException("can't convert Image to byte array, format " + image.getFormat());
        }
        Rect crop = image.getCropRect();
        int format = image.getFormat();
        int width = crop.width();
        int height = crop.height();
        Image.Plane[] planes = image.getPlanes();
        byte[] data = new byte[width * height * ImageFormat.getBitsPerPixel(format) / 8];
        byte[] rowData = new byte[planes[0].getRowStride()];
        int channelOffset = 0;
        int outputStride = 1;
        for (int i = 0; i < planes.length; i++) {
            switch (i) {
                case 0:
                    channelOffset = 0;
                    outputStride = 1;
                    break;
                case 1:
                    if (colorFormat == COLOR_FormatI420) {
                        channelOffset = width * height;
                        outputStride = 1;
                    } else if (colorFormat == COLOR_FormatNV21) {
                        channelOffset = width * height + 1;
                        outputStride = 2;
                    }
                    break;
                case 2:
                    if (colorFormat == COLOR_FormatI420) {
                        channelOffset = (int) (width * height * 1.25);
                        outputStride = 1;
                    } else if (colorFormat == COLOR_FormatNV21) {
                        channelOffset = width * height;
                        outputStride = 2;
                    }
                    break;
            }
            ByteBuffer buffer = planes[i].getBuffer();
            int rowStride = planes[i].getRowStride();
            int pixelStride = planes[i].getPixelStride();

            int shift = (i == 0) ? 0 : 1;
            int w = width >> shift;
            int h = height >> shift;
            buffer.position(rowStride * (crop.top >> shift) + pixelStride * (crop.left >> shift));
            for (int row = 0; row < h; row++) {
                int length;
                if (pixelStride == 1 && outputStride == 1) {
                    length = w;
                    buffer.get(data, channelOffset, length);
                    channelOffset += length;
                } else {
                    length = (w - 1) * pixelStride + 1;
                    buffer.get(rowData, 0, length);
                    for (int col = 0; col < w; col++) {
                        data[channelOffset] = rowData[col * pixelStride];
                        channelOffset += outputStride;
                    }
                }
                if (row < h - 1) {
                    buffer.position(buffer.position() + rowStride - length);
                }
            }
        }
        return data;
    }

    /**
     * 是否是支持的数据类型
     */
    private static boolean isImageFormatSupported(Image image) {
        int format = image.getFormat();
        switch (format) {
            case ImageFormat.YUV_420_888:
            case ImageFormat.NV21:
            case ImageFormat.YV12:
                return true;
        }
        return false;
    }
```
"csd-0"和"csd-1"是什么，对于H264视频的话，它对应的是sps和pps，对于AAC音频的话，对应的是ADTS，做音视频开发的人应该都知道，它一般存在于编码器生成的IDR帧之中。

得到的mediaFormat
```
mediaFormat in:{height=720, width=1280, csd-1=java.nio.ByteArrayBuffer[position=0,limit=7,capacity=7], mime=video/avc, csd-0=java.nio.ByteArrayBuffer[position=0,limit=13,capacity=13], color-format=2135033992}
```

### 存储图片的方法
Image类在Android API21及以后功能十分强大。

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
* [Android: Image类浅析(结合YUV_420_888)](https://www.polarxiong.com/archives/Android-Image%E7%B1%BB%E6%B5%85%E6%9E%90-%E7%BB%93%E5%90%88YUV_420_888.html)
* [Android MediaCodec stuff](http://bigflake.com/mediacodec/)
* [雷霄骅(leixiaohua1020)的专栏](http://blog.csdn.net/leixiaohua1020/)
* [[总结]FFMPEG视音频编解码零基础学习方法 - 雷霄骅](http://blog.csdn.net/leixiaohua1020/article/details/15811977)
* [Bilibili/ijkplayer - Github](https://github.com/Bilibili/ijkplayer)
