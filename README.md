# Hugo-Website

## 不要提交 reimu 子模块

## 克隆仓库记得克隆子模块

```bash
git clone --recurse-submodules <仓库地址>
```

如果已经克隆了主项目但忘记添加 `--recurse-submodules` 参数，可以通过以下命令初始化和更新子模块：

```bash
git submodule update --init --recursive
```

## 更新 reimu 到特定的 commit

```bash
cd themes/reimu
git reset --hard <目标commit_hash>
cd ../..
git add themes/reimu
git commit -m "chore: update reimu"
git push
```

## reimu 示例

[hugo-reimu-template](https://github.com/D-Sketon/hugo-reimu-template)

[效果](https://d-sketon.github.io/hugo-theme-reimu/)
