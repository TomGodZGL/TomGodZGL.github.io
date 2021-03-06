*此个人[博客](http://www.baidu.com)旨在技术交流，大家一起学习，欢迎大家指正*


##大型网站优化##
大型网站优化主要有以下两点：

- 网站静态化
- mysql优化

###判断大型网站的基本概念###

- pv值（page views）页面浏览量：
是指，一个网站的所有页面，在一天内被访问的总的次数。达到千万级别以上，几百万以上
- uv值（unique vistor）独立访客：是指一个网站，在一天内有多少个用户来访问我们的网站。一般几十万以上
- 独立ip：是指一个网站，在一天内有多少个独立ip来访问我们的网站

###大型网站带来的问题？###
- 大的访问量（大并发）

	并发：是指在某个时间点，有多少个用户同时访问某个地址
- 大的流量（带宽问题）


- 海量的数据存储

	网站从小到大的数据的存储，比如数据表的容量达到GT级别，带来查询速度变慢，要从海量的数据表里面快速的查找到数据，是我们优化的重点。

###大访问量问题###

使用负载均衡以及分层架构服务器

负载均衡说明：

软件：lvs(linux virtual server)linux虚拟服务， nginx（web服务器，负载均衡）

硬件：f5-bigip  :价格昂贵，立竿见影，效果非常好。一般是大的网游公司或大的门户网站使用

负载均衡器的策略：

- 轮询：载均衡把请求轮流转发给web服务器
- 加权：配置性能高的，分配的权重就越大，处理的请求就越多
- 最少连接：负载均衡把请求转发给最空闲的那台服务器
- ip哈希：同一地址的客户端，负载均衡把请求始终给同一台服务器

###大流量问题###
- 在web 服务器端，配置压缩，减少数据传输的数据量
	
	Apache上利用gzip压缩算法进行压缩的模块有两种：mod_gzip和mod_deflate. 

	具体的步骤：
	
	1）打开apache的httpd.conf配置文件，开启压缩模块
	
		LoadModeule deflate_module modules/mod_deflate.so	

	2）在虚拟主机里面，配置压缩的对象

		<ifmodule mod_deflate.c> 
			#配置压缩的级别，压缩级别为6，可选1-9，推荐为6
			DeflateCompressionLevel 6
			#压缩文本文件      
			AddOutputFilterByType  DEFLATE text/plain   
			#压缩html文件
			AddOutputFilterByType  DEFLATE text/html   
			#压缩xml文件
			AddOutputFilterByType  DEFLATE text/xml    
		</ifmodule> 
		注意：为什么要指定文件类型来压缩？
		压缩也是要耗费cpu资源的，图片/视频等文件，压缩效果不好，不要对其压缩。一般压缩的是文本格式的文件。
		DeflateCompressionLevel 指令来设置压缩级别。该指令的值可为1（压缩速度最快，最低的压缩质量）到9（最慢的压缩速度，压缩率最高）之间的整数，其默认值为6（压缩速度和压缩质量较为平衡的值）

- 合并文件（样式文件，js文件，背景图片文件），减少http的请求

- 把比较占用流量的资源（或不同的功能）单独部署服务器

###大存储问题###
- 使用缓存技术

	主要目的是：减少数据库的查询次数
	
	1）磁盘缓存（页面静态化）

	页面静态化，就是把一个动态（操作数据库）的页面，转换成一个.html页面。直接访问生成的.html页面。提高了访问速度，因为没有查询数据库
	
	2）内存缓存（redis/memcache）

	把数据缓存到服务器端的内存里面。下次访问时，直接从内存里面获取数据
- 对mysql优化

	1）设计角度：存储引擎的选择，字段类型选择，范式

	2）利用mysql自身的特性：索引，查询缓存，分区分表，存储过程，sql语句优化配置，

	3）部署大负载架构体系：主从复制，读写分离。

	4）硬件升级
	
	**注**： 此mysql优化推荐参考 [骑着笨鸟慢慢飞](http://superve.leanote.com/) 个人技术博客
	 
###页面静态化详解###

- 什么是页面静态化：

	就是把一个动态（操作数据库）的页面转换成一个.html页面。
	比如http://www.abc.com/goods.php?id=4;      
	可以变成http://www.abc.com/goods_id4.html
 
	页面静态化可以分为两种：真静态和伪静态。
	真静态：就是把一个动态的页面，实实在在的生成一个静态页面。
	伪静态：从形式上看是一个静态页面，实际上操作的还是动态页面。比如
	http://www.abc.com/news-sport_id12.html
	实际上操作是：
	http://www.abc.com/news.php?type=sport&id=12

- 实现方式

	真静态的实现方式：使用ob缓存技术，ob缓存，缓存的内容是响应的主体数据。

	在请求一个php的过程中，我们实际上经过三个缓存，程序缓存，ob缓存，浏览器缓存。

	伪静态的实现方式：通过web服务器的重写机制，比如请求 index.html =》 index.php

- 程序缓存

	该缓存是必须存在的，是无法关闭的。缓存的数据是响应的数据（响应的头和响应的主体数据）

- ob缓存

	ob就是 output_buffering:输出缓存，缓存的数据是响应的主体数据
	
	如果开辟了ob缓存，主体数据首先存储到ob缓存里面，头信息要存储到程序缓存（无论是否开启ob缓存），当代码执行完毕后，ob缓存里面的数据刷新（移动）到程序缓存，程序缓存再输出到浏览器缓存中，最后输出内容
- 如何开启 ob缓存

	1）使用ob_start()函数，针对当前页面有效

	2）通过使用php.ini的配置文件来开启ob缓存，针对所有的页面都有效

- ob相关的函数

	ob_get_contents();//获取ob缓存里面的内容

	ob_clean();//清空ob缓存里面的内容，不关闭ob缓存
	
	ob_end_clean();//清空ob缓存里面的内容，并关闭ob缓存

	ob_flush();//是把ob缓存里面的数据给刷新（移动）到程序缓存。不关闭ob缓存

	ob_end_flush();是把ob缓存里面的数据给刷新（移动）到程序缓存。并关闭ob缓存
	
	获取ob缓存里面的数据内容：

		$content = ob_get_contents();响应主体的数据
		file_put_contents(文件名称,’文件内容’);// 生成了一个静态页面

- 真静态案例
	
		<?php 
		
		    //接收参数
		    $goods_id = $_GET['goods_id'] + 0;
		    $lifetime = 24*3600*7;
		    //构造静态页面文件名
		    $filename = 'goods_' . $goods_id . '.html';
		
		    //判断文件是否存在且是否在生命周期中
		    if(file_exists($filename) && filemtime($filename) + $lifetime > time()) {
		        include $filename;
		        exit;
		    }
		    
		    //如果静态页面不存在，页面已经过期，则操作数据库
		    $link = mysql_connect('localhost:3306','root','root');
		    mysql_query('set names utf8');
		    mysql_query('use test');
		
		    $sql = "select * from sh_goods where g_id = $goods_id";
		    $res = mysql_query($sql);
		    $info = mysql_fetch_assoc($res);
		
		    ob_start();
		?>
		    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
		    <html xmlns="http://www.w3.org/1999/xhtml" lang="zh-CN">
		    <head>
		    <title>商品详情页</title>
		    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
		    <meta name="description" content="" />
		    <meta name="keywords" content="" />
		    <script type="text/javascript">
		
		    </script>
		
		    <style type="text/css">
		    </style>
		    </head>
		        <body>
		        <h1><?php echo $info['goods_name']?></h1>
		        <div><?php echo $info['goods_desc']?></div>
		        </body>
		    </html>
		
		<?php 
		    $content = ob_get_contents();//获取ob缓存里面的数据内容
		    file_put_contents($filename,$content);
		
		?>
###伪静态####

实际开发项目中，我们的页面不适合使用真静态，但是你不仅希望页面安全性高，同时利于seo，可以考虑使用伪静态技术

- 实现方式

	使用apache的重写模块，要开启重写模块（rewrite机制）。
	在重写模块中，要配置重写规则，在重写规则里面我们使用到正则表达式

- 如何开启重写模块

	打开 httpd.conf文件，进行配置修改。添加rewrite_module模块
		
		LoadModule rewrite_module modules/mod_rewrite.so

- 具体的重写规则的配置

	1）使用分布式文件配置，.htaccess文件

	2）在虚拟主机的配置文件中，要配置支持 .htaccess配置文件

		AllowOverride All
	
	3）在.htaccess文件中进行配置重写规则
		
		RewriteEngine  on 	开启重写引擎
		RewriteCond   		重写条件
		RewriteRule			重写规则

	4)RewriteCond重写条件语法

		RewriteCond 判断依据  条件表达式 [条件标志]
		
	5)案例说明

		<IfModule rewrite_module>
			#开启重写引擎
			RewriteEngine on
			#添加重写条件
			RewriteCond	%{REQUEST_FILENAME} !-f
			#添加重写规则
			RewriteRule index.html	index.php
		</IfModule>


