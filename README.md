# Hugo-Website
## 不要提交 reimu 子模块！！！
使用 copy-params.yml.ps1 复制文件，在进行任何提交之前，撤销 reimu 子模块中的全部更改。

## 更新 reimu
```bash
git submodule update --remote
cd themes/reimu
git fetch
cd ../..
```
如果 reimu 的 params.yml 有重大更新，可以这样做：

1. 使用 CSCode ，打开 themes/reimu/config/_default/params.yml
1. 打开命令面板（Ctrl + Shift + P），选择“文件: 比较活动文件与...”("File: Compare Active File With...")
1. 选择 params.yml
1. 这是界面分为两半，在两文件不同的地方，两边界面的中间，有个箭头“还原块”，可以快速地把想同步的内容同步。