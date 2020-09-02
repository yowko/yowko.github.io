---
title: "ASP.NET Web API 上傳檔案出現 415 Unsupported Media Type 錯誤"
date: 2017-12-25T23:33:00+08:00
lastmod: 2018-10-02T23:33:46+08:00
draft: false
tags: ["ASP.NET Web API"]
slug: "aspnet-web-api-415-unsupported-media"
aliases:
    - /2017/12/aspnet-web-api-415-unsupported-media.html
---
# ASP.NET Web API 上傳檔案出現 415 Unsupported Media Type 錯誤
最近專案前端高互動性的頁面採用 Vue.js 來呈現，讓原本習慣的開發流程跟方式都需求做些調整，像是 Vue.js 存取資料部份因為沒有 server render html 的需要就使用 Web API 來處理，當然過程中小問題不斷，其中一個就是透過 Vue.js 上傳檔案時遇到 415 Unsupported Media Type 的錯誤，所以紀錄一下解決方式，畢竟日後前後端拆分開發的機會應該會更多

## 錯誤訊息

*   訊息內容

    ```
    {
        "Message": "The request entity's media type 'multipart/form-data' is not supported for this resource.",
        "ExceptionMessage": "No MediaTypeFormatter is available to read an object of type 'HttpPostedFileBase' from content with media type 'multipart/form-data'.",
        "ExceptionType": "System.Net.Http.UnsupportedMediaTypeException",
        "StackTrace": "   at System.Net.Http.HttpContentExtensions.ReadAsAsync[T](HttpContent content, Type type, IEnumerable`1 formatters, IFormatterLogger formatterLogger, CancellationToken cancellationToken)\r\n   at System.Web.Http.ModelBinding.FormatterParameterBinding.ReadContentAsync(HttpRequestMessage request, Type type, IEnumerable`1 formatters, IFormatterLogger formatterLogger, CancellationToken cancellationToken)"
    }
    ```

*   錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/34341093-2c8325a0-e9cb-11e7-8250-6a819e5f175c.png)

## 解決方式一：從 HttpContext 中取得檔案

1.  api 程式碼

    ```cs
    public class UploadController : ApiController
    {
        public HttpResponseMessage Post()
        {
            //取得當前的 request 物件
            var httpRequest = HttpContext.Current.Request;
            //request 如有夾帶檔案
            if (httpRequest.Files.Count > 0)
            {
                //逐一取得檔案名稱
                foreach (string fileName in httpRequest.Files.Keys)
                {
                    //以檔案名稱從 request 的 Files 集合取得檔案內容
                    var file = httpRequest.Files[fileName];
                    //其他檔案處理
                }
                            
                return Request.CreateResponse(HttpStatusCode.OK);
            }
                        
            return Request.CreateResponse(HttpStatusCode.BadRequest);
        }
    }
    ```

2.  前端上傳內容

    *   Content-Type：application/x-www-form-urlencoded
    *   body 夾檔

        ![2body](https://user-images.githubusercontent.com/3851540/34341094-2cb1be42-e9cb-11e7-9b59-6a738c7e47b3.png)

3.  順利取得檔案內容

    ![3result1](https://user-images.githubusercontent.com/3851540/34341095-2ce054c8-e9cb-11e7-8a75-d6b4016f2da5.png)

    ![4result2](https://user-images.githubusercontent.com/3851540/34341096-2d0a1bb4-e9cb-11e7-818d-303d720587c4.png)

## 解決方式二：使用 MultipartFormDataStreamProvider

1.  API 程式碼

    ```cs
    public class UploadController : ApiController
    {
        public HttpResponseMessage Post()
        {
            // 如果 Content-Type 沒有 multipart/form-data 就回傳 415
            if (!Request.Content.IsMimeMultipartContent())
            {
                throw new HttpResponseException(HttpStatusCode.UnsupportedMediaType);
            }
            //檔案儲存路徑
            string root = HttpContext.Current.Server.MapPath("~/App_Data");
            //建立 MultipartFormDataStreamProvider instance
            var provider = new MultipartFormDataStreamProvider(root);
            try
            {
                //從 request content 中讀取檔案並以 BodyPart_{GUID} 格式寫至上方定義的路徑中
                Request.Content.ReadAsMultipartAsync(provider);
                            // 取得實際檔案內容
                foreach (MultipartFileData file in provider.FileData)
                {
                //實際檔案處理 
                }
                return Request.CreateResponse(HttpStatusCode.OK);
            }
            catch (System.Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.InternalServerError, e);
            }
        }
    }
    ```

2.  前端上傳內容
    *   Content-Type：multipart/form-data

        > 使用 postman 時只要上傳內容選擇 `file` 會自動加上 `multipart/form-data`

    *   body 夾檔

        ![2body](https://user-images.githubusercontent.com/3851540/34341094-2cb1be42-e9cb-11e7-9b59-6a738c7e47b3.png)

## 心得

[ASP.NET WebApi: MultipartDataMediaFormatter](https://www.codeproject.com/Tips/652633/ASP-NET-WebApi-MultipartDataMediaFormatter) 這篇文章有提到如果使用 HttpClient 上傳至 Web API 可以透過安裝套件來解決，有需要請自行參閱(與今天主題有些落差就不特別介紹，日後若有用到再另文紀錄)

話說今天紀錄的內容滿虛的：一來是發生原因不是很肯定(個人不負責任推測是 Web API 底層實作與 MVC 不同有關 )；二來是方法二用到的 `MultipartFormDataStreamProvider` 看了一些文件仍然未全然掌握用途，不過沒關係相信有天實力增強時應該就有契機可以學得透徹些了

# 參考資訊

1.  [How to post file to ASP.NET Web Api 2](https://stackoverflow.com/questions/33387764/how-to-post-file-to-asp-net-web-api-2)
2.  [Sending HTML Form Data in ASP.NET Web API: File Upload and Multipart MIME](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/sending-html-form-data-part-2?WT.mc_id=DOP-MVP-5002594)
3.  [MultipartFormDataStreamProvider 類別](https://msdn.microsoft.com/zh-tw/library/system.net.http.multipartformdatastreamprovider%28v=vs.118%29.aspx)
4.  [ASP.NET WebApi: MultipartDataMediaFormatter](https://www.codeproject.com/Tips/652633/ASP-NET-WebApi-MultipartDataMediaFormatter)
5.  [ASP.NET WebApi: MultipartDataMediaFormatter](https://www.codeproject.com/Tips/652633/ASP-NET-WebApi-MultipartDataMediaFormatter)
