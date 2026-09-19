
`shutil`可以简单地理解为sh + util，shell工具的意思。shutil模块是对os模块的补充，主要针对文件的拷贝、删除、移动、压缩和解压操作。
```python
shutil.copyfileobj(fsrc, fdst[, length])  # 将文件内容拷贝到另一个文件中

shutil.copyfile(src, dst)  # 拷贝文件
shutil.copymode(src, dst)  # 仅拷贝权限。内容、组、用户均不变
shutil.copystat(src, dst)  # 仅拷贝状态的信息，包括：mode bits, atime, mtime, flags
shutil.copy(src, dst)  # 拷贝文件和权限
shutil.copy2(src, dst)  # 拷贝文件和状态信息

shutil.ignore_patterns(*patterns)
shutil.copytree(src, dst, symlinks=False, ignore=None)  # 递归的去拷贝文件夹
>>> shutil.copytree('folder1', 'folder2', ignore=shutil.ignore_patterns('*.pyc', 'tmp*'))

shutil.rmtree(path[, ignore_errors[, onerror]])  # 递归的去删除文件

shutil.move(src, dst)  # 递归的去移动文件，它类似mv命令，其实就是重命名。

shutil.make_archive(base_name, format,...)  # 创建压缩包并返回文件路径，例如：zip、tar
- base_name： 压缩包的文件名，也可以是压缩包的路径。只是文件名时，则保存至当前目 录，否则保存至指定路径，
     如 data_bak                       =>保存至当前路径
     如：/tmp/data_bak =>保存至/tmp/
- format： 压缩包种类，“zip”, “tar”, “bztar”，“gztar”
- root_dir： 要压缩的文件夹路径（默认当前目录）
- owner： 用户，默认当前用户
- group： 组，默认当前组
- logger： 用于记录日志，通常是logging.Logger对象
```
