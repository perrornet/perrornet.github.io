---
title: python根据exif信息旋转图片
date: 2017-06-08 00:10:49
tags:
    - python
---
这几天在写业务代码有个需求：压缩图片。本来是很简单需求，三下五除二就写好了，但是在自己用手机上传图片的时候发现压缩后的图片都歪了，查了一下，原因是图片中的exif信息中的Orientation记录中图片的旋转角度。需要根据这个来旋转图片。

首先读取图片的exif信息：
```
from PIL import Image
img = Image.open('1.jpg')
# 先判断图片是否有exif信息
if hasattr(img, '_getexif'):
# 获取exif信息
dict_exif = img._getexif()
if dict_exif(274, 0) == 3:
# 旋转
new_img = img.rotate(-90)
elif dict_exif(274, 0) == 6:
# 旋转
new_img = img.rotate(180)
else:
new_img = img
else:
new_img = img
new_img.save('new_1.jpg', )
```
