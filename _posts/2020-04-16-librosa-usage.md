---
title: librosa
date: "2020-4-08 15:52:00 +0800"
tags: [音频处理]
---

# librosa用法
librosa是一个音频库，包括傅里叶变换、梅尔系数等一系列函数。准备记录一些用法。

## 输出一个文件
```python
import librosa
import os
cc_wav_path = os.path.join(os.path.abspath('.'), 'wav', 'cc-compare.wav')
sr_cc = librosa.audio.get_samplerate(cc_wav_path)
y_cc, sr = librosa.load(cc_wav_path, sr=sr_cc)
librosa.output.write_wav("out.wav", y_cc, sr)

```