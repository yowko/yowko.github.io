---
title: "製作 Selenium IDE 的 xUnit.net 2.0 版 Formatter"
date: 2017-06-07T21:00:00+08:00
lastmod: 2017-06-07T21:00:09+08:00
draft: false
tags: ["套件","xUnit","Tools","C#"]
slug: "selenium-ide-xunit-formatter"
aliases:
    - /2017/06/selenium-ide-xunitnet-20-formatter.html
---
# 製作 Selenium IDE 的 xUnit.net 2.0 版 Formatter
TDD 課程中，91 大介紹了 Selenium IDE 的用法，我的心得筆記請參考 [使用 Selenium IDE 與 C# 做 Web UI 測試](//blog.yowko.com/2017/06/selenium-ide-c-sharp-web-ui-test.html)，因為 Selenium IDE 預設只支援 Nunit，所以 91 大動手做了個 MSTest 的版本，詳細內容請參考 [Export to C#/WebDriver/MSTest](https://dotblogs.com.tw/hatelove/archive/2013/11/26/selenium-ide-export-to-csharp-webdriver-mstest.aspx)，當下就想到好像每次 xUnit 都被忽略，於是就興起自己做 xUnit Selenium IDE Formatter 的念頭，就來看看要怎麼修改吧

## 修改 formatter

細節就不多提，有興趣的人可以仔細研究，大意就是引用 xUnit 的 namespace 並使用 xUnit 的寫法來調整 ，程式碼在如下，也放上 [GitHub：C-sharp-xUnit.net-2.0-WebDriver](https://github.com/yowko/C-sharp-xUnit.net-2.0-WebDriver)

```
/*
* Formatter for Selenium 2 / WebDriver .NET (C#) client.
*/

if (!this.formatterType) {  // this.formatterType is defined for the new Formatter system
  // This method (the if block) of loading the formatter type is deprecated.
  // For new formatters, simply specify the type in the addPluginProvidedFormatter() and omit this
  // if block in your formatter.
  var subScriptLoader = Components.classes["@mozilla.org/moz/jssubscript-loader;1"].getService(Components.interfaces.mozIJSSubScriptLoader);
  subScriptLoader.loadSubScript('chrome://selenium-ide/content/formats/webdriver.js', this);
}

function testClassName(testName) {
  return testName.split(/[^0-9A-Za-z]+/).map(
      function(x) {
        return capitalize(x);
      }).join('');
}

function testMethodName(testName) {
  return "The" + capitalize(testName) + "Test";
}

function nonBreakingSpace() {
  return "\"\\u00a0\"";
}

function array(value) {
  var str = 'new String[] {';
  for (var i = 0; i < value.length; i++) {
    str += string(value[i]);
    if (i < value.length - 1) str += ", ";
  }
  str += '}';
  return str;
}

Equals.prototype.toString = function() {
  return this.e1.toString() + " == " + this.e2.toString();
};

Equals.prototype.assert = function() {
  return "Assert.Equal(" + this.e1.toString() + ", " + this.e2.toString() + ");";
};

Equals.prototype.verify = function() {
  return verify(this.assert());
};

NotEquals.prototype.toString = function() {
  return this.e1.toString() + " != " + this.e2.toString();
};

NotEquals.prototype.assert = function() {
  return "Assert.NotEqual(" + this.e1.toString() + ", " + this.e2.toString() + ");";
};

NotEquals.prototype.verify = function() {
  return verify(this.assert());
};

function joinExpression(expression) {
  return "String.Join(\",\", " + expression.toString() + ")";
}

function statement(expression) {
  return expression.toString() + ';';
}

function assignToVariable(type, variable, expression) {
  return capitalize(type) + " " + variable + " = " + expression.toString();
}

function ifCondition(expression, callback) {
  return "if (" + expression.toString() + ")\n{\n" + callback() + "}";
}

function assertTrue(expression) {
  return "Assert.True(" + expression.toString() + ");";
}

function assertFalse(expression) {
  return "Assert.False(" + expression.toString() + ");";
}

function verify(statement) {
  return  statement ;
}

function verifyTrue(expression) {
  return verify(assertTrue(expression));
}

function verifyFalse(expression) {
  return verify(assertFalse(expression));
}

RegexpMatch.patternToString = function(pattern) {
  if (pattern != null) {
    //value = value.replace(/^\s+/, '');
    //value = value.replace(/\s+$/, '');
    pattern = pattern.replace(/\\/g, '\\\\');
    pattern = pattern.replace(/\"/g, '\\"');
    pattern = pattern.replace(/\r/g, '\\r');
    pattern = pattern.replace(/\n/g, '(\\n|\\r\\n)');
    return '"' + pattern + '"';
  } else {
    return '""';
  }
};

RegexpMatch.prototype.toString = function() {
  return "Regex.IsMatch(" + this.expression + ", " + RegexpMatch.patternToString(this.pattern) + ")";
};

function waitFor(expression) {
  return "for (int second = 0;; second++) {\n" +
      indents(1) + 'if (second >= 60) Assert.Fail("timeout");\n' +
      indents(1) + "try\n" +
      indents(1) + "{\n" +
      (expression.setup ? indents(2) + expression.setup() + "\n" : "") +
      indents(2) + "if (" + expression.toString() + ") break;\n" +
      indents(1) + "}\n" +
      indents(1) + "catch (Exception)\n" +
      indents(1) + "{}\n" +
      indents(1) + "Thread.Sleep(1000);\n" +
      "}";
}

function assertOrVerifyFailure(line, isAssert) {
  var message = '"expected failure"';
  var failStatement = isAssert ? "Assert.Fail(" + message + ");" :
      "verificationErrors.Append(" + message + ");";
  return "try\n" +
      "{\n" +
      line + "\n" +
      failStatement + "\n" +
      "}\n" +
      "catch (Exception) {}\n";
}

function pause(milliseconds) {
  return "Thread.Sleep(" + parseInt(milliseconds, 10) + ");";  
}

function echo(message) {
  return "Console.WriteLine(" + xlateArgument(message) + ");";
}

function formatComment(comment) {
  return comment.comment.replace(/.+/mg, function(str) {
    return "// " + str;
  });
}

function keyVariable(key) {
  return "Keys." + key;
}

this.sendKeysMaping = {
  BKSP: "Backspace",
  BACKSPACE: "Backspace",
  TAB: "Tab",
  ENTER: "Enter",
  SHIFT: "Shift",
  CONTROL: "Control",
  CTRL: "Control",
  ALT: "Alt",
  PAUSE: "Pause",
  ESCAPE: "Escape",
  ESC: "Escape",
  SPACE: "Space",
  PAGE_UP: "PageUp",
  PGUP: "PageUp",
  PAGE_DOWN: "PageDown",
  PGDN: "PageDown",
  END: "End",
  HOME: "Home",
  LEFT: "Left",
  UP: "Up",
  RIGHT: "Right",
  DOWN: "Down",
  INSERT: "Insert",
  INS: "Insert",
  DELETE: "Delete",
  DEL: "Delete",
  SEMICOLON: "Semicolon",
  EQUALS: "Equal",

  NUMPAD0: "NumberPad0",
  N0: "NumberPad0",
  NUMPAD1: "NumberPad1",
  N1: "NumberPad1",
  NUMPAD2: "NumberPad2",
  N2: "NumberPad2",
  NUMPAD3: "NumberPad3",
  N3: "NumberPad3",
  NUMPAD4: "NumberPad4",
  N4: "NumberPad4",
  NUMPAD5: "NumberPad5",
  N5: "NumberPad5",
  NUMPAD6: "NumberPad6",
  N6: "NumberPad6",
  NUMPAD7: "NumberPad7",
  N7: "NumberPad7",
  NUMPAD8: "NumberPad8",
  N8: "NumberPad8",
  NUMPAD9: "NumberPad9",
  N9: "NumberPad9",
  MULTIPLY: "Multiply",
  MUL: "Multiply",
  ADD: "Add",
  PLUS: "Add",
  SEPARATOR: "Separator",
  SEP: "Separator",
  SUBTRACT: "Subtract",
  MINUS: "Subtract",
  DECIMAL: "Decimal",
  PERIOD: "Decimal",
  DIVIDE: "Divide",
  DIV: "Divide",

  F1: "F1",
  F2: "F2",
  F3: "F3",
  F4: "F4",
  F5: "F5",
  F6: "F6",
  F7: "F7",
  F8: "F8",
  F9: "F9",
  F10: "F10",
  F11: "F11",
  F12: "F12",

  META: "Meta",
  COMMAND: "Command"
};

/**
 * Returns a string representing the suite for this formatter language.
 *
 * @param testSuite  the suite to format
 * @param filename   the file the formatted suite will be saved as
 */
function formatSuite(testSuite, filename) {
  var suiteClass = /^(\w+)/.exec(filename)[1];
  suiteClass = suiteClass[0].toUpperCase() + suiteClass.substring(1);

  var formattedSuite = "using NXunit;\n"
      + "\n"
      + "namespace " + this.options.namespace + "\n"
      + '{\n'
      + indents(1) + "public class " + suiteClass + "\n"
      + indents(1) + '{\n'
      + indents(2) + "[Suite] public static TestSuite Suite\n"
      + indents(2) + '{\n'
      + indents(3) + "get\n"
      + indents(3) + '{\n'
      + indents(4) + 'TestSuite suite = new TestSuite("'+ suiteClass +'");\n';

  for (var i = 0; i < testSuite.tests.length; ++i) {
    var testClass = testSuite.tests[i].getTitle();
    formattedSuite += indents(4)
        + "suite.Add(new " + testClass + "());\n";
  }

  formattedSuite += indents(4) + "return suite;\n"
      + indents(3) + "}\n"
      + indents(2) + "}\n"
      + indents(1) + "}\n"
      + "}\n";

  return formattedSuite;
}

function defaultExtension() {
  return this.options.defaultExtension;
}

this.options = {
  receiver: "driver",
  showSelenese: 'false',
  namespace: "SeleniumTests",
  indent: '4',
  initialIndents:  '3',
  header:
  'using System;\n' +
          'using System.Text;\n' +
          'using System.Text.RegularExpressions;\n' +
          'using System.Threading;\n' +
          'using OpenQA.Selenium;\n' +
          'using OpenQA.Selenium.Firefox;\n' +
          'using OpenQA.Selenium.Support.UI;\n' +
    'using Xunit;\n' +
          '\n' +
          'namespace ${namespace}\n' +
          '{\n' +
          '    public class ${className} : IDisposable\n' +
          '    {\n' +
          '        private IWebDriver driver;\n' +
          '        private StringBuilder verificationErrors;\n' +
          '        private string baseURL;\n' +
          "        private bool acceptNextAlert = true;\n" +
          '        \n' +
          '        public ${className}()\n' +
          '        {\n' +
          '            ${receiver} = new FirefoxDriver();\n' +
          '            baseURL = "${baseURL}";\n' +
          '            verificationErrors = new StringBuilder();\n' +
          '        }\n' +
          '        \n' +
          '        public void Dispose()\n' +
          '        {\n' +
          '            try\n' +
          '            {\n' +
          '                ${receiver}.Quit();\n' +
          '            }\n' +
          '            catch (Exception)\n' +
          '            {\n' +
          '                // Ignore errors if unable to close the browser\n' +
          '            }\n' +
          '            Assert.Equal("", verificationErrors.ToString());\n' +
          '        }\n' +
          '        \n' +
          '        [Fact]\n' +
          '        public void ${methodName}()\n' +
          '        {\n',
  footer:
          '        }\n' +
          "        private bool IsElementPresent(By by)\n" +
          "        {\n" +
          "            try\n" +
          "            {\n" +
          "                driver.FindElement(by);\n" +
          "                return true;\n" +
          "            }\n" +
          "            catch (NoSuchElementException)\n" +
          "            {\n" +
          "                return false;\n" +
          "            }\n" +
          "        }\n" +
          '        \n' +
          "        private bool IsAlertPresent()\n" +
          "        {\n" +
          "            try\n" +
          "            {\n" +
          "                driver.SwitchTo().Alert();\n" +
          "                return true;\n" +
          "            }\n" +
          "            catch (NoAlertPresentException)\n" +
          "            {\n" +
          "                return false;\n" +
          "            }\n" +
          "        }\n" +
          '        \n' +
          "        private string CloseAlertAndGetItsText() {\n" +
          "            try {\n" +
          "                IAlert alert = driver.SwitchTo().Alert();\n" +
          "                string alertText = alert.Text;\n" +
          "                if (acceptNextAlert) {\n" +
          "                    alert.Accept();\n" +
          "                } else {\n" +
          "                    alert.Dismiss();\n" +
          "                }\n" +
          "                return alertText;\n" +
          "            } finally {\n" +
          "                acceptNextAlert = true;\n" +
          "            }\n" +
          "        }\n" +
          '    }\n' +
          '}\n',
  defaultExtension: "cs"
};
this.configForm = '<description>Variable for Selenium instance</description>' +
    '<textbox id="options_receiver" />' +
    '<description>Namespace</description>' +
    '<textbox id="options_namespace" />' +
    '<checkbox id="options_showSelenese" label="Show Selenese"/>';

this.name = "C# (WebDriver)";
this.testcaseExtension = ".cs";
this.suiteExtension = ".cs";
this.webdriver = true;

WDAPI.Driver = function() {
  this.ref = options.receiver;
};

WDAPI.Driver.searchContext = function(locatorType, locator) {
  var locatorString = xlateArgument(locator);
  switch (locatorType) {
    case 'xpath':
      return 'By.XPath(' + locatorString + ')';
    case 'css':
      return 'By.CssSelector(' + locatorString + ')';
    case 'id':
      return 'By.Id(' + locatorString + ')';
    case 'link':
      return 'By.LinkText(' + locatorString + ')';
    case 'name':
      return 'By.Name(' + locatorString + ')';
    case 'tag_name':
      return 'By.TagName(' + locatorString + ')';
  }
  throw 'Error: unknown strategy [' + locatorType + '] for locator [' + locator + ']';
};

WDAPI.Driver.prototype.back = function() {
  return this.ref + ".Navigate().Back()";
};

WDAPI.Driver.prototype.close = function() {
  return this.ref + ".Close()";
};

WDAPI.Driver.prototype.findElement = function(locatorType, locator) {
  return new WDAPI.Element(this.ref + ".FindElement(" + WDAPI.Driver.searchContext(locatorType, locator) + ")");
};

WDAPI.Driver.prototype.findElements = function(locatorType, locator) {
  return new WDAPI.ElementList(this.ref + ".FindElements(" + WDAPI.Driver.searchContext(locatorType, locator) + ")");
};

WDAPI.Driver.prototype.getCurrentUrl = function() {
  return this.ref + ".Url";
};

WDAPI.Driver.prototype.get = function(url) {
  if (url.length > 1 && (url.substring(1,8) == "http://" || url.substring(1,9) == "https://")) { // url is quoted
    return this.ref + ".Navigate().GoToUrl(" + url + ")";
  } else {
    return this.ref + ".Navigate().GoToUrl(baseURL + " + url + ")";
  }
};

WDAPI.Driver.prototype.getTitle = function() {
  return this.ref + ".Title";
};

WDAPI.Driver.prototype.getAlert = function() {
  return "CloseAlertAndGetItsText()";
};

WDAPI.Driver.prototype.chooseOkOnNextConfirmation = function() {
  return "acceptNextAlert = true";
};

WDAPI.Driver.prototype.chooseCancelOnNextConfirmation = function() {
  return "acceptNextAlert = false";
};

WDAPI.Driver.prototype.refresh = function() {
  return this.ref + ".Navigate().Refresh()";
};

WDAPI.Element = function(ref) {
  this.ref = ref;
};

WDAPI.Element.prototype.clear = function() {
  return this.ref + ".Clear()";
};

WDAPI.Element.prototype.click = function() {
  return this.ref + ".Click()";
};

WDAPI.Element.prototype.getAttribute = function(attributeName) {
  return this.ref + ".GetAttribute(" + xlateArgument(attributeName) + ")";
};

WDAPI.Element.prototype.getText = function() {
  return this.ref + ".Text";
};

WDAPI.Element.prototype.isDisplayed = function() {
  return this.ref + ".Displayed";
};

WDAPI.Element.prototype.isSelected = function() {
  return this.ref + ".Selected";
};

WDAPI.Element.prototype.sendKeys = function(text) {
  return this.ref + ".SendKeys(" + xlateArgument(text) + ")";
};

WDAPI.Element.prototype.submit = function() {
  return this.ref + ".Submit()";
};

WDAPI.Element.prototype.select = function(selectLocator) {
  if (selectLocator.type == 'index') {
    return "new SelectElement(" + this.ref + ").SelectByIndex(" + selectLocator.string + ")";
  }
  if (selectLocator.type == 'value') {
    return "new SelectElement(" + this.ref + ").SelectByValue(" + xlateArgument(selectLocator.string) + ")";
  }
  return "new Select(" + this.ref + ").SelectByText(" + xlateArgument(selectLocator.string) + ")";
};


WDAPI.ElementList = function(ref) {
  this.ref = ref;
};

WDAPI.ElementList.prototype.getItem = function(index) {
  return this.ref + "[" + index + "]";
};

WDAPI.ElementList.prototype.getSize = function() {
  return this.ref + ".Count";
};

WDAPI.ElementList.prototype.isEmpty = function() {
  return this.ref + ".Count == 0";
};

WDAPI.Utils = function() {
};

WDAPI.Utils.isElementPresent = function(how, what) {
  return "IsElementPresent(" + WDAPI.Driver.searchContext(how, what) + ")";
};

WDAPI.Utils.isAlertPresent = function() {
  return "IsAlertPresent()";
};
```

## 匯入 Selenium IDE

> 準備好 formatter 的執行語法後就需要將語法匯入 Selenium IDE 備用

1.  Selenium IDE 主選單 Options --> Options

    ![1options](https://user-images.githubusercontent.com/3851540/26862277-78321c4c-4b7c-11e7-8b40-9b8ce2cfc72c.png)

2.  Formats --> Add

    ![2add](https://user-images.githubusercontent.com/3851540/26862278-7833bc64-4b7c-11e7-802a-d16319e15f16.png)

3.  給個顯示名稱 --> 再把 script 貼進編輯視窗中 --> Save

    ![3name](https://user-images.githubusercontent.com/3851540/26862279-78359e12-4b7c-11e7-9802-4e53d40140fc.png)

4.  存檔後，需要重開 Options --> Options ，剛加入的 formatter 才會出現

    ![4newone](https://user-images.githubusercontent.com/3851540/26862280-783ed784-4b7c-11e7-87cd-733e9c36dee5.png)

## 如何使用

使用細節可以參考 [使用 Selenium IDE 與 C# 做 Web UI 測試](//blog.yowko.com/2017/06/selenium-ide-c-sharp-web-ui-test.html) 步驟大致如下：

1.  使用 Selenium IDE 錄製 web 操作
2.  匯出 `.cs` 並指定 foramtter
3.  將匯出的 `.cs` 加入測試專案
4.  引用適合的 NuGet packages


# 參考資訊

1.  [使用 Selenium IDE 與 C# 做 Web UI 測試](//blog.yowko.com/2017/06/selenium-ide-c-sharp-web-ui-test.html)
2.  [Export to C#/WebDriver/MSTest](https://dotblogs.com.tw/hatelove/archive/2013/11/26/selenium-ide-export-to-csharp-webdriver-mstest.aspx)
3.  [GitHub：C-sharp-xUnit.net-2.0-WebDriver](https://github.com/yowko/C-sharp-xUnit.net-2.0-WebDriver)
