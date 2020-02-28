---
title: "Ansible 透過 Http Status Code 當做檢核條件"
date: 2020-02-26T21:30:00+08:00
lastmod: 2020-02-26T21:30:31+08:00
draft: false
tags: ["Ansible"]
slug: "ansible-when-http-status-code"
---

## Ansible 透過 Http Status Code 當做檢核條件

最近 Ansible 使用的機會較多，簡單紀錄一下平常可能遇到的情境與解決方式，一般情況下我都是透過單一個 playbook 來處理某個工作，如果需要多個步驟時再透過 task 來切分，但有時候會發現前一個動作雖然成功執行了，但就商業邏輯的角度來看還不能算是可以正常提供服務，所以接著執行後面動作時就會引發一連串錯誤，今天筆記就是為了避免這種狀況

情境模擬：打算執行某個 job 但 Jenkins 重啟後仍未完成 warm-up，造成後續動作也就會失敗

## 基本環境說明

1. jenkins 2.204.2
2. ansible 2.7.8
3. absible script

   - inventory.ini

    ```ini
    [jenkins]
    jenkins1 ansible_host=192.168.1.112 ip=192.168.1.112  ansible_user=yowko ansible_password=password ansible_become_password=password
    ```

   - install.yml

    ```yml
    ---
    - name: Trigger Build
      hosts: jenkins
      tasks:
        - name: "Trigger Build"
          shell: curl http://localhost:8080/job/Test/build?token=67c2b2b3
    ```

## 遇到問題

- 錯誤訊息

    ```txt
    [WARNING]: Consider using the get_url or uri module rather than running
    'curl'.  If you need to use command because get_url or uri is insufficient you
    can add 'warn: false' to this command task or set 'command_warnings=False' in
    ansible.cfg to get rid of this message.

    fatal: [jenkins1]: FAILED! => {
        "changed": true, 
        "cmd": "curl -I http://localhost:8080/job/Test/build?token=67c2b2b3", 
        "delta": "0:00:00.066076", 
        "end": "2020-02-28 14:37:53.870970", 
        "invocation": {
            "module_args": {
                "_raw_params": "curl -I http://localhost:8080/job/Test/build?   token=67c2b2b3", 
                "_uses_shell": true, 
                "argv": null, 
                "chdir": null, 
                "creates": null, 
                "executable": null, 
                "removes": null, 
                "stdin": null, 
                "warn": true
            }
        }, 
        "msg": "non-zero return code", 
        "rc": 7, 
        "start": "2020-02-28 14:37:53.804894", 
        "stderr": "  % Total    % Received % Xferd  Average Speed   Time    Time     Time   Current\n                                 Dload  Upload   Total   Spent    Left      Speed\n\r  0     0    0     0    0     0      0      0 --:--:-- --:--:--    --:--:--     0curl: (7) Failed connect to localhost:8080; 連線被拒絕", 
        "stderr_lines": [
            "  % Total    % Received % Xferd  Average Speed   Time    Time     Time     Current", 
            "                                 Dload  Upload   Total   Spent    Left  Speed", 
            "", 
            "  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--       0curl: (7) Failed connect to localhost:8080; 連線被拒絕"
        ], 
        "stdout": "", 
        "stdout_lines": []
    }
    ```

- 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/75562152-15223380-5a83-11ea-994e-56b0f83de3ae.png)

## 使用語法

```yml
---
- name: Trigger Build
  hosts: jenkins
  tasks:
    - name: "Wait for Jenkins to Trigger Build"
      uri:
        url: "http://localhost:8080/job/Test/build?token=67c2b2b3"
        status_code: 201
      register: result
      until: result.status == 201
      retries: 60
      delay: 1
```

## 心得

上面的語法是精簡版，可以拆解為兩個步驟：

1. 先檢查服務是否正確運作
2. 再執行目標操作

但我自己遇到的問題比較常是不好明確定義服務正常運行，或是服務正常運作跟目標是否可以執行還是有些落差

## 參考資訊

1. [mikeifomin/wait_for_http.yml](https://gist.github.com/mikeifomin/67e233cd461331de16707ef59a07e372)
2. [How to check for a certain Status Code (4xx) in Ansible?](https://stackoverflow.com/questions/53009387/how-to-check-for-a-certain-status-code-4xx-in-ansible)
