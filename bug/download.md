# 1、当某些直链可以直接打开的文件(例如 .txt)放进压缩包时, 部门电脑下载后无法解压

## 问题出现原因
    B2C C端客户上传了一些txt的文件, B端客户需要查阅这些文件, 发现无法解压
### 出现问题的代码
```php
<?php
$zip = new ZipArchive();
$zip->open($tmpFile,ZipArchive::CREATE);
$file = file_get_contents('test.txt');
$zip->addFromString(sprintf("%s.%s",'test','txt'), $file);
$zip->close();
header("Content-Type:application/zip");
header('Content-disposition: attachment; filename='.$fileName);
header('Content-Length: ' . filesize($tmpFile));

ob_end_clean();
readfile($tmpFile);
unlink($tmpFile);

```
### 原因分析
    由于txt文件在浏览器上能直接打开并输出其内容, 
    而php由属于脚本语言, 
    故在file_get_contents获取到的内容后无法中断而导致问题的出现
    ps: 于2021-08-07测试了一下出问题的代码发现新版chorme已经不会出现这种情况
### 解决方案
    在最后一行代码中最后添加 exit(); 让程序退出即可

# 2、大文件下载内存不足的问题
## 问题出现原因
    由于逐行读取文件且内存未释放, 导致内存超出
    Fatal error: Allowed memory size of 134217728 bytes exhausted (tried to allocate 1099 bytes) in
### 出现问题的代码
```php
<?php
readfile($file_name);
```
### 解决方案
    使用切片下载, 值得注意的是缓存区分多层, 需逐个缓存区关闭
```php
<?php
$CHUNK_SIZE = 1024 * 1024 * 2; // 2M
$handle = fopen($file_name, 'rb');

fseek($handle, 0);
if ($handle === false) {
    return false;
}
while (@ob_end_flush()); // 清除所有缓存区
// 开启缓冲区
while (!feof($handle)) {
    echo fread($handle, $CHUNK_SIZE);
    @ob_flush();
    flush();
}
```