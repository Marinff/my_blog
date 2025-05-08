---
title: lsb
date: '2025-05-05 09:34:43'
updated: '2025-05-08 23:53:37'
permalink: /post/lsb-1x9l1l.html
comments: true
toc: true
---



# lsb

## LSB隐写笔记

---

最近给新生出题，研究了一下lsb的出题，本着写都写了，发挥最大作用，水篇博客先

### lsb原理

---

* 这里单值图片的lsb，音频的lsb还没研究

在一张图片中，一个像素的颜色由RGB三个通道的数据影响，一个通道的数据有8bit，即256种，那么三个通道组合起来，就有256^3种颜色，如果我们对这三个通道的最低位的数据修改一下，人眼是看不出颜色的区别的
最基本的lsb就是这样，将要隐藏的数据转换为二进制，然后按RGB的顺序轮流存储到图片颜色的最后一位

更进一步的lsb也是基于这个基础上，比如只存储到R通道，或RG通道，又或者不是存储在最后一位的数据，而是存储在第7，6位

使用python脚本简单实现了一些功能用来实现lsb（写的很烂，大佬轻喷orz

```python
import numpy as np
import PIL.Image as Image
import math


class info(object):
    def __init__(self,path):
        self.picture = Image.open(path)
        # 获取图片数据
        self.pic_data = np.array(self.picture)
        # 获取rgb各通道的数组，备用
        self.r_info = self.pic_data[:,:,0]
        self.g_info = self.pic_data[:,:,1]
        self.b_info = self.pic_data[:,:,2]
    
    # 获取单通道隐写的数据
    def one_lsb_info(self,rgb):
        if rgb == 'r':
            # 将多维数组变为序列
            r_data = self.r_info.ravel().tolist()
            return r_data
        if rgb == 'g':
            g_data = self.g_info.ravel().tolist()
            return g_data
        elif rgb == 'b':
            b_data = self.b_info.ravel().tolist()
            return b_data
    
    # 获取双通道隐写的数据
    def two_lsb_info(self,rgb):
        if rgb == 'rg':
            rg_data = np.stack((self.r_info,self.g_info),axis=-1)
            rg_data = rg_data.ravel().tolist()
            return rg_data
        elif rgb == 'rb':
            rb_data = np.stack((self.r_info,self.b_info),axis=-1)
            rb_data = rb_data.ravel().tolist()
            return rb_data
        elif rgb == 'gb':
            gb_data = np.stack((self.g_info,self.b_info),axis=-1)
            gb_data = gb_data.ravel().tolist()
            return gb_data
    
    # 获取全通道隐写的数据
    def all_lsb_info(self):
        img_data = self.pic_data.ravel().tolist()
        return img_data
    
    def insert_secret(self,secret,rgb_data,plane=0):
        """
        secret：要隐藏的信息
        rgb_data：图片的颜色信息
        plane：隐写的位，默认0就是修改最后一位的数据
        """
        index = 0
        res_data = []

        for i in range(len(secret)):
            # 将要隐藏的信息修改为二进制值
            secret_info = ord(secret[i])
            # 填充为8位二进制
            bin_secret = bin(secret_info)[2:].zfill(8)

            insert_data = rgb_data[index*8:(index+1)*8]
            
            for i in range(8):
                # 将颜色的列表转换为二进制，用来存储信息
                bin_data = bin(insert_data[i])[2:].zfill(8)
                if bin_secret[i] == '0':
                    bin_data = bin_data[:7-plane] + '0' + bin_data[8-plane:]
                elif bin_secret[i] == '1':
                    bin_data = bin_data[:7-plane] + '1' + bin_data[8-plane:]
                res_data.append(int(bin_data,2))

            # 修改下次循环加入数据的位置
            index += 1

        res_data += rgb_data[index*8:]
        return res_data
	
    # 将图片信息保存到新图片中
    def save_new_pic(self,res_data,rgb):
        h,w = self.pic_data.shape[:2]
        
        new_image_data = np.zeros((h, w, 3), dtype=np.uint8)
        if rgb == 'r':
            new_image_data[:, :, 0] = np.array(res_data).reshape((h, w))
            new_image_data[:, :, 1] = self.g_info
            new_image_data[:, :, 2] = self.b_info
        if rgb == 'g':
            new_image_data[:, :, 0] = self.r_info
            new_image_data[:, :, 1] = np.array(res_data).reshape((h, w))
            new_image_data[:, :, 2] = self.b_info
        elif rgb == 'b':
            new_image_data[:, :, 0] = self.r_info
            new_image_data[:, :, 1] = self.g_info
            new_image_data[:, :, 2] = np.array(res_data).reshape((h, w))
        elif rgb == 'rg':
            new_rg = np.array(res_data).reshape(-1,2)
            new_r = new_rg[:,0].reshape(self.r_info.shape)
            new_g = new_rg[:,1].reshape(self.g_info.shape)

            new_image_data = np.stack((new_r,new_g,self.b_info),axis=-1).astype(np.uint8)
        elif rgb == 'rb':
            new_rb = np.array(res_data).reshape(-1,2)
            new_r = new_rb[:,0].reshape(self.r_info.shape)
            new_b = new_rb[:,1].reshape(self.b_info.shape)

            new_image_data = np.stack((new_r,self.g_info,new_b),axis=-1).astype(np.uint8)
        elif rgb == 'gb':
            new_gb = np.array(res_data).reshape(-1,2)
            new_g = new_gb[:,0].reshape(self.g_info.shape)
            new_b = new_gb[:,1].reshape(self.b_info.shape)

            new_image_data = np.stack((self.r_info,new_g,new_b),axis=-1).astype(np.uint8)

        elif rgb == 'rgb':
            new_image_data = np.array(res_data).astype(np.uint8).reshape(self.pic_data.shape)

        new_img = Image.fromarray(new_image_data)
        new_img.save('res.png')

    # 利用stegsolve可以查看图像通道的功能，将二维码数据隐藏到图片中
    def insert_qrcode(self,rgb_data,secret,start_row=0,start_col=0):
        """
        secret: 要隐藏的二维码
        rgb: 要隐藏的图片的通道
        
        start_row: 隐藏的二维码的高度
        start_col: 隐藏的二维码的宽度 
        这两个参数默认是0,可以不修改
        
        """
        index = 0
        wei,hei = self.picture.size
        res_data = []

        s_len = int(math.sqrt(len(secret)))
        
        for i in range(hei):
            insert_data = rgb_data[i*wei:(i+1)*wei]

            for j in range(wei):
                if i >= start_row and j >= start_col:
                    if (i-start_row)*s_len + (j-start_col) < len(secret) and (j-start_col) < s_len:  
                        bin_data = bin(insert_data[j])[2:].zfill(8)  
                        # 这个0和1可以看情况修改，有些二维码给的是反的
                        if secret[(i-start_row)*s_len + (j-start_col)] == '0':
                            bin_data = bin_data[:7] + '1'
                        elif secret[(i-start_row)*s_len +(j-start_col)] == '1':
                            bin_data = bin_data[:7] + '0'
                        
                        res_data.append(int(bin_data,2))
                    else:
                        bin_data = bin(insert_data[j])[2:].zfill(8)
                        bin_data = bin_data[:7] + '1'
                        res_data.append(int(bin_data,2))
                else:
                    bin_data = bin(insert_data[j])[2:].zfill(8)
                    bin_data = bin_data[:7] + '1'
                    res_data.append(int(bin_data,2))
        return res_data
    

# 写来简化操作的类，每次要隐写只需要修改参数或方法
class oper(object):
    def one(self,pic,secret,rgb,plane=0):
        rgb_data = pic.one_lsb_info(rgb)
        res_data = pic.insert_secret(secret,rgb_data,plane)
        pic.save_new_pic(res_data,rgb)
    def two(self,pic,secret,rgb,plane=0):
        rgb_data = pic.two_lsb_info(rgb)
        res_data = pic.insert_secret(secret,rgb_data,plane)
        pic.save_new_pic(res_data,rgb)
    def all(self,pic,secret,plane=0):
        rgb_data = pic.all_lsb_info()
        res_data = pic.insert_secret(secret,rgb_data,plane)
        pic.save_new_pic(res_data,'rgb')

    def insert_pic(self,pic,rgb,secret):
        start_row = 40
        start_col = 40
        rgb_data = pic.one_lsb_info(rgb)
        res_data = pic.insert_qrcode(rgb_data,secret,start_row,start_col)
        pic.save_new_pic(res_data,rgb)

        
# 临时起意想的一个小功能，把黑白图片转成二进制，往lsb里塞图片的一个小想法
def pic_turn_to_code(hide_pic_path):
    img = Image.open(hide_pic_path)
    w,h = img.size

    pixels = img.load()
    code = ""
    for i in range(h):
        for j in range(w):
           if pixels[j,i] == (255,255,255,255):
               code += '0'
           elif pixels[j,i] != (255,255,255,255):
               code += '1'
    return code 


if __name__ == '__main__':
    """
    picture_path: lsb图片路径
    hide_pic_path: 要隐藏的黑白图的路径
    secret: 要隐藏的信息
    rgb: 要隐藏信息的rgb通道,在对应通道要修改rgb值
    op.one: 单通道隐写
    op.two: 双通道隐写
    op.all: 全通道隐写
    op.insert_pic: 隐写黑白图片
    """

    picture_path = 'lsb.jpg'
    secret = r"aurora{mairn_like_lsb}"
    rgb = "r"
    plane = 0

    
    pic = info(picture_path)    
    op = oper()

    # op.one(pic,secret,rgb,plane)
    # op.two(pic,secret,rgb)
    # op.all(pic,secret)

    hide_pic_path = '1.png'
    secret = pic_turn_to_code(hide_pic_path)
    op.insert_pic(pic,rgb,secret)


```
