# Amazon Linux 2023でEC2にMySQL 8.0をインストールした際`Error: GPG check FAILED`とエラーになる
[参考サイト](https://it-slroom.blog/aws/al2023-mysql8-install/)

### エラー内容
`Error: GPG check FAILED` と表示される。
GPGキーが正しくないか、EC2内に存在していないためにエラーが発生している。

![image](https://github.com/user-attachments/assets/6c54327c-41db-448d-9d4b-045666dd0f44)

`/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022` 

### 解決
Amazon Linux 2023では、MySQLのGPGキー(2023)を取り込む必要があります。
```
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
```

![image](https://github.com/user-attachments/assets/1dd08dc4-95e7-46be-a1df-93fcb95b2706)
