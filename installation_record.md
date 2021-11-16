## linux环境下gmssl安装步骤
```powershell
./config
make
make test
sudo make install
```

## 安装过程问题
### 问题1

**问题**: 执行`./config`命令报错
```powershell
rootrebecca@rootrebecca:GmSSL-master$ ./config 
Operating system: x86_64-whatever-linux2
"glob" is not exported by the File::Glob module
Can't continue after import errors at ./Configure line 18.
BEGIN failed--compilation aborted at ./Configure line 18.
"glob" is not exported by the File::Glob module
Can't continue after import errors at ./Configure line 18.
BEGIN failed--compilation aborted at ./Configure line 18.
This system (linux-x86_64) is not supported. See file INSTALL for details.
```

**问题原因**
```powershell
rootrebecca@rootrebecca:下载$ perl -v
This is perl 5, version 30, subversion 0 (v5.30.0) built for x86_64-linux-gnu-thread-multi
...
```
由于在perl 5.30版本中，`File::Glob::glob()`移除了，所以使用`File::Glob::bsd_glob()`. 参考[链接](https://github.com/wazuh/wazuh/issues/4054)

**解决办法**

```powershell
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/Configure
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/test/build.info
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/test/run_tests.pl
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/test/recipes/40-test_rehash.t
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/test/recipes/80-test_ssl_new.t
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/test/recipes/90-test_fuzz.t
sed -i "s#'File::Glob' => qw/glob/;#'File::Glob' => qw/bsd_glob/;#g" ./GmSSL/util/process_docs.pl
```

### 问题2

**问题**: 执行`gmssl`命令报错
```powershell
rootrebecca@rootrebecca:~$ gmssl 
gmssl: symbol lookup error: gmssl: undefined symbol: BIO_debug_callback, version OPENSSL_1_1_0d
```

**解决办法**
```powershell
rootrebecca@rootrebecca:~$ export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
rootrebecca@rootrebecca:~$ gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
```

## 一些说明

### 说明1
`gmssl`安装后，`openssl`命令被软连接到`gmssl`命令上了，即`ln -s gmssl openssl`
```powershell
root@rootrebecca:~# ll /usr/local/bin/|grep -Ei "openssl|gmssl"
-rwxr-xr-x  1 root        root          879136 11月 16 15:36 gmssl*
lrwxrwxrwx  1 root        root               5 11月 16 17:12 openssl -> gmssl*

rootrebecca@rootrebecca:~$ gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
rootrebecca@rootrebecca:~$
rootrebecca@rootrebecca:~$ openssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
```

如果需要恢复到原始的`openssl`命令，执行以下解除软连接命令
```powershell
root@rootrebecca:~# cd /usr/local/bin
root@rootrebecca:bin# unlink openssl
root@rootrebecca:bin# ll|grep -Ei "openssl|gmssl"
-rwxr-xr-x  1 root        root          879136 11月 16 15:36 gmssl*
```

执行下面命令查看两个命令版本(在link被解除的情况下查看)
```powershell
root@rootrebecca:bin# export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
root@rootrebecca:bin# gmssl version
GmSSL 2.5.4 - OpenSSL 1.1.0d  19 Jun 2019
root@rootrebecca:bin#
root@rootrebecca:bin# openssl version
openssl: /usr/local/lib/libssl.so.1.1: version `OPENSSL_1_1_1' not found (required by openssl)
openssl: /usr/local/lib/libssl.so.1.1: version `OPENSSL_1_1_0' not found (required by openssl)
openssl: /usr/local/lib/libcrypto.so.1.1: version `OPENSSL_1_1_1' not found (required by openssl)
openssl: /usr/local/lib/libcrypto.so.1.1: version `OPENSSL_1_1_0' not found (required by openssl)
root@rootrebecca:bin#
root@rootrebecca:bin# unset LD_LIBRARY_PATH
root@rootrebecca:bin# /usr/bin/openssl version
OpenSSL 1.1.1f  31 Mar 2020
```