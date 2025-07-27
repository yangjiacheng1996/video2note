# extract-video-ppt
项目地址 https://github.com/wudududu/extract-video-ppt
本项目将视频中的ppt导出为pdf文件，或者直接把视频转成pdf。
## 安装
```
# install from pypi
pip install extract-video-ppt

# or local
python ./setup.py install

# or local user
python ./setup.py install --user

````

## 使用
```
# help info
evp --help
# example
evp --similarity 0.6 --pdfname hello.pdf --start_frame 0:00:09 --end_frame 00:00:30 ./ ./test.mp4
# similarity: The similarity between this frame and the previous frame is less than this value and this frame will be saveed, default: 0.6
# pdfname: the name for export pdf 
# start_frame: start frame time point, default = '00:00:00'
# end_frame: end frame time point, default = 'INFINITY'

````