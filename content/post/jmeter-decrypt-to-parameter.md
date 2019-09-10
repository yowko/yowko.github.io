---
title: "[JMeter] 從 Http Response Body 解密 Token 做為 Http Request Header 參數"
date: 2019-09-10T01:30:00+08:00
lastmod: 2019-09-10T01:30:31+08:00
draft: false
tags: ["JMeter"]
slug: "jmeter-decrypt-to-parameter"
---

## [JMeter] 從 Http Response Body 解密 Token 做為 Http Request Header 參數

之前筆記 [[JMeter] 取得 JSON Token 做為 Http Request Header 參數](https://blog.yowko.com/jmeter-json-parameter/) 紀錄到如何在測試目標 Api 前先執行 login 取得 token 後當做目標 api 的 header，當時有提到實際應用時常都會加密保護，過去比較常見的做法是 BeanShell，這次就試試 JSR223 吧

## 基本環境說明

1. macOS Mojave 10.14.6
2. JMeter 5.1.1
3. 模擬 auth 及回傳加密

    > 使用 base64 加密來模擬

    ```cs
    [Route("api/[controller]")]
    [ApiController]
    public class AuthController : ControllerBase
    {

        // POST api/values
        [HttpPost]
        public string Post([FromBody] AuthRequest model)
        {
            var txt = JsonConvert.SerializeObject(new AuthResponse()
            {
                Token = "QpwL5tke4Pnpja7X4"
            });
            var encodedBytes = System.Text.Encoding.UTF8.GetBytes(txt);
            var encodedTxt = Convert.ToBase64String(encodedBytes);

            return encodedTxt;
        }
    }

    public class AuthRequest
    {
        public string Email { get; set; }

        public string Password { get; set; }
    }

    public class AuthResponse
    {
        public string Token { get; set; }
    }
    ```

4. 模擬驗證及回傳

    ```cs
    [Route("api/[controller]")]
    [ApiController]
    public class UserController : ControllerBase
    {
        // GET
        [HttpPost]
        public AddUserResponse Post([FromHeader]string token,[FromBody] AddUserRequest model)
        {
            if (string.IsNullOrWhiteSpace(token) || String.CompareOrdinal(token, "QpwL5tke4Pnpja7X4") != 0) return null;
            return new AddUserResponse(){Id="497",CreatedAt = DateTime.Now.ToString()};
        }
    }

    public class AddUserRequest
    {
        public string Name { get; set; }

        public string Job { get; set; }
    }
    public class AddUserResponse
    {
        public string Id { get; set; }

        public string CreatedAt { get; set; }
    }
    ```

## JMeter 設定

1. Test Plan 右鍵 --> Add --> Threads (Users) --> Thread Group

    ![1addthread](https://user-images.githubusercontent.com/3851540/64123783-e11dcc80-cdd7-11e9-9259-e8d7229b94a1.png)

2. 建立 Login Logic Controller

    > 用來區隔不同目的設定

    ![2logiccontroller](https://user-images.githubusercontent.com/3851540/64624509-54998c80-d41d-11e9-88f8-44773a82ef5e.png)

    ![3onceonly](https://user-images.githubusercontent.com/3851540/64624510-54998c80-d41d-11e9-8e12-9a7f91612727.png)

3. 在 Login Logic Controller 下建立 HTTP Request - Login

    > 用來實際取得加密 token

    ![3request](https://user-images.githubusercontent.com/3851540/64123789-e1b66300-cdd7-11e9-8544-1ff01b486a2c.png)

    ![1loginrequest](https://user-images.githubusercontent.com/3851540/64624507-54998c80-d41d-11e9-9ab7-eea8dede8235.png)

4. 在 HTTP Request - Login 下建立 Http Header Manager - Login

    > 用來管理取得 token 的 request header，建立在 HTTP Request 代表該 header 僅適用於該 HTTP Request，不會影響到其他 HTTP Request

    ![2header](https://user-images.githubusercontent.com/3851540/64123784-e11dcc80-cdd7-11e9-92fc-7599b9ec29f1.png)

    ![2loginheader](https://user-images.githubusercontent.com/3851540/64123785-e1b66300-cdd7-11e9-8e80-cfede5d482f9.png)

5. 在 HTTP Request - Login 下建立 JSR223 PostProcessor

    > 取得回傳的加密 JSON 內容，並解密後設定為 JMeter 變數

    ![4postprocessor](https://user-images.githubusercontent.com/3851540/64624512-55322300-d41d-11e9-85c5-6bf256662976.png)

    ![5processorsetting](https://user-images.githubusercontent.com/3851540/64624513-55322300-d41d-11e9-942e-8d6e3f1709bc.png)

    - 使用 java (也可以使用 groovy，javascript)
    - 使用 java 來解密以及設定 JMeter 變數

        ```java
        import org.apache.commons.codec.binary.Base64;
        import net.sf.json.JSONObject;

        String string = "eyJUb2tlbiI6IlFwd0w1dGtlNFBucGphN1g0In0=";
        byte[] byteArray = Base64.decodeBase64(string.getBytes());
        String decodedString = new String(byteArray);

        JSONObject json = JSONObject.fromObject(decodedString);

        vars.put("token", json.getString("Token"));
        ```

6. 建立測試目標 HTTP Request - Create User

    > 實際執行測試目標

    ![6createrequest](https://user-images.githubusercontent.com/3851540/64624515-55322300-d41d-11e9-89d2-82ec152dd158.png)

7. 在測試目標 HTTP Request 下建立 Http Header Manager - Create User

    > 用來管理實際執行目標 api 的 header 設定, 並使用上方 JSR223 PostProcessor 設定的參數

    ![7createuserheader](https://user-images.githubusercontent.com/3851540/64624516-55cab980-d41d-11e9-9a72-3ea437a09ee4.png)

8. 加入 View Results Tree

    > 用來檢視各步驟的執行結果

    ![8viewresult](https://user-images.githubusercontent.com/3851540/64123794-e2e79000-cdd7-11e9-9c6f-5c7661dcdbe0.png)

9. 實際執行測試

    可以從 View Results Tree 看到 HTTP Request - Login 取得加密的 token : `eyJUb2tlbiI6IlFwd0w1dGtlNFBucGphN1g0In0=`，並可以看到 HTTP Request - Create User 的 request header 也正確使用 `Token: QpwL5tke4Pnpja7X4`

    ![8resultencrypt](https://user-images.githubusercontent.com/3851540/64624518-55cab980-d41d-11e9-964c-e5985f524eaa.png)

    ![9resultheader](https://user-images.githubusercontent.com/3851540/64624519-55cab980-d41d-11e9-90aa-9ac7b6c2e6aa.png)

10. 完整設定

    ![10architech](https://user-images.githubusercontent.com/3851540/64624520-55cab980-d41d-11e9-93a5-9e6baf042703.png)

## 心得

記得實際在修改專案時我一直遇到無法在 Http Header Manager - Create User 中取得 JSR223 PostProcessor 設定的 `token` 變數值，但筆記時卻一切正確 ？！ 難道是人品變好了 ~~ 但這邊還是紀錄一下該如何解決：在 JSR223 PostProcessor 後再加一個 Http Header Manager 去讀取 `token` 變數，實際執行 HTTP Request - Create User 時就可以正確拿到 `token` 變數，雖然不知道原因為何(猜測是 bug)，但實際使用是有效的 - 可以參考 [JMeter Alter HTTP Headers During Test](https://stackoverflow.com/a/43283700/3600583)

## 參考資訊

1. [JMeter Alter HTTP Headers During Test](https://stackoverflow.com/a/43283700/3600583)
2. [[JMeter] 取得 JSON Token 做為 Http Request Header 參數](https://blog.yowko.com/jmeter-json-parameter/)
