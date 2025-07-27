# 官方项目地址
https://github.com/openai/whisper

# 安装
```
# whisper
pip install -U openai-whisper

# ffmpeg
apt update && sudo apt install ffmpeg

# rust
pip install setuptools-rust
````

# whisper的六大模型
| Size    | Parameters | English-only model | Multilingual model | Required VRAM | Relative speed |
|---------|------------|---------------------|---------------------|--------------|----------------|
| tiny    | 39 M       | tiny.en             | tiny                | ~1 GB        | ~10x           |
| base    | 74 M       | base.en             | base                | ~1 GB        | ~7x            |
| small   | 244 M      | small.en            | small               | ~2 GB        | ~4x            |
| medium  | 769 M      | medium.en           | medium              | ~5 GB        | ~2x            |
| large   | 1550 M     | N/A                 | large               | ~10 GB       | 1x             |
| turbo   | 809 M      | N/A                 | turbo               | ~6 GB        | ~8x            |

一般选择turbo就行，显存足够上large。
# 使用
语音转文字，默认采用English语言去识别音频
```
# 同时转3个音频文件
whisper audio.flac audio.mp3 audio.wav --model turbo
```

语音转文字的同时，指定音频中的语言。whisper会根据--language的语言做解析。
```
whisper japanese.wav --language Japanese
whisper /root/00课程介绍与学习指南.mp4   --model turbo  --language Cantonese 
```
语音转文字后，翻译成英文
```
whisper japanese.wav --model medium --language Japanese --task translate
```
查看命令帮助：
```
whisper --help
```

这个工具也可以识别mp4格式的语音。