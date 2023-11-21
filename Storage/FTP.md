# FTP

## login

```bash
ftp [options] [IP]

sftp -P $port username@$hostname
```

| FTP Command Options | **Description**                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------- |
| **`-4`**            | Use only IPv4.                                                                                          |
| **`-6`**            | Use only IPv6.                                                                                          |
| **`-e`**            | Disables command editing and history support.                                                           |
| **`-p`**            | Uses passive mode for data transfers, allowing you to use FTP despite a firewall that might prevent it. |
| **`-i`**            | Turns off interactive prompting during multiple file transfers.                                         |
| **`-n`**            | Disables auto-login attempts on initial connection.                                                     |
| **`-g`**            | Disables file name globbing.                                                                            |
| **`-v`**            | Enables verbose output.                                                                                 |
| **`-d`**            | Enables debugging.                                                                                      |

## FTP commands

```bash
# list directories
ls
# 传送src.txt到本地并且文件名不变化
get src.tx
# 传送src.txt到本地并且文件名变为dest.txt
get src.txt dest.txt
# The mget command allows you to transfer multiple files at the same time.
# For example, transferring test01.txt, test02.txt, and test03.txt from 
# the Example directory
mget test01.txt test02.txt test03.txt

# 传送本地文件/home/ftp/my-file.txt到ftp当前目录，文件名不变
put /home/ftp/my-file.txt
# 传送本地文件到ftp服务器，文件名变更为my-file-0.txt
put /home/ftp/my-file.txt my-file-0.txt
# Use the mput command to transfer multiple files to the remote system. 
# For example, transfer test04.txt, test05.txt, and test06.txt with
mput test04.txt test05.txt test06.txt

# Rename Files
# Use the rename command to rename files on the remote server. 
# The rename command uses the following syntax
rename [old file name] [new file name]
rename sample01.txt sample_file01.txt

# Delete Files
# The delete command allows you to delete a file on the remote system. 
# It uses the following syntax:
delete [file name]
delete sample_file01.txt
# Using the mdelete command allows you to delete multiple files at the 
# same time by adding the file names after the command:
mdelete test04.txt test05.txt test06.txt

# sftp采用rm和rmdir命令进行删除
rm myfile.txt


```

## References



[linux-ftp](https://phoenixnap.com/kb/linux-ftp)

[How to Use SFTP Command to Transfer Files | Linuxize](https://linuxize.com/post/how-to-use-linux-sftp-command-to-transfer-files/)
