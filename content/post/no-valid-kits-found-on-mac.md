---
title: "在 macOS 上的 Qt Creator 中出現 No valid kits found"
date: 2019-01-27T00:30:00+08:00
lastmod: 2021-11-02T00:30:31+08:00
draft: false
tags: ["Tools","macOS"]
slug: "no-valid-kits-found-on-mac"
---
## 在 macOS 上的 Qt Creator 中出現 No valid kits found

這是在 mac 上嘗試 build Redis Desktop Manager 時遇到的狀況，mac 本來就不是熟悉的操作系統  還要透過 Qt 來 build，早就有心理準備問題不會少，順手紀錄一下解決方式  相信日後會有些幫助的

## 基本環境說明

1. macOS Mojave 10.14.2
2. Qt 5.12.0
3. Qt Creator 4.8.1

## 問題描述

透過 Qt Creator 開啟 .pro 檔出現 project 需要 configure 與 No valid kits found

![1novalid](https://user-images.githubusercontent.com/3851540/51799857-c68fdd80-2261-11e9-8b36-3d9990f55568.png)

## 解決方式

1. 點擊 `options` 進行環境設定

    ![2options](https://user-images.githubusercontent.com/3851540/51799858-c68fdd80-2261-11e9-87e0-454685870b0f.png)

2. 點擊 `桌面 (default)` 確認 Qt 版本

    ![3qtversion](https://user-images.githubusercontent.com/3851540/51799859-c68fdd80-2261-11e9-8699-59a89e5583ba.png)

    > `無` 表示 Qt Creator 找不到正確的 Qt 版本

3. 點擊 `Qt 版本` 旁的 `Manage…` 設定

    ![4manage](https://user-images.githubusercontent.com/3851540/51799860-c68fdd80-2261-11e9-8e06-1094976e21a5.png)

4. 新增 qmake 位置

    > 我的位置為 `/usr/local/Cellar/qt/5.12.0/bin/qmake`

    ![5add](https://user-images.githubusercontent.com/3851540/51799861-c7287400-2261-11e9-9e5b-1c8afc985fdf.png)

    ![6added](https://user-images.githubusercontent.com/3851540/51799862-c7287400-2261-11e9-9488-22c658331202.png)

5. 記得將 Kits 的 Qt 版本選為剛手動新增的設定值

    ![7changeqtversion](https://user-images.githubusercontent.com/3851540/51799863-c7287400-2261-11e9-93cb-2e5deefcaf16.png)

6. 完成設定：出現可用的 kit

    ![8selectkit](https://user-images.githubusercontent.com/3851540/51799864-c7287400-2261-11e9-981d-e6ad294f3cbf.png)

## 前後比較

- 設定前： No valid kits found

    ![1novalid](https://user-images.githubusercontent.com/3851540/51799857-c68fdd80-2261-11e9-8b36-3d9990f55568.png)
- 設定後

    ![8selectkit](https://user-images.githubusercontent.com/3851540/51799864-c7287400-2261-11e9-981d-e6ad294f3cbf.png)

## 心得

以前總覺得在 mac 上的軟體開發工作相較於 Windows 是比較簡單的，以為 open source 專案都是以 mac 環境為主，主觀以為在 mac 上的設定應該早就被克服而少了許多繁瑣的環境設定問題，但就這次經驗看來過去的想法比較像是先入為主的迷思，基本環境設定還是免不了的

## 參考資訊

1. [首次安裝Qt後，創建項目時出現“No valid kits found” 的解決辦法](https://blog.csdn.net/aseity/article/details/55095052)
