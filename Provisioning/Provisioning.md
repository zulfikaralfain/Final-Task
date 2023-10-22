# FINAL TASK
## Provisioning

### Appserver
Disini kita buat multipass terlebih dahulu untuk membuat ansible nya, karna saya sudah mempunyai multipas maka saya tinggal masuk kedalam multipassnya. Jika sudah kita buat file, disini saya membuat tiga file yaitu *appserver* *gateway* *resource*. Jika suda kita masuk ke dalam file appserver, lalu dsini saya membuat script Docker yang dimana script ini akan menginstal Docker di dalam appserver.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/1.png?raw=true)
```
- hosts: all # masukan hosts yang ingin tuju (disini saya memasukan ke semua hosts)
  become: true
  vars:
    container_count: 4
    default_container_name: docker
    default_container_image: ubuntu
    default_container_command: sleep 1d

  tasks:
    - name: Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: yes

    - name: Correct Problem
      command: sudo dpkg --configure -a

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: yes

    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: yes

    - name: Install Docker Module for Python
      pip:
        name: docker

    - name: add user docker group
      user:
        name: zulfikar
        groups: docker
        append: yes

    - name: add user docker group
      user:
        name: zulfikar
        groups: docker
        append: yes
```
Jika sudah membuat script, langkah selanjutnya adalah menginstalnya menggunakan perintah `ansible-playbook nama_file.yml`, 
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/2.png?raw=true)
```
ansibel-playbook install-docker.yml
```
Langkah selanjutnya adalah mendeploy aplikasi yang bernama Node Exporter. Sebelum mendeploy, kita lakukan hal yang sama seperti menginstal Docker yaitu membuat script untuk mendeploy nodenya.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/16.png?raw=true)
```
- name: Deploy Node Exporter with Docker
  hosts: all
  become: true
  tasks:
    - name: Pull the bitnami/node-exporter Docker image
      docker_image:
        name: bitnami/node-exporter
        source: pull

    - name: Run the Node Exporter container
      docker_container:
        name: node-exp
        image: bitnami/node-exporter
        state: started
        restart_policy: unless-stopped
        published_ports:
          - "9100:9100"
```
Selanjutya kita jalankan *ansible-palybook* nya.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/3.png?raw=true)
```
ansible-playbook deploy-node.yml
```
Selanjutnya kita clone github, disini kita clone yang bernama *Dumbmerch*. Untuk mengclone github menggunakan ansible, kita lakukan hal yang serupa. yaitu membuat script untuk mengclone github Dumbmerch.
```
- name: Clone fe-dumbmerch and be-dumbmerch Repositories and Add Remotes
  hosts: appserver

  tasks:
    - name: Clone fe-dumbmerch repository
      git:
        repo: https://github.com/demo-dumbways/fe-dumbmerch.git
        dest: /home/zulfikar/fe-dumbmerch

    - name: Add remote "upstream" for fe-dumbmerch
      command: git remote add upstream https://github.com/demo-dumbways/fe-dumbmerch.git
      args:
        chdir: /home/zulfikar/fe-dumbmerch
      ignore_errors: yes

    - name: Clone be-dumbmerch repository
      git:
        repo: https://github.com/demo-dumbways/be-dumbmerch.git
        dest: /home/zulfikar/be-dumbmerch

    - name: Add remote "upstream" for be-dumbmerch
      command: git remote add upstream https://github.com/demo-dumbways/be-dumbmerch.git
      args:
        chdir: /home/zulfikar/be-dumbmerch

    - name: Copy file Dockerfile (backend)
      copy:
        src: ~/ansible/appserver/Dockerfile
        dest: /home/zulfikar/be-dumbmerch/Dockerfile
        owner: zulfikar
        group: zulfikar

    - name: Copy file Dockerfile (Frontend)
      copy:
        src: ~/ansible/resource/Dockerfile
        dest: /home/zulfikar/fe-dumbmerch
        owner: zulfikar
        group: zulfikar

    - name: Copy file docker-compose
      copy:
        src: ~/ansible/resource/docker-compose.yml
        dest: /home/zulfikar/docker-compose.yml
        owner: zulfikar
        group: zulfikar
```
Selanjutnya kita jalankan *ansible-playbook* nya.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/11.png?raw=true)
```
ansible-playbook deploy-dumbmerch.yml
```
Selanjutnya yaitu mendeploy aplikasi yang bernama Prometheus dan Grafana. Kita buat script untuk mendeploy aplikasi Prometheus dan Grafana di dalam docker.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/9.png?raw=true)
```
- name: Deploy Monitoring
  hosts: appserver
  become: true
  tasks:
    - name: Create directory for Prometheus config
      file:
        path: /home/zulfikar/prometheus
        state: directory
        mode: '0755'

    - name: Copy prometheus.yml configuration file
      copy:
        src: /home/ubuntu/ansible/resource/prometheus.yml
        dest: /home/zulfikar/prometheus/prometheus.yml
        mode: "0644"
    - name: Pull bitnami/prometheus Docker image
      docker_image:
        name: bitnami/prometheus
        source: pull

    - name: Run Prometheus container
      docker_container:
        name: prometheus
        image: bitnami/prometheus
        state: started
        ports:
          - "9090:9090"
        volumes:
          - /home/zulfikar/prometheus:/etc/prometheus
        container_default_behavior: compatibility

    - name: Pull Grafana Docker image
      docker_image:
        name: grafana/grafana
        source: pull

    - name: Run Grafana container
      docker_container:
        name: grafana
        image: grafana/grafana
        state: started
        ports:
          - "1404:3000"
```
Jika sudah kita jalankan *ansible-playbook*
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/10.png?raw=true)
```
ansible-playbook deploy-monitoring.yml
```

