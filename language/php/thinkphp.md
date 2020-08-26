# Thinkphp

## 

```
- A快速实例化Action类库
- B执行行为类
- C配置参数存取方法
- D快速实例化Model类库
- F快速简单文本数据存取方法
- L 语言参数存取方法
- M快速高性能实例化模型
- R快速远程调用Action类方法
- S快速缓存存取方法
- U URL动态生成和重定向方法
- W 快速Widget输出方法
- D函数实例化的是你当前项目的Lib/Model下面的模块。如果该模块不存在的话，直接返回实例化Model的对象(意义就与M()函数相同)。而M只返回，实例化Model的对象。它的$name参数作为数据库的表名来处理对数据库的操作。
```

## How to forward to one new page

```php
    <form action="{:U('Home/Index/qrcode')}" method="post">
        输入连接：http://<input type="text" name="url" value="baijunyao.com">
        <input type="submit" value="生成二维码">
    </form>
```
```html
<a href="{:U('Admin/Rule/add_admin')}">添加管理员</a>
```

```
        $url=U('Api/Weixinpay/pay',array('out_trade_no'=>$out_trade_no));
        // 前往支付
        redirect($url);
```

```php
$this->success('添加成功',U('Admin/Rule/admin_user_list'));
```

```php
	$user=M('users');
	$sql = $user->select();
	var_dump($sql);
```

```php
        define('NOW_TIME', $_SERVER['REQUEST_TIME']);
        define('REQUEST_METHOD', $_SERVER['REQUEST_METHOD']);
        define('IS_GET', REQUEST_METHOD == 'GET' ? true : false);
        define('IS_POST', REQUEST_METHOD == 'POST' ? true : false);
        define('IS_PUT', REQUEST_METHOD == 'PUT' ? true : false);
        define('IS_DELETE', REQUEST_METHOD == 'DELETE' ? true : false);
```

```
return array(

//*************************************数据库设置*************************************
    'DB_TYPE'               =>  'mysqli',                 // 数据库类型
    'DB_HOST'               =>  '127.0.0.1',     // 服务器地址
    'DB_NAME'               =>  'admin',     // 数据库名
    'DB_USER'               =>  'johnson',     // 用户名
    'DB_PWD'                =>  '073736',      // 密码
    'DB_PORT'               =>  '3306',     // 端口
    'DB_PREFIX'             =>  'admin_',   // 数据库表前缀
);
```
UsersModel.class.php

<?php
namespace Common\Model;
use Common\Model\BaseModel;


```php
    public function addData($data){
        // 对data数据进行验证
        var_dump($data);
        if(!$data=$this->create($data)){
            // 验证不通过返回错误
            return false;
        }else{
            // 验证通过
            $result=$this->add($data);
            return $result;
        }
    }
```