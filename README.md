# Hugo-Website
## 不要提交 reimu 子模块！！！
使用 copy-params.yml.ps1 复制文件，在进行任何提交之前，撤销 reimu 子模块中的全部更改。

## 克隆仓库记得克隆子模块
```bash
git clone --recurse-submodules <仓库地址>
```

如果已经克隆了主项目但忘记添加 `--recurse-submodules` 参数，可以通过以下命令初始化和更新子模块：

```bash
git submodule update --init --recursive
```

## 更新 reimu
```bash
git submodule update --remote
cd themes/reimu
git fetch
cd ../..
git add *
git commit -m "Update reimu"
git push
```
如果 reimu 的 params.yml 有重大更新，可以这样做：

1. 使用 CSCode ，打开 themes/reimu/config/_default/params.yml
1. 打开命令面板（Ctrl + Shift + P），选择“文件: 比较活动文件与...”("File: Compare Active File With...")
1. 选择 params.yml
1. 这是界面分为两半，在两文件不同的地方，两边界面的中间，有个箭头“还原块”，可以快速地把想同步的内容同步。
