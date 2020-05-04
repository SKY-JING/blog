<div class="my_title" style="text-align: center; font-weight: 700; font-size: 2em;">php中转</div>

前端开发的时候经常可能遇到的一个问题就是接口请求跨域，即我们的前端页面和需要请求的接口不在同一个域下。

此时我们通常想到的办法是用jsonp格式发送请求，但是有时候接口是第三方提供的，并不支持jsonp。这时我们可以采用php这类后台服务器语言对请求进行中转便可达到想要的结果。

下面是自己模拟的一个中转，支持get形式和post形式，支持form data请求和payload请求，可以中转http接口和https接口：

```php
<?php
  // 设置请求头
  // header('Content-type:text/html;charset=utf-8');
  // 根据ajax设置数据提交类型和方式获取数据，动态设置请求方式
  if($_SERVER['REQUEST_METHOD'] == 'POST') {
    $ispost = 1; // post
    if($_SERVER['CONTENT_TYPE'] == 'application/json') {
      $type = 1;
      $data = json_decode(file_get_contents( "php://input"), true); // request payload
    } else {
      $type = 0;
      $data = $_POST; // form data
      echo $data;
    }
  } else if ($_SERVER['REQUEST_METHOD']=='GET') {
    $type = 0;
    $ispost = 0; // get
    $data = $_GET;
  }

  // 获取数据中的url和其他参数
  foreach($data as $key=>$val) {
    if($key == 'url') {
      $url = $val;
    }else {
      $params[$key] = $val;
    }
  }

  // 如果获取参数时参数为空，初始化参数
  if (!isset($params)) {
    if($type == 1) { // payload形式
      $params = (object) array();
    } else { // form data形式
      $params = '';
    }
  }

  // 检测请求地址是http的还是https的
  if(strstr($url,"https")) {
    $isHttps = 1;
  }else {
    $isHttps = 0;
  }

  // 参数是否设置（因为params提前定义了，所以此处不需要再判断，只需要根据数据类型进行合适的编码）
  if ($type == 1) {
    $paramstring = json_encode($params);
  } else {
    $paramstring = http_build_query($params);
  }

  $content = juhecurl($url, $paramstring, $ispost, $type, $isHttps);
  echo $content;

  /**
   * 请求接口返回内容
   * @param  string $url [请求的URL地址]
   * @param  string $params [请求的参数]
   * @param  int $ipost [是否采用POST形式]
   * @return  string
   */
  function juhecurl($url, $params=false, $ispost=1, $type = 0, $isHttps = 0){
    $httpInfo = array();
    $ch = curl_init();
    //curl_setopt( $ch, CURLOPT_HTTP_VERSION , CURL_HTTP_VERSION_1_1 );
    //curl_setopt( $ch, CURLOPT_USERAGENT , 'JuheData' );
    if($isHttps == 1) { // 如果是请求地址是https，加入下面两句避免https签名认证
      curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
      curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
    }
    curl_setopt( $ch, CURLOPT_CONNECTTIMEOUT , 60 ); // 链接超时时间
    curl_setopt( $ch, CURLOPT_TIMEOUT , 60); // 执行超时时间
    curl_setopt( $ch, CURLOPT_RETURNTRANSFER , true ); // 将结果保存在数据中，不直接输出
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true); // 设置可跟踪爬取重定向页面
    if( $ispost )
    {
      curl_setopt( $ch , CURLOPT_POST , true ); // 启动post
      curl_setopt( $ch , CURLOPT_POSTFIELDS , $params ); // post参数
      curl_setopt( $ch , CURLOPT_URL , $url ); // post地址
      if($type) {
        curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/json'));
      }
    }
    else
    {
      if($params){
        curl_setopt( $ch , CURLOPT_URL , $url.'?'.$params ); // 默认get方式，带参数
      }else{
        curl_setopt( $ch , CURLOPT_URL , $url); // 默认get方式，不带参数
      }
    }
    $response = curl_exec( $ch ); // 执行请求
    if ($response === FALSE) {
      return false;
    }
    $httpCode = curl_getinfo( $ch , CURLINFO_HTTP_CODE ); // 获取http状态码
    $httpInfo = array_merge( $httpInfo , curl_getinfo( $ch ) ); // 将状态码和数据合并
    curl_close( $ch ); // 关闭会话
    return $response;
  }
?>
```

将此中转文件放入应用，然后按下面格式进行ajax请求即可：

```js
// payload形式
$.ajax({
  url: '中转文件路径',
  type: 'POST',
  contentType: 'application/json',
  data: JSON.stringify({"url": '原始请求地址'}), // 参数需要将原始路径带入
  success: function(data) {
    console.log(data);
  }
});

// form data形式
$.ajax({
  url: '中转文件路径',
  type: 'POST',
  data: {
    "url": '原始请求地址' // 参数需要将原始路径带入
  },
  success: function(data) {
    console.log(data);
  }
});

// get请求
$.ajax({
  url: '中转文件路径',
  type: 'GET',
  data: {
    "url": '原始请求地址' // 参数需要将原始路径带入
  },
  success: function(data) {
    console.log(data);
  }
});
```