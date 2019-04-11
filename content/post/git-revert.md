---
title: "Git 如何還原已經 push 的 commit"
date: 2017-09-17T22:34:00+08:00
lastmod: 2018-09-26T22:34:59+08:00
draft: false
tags: ["Git"]
slug: "git-revert"
aliases:
    - /2017/09/git-revert.html
---
# Git 如何還原已經 push 的 commit
同事在完成新功能的開發後，已經將 feature branch merge 至 master 中也 push 至 Git server 上，並通過了 CI server 的檢查，正等待著良辰吉時的到來，準備上線至 production 接著就是再次宣告完成一個需求功能的開發，但 user 卻因為種種因素決定放棄該功能，這時候該怎麼辦勒？

做法有好幾個，其中針對誤推帳密等敏感資訊的情況或是可以接受 commit history 遺失，最容易的方式就是重建新的 commit 然後使用 force push 來清除錯誤的 commit 了，但這樣的方式會讓線圖中斷也會造成其他人需要重新 clone，並不是好的做法，所以紀錄一下其他做法

## 情境說明

1.  已經將 `NewFeature` merge 至 `master` 並 push 至 Git Server 上
2.  user 打算將 `NewFeature` 所有 change 都抽掉


## 可以怎麼做？

Git 是版控系統，既然是需求面變更也應該在版控上留下這個紀錄，所以下列的操作概念會是將已經 merge 的 commit 加上一個反向的 commit 代表需求變更的結果，而不是直接消滅已經完成的 commit，讓整個 commit 歷程可以看得出曾經做過程式碼移除的動作，如果是直接移除已經不需要的 commit 日後要追蹤就不是那麼容易了

1.  使用 TortoiseGit
    *   merge `NewFeature` branch 後沒有後續修改或是後續修改不是基於需要移除的 commit

        ![1init](https://user-images.githubusercontent.com/3851540/30521708-b9128cf4-9bf6-11e7-9fc9-60f3728747f2.png)

        *   直接 revert merge commit 即可，但需要留意選擇正確的 commit

            ![2revert](https://user-images.githubusercontent.com/3851540/30521709-b93adb96-9bf6-11e7-8bc9-4516a1fde783.png)

        *   選擇的原則就是挑要保留的(ex.想拿掉 feature branch 就選 `parent1`)

            ![3reverted](https://user-images.githubusercontent.com/3851540/30521711-b95c9164-9bf6-11e7-80c5-054958b95131.png)

    *   merge `NewFeature` branch 後有基於需要移除的 commit 做的修改

        ![4case2](https://user-images.githubusercontent.com/3851540/30521710-b95c9358-9bf6-11e7-89d2-81535c06056f.png)

        *   revert merge commit，但會出現錯誤

            ![5revertcase2](https://user-images.githubusercontent.com/3851540/30521714-b95eed56-9bf6-11e7-9980-e566f75eb2f5.png)

            ![6reverterror](https://user-images.githubusercontent.com/3851540/30521713-b95e3500-9bf6-11e7-8076-42073e6500a9.png)

        *   依實際需求解決衝突

            ![7confict](https://user-images.githubusercontent.com/3851540/30521712-b95cffbe-9bf6-11e7-9dc5-31fb21fdf72b.png)

2.  使用 git revert 指令
    *   merge `NewFeature` branch 後沒有後續修改或是後續修改不是基於需要移除的 commit
        *   先用 `git log` 確認需要移除的 commit sha1

            ![8gitlog](https://user-images.githubusercontent.com/3851540/30521715-b961e4e8-9bf6-11e7-9ca5-7efbd4fa611c.png)

        *   執行 git revert {commit id} -m {parent NO}

             ![9gitrevert](https://user-images.githubusercontent.com/3851540/30521717-b982db6c-9bf6-11e7-953f-93931879ccc7.png)

    *   merge `NewFeature` branch 後有基於需要移除的 commit 做的修改
        *   執行 git revert {commit id} -m {parent NO}

            ![10gitreverterror](https://user-images.githubusercontent.com/3851540/30521716-b982b38a-9bf6-11e7-9bef-bf4131b79e04.png)

        *   依實際需求解決衡突

            ![11gitconflict](https://user-images.githubusercontent.com/3851540/30521718-b985c73c-9bf6-11e7-97f6-b0151c1a17f7.png)

*   重新 commit

    > 上面兩種方式都可以協助將已經 commit 的內容做反向還原，但還原後還是需要自行 commit 跟 push 才可以更新 Git server 上的內容

## 心得

在同事遇到需要排除已經花時間開發完成的功能前，我壓根沒有想過會出現這樣的需求：怎麼會在花了成本把功能完成後才決定捨棄功能勒？！ 要嘛就是提出需求的過程沒有想清楚，不然就是製作功能的成本太低造成放棄已經完成的功能不痛不癢

回到今天介紹的功能，之前比較普遍遇到的情境是不小心把機敏資訊公開在 GitHub 等公開平常上而造成資安問題，針對這樣的情況一般都是透過 force push 強制將之前的 commit 移除，避免從 commit history 找到過去的內容，跟同事的狀況不太一樣，如果是因為需求變更而需要移除程式碼就建議使用 git revert 的做法，在既有的程式碼基礎上加入反向的 commit 以利日後追蹤

# 參考資訊

1.  [git-revert](https://git-scm.com/docs/git-revert)
