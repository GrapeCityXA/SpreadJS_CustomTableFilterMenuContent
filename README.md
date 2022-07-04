# SpreadJS_CustomTableFilterMenuContent
在纯前端在线表格中实现自定义表格筛选菜单内容功能
# SpreadJS_CustomTableFilterMenuContent

### SpreadJS 示例，自定义表格筛选菜单内容
该示例包括使用 SpreadJS API 的演示脚本，可用于实现自定义表格筛选菜单内容。
有关 SpreadJS API 的更多信息，请参阅[SpreadJS API指南]( https://demo.grapecity.com.cn/spreadjs/help/api/) 和[帮助手册]( https://help.grapecity.com.cn/pages/viewpage.action?pageId=5963808)。



### 运行步骤
1、在开始之前，请确保您已满足以下先决条件：
要运行 SpreadJS，浏览器必须支持 HTML5，客户端导入和导出 Excel 需要 IE10及以上。
请先了解 [SpreadJS 的产品使用环境]( https://www.grapecity.com.cn/developer/spreadjs/selection-guide/product-use-environment)，并申请临时部署授权激活
安装并更新NodeJS和NPM
2、克隆或下载此代码库
3、初始化控件，并运行示例脚本
#### 控件初始化
首先，创建一个新页面，并在页面上输入以下代码：
```
<!DOCTYPE html>
    <html>
    <head>
        <title>SpreadJS HTML Test Page</title>
```
2、在页面中添加对 SpreadJS 的引用。代码如下。需要注意的是，SpreadJS 提供压缩过
```
//（minified）的 JavaScript 文件和和用于调试的文件：
<script src="[Your_Scripts_Path]/gc.spread.sheets.all.xxxx.min.js" type="text/javascript"></script>
```
3、添加 CSS 文件以改变Spread.JS 的外观。默认的CSS文件名为： 
gc.spread.sheets.xxxx.css，里面包含了所有的默认样式。该 CSS 文件将会影响滚动条，筛选框及其子元素，单元格和下方标签栏的样式。引入 CSS 的代码如下：
```
//<link href="[Your_CSS_Path]/gc.spread.sheets.xxxx.css" rel="stylesheet" type="text/css"/>
```
4、添加产品授权，代码为（本地测试可以不添加）：
```
GC.Spread.Sheets.LicenseKey = "xxx";
```
5. 添加控件初始化代码。本例会在一个 id 为 “ss” 的 DOM 元素上初始化 SpreadJS：
```
<script type="text/javascript">
// Add your license
// If run this in local for testing, remove or comment below code
 GC.Spread.Sheets.LicenseKey = "xxx";

// Add your code
 window.onload = function(){
var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"),{sheetCount:3});
var sheet = spread.getActiveSheet();
 }
</script>
</head>
<body>
```
6、 创建一个 id 为 “ss” 的元素，SpreadJS 将在该 DOM 中初始化：
```
<div id="ss" style="height: 500px; width: 800px"></div>
</body>
</html>
```
#### 示例代码
```
HTML：
<p class="title">自定义筛选菜单</p>
<h6>E2单元格的筛选弹出框框是自定义的弹出框，F2、G2是默认筛选弹出框</h6>
<div id='ss'></div>

CSS：
#ss {
    height: 400px;
    width: 100%
}

p.title{
    color: #336699;
    text-align: center;
}

JavaScript：
// Title:自定义筛选对话框
// Description：点击单元格E2筛选按钮弹出自定义弹出框
// Tag:筛选框、对话框、自定义
GC.Spread.Common.CultureManager.culture('zh-cn');
var ns = GC.Spread.Sheets;

function customFilterDialog(sheet, filterHitInfo) {
    this._sheet = sheet;
    this._filterHitInfo = filterHitInfo;
    this._container = null;
    this.init();
}

customFilterDialog.prototype.init = function() {
    var $overlay = $("<div><div style='position: absolute;width:180px; border: 1px solid;background-color: #fff;height:100px;backgroundcolor:gray;'><label style='position: absolute;top:5px;left:5px;'>最小值</label><input id='min' style='position: absolute;left: 60px;top: 5px;width:100px;height:20px' min='0' max='100' /><label style='position: absolute;top:35px;left:5px;'>最大值</label><input id='max' style='position: absolute;width:100px;left: 60px;height:20px;top:35px;' min='0' max='100' /><button id='filter' style='position: absolute;width:100px; height:30px;top:66px;left:50px;'>确定</button></div></div>");
    $overlay.css("width", 100000);
    $overlay.css("height", 100000);
    $overlay.css("left", 0);
    $overlay.css("top", 0);
    $overlay.css("z-index", 100000)
    $overlay.css("position", "absolute");
    $overlay.css("display", "hidden");
    this._container = $overlay[0];
    $overlay.appendTo($(document.body));
}

customFilterDialog.prototype.open = function() {
    var sheet = this._sheet,
        tempSpread = sheet.getParent(),
        self = this;
    $(self._container).css("display", "display");
    var x = self._filterHitInfo.x + self._filterHitInfo.width + tempSpread.getHost().offsetLeft;
    var y = self._filterHitInfo.y + self._filterHitInfo.height + tempSpread.getHost().offsetTop;
    $(self._container).children().css({
        "left": x,
        "top": y
    });
    if (window.filterMaxValue) {
        $(self._container).children().val(window.filterMaxValue);
    }
    $(self._container).bind("mousedown", function(event) {
        if (event.target === self._container) {
            self.close();
        }
    });
    document.getElementById('filter').addEventListener('click', function() {
        self.doFilter();
        self.close();
    });
}

customFilterDialog.prototype.close = function() {
    window.filterMaxValue = +$(this._container).children().val();
    $(this._container).remove();
    this._container = null;
}

customFilterDialog.prototype.doFilter = function() {
    var colIndex = this._filterHitInfo.col;
    var drf = this._filterHitInfo.rowFilter;
    drf.removeFilterItems(colIndex);

    //When close, create condition with the value which fetched from the dialog UI.
    minCondition = new ns.ConditionalFormatting.Condition(ns.ConditionalFormatting.ConditionType.cellValueCondition, {
        compareType: ns.ConditionalFormatting.GeneralComparisonOperators.greaterThan,
        //  expected: +$(this._container).children()[0].min
        expected: document.getElementById("min").value
    });
    maxCondition = new ns.ConditionalFormatting.Condition(ns.ConditionalFormatting.ConditionType.cellValueCondition, {
        compareType: ns.ConditionalFormatting.GeneralComparisonOperators.lessThan,
        // expected: +$(this._container).children().val()
        expected: document.getElementById("max").value
    });
    relationCondition = new ns.ConditionalFormatting.Condition(ns.ConditionalFormatting.ConditionType.relationCondition, {
        compareType: ns.ConditionalFormatting.LogicalOperators.and,
        item1: minCondition,
        item2: maxCondition
    });
    drf.addFilterItem(colIndex, relationCondition);

    this._sheet.suspendPaint(true);
    //Execute the filter behavior.
    drf.filter(colIndex);
    this._sheet.resumePaint(false);
}

//overwrite openFilterDialog and create our own dialog here.

var oldOpenFilterDialog = GC.Spread.Sheets.Filter.HideRowFilter.prototype.openFilterDialog;
GC.Spread.Sheets.Filter.HideRowFilter.prototype.openFilterDialog = function(filterButtonHitInfo) {
    var sheet = GC.Spread.Sheets.findControl("ss").getActiveSheet();
    console.log(filterButtonHitInfo);
    if (filterButtonHitInfo.col == 4) { //第四列自定义筛选弹框
        var filterDialog = new customFilterDialog(sheet, filterButtonHitInfo);
        filterDialog.open();
    } else {
        oldOpenFilterDialog.apply(this, [filterButtonHitInfo]);
    }
}

$(document).ready(function() {
    var spread = new GC.Spread.Sheets.Workbook(document.getElementById('ss'), {
        sheetCount: 3
    });
    var sheet = spread.getActiveSheet();
    var salesData = [
        ["SalesPers", "Birth", "Region", "SaleAmt", "ComPct", "ComAmt"],
        ["Joe", "2000/01/23", "North", 260, 0.1, 26],
        ["Robert", "1988/08/21", "South", 660, 0.15, 99],
        ["Michelle", "1995/08/03", "East", 940, 0.15, 141],
        ["Erich", "1994/05/23", "West", 410, 0.12, 49.2],
        ["Dafna", "1992/07/21", "North", 800, 0.15, 120],
        ["Rob", "1995/11/03", "South", 900, 0.15, 135],
        ["Jonason", "1987/02/11", "West", 300, 0.17, 110],
        ["Enana", "1997/04/01", "West", 310, 0.16, 99.2],
        ["Dania", "1997/02/15", "North", 500, 0.10, 76],
        ["Robin", "1991/12/28", "East", 450, 0.18, 35]
    ];
    sheet.setArray(1, 1, salesData);
    sheet.setColumnWidth(2, 100);
    var filter = new GC.Spread.Sheets.Filter.HideRowFilter(new GC.Spread.Sheets.Range(2, 4, salesData.length - 1, salesData[0].length - 3));
    sheet.rowFilter(filter);

});

```

#### 关于 SpreadJS
[SpreadJS]( https://www.grapecity.com.cn/developer/spreadjs) 是一款基于 HTML5 的纯前端表格控件，兼容 450 多种 Excel 公式，具备“高性能、跨平台、与 Excel 高度兼容”的产品特性。使用 SpreadJS，可直接在 Angular、 React、 Vue 等前端框架中实现高效的模板设计、在线编辑和数据绑定等功能，为最终用户提供高度类似 Excel 的使用体验。


