wordpress
=========

## install

### apt

1. 安装wordpress

   ```sh
   sudo apt install wordpress
   ```

   2. 配置

      ```sh
      cat > /etc/apache2/sites-available/wordpress.conf <<EOF
      Alias /blog /usr/share/wordpress
      <Directory /usr/share/wordpress>
          Options FollowSymLinks
          AllowOverride Limit Options FileInfo
          DirectoryIndex index.php
          Order allow,deny
          Allow from all
      </Directory>
      <Directory /usr/share/wordpress/wp-content>
          Options FollowSymLinks
          Order allow,deny
          Allow from all
      </Directory>
      EOF
      ```

    3. 启动服务

       ```sh
       sudo a2ensite wordpress
       systemctl reload apache2
       sudo systemctl reload apache2
       sudo a2enmod rewrite
       sudo service apache2 reload
       ```

    4. 安装mysql

       ```
       sudo apt install mysql-server php-mysql libapache2-mod-php
       mysql -u root
       ```

    5. 配置mysql

       ```
       $ sudo mysql -u root
       Welcome to the MySQL monitor.  Commands end with ; or \g.
       Your MySQL connection id is 7
       Server version: 5.7.20-0ubuntu0.16.04.1 (Ubuntu)
       
       Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
       
       Oracle is a registered trademark of Oracle Corporation and/or its
       affiliates. Other names may be trademarks of their respective
       owners.
       
       Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
       
       mysql> CREATE DATABASE wordpress;
       Query OK, 1 row affected (0,00 sec)
       
       mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost IDENTIFIED BY 'xxx';
       Query OK, 1 row affected (0,00 sec)
       
       mysql> FLUSH PRIVILEGES;
       Query OK, 1 row affected (0,00 sec)
       
       mysql> quit
       sudo service mysql restart
       ```
       
    6. 配置数据库链接
   
    ```php
       cat >/etc/wordpress/config-bjrdc212.php <<EOF
    <?php
       define('DB_NAME', 'wordpress');
       define('DB_USER', 'wordpress');
       define('DB_PASSWORD', '<your-password>');
       define('DB_HOST', 'localhost');
       define('DB_COLLATE', 'utf8_general_ci');
       define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
       define('WPLANG', 'zh_CN');
       ?>
       EOF
    ```
   
    7. 访问
   
    ```
    bjrdc212/blog
    http://bjrdc212/blog/wp-admin/
    ```
   
    8. ftp
   
   [vstfp]: Ivsftpd.md
   
    

## 插件

> 1. on ftp
>
>    老版本，目前测试是<5的版本需要ftp安装插件
>
> 2. 手动
>
>    在没有ftp的情况下可以手动的安装插件，方式如下
>
>    1. 在官网`https://cn.wordpress.org/plugins/wp-statistics/`或者`https:/www.wordpress.org`搜索并下载需要的插件
>    2. 将插件上传到服务器上wp的`wordpress/wp-content/plugins`目录下
>    3. 在wordpress的管理界面上启用该插件。
>
> 3. 5.4. 之后的版本不需要ftp，但是需要配置php的 `extension=openssl`，可能出的问题
>
>    ```
>    installation failed: download failed. no working transports found
>    ```
>
>    需要安装with-openssl
>
>    ```
>    ./configure --with-apxs2=/home/bjrdc/software/httpd-2.4.42/bin/apxs --with-zlib --prefix=/home/bjrdc/software/php-7.3.12 --with-mysqli=/usr/local/mysql/bin/mysql_config --without-sqlite3 --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --without-pdo-sqlite --prefix=/home/bjrdc/software/php --with-openssl
>    ```
>
> 4. 两个好用的插件
>
>    **WP Statistics**
>
>    

### 主题

#### 增加文章二维码

在 single.php 里或者其它文章显示文件里在自己喜欢的位置添加如下代码就可以了

