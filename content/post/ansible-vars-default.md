---
title: "Ansible 變數不存在時指定預設值"
date: 2021-11-09T00:30:00+08:00
lastmod: 2021-11-09T00:30:31+08:00
draft: false
tags: ["Ansible"]
slug: "ansible-vars-default"
---

## Ansible 變數不存在時指定預設值

這個技巧之前就已經頻繁用在專案上了，原本也以為掌握度相當高，不過隔了好陣子沒有寫 ansible script，當下要用時卻又腦袋打結不知道該如何下手，想了想雖然簡單但為了方便日後搜尋還是決定簡單筆記一下

## 基本環境設定

1. macOS Big Sur 11.6
2. ansible 2.11.9
3. python 3.9.7 (default, Sep  3 2021, 12:37:55) [Clang 12.0.5 (clang-1205.0.22.9)]
4. jinja 3.0.1

## 問題描述

- 直接存取未指定的變數

    ```ini
    - hosts: localhost
      tasks:
      - name: DEBUG
        debug:
          msg: "{{ test }}"
    ```

- 出現未定義 `undefined variable` 錯誤

    ```log
    fatal: [localhost]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'test' is undefined\n\nThe error appears to be in '/Users/yowko.tsai/sourcecode/it-automation/test.yml': line 37, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: DEBUG\n    ^ here\n"}
    ```

## 使用方式

1. value type : `boolean`,`int`,`string`,`float`

    - 語法

        ```ini
        - hosts: localhost
          tasks:
          - name: DEBUG
            debug:
              msg: "{{ test|default(對應型別的預設值) }}"
        ```

    - 範例

        > 以 bool 為例

        ```ini
        - hosts: localhost
        tasks:
        - name: DEBUG
            debug:
            msg: "{{ test|default(false) }}"
        ```

2. refernce type : `dict`、`list`

    ```ini
    - hosts: localhost
      tasks:
      - name: DEBUG
        debug:
          msg: "{{ item }}"
        with_items: "{{testdic|default({})}}"
    ```

3. 特殊使用：`omit`

    > `default(omit)` 表示當變數 undifined 時即不送出該行設定值；以下方 script 為例，testvar undifined 於是 debug module 就不會送出 msg value，而 msg 在 debug module 的預設值為 `Hello world!` 最後就是 print 出 `Hello world!`

    ```ini
    - hosts: localhost
      tasks:
      - name: DEBUG
        debug:
          msg: "{{ testvar|default(omit) }}"
    ```

    ![1msgomit](https://user-images.githubusercontent.com/3851540/140860041-e8928a31-ceff-470e-add4-bdb84cf9de43.png)

## 心得

原本想要快速紀錄一下的，結果發現記憶力衰退嚴重，又重新查了些資料才又慢慢回憶起來，不過幸虧有先下定決心要筆記，這次查資料還是覺得不是很好查，幸好下次知道哪邊可以快速找到了

## 參考資訊

1. [debug - Print statements during execution](https://docs.ansible.com/ansible/2.4/debug_module.html)
2. [In Ansible, determine the type of a value, and casting those values to other types](https://jon.sprig.gs/blog/post/1801)
3. [Making variables optional](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#making-variables-optional)
