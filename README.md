# video2note
本项目教你如何将本地视频自动转换成笔记。cpu即可运行。

# 环境安装
```
# pip
python3 -m venv venv
source venv/bin/activate
python -m pip install -U pip setuptools wheel
pip install -r requirements.txt

# ffmpeg on Debian/Ubuntu
apt update && sudo apt install ffmpeg
```

# 文件转换
假设你的视频文件路径是 /path/to/sample.mp4 ：
```
# 提取视频关键帧，将关键帧截图全部保存到note.pdf文件
evp --similarity 0.6 --pdfname note.pdf  ./   /path/to/sample.mp4
# 当前目录下就会有个note.pdf

# 提取视频中的音频，将音频转成文字。--language制定视频语音使用哪国的语言
whisper /path/to/sample.mp4   --model turbo  --language Chinese
# 初次转换时间会很长，会自动通过外网下载turbo模型（1.2GB），保存到本地~/.cache/whisper中
# whisper --help 查看支持哪些语言
```

# 语音文字纠错
whisper语音转文字的错误率在15%左右，需要借用大模型纠错。<br>
将note.pdf和语音转文字txt文件上传给大模型。比如腾讯元宝deepseek、豆包等。<br>
纠错prompt如下：
```
你好，我通过工具提取了一个视频的关键帧，将所有关键帧图片保存到pdf中，已经将pdf文件上传给你。
同时我使用whisper将该视频的音频部分转换成了文字，但是部分文字可能存在转换错误的问题，txt文件也已经上传给你。
现在请你根据pdf中的关键帧图片的信息，对音频转文字部分进行纠错。请直接返回纠错的结果，不要返回其他不相关的话语。
```
市面上很多LLM不支持多模态，腾讯元宝的deepseek会自动将图片进行OCR识别。通过OCR文字，对语音识别文字进行纠错。

# 其他好用的功能
正在想...


