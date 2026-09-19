# 概览

# 常用方法
```python
from pathlib import Path

# 获取路径
Path('D:/demo/test.py')
Path.cwd()  # 档期那工作路径
Path.home()  # 用户home目录
Path(__file__)  # 当前文件路径，cmd和pycharm结果不同

# 文件方法
file = Path('/test.py')
file..resolve()  # 获取绝对路径 D:/demo/test.py
file.stat()  # 文件属性，如文件大小，创建时间，修改时间等
file.stat().st_size
file.stat().st_atime
file.stat().st_ctime
file.stat().st_mtime

# 路径组成
file.name  # test.py
file.stem  # test
file.suffix  # .py
file.parent  # D:/demo/
file.parents  # 所有父目录
file.anchor  # 锚，目录前面的部分 C:\ 或者 /

# 路径扫描
path = Path.cwd()
path.iterdir()  # 扫描某个目录下的所有路径（文件和子目录)
path.glob('*.txt')  # 当前目录所有txt文件
path.rglob('*.txt')  # 当前目录及所有子目录txt文件
path.match('*.txt')  # 路径是否符合规则

# 路径拼接
Path.home() / 'dir' / 'file.txt'
Path.home().joinpath('dir', 'file.txt')

# 路径判断
path.is_file()  # 是否文件
path.is_dir()  # 是否路径
path.exists  # 是否存在
```
# 文件操作

对文件进行操作最好还是用 shutil 模块
```python
# 文件操作
file.touch(exist_ok=True)  # 创建文件
file.unlink()  # 删除文件
path.mkdir(parents=True)  # 创建目录， parents为True可创建多级目录，否则只能创建一层
path.rmdir()  # 删除目录没有提示，一次只删除一级目录，且当前目录必须为空

# 文件IO
with open(file) as f:  # 打开文件
    pass
with file.open() as f:  # 打开文件
    pass
file.read_text()  # 读取文本，已封装，无需重复去打开文件和管理文件的关闭了
file.read_bytes()  # 读取 bytes
file.write_text()  # 写入文本，w模式
file.write_bytes()  # 写入bytes，w模式

# 移动文件
txth = Path('archive/demo.txt')
res = txt.replace('new_demo.txt')  # 把 archive 目录下的 demo.txt 文件移动到当前工作目录，并重命名为 new_demo.txt

# 重命名文件
txt = Path('archive/demo.txt')
txt_ = txt.with_name('new.txt')
txt.replace(txt_)

# 修改后缀名
txt = Path('archive/demo.txt')
txt_ = txt.with_suffix('.json')
txt.replace(txt_)
```
