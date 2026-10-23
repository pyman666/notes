# shutil

`shutil`可以简单地理解为sh + util，shell工具的意思。`shutil`模块是对`os`模块的补充，主要针对文件的拷贝、删除、移动、压缩和解压操作。

**拷贝**

```python
shutil.copyfileobj(fsrc, fdst)  # 将文件内容拷贝到另一个文件中
shutil.copyfile(src, dst)  # 拷贝文件
shutil.copymode(src, dst)  # 仅拷贝权限。内容、组、用户均不变
shutil.copystat(src, dst)  # 仅拷贝状态的信息，包括：mode bits, atime, mtime, flags
shutil.copy(src, dst)  # 拷贝文件和权限
shutil.copy2(src, dst)  # 拷贝文件和状态信息

shutil.copytree(src, dst, symlinks=False, ignore=None)  # 递归的去拷贝文件夹
shutil.copytree('folder1', 'folder2', ignore=shutil.ignore_patterns('*.pyc', 'tmp*'))
```

**删除**

`shutil.rmtree(path[, ignore_errors[, onerror]])  # 递归的去删除文件`

**移动**

`shutil.move(src, dst)  # 递归的去移动文件，它类似mv命令，其实就是重命名。`

**压缩**

`shutil.make_archive(base_name, format,...)  # 创建压缩包并返回文件路径，例如：zip、tar`
