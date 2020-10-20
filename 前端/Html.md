## Css

bootstraptable的表格中columns属性通过rownum调整每列所占行数，用以下css调整样式居中。

```css
.table thead tr th{
    vertical-align: middle !important;
}
// !important '覆盖此元素其他样式'
```



## iframe

html用来存放前端下载的文件

```html
<iframe src="" style="display: none;" id="Hiframe"></iframe>
```

js中用如下方法添加文件

```javascript
$("#Hiframe").attr("src", "score/extMonthPayroll?cost_month="+cost_month);
```