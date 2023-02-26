# step by step manual  
## 0. Overview  
System architecture  
![System architecture](../docs/images/rproxy-outline.png)   

## 1. web server A  
```bash
echo "<html><head><title> A </title><meta http-equiv='Content-Type' content='text/html; charset=utf-8' /></head><body><h1> A </h1></body></html>" > ~/index.html 
# nginx web server run with index.html  
sudo docker run -d --rm -v ~/index.html:/usr/share/nginx/html/index.html -p 8881:80 --name web-a nginx  
```

## 2. jupyter lab web server  
```bash
# sudo docker run -itd --rm --name jupyter -p 8888:8888 shwsun/jupyter-spark jupyter lab --allow-root --ip='*' --NotebookApp.allow_origin='*' --NotebookApp.notebook_dir='/tf' --workspace='/tf' --NotebookApp.token='' --NotebookApp.password='' --NotebookApp.base_url='/jupyter'
sudo docker run -itd --rm --name jupyter -p 8888:8888 shwsun/jupyter-spark jupyter lab --allow-root --ip='*' --NotebookApp.allow_origin='*' --NotebookApp.notebook_dir='/tf' --workspace='/tf' --NotebookApp.token='' --NotebookApp.password=''  --NotebookApp.base_url='/jupyter'   
# # Ctrl + p, Ctrl + q 
```
## 3. jupyter lab web server 2    
```bash
sudo docker run -itd --rm --name jupyter -p 8888:8888 shwsun/jupyter-spark jupyter lab --allow-root --ip='*' --NotebookApp.allow_origin='*' --NotebookApp.notebook_dir='/tf' --workspace='/tf' --NotebookApp.token='' --NotebookApp.password=''  --NotebookApp.base_url='/jupyter2'   
# # Ctrl + p, Ctrl + q 
```
## 4. http proxy  
- run apache reverse proxy server  
```bash
# 3. apache docker run
sudo docker run -d --rm --name proxy -p 80:80 httpd
# sudo docker run -d --rm --name proxy -p 80:80  httpd:alpine3.17
sudo docker exec -it proxy /bin/bash

# 필수 라이브러리 설치 : vi, ps  
apt-get update 
apt-get install -y vim 
apt-get install -y procps
# hosts 편집  

# httpd conf 편집  : ServerName, mod_proxy, mod_proxy_http, access_log, error_log , Include vhosts.conf
# vi /usr/local/apache2/conf/httpd.conf  
# mod_proxy.so, mod_proxy_http.so, mod_proxy_wstunnel.so
# vi /usr/local/apache2/conf/extra/httpd-vhosts.conf  
sudo docker cp httpd.conf proxy:/usr/local/apache2/conf/httpd.conf
sudo docker cp ./extra/httpd-vhosts.conf proxy:/usr/local/apache2/conf/extra/httpd-vhosts.conf 

# apache 재시작  
/usr/local/apache2/bin/apachectl restart
```

