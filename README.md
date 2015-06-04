# xsshtml-demo

关于过滤结果存在`<p>`标签对的问题，目前先用临时方案来解决

```php
/**
 * 支持单个或批量
 * @param array $items 待处理数组，引用变量。若传入字符串，则当做单个来处理。
 * @param bool $fullfilter 默认处理数组全部元素
 * @param array $needKeys 需要处理的元素所在的键，该参数仅在$fullfilter=false时有效
 */
function xssFilter(&$items = array(), $fullfilter = true, $needKeys = array()){
    $filter = function(&$v){
        $xss = new XssHtml($v);
        $v = $xss->getHtml();
        $v = preg_replace(array('/\\n\<p\>/is', '/\<\/p\>\\n/is'), '', $v);
    };
    if (is_string($items)) {
        $filter($items);
        return;
    }
    foreach ($items as $k => &$v) {
        if ($fullfilter || in_array($k, $needKeys) && is_string($v)) {
            $filter($v);
        }
    }
}
```

调用示例：

```php
$data = array(
    'a' => '<script>alert()</script>', 
    'b' => 2,
    'c' => 'javascript:void(0)',
);
$data1 = $data2 = $data;
$data3 = '<div onload="javascript:location.href='/';"></div>';

xssFilter($data1); //批量处理：针对全部字段过滤
xssFilter($data2, false, array('a')); //批量处理：针对a字段对应的内容进行过滤
xssFilter($data3); //单个字符串处理

var_dump(compact('data1', 'data2', 'data3'));
```

[→本家传送门](https://github.com/phith0n/XssHtml/issues/2)
