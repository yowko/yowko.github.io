---
title: "TimeZoneInfo 在 Mac/Linux 找不到 time zone ID"
date: 2019-05-10T21:30:00+08:00
lastmod: 2019-05-10T21:30:31+08:00
draft: flase
tags: ["C#","Linux","Mac"]
slug: "timezoneinfo-time-zone-id-not-found"
---
# TimeZoneInfo 在 Mac/Linux 找不到指定 time zone ID

同事提到 TimeZoneInfo 的操作在 Linux server 會出現錯誤，想說我怎麼沒遇到立馬試試，於是發現原來是 time zone ID 在 Windows 與 Linux 不同造成的 XD 雖說可以理解，但不免還是有些違詞：微軟這類的問題在過去可是層出不窮，就是要跟別人不一樣 XD 幸虧這個狀況近幾年有大幅改善 (我個人最有感的就是 docker command 的 porting，不同平台有幾乎一致的使用體驗)，問題既然已經出現就得要找解決方式，紀錄一下用法備查

## 基本環境說明

1. Mac/Linux : macOS Mojave 10.14.4
2. Windows : https://dotnetfiddle.net/ 與 Windows 10 Version 1803 (OS Build 17134.590)
3. .NET Core 2.2.101
4. NuGet 套件
   - TimeZoneConverter 3.1.0
5. 模擬程式碼

    ```cs
    TimeZoneInfo timeZone = TimeZoneInfo.FindSystemTimeZoneById("China Standard Time");
	var utcNow = DateTime.UtcNow;
    Console.WriteLine(utcNow);
    Console.WriteLine(TimeZoneInfo.ConvertTime(utcNow, timeZone));
    ```

5. Windows 平台結果

    ```
    5/10/2019 7:05:46 AM
    5/10/2019 3:05:46 PM
    ```


## 錯誤訊息

