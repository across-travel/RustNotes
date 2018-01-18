---
title: Android 使用OpenGL，记录一些API
date: 2018-02-18 20:52:18
category: Android_note
toc: true
---

* win7
* Android Studio 3.0.1

在Android中使用OpenGL，记录一些API。

`GLES20.glGetError()`

如果函数参数不符或者不符合当前上下文设置的状态，则会导致 OpenGL Error。
以error code来表示。绝大多数情况下 OpenGL functions 产生 errors，则不会生效。少数有效。
OpenGL Error 存储在一个队列中，直到该错误被处理。
因此，如果你不定期的检测错误，你将不会知道某个函数某个函数的调用触发了错误。
错误检测应该定期检测，确保知道错误的详细信息。


GLES20.glCreateShader
GLES20.glShaderSource
GLES20.glCompileShader
GLES20.glDeleteShader
GLES20.glGetShaderInfoLog

GLES20.glGetAttribLocation
GLES20.glGetUniformLocation

GLES20.glDeleteTextures
GLES20.glGenTextures
GLES20.glBindTexture
GLES20.glTexImage2D
GLES20.glTexParameterf

GLES20.glUseProgram
GLES20.glVertexAttribPointer
GLES20.glEnableVertexAttribArray
GLES20.glActiveTexture
GLES20.glBindTexture
GLES20.glUniform1i
GLES20.glDrawArrays
GLES20.glFinish

GLES20.glDisableVertexAttribArray
GLES20.glTexParameteri

## 参考资料

* [OpenGL - The Industry Standard for High Performance Graphics](https://www.opengl.org/)
