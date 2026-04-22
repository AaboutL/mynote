---
title: macbook中clion长按hjkl无效的解决方法
updated: 2022-08-14 08:49:27Z
created: 2022-08-10 03:49:40Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

## clion
按照网上的方法：
defaults write com.jetbrains.intellij ApplePressAndHoldEnabled -bool false
然后重启clion，不起作用

单独设置clion：
defaults write com.jetbrains.Clion ApplePressAndHoldEnabled -bool false
然后重启clion，有效。

## vscode
defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false