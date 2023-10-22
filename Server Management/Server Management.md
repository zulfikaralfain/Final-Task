# FINAL TASK
## Server Management

Disini saya masuk kedalam *ssh* yang ada di multipass saya, lalu saya buat config untuk appserver dan gateway
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Server%20Management/assets/1.png?raw=true)
```
host gateway
    HostName 103.150.93.47
    User zulfikar

host appserver
    Hostname 103.175.221.150
    User zulfikar
    ProxyCommand ssh gateway -W %h:%p
    IdentityFile /home/ubuntu/.ssh/id_rsa
```
Selanjutnya kita masuk kedalam VM appserver dan gateway lalu kita ubah *PasswordAuthentication* menjadi no 
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Server%20Management/assets/5.png?raw=true)
```
sudo nano /etc/ssh/sshd_config
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Server%20Management/assets/4.png?raw=true)
*PasswordAuthentication no*
Jika sudah kita masuk ke dalam multipass dan coba jalankan VM appserver dan gateway melalui multipass

Appserver
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Server%20Management/assets/2.png?raw=true)

Gateway
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Server%20Management/assets/3.png?raw=true)