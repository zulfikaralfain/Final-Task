# FINAL TASK
## Monitoring


Disini saya menggunakan template membuat tampilan dashboardnya, tapi sebelum itu kita atur dulu data sourcenya prometheus-nya agar nanti ketika membuat dashboard dan alerting nya kebaca 
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/1.png?raw=true)
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/2.png?raw=true)
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/3.png?raw=true)
Lalu disini saya mengimport dashboard yang ada di web grafananya
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/4.png?raw=true)
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/5.png?raw=true)
Selanjutnya disini untuk contact pointnya saya menggunakan discord yang dimana nanti pesan alertingnya mengirim notifikasi ke discord
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/6.png?raw=true)
Disini kita masuk ke alerting rulesnya dan kita buat 3 alerting yaitu CPU, RAM dan Free disk.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/7.png?raw=true)
```
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle",job="grafana"}[5m])) * 100)
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/9.png?raw=true)
```
100 * (1 - ((avg_over_time(node_memory_MemFree_bytes[10m]) + avg_over_time(node_memory_Buffers_bytes[10m])) / avg_over_time(node_memory_MemTotal_bytes[10m])))
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/10.png?raw=true)
```
max(100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)) by (instance)
```
Jika berhasil maka akan terkirim notifikasinya ke discord
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Monitoring/assets/8.png?raw=true)