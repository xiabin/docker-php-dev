docker-php-dev
===============================

基于docker的php开发环境，内部可以定制php任何版本，xdebug,mysql，nginx，reddis,memcached,mongodb。目前只用到这些，可以根据个人情况酌情添加。
#目录结构
```$xslt
mysql                       mysql挂载
nginx                       nginx镜像
    Dockerfile
    nginx.conf
php 
    php56                   php5.6镜像
    php7                    php7镜像
sites                       存放nginx的虚拟主机配置
    default.vhost           配置文件
www                         站点根目录
```


#使用
```$xslt
cd /path/to/your/project && docker-compose build #构建镜像
docker-compose up -d #运行容器
```
然后访问`http://localhost`会看到phpinfo页面

#AQ
### 1. 如何添加对应的php扩展
docker提供了`docker-php-ext-configure`,`docker-php-ext-install`,`docker-php-ext-enable`来管理扩展
例如：
```$xslt
FROM php:7.0-fpm
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
```
或者 pecl扩展安装,例如：
```$xslt
FROM php:7.1-fpm
RUN pecl install redis-3.1.0 \
    && pecl install xdebug-2.5.0 \
    && docker-php-ext-enable redis xdebug
FROM php:5.6-fpm
RUN apt-get update && apt-get install -y libmemcached-dev zlib1g-dev \
    && pecl install memcached-2.2.0 \
    && docker-php-ext-enable memcached
```
更多请移步[https://hub.docker.com/_/php/](https://hub.docker.com/_/php/)
### 2.docker镜像设置时区为`Asia/Shanghai`
利用docker-compose构建的时候目前采用的是利用dockerfile来修改,添加如下命令：
```$xslt
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
或者在你启动容器后手动切换时区

### 3. mac下xdebug调试的问题
xdebug在mac下由于mac下docker的特性，配置远程调试必须填写在服务端配置远程调试ip。