```php
<?php
/**
 * The template for displaying all single posts
 *
 * @link https://developer.wordpress.org/themes/basics/template-hierarchy/#single-post
 *
 * @package WordPress
 * @subpackage Twenty_Seventeen
 * @since 1.0
 * @version 1.0
 */

get_header(); ?>

<div class="wrap">
	<div id="primary" class="content-area">
		<main id="main" class="site-main" role="main">

			<?php
			/* Start the Loop */
			while ( have_posts() ) : the_post();

				get_template_part( 'template-parts/post/content', get_post_format() );
/*
				// If comments are open or we have at least one comment, load up the comment template.
				if ( comments_open() || get_comments_number() ) :
					comments_template();
				endif;

				the_post_navigation( array(
					'prev_text' => '<span class="screen-reader-text">' . __( 'Previous Post', 'twentyseventeen' ) . '</span><span aria-hidden="true" class="nav-subtitle">' . __( 'Previous', 'twentyseventeen' ) . '</span> <span class="nav-title"><span class="nav-title-icon-wrapper">' . twentyseventeen_get_svg( array( 'icon' => 'arrow-left' ) ) . '</span>%title</span>',
					'next_text' => '<span class="screen-reader-text">' . __( 'Next Post', 'twentyseventeen' ) . '</span><span aria-hidden="true" class="nav-subtitle">' . __( 'Next', 'twentyseventeen' ) . '</span> <span class="nav-title">%title<span class="nav-title-icon-wrapper">' . twentyseventeen_get_svg( array( 'icon' => 'arrow-right' ) ) . '</span></span>',
				) );
*/
			endwhile; // End of the loop.
			?>
<img id="qy"/>


		</main><!-- #main -->
	</div><!-- #primary -->
	<?php get_sidebar(); ?>
</div><!-- .wrap -->
<script>
document.getElementById("qy").src="http://api.qrserver.com/v1/create-qr-code/?size=100x100&data="+window.location.href
</script>
<?php get_footer();


```

第二种二维码方式

```php
<img src="http://api.qrserver.com/v1/create-qr-code/?size=100x100&data=<?php echo 'http://'.$_SERVER['HTTP_HOST'].':10080'.$_SERVER['PHP_SELF'].'?'.$_SERVER['QUERY_STRING']; ?>" alt="QR: <?php the_title(); ?>"/>
```

#### 修改主题 行距

```
.post p{
	line-height: 2.0em;}
```



## 反向代理

> ### 第一种方式
>
> 从网上找的反向代理的方法不太好用，最终统计的方式为通过nginx+subfilter的方式实现，具体配置过程如下
>
> 1. 安装nginx的时候，将subfilter加上，命令如下
>
>    ```
>    ./configure --prefix=/home/bjrdc/software/nginx --with-pcre --with-http_sub_module
>    ```
>
> 2. 配置反向代理和subfilter
>
>    ```
>            location /blog/ {
>                proxy_set_header Accept-Encoding "";
>                proxy_pass http://bjrdc212/blog/;
>                sub_filter 'bjrdc212' '211.157.177.101:10080';
>                sub_filter_once off;
>                sub_filter_types *;
>                proxy_redirect     off;
>                proxy_set_header   Host             $host;
>                proxy_set_header   X-Real-IP        $remote_addr;
>                proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
>            }
>    ```
>
> 3. 并不需要修改wordpress的wp-config.php代码和修改wp的home和siteurl参数，仍然为
>
>    ```
>    WordPress地址（URL）：http://bjrdc212/blog
>    站点地址（URL）：http://bjrdc212/blog
>    ```
>
>    这种方式的问题是：**在仪表盘上看到的地址和最终访问的地址不同**
>
> ### 第二种方式（解决无限redirect问题）
>
> 需要在仪表盘中的常规中将地址修改为反响代理之后的地址
>
> ```
> WordPress地址（URL）：http://211.157.../blog
> 站点地址（URL）：http://211.157..../blog
> ```
>
> 
>
> 可以使用手动安装httpd+nginx的方式，仍然出现无限redirect，通过修改如下代码来实现
>
> ```
> vi wordpress/wp-includes/template-loader.php
> ```
>
> 注释掉开头的redirect的代码
>
> ```
> //      do_action( 'template_redirect' );
> ```
>
> 



## 异常处理

### 恢复地址

```
mysql> update wp_options SET option_value='http://bjrdc212/blog' WHERE option_name='siteurl';
mysql> update wp_options SET option_value='http://bjrdc212/blog' WHERE option_name='home';
```

使用 search 恢复数据



```
 php srdb.cli.php  -h localhost -n wordpress  -u wordpress -p "xxxx" -r "http://bjrdc212/" -s "http://211.157.177.101:10080/"
```

