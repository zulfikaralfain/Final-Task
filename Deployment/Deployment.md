# FINAL TASK
## Deployment

### Dumbmarch
Pertama kita masuk ke dalam file *fe-dumbmarch*, jika sudah kita buat Dockerfile untuk file yang dimana nanti untuk menyambungkan ke docker compose
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/7.png?raw=true)
```
FROM node:16-alpine as build
WORKDIR /dumbmerch-fe
COPY . /dumbmerch-fe
RUN yarn install

FROM node:16-alpine
COPY --from=build /dumbmerch-fe /dumbmerch-fe
WORKDIR /dumbmerch-fe
EXPOSE 3000
CMD ["yarn","run","start"]
```
Jika sudah kita buat *.env* file untuk fe-dumbmarch
```
nano .env
```
```
REACT_APP_BASEURL=https://api.zulfi.studentdumbways.my.id/api/v1
```
Langkah selanjutnya kita keluar dari file *fe-dumbmarch* dan masuk ke *be-dumbmarch*, jika sudah kita buat sama seperti *fe-dumbmarch* yaitu membuat Dockerfile.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/8.png?raw=true)
```
FROM golang:1.18-alpine
WORKDIR /app
COPY . .
RUN go build -o /dumbmerch
EXPOSE 5000
CMD [ "/dumbmerch" ]
```
Jika sudah kita ubah *.env* file yang ada di *be-dumbmerch*
```
nano .env
```
```
DB_HOST=103.175.221.150
DB_USER=zulfikar
DB_PASSWORD=alfhabet
DB_NAME=dumbmerch
DB_PORT=5432
PORT=5000
```
JIka sudah kita keluar dari file *be-dumbmerch* dan kita buat yang namanya *docker-compose.yml*
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/2.png?raw=true)
```
version: '3.8'
services:
  frontend:
    build: ./fe-dumbmerch
    container_name: frontend-container
    ports:
      - "3000:3000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  backend:
    build: ./be-dumbmerch
    container_name: backend-container
    ports:
      - "5000:5000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  postgres:
    image: "postgres"
    container_name: database-container
    environment:
      POSTGRES_PASSWORD: alfhabet
      POSTGRES_USER: zulfikar
      POSTGRES_DB: dumbmerch
    volumes:
      - "./data/postgresql:/var/lib/postgresql/data"
    networks:
      - network
    ports:
      - "5432:5432"
    restart: unless-stopped

networks:
   network:
```
Jika sudah kita jalankan perintahnya
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/3.png?raw=true)
```
docker compose up -d
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/4.png?raw=true)
```
docker ps -a
```
Jika berjalan maka tampilannya akan seperti ini
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/18.png?raw=true)
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/19.png?raw=true)
Selanjutnya kita masuk kedalam repository *fe-dumbmerch* dan *be-dumbmerch* terus kita buat CI/CD Git action. Kita mulai dulu yang *fe-dumbmerch*, sebelum ke git action kita klik setting yang ada di repo *fe-dumbmarch* lalu klik secret and variable
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/11.png?raw=true)
Lalu kita klik new repository secret dan kita buat *DOCKERHUB_TOKEN* dan *DOCKERHUB_USERNAME*. Untuk *DOCERHUB_USERNAME* kita tulis nama username kita yang ada di docker hub untuk token kita bisa buat *new token* yang ada di docker hub dengan cara masuk kedalam docker lalu klik account settings terus klik security dan klik new acces token.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/12.png?raw=true)
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/20.png?raw=true)
Jika sudah kita masuk ke dalam Git Action, lalu klik set up a workfile yourself, jika sudah kita masukan script CI/CD Git action nya. Disini saya buat di production dan staging
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/10.png?raw=true)
```
name: CI/CD Pipeline

on:
  push:
    branches: [production]
  pull_request:
    branches: [production]
  workflow_dispatch:
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'

#    - name: Git Pull
 #     run: |
  #      cd /github/home/fe-dumbmerch
   #     git pull

    - name: Build and Push Docker Image
      run: |
        docker build -t zulfikaralfain/fe-dumbmerch .
        docker push zulfikaralfain/fe-dumbmerch
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/zulfikar/fe-dumbmerch
          docker compose up -d
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/16.png?raw=true)
```
name: CI/CD Pipeline

on:
  push:
    branches: [staging]
  pull_request:
    branches: [staging]
  workflow_dispatch:
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'

#    - name: Git Pull
 #     run: |
  #      cd /github/home/fe-dumbmerch
   #     git pull

    - name: Build and Push Docker Image
      run: |
        docker build -t zulfikaralfain/fe-dumbmerch .
        docker push zulfikaralfain/fe-dumbmerch
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/zulfikar/fe-dumbmerch
          docker compose up -d
```
Jika sudah klik commit changes. Jika berhasil maka tampilannya akan seperti ini.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/13.png?raw=true)
Untuk *be-dumbmarch* kita lakukan hal yang sama pada *fe-dumbmerch* yaitu menambahkan *DOCKERHUB_TOKEN* dan *DOCKERHUB_USERNAME* di new repository secret.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/15.png?raw=true)
Lalu kita masuk ke Git Action kita buat Script CI/CD nya dan sama seperti *fe-dumbmerch* sebelumnya.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/14.png?raw=true)
```
name: CI/CD Pipeline

on:
  push:
    branches: [production]
  pull_request:
    branches: [production]
  workflow_dispatch:
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'

#    - name: Git Pull
 #     run: |
  #      cd /github/home/fe-dumbmerch
   #     git pull

    - name: Build and Push Docker Image
      run: |
        docker build -t zulfikaralfain/be-dumbmerch .
        docker push zulfikaralfain/be-dumbmerch
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/zulfikar/be-dumbmerch
          docker compose up -d
```
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/16.png?raw=true)
```
name: CI/CD Pipeline

on:
  push:
    branches: [staging]
  pull_request:
    branches: [staging]
  workflow_dispatch:
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'

#    - name: Git Pull
 #     run: |
  #      cd /github/home/fe-dumbmerch
   #     git pull

    - name: Build and Push Docker Image
      run: |
        docker build -t zulfikaralfain/be-dumbmerch .
        docker push zulfikaralfain/be-dumbmerch
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.150
        username: zulfikar
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/zulfikar/be-dumbmerch
          docker compose up -d
```
Jika kita sudah bisa menjalankan CI/CD Git action, maka repository *fe-dumbmerch* dan *be-dumbmerch* kita akan ada di docker hub kita.
![alt text](https://github.com/zulfikaralfain/Final-Task/blob/main/Deployment/assest/21.png?raw=true)