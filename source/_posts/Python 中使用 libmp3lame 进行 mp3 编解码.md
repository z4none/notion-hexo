---
updated: '2014-07-09 00:00:00'
categories: code
excerpt: 最近在研究 Python 的 mp3 编解码， 可能是 mp3 格式专利问题，这方面可用并且还在维护的 Python 库没发现几个
date: '2014-07-09 00:00:00'
tags:
  - Python
urlname: python-lame
title: Python 中使用 libmp3lame 进行 mp3 编解码
---

最近在研究 Python 的 mp3 编解码， 可能是 mp3 格式专利问题，这方面可用并且还在维护的 Python 库没发现几个


试了一下 pymedia 倒是可以，但是它官方支持的版本只到 Python 2.4 


找了个第三方的 for Python 2.7 版本的， 发现它在 windows 下的实现中输出缓冲区大小写死成 10000 字节，导致解码的实时性差， 努力想把官方工程修改后迁移到 2.7 版本，最后还是放弃了… 


也尝试直接用 Lame.exe 用管道收发数据，但是 Windows 下 Python 2.X 貌似不支持管道的异步读写，最后也放弃了 好在我只用考虑 windows 平台，最后通过 ctype 调用 libmp3lame.dll 中的方法实现了我要的功能 


代码很简单，只考虑了我需要的音频格式


```text
#coding:utf-8

import time
import ctypes

class LameEncoder():
    def __init__(self, sample_rate, channel_count, bit_rate):
        self.dll  = ctypes.CDLL("libmp3lame.dll")
        self.lame = self.dll.lame_init()
        self.dll.lame_set_in_samplerate(self.lame, sample_rate);
        self.dll.lame_set_num_channels(self.lame, channel_count);
        self.dll.lame_set_brate(self.lame, bit_rate);
        self.dll.lame_set_quality(self.lame, 3);
        self.dll.lame_init_params(self.lame);

    def encode(self, pcm_data):
        sample_count    = len(pcm_data) /2
        output_buff_len = int(1.25 * sample_count + 7200)
        output_buff     = ctypes.create_string_buffer(output_buff_len)
        output_size     = self.dll.lame_encode_buffer(self.lame, ctypes.create_string_buffer(pcm_data), 0, sample_count, output_buff, output_buff_len);
        return output_buff.raw[:output_size]

class LameDecoder():
    def __init__(self, sample_rate, channel_count, bit_rate):
        self.dll  = ctypes.CDLL("libmp3lame.dll")
        self.lame = self.dll.lame_init()
        self.hip  = self.dll.hip_decode_init()
        self.dll.lame_set_in_samplerate(self.lame, sample_rate)
        self.dll.lame_set_num_channels(self.lame, channel_count)
        self.dll.lame_set_brate(self.lame, bit_rate)
        self.dll.lame_set_mode(self.lame, 3)
        self.dll.lame_set_quality(self.lame, 3)
        self.dll.lame_init_params(self.lame)
        self.dll.lame_get_framesize(self.lame)

    def decode(self, mp3_data):
        output_buff_len =  self.dll.lame_get_framesize(self.lame) * 2
        output_buff     = ctypes.create_string_buffer(output_buff_len)
        output_size     = self.dll.hip_decode1(self.hip, ctypes.create_string_buffer(mp3_data), len(mp3_data), output_buff, 0);
        return output_buff.raw[:output_size * 2]

    def flush(self):
        output_buff_len =  self.dll.lame_get_framesize(self.lame) * 2
        output_buff     = ctypes.create_string_buffer(output_buff_len)
        output_size     = self.dll.hip_decode1(self.hip, ctypes.create_string_buffer(""), 0, output_buff, 0);
        return output_buff.raw[:output_size * 2]

if __name__ == "__main__":
    def test_enc():
        print "test encode ..."
        lame = LameEncoder(8000, 1, 8)
        input_file  = open("1.pcm", "rb")
        output_file = open("1.mp3", "wb+")
        while 1:
            data = input_file.read(256)
            if data:
                output = lame.encode(data)
                output_file.write(output)
            else:
                break
        input_file.close()
        output_file.close()
        print "test encode done"

    def test_dec():
        print "test decode ..."
        lame = LameDecoder(8000, 1, 8)
        input_file  = open("1.mp3", "rb")
        output_file = open("2.pcm", "wb+")
        while 1:
            data = input_file.read(512)
            if data:
                output = lame.decode(data)
                if output:
                    output_file.write(output)
                    while 1:
                        output = lame.flush()
                        if output:
                            output_file.write(output)
                        else:
                            break
            else:
                break
        input_file.close()
        output_file.close()
        print "test decode done"

    test_enc()
    test_dec()
```