### Gateway
Sekarang kita masuk ke dalam file gateway, di dalam file gateway kita buat script untuk menginstal Nginx, yang dimana Nginx ini akan menjadi akses untuk masuk kedalam aplikasi seperti yang sudah kita deploy.Jika sudah membuat script langkah selanjutnya yaitu menjalankan perintahnya
```
- name: Update nginx
  hosts: gateway
  tasks:
    - name: Ensure nginx is at the latest version
      apt:
        name: nginx
        state: latest
        update_cache: yes
      become: yes

    - name: Start nginx
      service:
        name: nginx
        state: started
      become: yes

    - name: Copy the nginx config wayshub
      copy:
        src: ~/ansible/resource/rproxy.conf
        dest: /etc/nginx/sites-enabled/rproxy.conf
      become: yes

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      become: yes
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/7.png?raw=true)
```
ansible-playbook install-nginx.yml
```
Selanjutnya kita buat rproxy.conf, untuk membuat rproxy, kita masuk ke file resource. Jika sudah kita buat rproxynya yaitu `nano rproxy.conf`.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/5.png?raw=true)
```
server {
    server_name nodaapser.zulfi.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.150:9100;
    }
}

server {
    server_name nodgate.zulfi.studentdumbways.my.id;

    location / {
             proxy_pass http://103.150.93.47:9100;
    }
}

server {
    server_name pro.zulfi.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.150:9090;
    }
}

server {
    server_name gra.zulfi.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.150:1404;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    server_name zulfi.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.150:3000;
    }
}

server {
    server_name api.zulfi.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.150:5000;
    }
}
```
Karna kita sudah membuat rproxynya. Maka kita instal certbort yang dimana certbot ini fungsinya untuk mengamankan browser yang ingin kita jalankan. Sebelum menginstal certbot kita buat dulu script nya. 
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/4.png?raw=true)
```
- name: Install Certbot with Snap
  hosts: gateway
  tasks:
    - name: Install Certbot with Snap
      snap:
        name: certbot
        classic: yes
        state: present
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx dumbmerch-fe
      command: certbot --nginx -d zulfi.studentdumbways.my.id --non-interactive --agree-tos --email zulfikar.alfain1@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx dumbmerch-be
      command: certbot --nginx -d api.zulfi.studentdumbways.my.id --non-interactive --agree-tos --email zulfikar.alfain1@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx prometheus
      command: certbot --nginx -d pro.zulfi.studentdumbways.my.id --non-interactive --agree-tos --email zulfikar.alfain1@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx grafana
      command: certbot --nginx -d gra.zulfi.studentdumbways.my.id --non-interactive --agree-tos --email zulfikar.alfain1@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx node-appserver
      command: certbot --nginx -d nodaapser.zulfi.studentdumbways.my.id --non-interactive --agree-tos --email zulfikar.alfain1@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx node-gateway
      command: certbot --nginx -d nodgate.zulfi.studentdumbways.my.id --non-interactive --agree-tos --email zulfikar.alfain1@gmail.com
      become: yes

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      become: yes
```
Jika sudah kita instal certbotnya.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Provisioning/assets/8.png?raw=true)
```
ansible-playbook certbot.yml
```