1. 訊息內容

    ```
    Unhandled Exception: System.TimeZoneNotFoundException: The time zone ID 'China Standard Time' was not found on the local computer. ---> System.IO.FileNotFoundException: Could not find file '/usr/share/zoneinfo/China Standard Time'.
   at Interop.ThrowExceptionForIoErrno(ErrorInfo errorInfo, String path, Boolean isDirectory, Func`2 errorRewriter)
   at Microsoft.Win32.SafeHandles.SafeFileHandle.Open(String path, OpenFlags flags, Int32 mode)
   at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, FileOptions options)
   at Internal.IO.File.ReadAllBytes(String path)
   at System.TimeZoneInfo.TryGetTimeZoneFromLocalMachine(String id, TimeZoneInfo& value, Exception& e)
   --- End of inner exception stack trace ---
   at System.TimeZoneInfo.FindSystemTimeZoneById(String id)
   at TestFluentd.Program.Main(String[] args) in /Users/yowko.tsai/Program.cs:line 30

    Process finished with exit code 6.
    ```

2. 錯誤截圖

    ![1error](https://user-images.githubusercontent.com/3851540/57514601-7554b700-7343-11e9-8173-7dbda44cc07b.png)

## 解決方式 1 : 使用套件 `TimeZoneConverter`

1. 安裝套件

    - Package Manager
    
        ```
        Install-Package TimeZoneConverter
        ```
    
    - .NET CLI

        ```
        dotnet add package TimeZoneConverter
        ```

2. 實際使用方式

    ```cs
    TimeZoneInfo timeZone =TZConvert.GetTimeZoneInfo("China Standard Time");
    var utcNow = DateTime.UtcNow;
    Console.WriteLine(utcNow);
    Console.WriteLine(TimeZoneInfo.ConvertTime(utcNow, timeZone));
    ```

## 解決方式 2 : 使用 TZ database name

1. Windows time zone 與 TZ ID 對應

    **Windows**|**TZ ID**
    :-----:|:-----:
    AUS Central Standard Time|Australia/Darwin
    AUS Central Standard Time|Australia/Darwin
    AUS Eastern Standard Time|Australia/Sydney
    AUS Eastern Standard Time|Australia/Sydney Australia/Melbourne
    Afghanistan Standard Time|Asia/Kabul
    Afghanistan Standard Time|Asia/Kabul
    Alaskan Standard Time|America/Anchorage
    Alaskan Standard Time|America/Anchorage America/Juneau America/Nome America/Sitka America/Yakutat
    Aleutian Standard Time|America/Adak
    Aleutian Standard Time|America/Adak
    Altai Standard Time|Asia/Barnaul
    Altai Standard Time|Asia/Barnaul
    Arab Standard Time|Asia/Riyadh
    Arab Standard Time|Asia/Bahrain
    Arab Standard Time|Asia/Kuwait
    Arab Standard Time|Asia/Qatar
    Arab Standard Time|Asia/Riyadh
    Arab Standard Time|Asia/Aden
    Arabian Standard Time|Asia/Dubai
    Arabian Standard Time|Asia/Dubai
    Arabian Standard Time|Asia/Muscat
    Arabian Standard Time|Etc/GMT-4
    Arabic Standard Time|Asia/Baghdad
    Arabic Standard Time|Asia/Baghdad
    Argentina Standard Time|America/Buenos\_Aires
    Argentina Standard Time|America/Buenos\_Aires America/Argentina/La\_Rioja America/Argentina/Rio\_Gallegos America/Argentina/Salta America/Argentina/San\_Juan America/Argentina/San\_Luis America/Argentina/Tucuman America/Argentina/Ushuaia America/Catamarca America/Cordoba America/Jujuy America/Mendoza
    Astrakhan Standard Time|Europe/Astrakhan
    Astrakhan Standard Time|Europe/Astrakhan Europe/Ulyanovsk
    Atlantic Standard Time|America/Halifax
    Atlantic Standard Time|Atlantic/Bermuda
    Atlantic Standard Time|America/Halifax America/Glace\_Bay America/Goose\_Bay America/Moncton
    Atlantic Standard Time|America/Thule
    Aus Central W. Standard Time|Australia/Eucla
    Aus Central W. Standard Time|Australia/Eucla
    Azerbaijan Standard Time|Asia/Baku
    Azerbaijan Standard Time|Asia/Baku
    Azores Standard Time|Atlantic/Azores
    Azores Standard Time|America/Scoresbysund
    Azores Standard Time|Atlantic/Azores
    Bahia Standard Time|America/Bahia
    Bahia Standard Time|America/Bahia
    Bangladesh Standard Time|Asia/Dhaka
    Bangladesh Standard Time|Asia/Dhaka
    Bangladesh Standard Time|Asia/Thimphu
    Belarus Standard Time|Europe/Minsk
    Belarus Standard Time|Europe/Minsk
    Bougainville Standard Time|Pacific/Bougainville
    Bougainville Standard Time|Pacific/Bougainville
    Canada Central Standard Time|America/Regina
    Canada Central Standard Time|America/Regina America/Swift\_Current
    Cape Verde Standard Time|Atlantic/Cape\_Verde
    Cape Verde Standard Time|Atlantic/Cape\_Verde
    Cape Verde Standard Time|Etc/GMT+1
    Caucasus Standard Time|Asia/Yerevan
    Caucasus Standard Time|Asia/Yerevan
    Cen. Australia Standard Time|Australia/Adelaide
    Cen. Australia Standard Time|Australia/Adelaide Australia/Broken\_Hill
    Central America Standard Time|America/Guatemala
    Central America Standard Time|America/Belize
    Central America Standard Time|America/Costa\_Rica
    Central America Standard Time|Pacific/Galapagos
    Central America Standard Time|America/Guatemala
    Central America Standard Time|America/Tegucigalpa
    Central America Standard Time|America/Managua
    Central America Standard Time|America/El\_Salvador
    Central America Standard Time|Etc/GMT+6
    Central Asia Standard Time|Asia/Almaty
    Central Asia Standard Time|Antarctica/Vostok
    Central Asia Standard Time|Asia/Urumqi
    Central Asia Standard Time|Indian/Chagos
    Central Asia Standard Time|Asia/Bishkek
    Central Asia Standard Time|Asia/Almaty Asia/Qostanay
    Central Asia Standard Time|Etc/GMT-6
    Central Brazilian Standard Time|America/Cuiaba
    Central Brazilian Standard Time|America/Cuiaba America/Campo\_Grande
    Central Europe Standard Time|Europe/Budapest
    Central Europe Standard Time|Europe/Tirane
    Central Europe Standard Time|Europe/Prague
    Central Europe Standard Time|Europe/Budapest
    Central Europe Standard Time|Europe/Podgorica
    Central Europe Standard Time|Europe/Belgrade
    Central Europe Standard Time|Europe/Ljubljana
    Central Europe Standard Time|Europe/Bratislava
    Central European Standard Time|Europe/Warsaw
    Central European Standard Time|Europe/Sarajevo
    Central European Standard Time|Europe/Zagreb
    Central European Standard Time|Europe/Skopje
    Central European Standard Time|Europe/Warsaw
    Central Pacific Standard Time|Pacific/Guadalcanal
    Central Pacific Standard Time|Antarctica/Macquarie
    Central Pacific Standard Time|Pacific/Ponape Pacific/Kosrae
    Central Pacific Standard Time|Pacific/Noumea
    Central Pacific Standard Time|Pacific/Guadalcanal
    Central Pacific Standard Time|Pacific/Efate
    Central Pacific Standard Time|Etc/GMT-11
    Central Standard Time|America/Chicago
    Central Standard Time|America/Winnipeg America/Rainy\_River America/Rankin\_Inlet America/Resolute
    Central Standard Time|America/Matamoros
    Central Standard Time|America/Chicago America/Indiana/Knox America/Indiana/Tell\_City America/Menominee America/North\_Dakota/Beulah America/North\_Dakota/Center America/North\_Dakota/New\_Salem
    Central Standard Time|CST6CDT
    Central Standard Time (Mexico)|America/Mexico\_City
    Central Standard Time (Mexico)|America/Mexico\_City America/Bahia\_Banderas America/Merida America/Monterrey
    Chatham Islands Standard Time|Pacific/Chatham
    Chatham Islands Standard Time|Pacific/Chatham
    China Standard Time|Asia/Shanghai
    China Standard Time|Asia/Shanghai
    China Standard Time|Asia/Hong\_Kong
    China Standard Time|Asia/Macau
    Cuba Standard Time|America/Havana
    Cuba Standard Time|America/Havana
    Dateline Standard Time|Etc/GMT+12
    Dateline Standard Time|Etc/GMT+12
    E. Africa Standard Time|Africa/Nairobi
    E. Africa Standard Time|Antarctica/Syowa
    E. Africa Standard Time|Africa/Djibouti
    E. Africa Standard Time|Africa/Asmera
    E. Africa Standard Time|Africa/Addis\_Ababa
    E. Africa Standard Time|Africa/Nairobi
    E. Africa Standard Time|Indian/Comoro
    E. Africa Standard Time|Indian/Antananarivo
    E. Africa Standard Time|Africa/Mogadishu
    E. Africa Standard Time|Africa/Juba
    E. Africa Standard Time|Africa/Dar\_es\_Salaam
    E. Africa Standard Time|Africa/Kampala
    E. Africa Standard Time|Indian/Mayotte
    E. Africa Standard Time|Etc/GMT-3
    E. Australia Standard Time|Australia/Brisbane
    E. Australia Standard Time|Australia/Brisbane Australia/Lindeman
    E. Europe Standard Time|Europe/Chisinau
    E. Europe Standard Time|Europe/Chisinau
    E. South America Standard Time|America/Sao\_Paulo
    E. South America Standard Time|America/Sao\_Paulo
    Easter Island Standard Time|Pacific/Easter
    Easter Island Standard Time|Pacific/Easter
    Eastern Standard Time|America/New\_York
    Eastern Standard Time|America/Nassau
    Eastern Standard Time|America/Toronto America/Iqaluit America/Montreal America/Nipigon America/Pangnirtung America/Thunder\_Bay
    Eastern Standard Time|America/New\_York America/Detroit America/Indiana/Petersburg America/Indiana/Vincennes America/Indiana/Winamac America/Kentucky/Monticello America/Louisville
    Eastern Standard Time|EST5EDT
    Eastern Standard Time (Mexico)|America/Cancun
    Eastern Standard Time (Mexico)|America/Cancun
    Egypt Standard Time|Africa/Cairo
    Egypt Standard Time|Africa/Cairo
    Ekaterinburg Standard Time|Asia/Yekaterinburg
    Ekaterinburg Standard Time|Asia/Yekaterinburg
    FLE Standard Time|Europe/Kiev
    FLE Standard Time|Europe/Mariehamn
    FLE Standard Time|Europe/Sofia
    FLE Standard Time|Europe/Tallinn
    FLE Standard Time|Europe/Helsinki
    FLE Standard Time|Europe/Vilnius
    FLE Standard Time|Europe/Riga
    FLE Standard Time|Europe/Kiev Europe/Uzhgorod Europe/Zaporozhye
    Fiji Standard Time|Pacific/Fiji
    Fiji Standard Time|Pacific/Fiji
    GMT Standard Time|Europe/London
    GMT Standard Time|Atlantic/Canary
    GMT Standard Time|Atlantic/Faeroe
    GMT Standard Time|Europe/London
    GMT Standard Time|Europe/Guernsey
    GMT Standard Time|Europe/Dublin
    GMT Standard Time|Europe/Isle\_of\_Man
    GMT Standard Time|Europe/Jersey
    GMT Standard Time|Europe/Lisbon Atlantic/Madeira
    GTB Standard Time|Europe/Bucharest
    GTB Standard Time|Asia/Famagusta Asia/Nicosia
    GTB Standard Time|Europe/Athens
    GTB Standard Time|Europe/Bucharest
    Georgian Standard Time|Asia/Tbilisi
    Georgian Standard Time|Asia/Tbilisi
    Greenland Standard Time|America/Godthab
    Greenland Standard Time|America/Godthab
    Greenwich Standard Time|Atlantic/Reykjavik
    Greenwich Standard Time|Africa/Ouagadougou
    Greenwich Standard Time|Africa/Abidjan
    Greenwich Standard Time|Africa/Accra
    Greenwich Standard Time|Africa/Banjul
    Greenwich Standard Time|Africa/Conakry
    Greenwich Standard Time|Africa/Bissau
    Greenwich Standard Time|Atlantic/Reykjavik
    Greenwich Standard Time|Africa/Monrovia
    Greenwich Standard Time|Africa/Bamako
    Greenwich Standard Time|Africa/Nouakchott
    Greenwich Standard Time|Atlantic/St\_Helena
    Greenwich Standard Time|Africa/Freetown
    Greenwich Standard Time|Africa/Dakar
    Greenwich Standard Time|Africa/Lome
    Haiti Standard Time|America/Port-au-Prince
    Haiti Standard Time|America/Port-au-Prince
    Hawaiian Standard Time|Pacific/Honolulu
    Hawaiian Standard Time|Pacific/Rarotonga
    Hawaiian Standard Time|Pacific/Tahiti
    Hawaiian Standard Time|Pacific/Johnston
    Hawaiian Standard Time|Pacific/Honolulu
    Hawaiian Standard Time|Etc/GMT+10
    India Standard Time|Asia/Calcutta
    India Standard Time|Asia/Calcutta
    Iran Standard Time|Asia/Tehran
    Iran Standard Time|Asia/Tehran
    Israel Standard Time|Asia/Jerusalem
    Israel Standard Time|Asia/Jerusalem
    Jordan Standard Time|Asia/Amman
    Jordan Standard Time|Asia/Amman
    Kaliningrad Standard Time|Europe/Kaliningrad
    Kaliningrad Standard Time|Europe/Kaliningrad
    Korea Standard Time|Asia/Seoul
    Korea Standard Time|Asia/Seoul
    Libya Standard Time|Africa/Tripoli
    Libya Standard Time|Africa/Tripoli
    Line Islands Standard Time|Pacific/Kiritimati
    Line Islands Standard Time|Pacific/Kiritimati
    Line Islands Standard Time|Etc/GMT-14
    Lord Howe Standard Time|Australia/Lord\_Howe
    Lord Howe Standard Time|Australia/Lord\_Howe
    Magadan Standard Time|Asia/Magadan
    Magadan Standard Time|Asia/Magadan
    Magallanes Standard Time|America/Punta\_Arenas
    Magallanes Standard Time|Antarctica/Palmer
    Magallanes Standard Time|America/Punta\_Arenas
    Marquesas Standard Time|Pacific/Marquesas
    Marquesas Standard Time|Pacific/Marquesas
    Mauritius Standard Time|Indian/Mauritius
    Mauritius Standard Time|Indian/Mauritius
    Mauritius Standard Time|Indian/Reunion
    Mauritius Standard Time|Indian/Mahe
    Middle East Standard Time|Asia/Beirut
    Middle East Standard Time|Asia/Beirut
    Montevideo Standard Time|America/Montevideo
    Montevideo Standard Time|America/Montevideo
    Morocco Standard Time|Africa/Casablanca
    Morocco Standard Time|Africa/El\_Aaiun
    Morocco Standard Time|Africa/Casablanca
    Mountain Standard Time|America/Denver
    Mountain Standard Time|America/Edmonton America/Cambridge\_Bay America/Inuvik America/Yellowknife
    Mountain Standard Time|America/Ojinaga
    Mountain Standard Time|America/Denver America/Boise
    Mountain Standard Time|MST7MDT
    Mountain Standard Time (Mexico)|America/Chihuahua
    Mountain Standard Time (Mexico)|America/Chihuahua America/Mazatlan
    Myanmar Standard Time|Asia/Rangoon
    Myanmar Standard Time|Indian/Cocos
    Myanmar Standard Time|Asia/Rangoon
    N. Central Asia Standard Time|Asia/Novosibirsk
    N. Central Asia Standard Time|Asia/Novosibirsk
    Namibia Standard Time|Africa/Windhoek
    Namibia Standard Time|Africa/Windhoek
    Nepal Standard Time|Asia/Katmandu
    Nepal Standard Time|Asia/Katmandu
    New Zealand Standard Time|Pacific/Auckland
    New Zealand Standard Time|Antarctica/McMurdo
    New Zealand Standard Time|Pacific/Auckland
    Newfoundland Standard Time|America/St\_Johns
    Newfoundland Standard Time|America/St\_Johns
    Norfolk Standard Time|Pacific/Norfolk
    Norfolk Standard Time|Pacific/Norfolk
    North Asia East Standard Time|Asia/Irkutsk
    North Asia East Standard Time|Asia/Irkutsk
    North Asia Standard Time|Asia/Krasnoyarsk
    North Asia Standard Time|Asia/Krasnoyarsk Asia/Novokuznetsk
    North Korea Standard Time|Asia/Pyongyang
    North Korea Standard Time|Asia/Pyongyang
    Omsk Standard Time|Asia/Omsk
    Omsk Standard Time|Asia/Omsk
    Pacific SA Standard Time|America/Santiago
    Pacific SA Standard Time|America/Santiago
    Pacific Standard Time|America/Los\_Angeles
    Pacific Standard Time|America/Vancouver America/Dawson America/Whitehorse
    Pacific Standard Time|America/Los\_Angeles America/Metlakatla
    Pacific Standard Time|PST8PDT
    Pacific Standard Time (Mexico)|America/Tijuana
    Pacific Standard Time (Mexico)|America/Tijuana America/Santa\_Isabel
    Pakistan Standard Time|Asia/Karachi
    Pakistan Standard Time|Asia/Karachi
    Paraguay Standard Time|America/Asuncion
    Paraguay Standard Time|America/Asuncion
    Romance Standard Time|Europe/Paris
    Romance Standard Time|Europe/Brussels
    Romance Standard Time|Europe/Copenhagen
    Romance Standard Time|Europe/Madrid Africa/Ceuta
    Romance Standard Time|Europe/Paris
    Russia Time Zone 10|Asia/Srednekolymsk
    Russia Time Zone 10|Asia/Srednekolymsk
    Russia Time Zone 11|Asia/Kamchatka
    Russia Time Zone 11|Asia/Kamchatka Asia/Anadyr
    Russia Time Zone 3|Europe/Samara
    Russia Time Zone 3|Europe/Samara
    Russian Standard Time|Europe/Moscow
    Russian Standard Time|Europe/Moscow Europe/Kirov Europe/Volgograd
    Russian Standard Time|Europe/Simferopol
    SA Eastern Standard Time|America/Cayenne
    SA Eastern Standard Time|Antarctica/Rothera
    SA Eastern Standard Time|America/Fortaleza America/Belem America/Maceio America/Recife America/Santarem
    SA Eastern Standard Time|Atlantic/Stanley
    SA Eastern Standard Time|America/Cayenne
    SA Eastern Standard Time|America/Paramaribo
    SA Eastern Standard Time|Etc/GMT+3
    SA Pacific Standard Time|America/Bogota
    SA Pacific Standard Time|America/Rio\_Branco America/Eirunepe
    SA Pacific Standard Time|America/Coral\_Harbour
    SA Pacific Standard Time|America/Bogota
    SA Pacific Standard Time|America/Guayaquil
    SA Pacific Standard Time|America/Jamaica
    SA Pacific Standard Time|America/Cayman
    SA Pacific Standard Time|America/Panama
    SA Pacific Standard Time|America/Lima
    SA Pacific Standard Time|Etc/GMT+5
    SA Western Standard Time|America/La\_Paz
    SA Western Standard Time|America/Antigua
    SA Western Standard Time|America/Anguilla
    SA Western Standard Time|America/Aruba
    SA Western Standard Time|America/Barbados
    SA Western Standard Time|America/St\_Barthelemy
    SA Western Standard Time|America/La\_Paz
    SA Western Standard Time|America/Kralendijk
    SA Western Standard Time|America/Manaus America/Boa\_Vista America/Porto\_Velho
    SA Western Standard Time|America/Blanc-Sablon
    SA Western Standard Time|America/Curacao
    SA Western Standard Time|America/Dominica
    SA Western Standard Time|America/Santo\_Domingo
    SA Western Standard Time|America/Grenada
    SA Western Standard Time|America/Guadeloupe
    SA Western Standard Time|America/Guyana
    SA Western Standard Time|America/St\_Kitts
    SA Western Standard Time|America/St\_Lucia
    SA Western Standard Time|America/Marigot
    SA Western Standard Time|America/Martinique
    SA Western Standard Time|America/Montserrat
    SA Western Standard Time|America/Puerto\_Rico
    SA Western Standard Time|America/Lower\_Princes
    SA Western Standard Time|America/Port\_of\_Spain
    SA Western Standard Time|America/St\_Vincent
    SA Western Standard Time|America/Tortola
    SA Western Standard Time|America/St\_Thomas
    SA Western Standard Time|Etc/GMT+4
    SE Asia Standard Time|Asia/Bangkok
    SE Asia Standard Time|Antarctica/Davis
    SE Asia Standard Time|Indian/Christmas
    SE Asia Standard Time|Asia/Jakarta Asia/Pontianak
    SE Asia Standard Time|Asia/Phnom\_Penh
    SE Asia Standard Time|Asia/Vientiane
    SE Asia Standard Time|Asia/Bangkok
    SE Asia Standard Time|Asia/Saigon
    SE Asia Standard Time|Etc/GMT-7
    Saint Pierre Standard Time|America/Miquelon
    Saint Pierre Standard Time|America/Miquelon
    Sakhalin Standard Time|Asia/Sakhalin
    Sakhalin Standard Time|Asia/Sakhalin
    Samoa Standard Time|Pacific/Apia
    Samoa Standard Time|Pacific/Apia
    Sao Tome Standard Time|Africa/Sao\_Tome
    Sao Tome Standard Time|Africa/Sao\_Tome
    Saratov Standard Time|Europe/Saratov
    Saratov Standard Time|Europe/Saratov
    Singapore Standard Time|Asia/Singapore
    Singapore Standard Time|Asia/Brunei
    Singapore Standard Time|Asia/Makassar
    Singapore Standard Time|Asia/Kuala\_Lumpur Asia/Kuching
    Singapore Standard Time|Asia/Manila
    Singapore Standard Time|Asia/Singapore
    Singapore Standard Time|Etc/GMT-8
    South Africa Standard Time|Africa/Johannesburg
    South Africa Standard Time|Africa/Bujumbura
    South Africa Standard Time|Africa/Gaborone
    South Africa Standard Time|Africa/Lubumbashi
    South Africa Standard Time|Africa/Maseru
    South Africa Standard Time|Africa/Blantyre
    South Africa Standard Time|Africa/Maputo
    South Africa Standard Time|Africa/Kigali
    South Africa Standard Time|Africa/Mbabane
    South Africa Standard Time|Africa/Johannesburg
    South Africa Standard Time|Africa/Lusaka
    South Africa Standard Time|Africa/Harare
    South Africa Standard Time|Etc/GMT-2
    Sri Lanka Standard Time|Asia/Colombo
    Sri Lanka Standard Time|Asia/Colombo
    Sudan Standard Time|Africa/Khartoum
    Sudan Standard Time|Africa/Khartoum
    Syria Standard Time|Asia/Damascus
    Syria Standard Time|Asia/Damascus
    Taipei Standard Time|Asia/Taipei
    Taipei Standard Time|Asia/Taipei
    Tasmania Standard Time|Australia/Hobart
    Tasmania Standard Time|Australia/Hobart Australia/Currie
    Tocantins Standard Time|America/Araguaina
    Tocantins Standard Time|America/Araguaina
    Tokyo Standard Time|Asia/Tokyo
    Tokyo Standard Time|Asia/Jayapura
    Tokyo Standard Time|Asia/Tokyo
    Tokyo Standard Time|Pacific/Palau
    Tokyo Standard Time|Asia/Dili
    Tokyo Standard Time|Etc/GMT-9
    Tomsk Standard Time|Asia/Tomsk
    Tomsk Standard Time|Asia/Tomsk
    Tonga Standard Time|Pacific/Tongatapu
    Tonga Standard Time|Pacific/Tongatapu
    Transbaikal Standard Time|Asia/Chita
    Transbaikal Standard Time|Asia/Chita
    Turkey Standard Time|Europe/Istanbul
    Turkey Standard Time|Europe/Istanbul
    Turks And Caicos Standard Time|America/Grand\_Turk
    Turks And Caicos Standard Time|America/Grand\_Turk
    US Eastern Standard Time|America/Indianapolis
    US Eastern Standard Time|America/Indianapolis America/Indiana/Marengo America/Indiana/Vevay
    US Mountain Standard Time|America/Phoenix
    US Mountain Standard Time|America/Dawson\_Creek America/Creston America/Fort\_Nelson
    US Mountain Standard Time|America/Hermosillo
    US Mountain Standard Time|America/Phoenix
    US Mountain Standard Time|Etc/GMT+7
    UTC|Etc/GMT
    UTC|America/Danmarkshavn
    UTC|Etc/GMT Etc/UTC
    UTC+12|Etc/GMT-12
    UTC+12|Pacific/Tarawa
    UTC+12|Pacific/Majuro Pacific/Kwajalein
    UTC+12|Pacific/Nauru
    UTC+12|Pacific/Funafuti
    UTC+12|Pacific/Wake
    UTC+12|Pacific/Wallis
    UTC+12|Etc/GMT-12
    UTC+13|Etc/GMT-13
    UTC+13|Pacific/Enderbury
    UTC+13|Pacific/Fakaofo
    UTC+13|Etc/GMT-13
    UTC-02|Etc/GMT+2
    UTC-02|America/Noronha
    UTC-02|Atlantic/South\_Georgia
    UTC-02|Etc/GMT+2
    UTC-08|Etc/GMT+8
    UTC-08|Pacific/Pitcairn
    UTC-08|Etc/GMT+8
    UTC-09|Etc/GMT+9
    UTC-09|Pacific/Gambier
    UTC-09|Etc/GMT+9
    UTC-11|Etc/GMT+11
    UTC-11|Pacific/Pago\_Pago
    UTC-11|Pacific/Niue
    UTC-11|Pacific/Midway
    UTC-11|Etc/GMT+11
    Ulaanbaatar Standard Time|Asia/Ulaanbaatar
    Ulaanbaatar Standard Time|Asia/Ulaanbaatar Asia/Choibalsan
    Venezuela Standard Time|America/Caracas
    Venezuela Standard Time|America/Caracas
    Vladivostok Standard Time|Asia/Vladivostok
    Vladivostok Standard Time|Asia/Vladivostok Asia/Ust-Nera
    W. Australia Standard Time|Australia/Perth
    W. Australia Standard Time|Antarctica/Casey
    W. Australia Standard Time|Australia/Perth
    W. Central Africa Standard Time|Africa/Lagos
    W. Central Africa Standard Time|Africa/Luanda
    W. Central Africa Standard Time|Africa/Porto-Novo
    W. Central Africa Standard Time|Africa/Kinshasa
    W. Central Africa Standard Time|Africa/Bangui
    W. Central Africa Standard Time|Africa/Brazzaville
    W. Central Africa Standard Time|Africa/Douala
    W. Central Africa Standard Time|Africa/Algiers
    W. Central Africa Standard Time|Africa/Libreville
    W. Central Africa Standard Time|Africa/Malabo
    W. Central Africa Standard Time|Africa/Niamey
    W. Central Africa Standard Time|Africa/Lagos
    W. Central Africa Standard Time|Africa/Ndjamena
    W. Central Africa Standard Time|Africa/Tunis
    W. Central Africa Standard Time|Etc/GMT-1
    W. Europe Standard Time|Europe/Berlin
    W. Europe Standard Time|Europe/Andorra
    W. Europe Standard Time|Europe/Vienna
    W. Europe Standard Time|Europe/Zurich
    W. Europe Standard Time|Europe/Berlin Europe/Busingen
    W. Europe Standard Time|Europe/Gibraltar
    W. Europe Standard Time|Europe/Rome
    W. Europe Standard Time|Europe/Vaduz
    W. Europe Standard Time|Europe/Luxembourg
    W. Europe Standard Time|Europe/Monaco
    W. Europe Standard Time|Europe/Malta
    W. Europe Standard Time|Europe/Amsterdam
    W. Europe Standard Time|Europe/Oslo
    W. Europe Standard Time|Europe/Stockholm
    W. Europe Standard Time|Arctic/Longyearbyen
    W. Europe Standard Time|Europe/San\_Marino
    W. Europe Standard Time|Europe/Vatican
    W. Mongolia Standard Time|Asia/Hovd
    W. Mongolia Standard Time|Asia/Hovd
    West Asia Standard Time|Asia/Tashkent
    West Asia Standard Time|Antarctica/Mawson
    West Asia Standard Time|Asia/Oral Asia/Aqtau Asia/Aqtobe Asia/Atyrau Asia/Qyzylorda
    West Asia Standard Time|Indian/Maldives
    West Asia Standard Time|Indian/Kerguelen
    West Asia Standard Time|Asia/Dushanbe
    West Asia Standard Time|Asia/Ashgabat
    West Asia Standard Time|Asia/Tashkent Asia/Samarkand
    West Asia Standard Time|Etc/GMT-5
    West Bank Standard Time|Asia/Hebron
    West Bank Standard Time|Asia/Hebron Asia/Gaza
    West Pacific Standard Time|Pacific/Port\_Moresby
    West Pacific Standard Time|Antarctica/DumontDUrville
    West Pacific Standard Time|Pacific/Truk
    West Pacific Standard Time|Pacific/Guam
    West Pacific Standard Time|Pacific/Saipan
    West Pacific Standard Time|Pacific/Port\_Moresby
    West Pacific Standard Time|Etc/GMT-10
    Yakutsk Standard Time|Asia/Yakutsk
    Yakutsk Standard Time|Asia/Yakutsk Asia/Khandyga

2. 實際使用

    ```cs
    TimeZoneInfo timeZone = TimeZoneInfo.FindSystemTimeZoneById("Asia/Shanghai");
    var utcNow = DateTime.UtcNow;
    Console.WriteLine(utcNow);
    Console.WriteLine(TimeZoneInfo.ConvertTime(utcNow, timeZone));
    ```

3. 通吃雙平台

    ```cs
    TimeZoneInfo timeZone;
    try {
        timeZone = TimeZoneInfo.FindSystemTimeZoneById("China Standard Time");
    }
    catch (Exception ex)
    {
        timeZone = TimeZoneInfo.FindSystemTimeZoneById("Asia/Shanghai");
    }

    var utcNow = DateTime.UtcNow;
    Console.WriteLine(utcNow);
    Console.WriteLine(TimeZoneInfo.ConvertTime(utcNow, timeZone));
    ```

## 心得 

`解法 1 ： 使用套件` 比較方便，缺點是對套件有相依，需要多管理成本

`解法 2 ： 使用 TZ database name` 沒有套件相依問題，但跨平台應用的寫法上可能需要調整，try catch 很方便，不過效能的耗損在頻繁使用下是無法忽略

# 參考資訊
1. [mj1856/TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter)
2. [FindSystemTimeZoneById doesn't support "Eastern Standard Time" on Ubuntu](https://github.com/dotnet/corefx/issues/16871)
3. [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
4. [Zone → Tzid](http://www.unicode.org/cldr/charts/latest/supplemental/zone_tzid.html)