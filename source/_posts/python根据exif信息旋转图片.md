---
title: python根据exif信息旋转图片
date: 2017-06-08 00:10:49
tags:
    - python
---
最近写业务代码时遇到了一个需求：压缩图片。这本应该是一个简单的需求，三下五除二就能解决。但是当我用手机上传图片时，发现压缩后的图片都歪了。经过查找，发现问题出在图片的exif信息中的Orientation记录，它记录了图片的旋转角度。因此，需要根据这个角度来旋转图片。

首先需要读取图片的exif信息，代码如下：

```
from PIL import Image

img = Image.open('1.jpg')

# 先判断图片是否有exif信息
if hasattr(img, '_getexif'):
    # 获取exif信息
    dict_exif = img._getexif()
    if dict_exif.get(274, 0) == 3:
        # 旋转
        new_img = img.rotate(-90)
    elif dict_exif.get(274, 0) == 6:
        # 旋转
        new_img = img.rotate(180)
    else:
        new_img = img
else:
    new_img = img

new_img.save('new_1.jpg')

```

通过这个代码，我们可以获取图片的exif信息，并根据其中的Orientation记录来旋转图片。最终，我们得到了正确的压缩后的图片。