- 防盗链

	1)为了防止外部使用网站图片资源可以添加水印
	
	2）利用重写机制，根据HTTP_PEFERER

		<IfModule rewrite_module>
			RewriteEngine on
			#判断URL的上一次访问是否本域名地址
			RewriteCond  %{HTTP_PEFERER} -www.demo.com[NC]
			#如果不会本网站访问，则禁止访问
			RewriteRule  \.(jpg|png|jpeg|gif|bmp)	-[F]
		</IfModule>


##商品浏览记录代码实现##

**获取商品历史浏览记录，利用thinkPHP框架开发，基于cookie实现**

	public function getHistory(){
		//判断cookie中时候有history数据存在
		$list = isset($_COOKIE['history']) ? unserialize($_COOKIE['history']) : array();
		$goods_id = (int)$_GET['id'];
		
		//判断用户是否浏览了商品
		if(!empty($goods_id)){
			if(!empty($list)){		
				//获取商品ID
				$ids = array();
				foreach($list as $v){
					$ids[] = $v['goods_id'];
				}
				//判断商品是否已经存在,则删除
				$index = array_search($goods_id, $ids);
				if($index !== false){
					$history = $list[$index];
					unset($list[$index]);
				}else{
					//根据商品ID获取商品详细信息
					$goodsinfo = M('Goods')->field('goods_name,shop_price,goods_thumb')->where("id = $goods_id")->find();
					//构造新数组，并添加到浏览记录中
					$history = array(
							'goods_id'		=>	$goods_id,
							'goods_name'	=>	$goodsinfo['goods_name'],
							'shop_price'	=>	$goodsinfo['shop_price'],
							'goods_thumb'	=>	$goodsinfo['goods_thumb']
					);
				}
				//判断商品记录是否大于3条
				if(count($list) > 3){
					array_pop($list);
				}
				//往数组的头部添加数据
				array_unshift($list, $history);							
			}else{
				//根据商品ID获取商品详细信息
				
				$goodsinfo = M('Goods')->field('goods_name,shop_price,goods_thumb')->where("id = $goods_id")->find();
				
				//构造新数组，并添加到浏览记录中
				$history = array(
						'goods_id'		=>	$goods_id,
						'goods_name'	=>	$goodsinfo['goods_name'],
						'shop_price'	=>	$goodsinfo['shop_price'],
						'goods_thumb'	=>	$goodsinfo['goods_thumb']
				);
				//往数组的头部添加数据
				array_unshift($list, $history);
			}
			
			//获取商品信息并将其放入cookie中，用于浏览历史使用
			setcookie('history',serialize($list),time()+24*3600,'/');			
		}
		//将数组返回
		return $list;	
	}
