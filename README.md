# Nginx-Lua-FastDFS-GraphicsMagick

我们通常使用FastDFS来当作图片服务器，客户端通过Nginx请求图片。某些情况下，客户端对请求的图片有尺寸要求，<br />如  `http://ip/group1/M00/00/00/wKhnBFiH_2KAOAH-AA1rIuRd3Es633.jpg_500x600.jpg`。<br />这时我们可以使用GraphicsMagick工具动态的修改图片以满足客户端的需求。<br/>
这里使用lua脚本，调用GraphicsMagick的gm命令动态处理图片。<br/>

 # 整体思路:
1、首先服务器需要有lua环境<br />
2、Nginx扩展支持lua，可调用lua脚本<br />
3、lua脚本中定义gm命令及参数<br />

# 安装配置:
1、安装lua环境<br />
`tar zxvf LuaJIT-2.0.2.tar.gz`<br />
`cd LuaJIT-2.0.2`<br />
`make && make install`<br />

2、配置lua环境变量<br />
`export LUAJIT_LIB=/usr/local/lib`<br />
`export LUAJIT_INC=/usr/local/include/luajit-2.0`<br />

3、安装GraphicsMagick<br />
`tar zxvf GraphicsMagick-1.3.21.tar.gz`<br />
`cd GraphicsMagick`<br />
`./configure --prefix=/data/local/GraphicsMagick --enable-shared`<br />
`make && make install`<br />

查看GraphicsMagick 支持的文件类型:<br />
`/data/local/GraphicsMagick/bin/gm -version`<br />


```ruby

Feature Support:
  Native Thread Safe       yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     yes
  DPS                      no
  FlashPix                 no
  FreeType                 no
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                no
  JPEG                     yes
  Little CMS               no
  Loadable Modules         no
  OpenMP                   yes (201107)
  PNG                      yes
  TIFF                     no
  TRIO                     no
  UMEM                     no
  WebP                     no
  WMF                      no
  X11                      no
  XML                      no
  ZLIB                     yes

Host type: x86_64-unknown-linux-gnu

```

若PNG、JPEG等不支持，可以在编译GraphicsMagick的时候，使用 `--with-png=yes`

4、准备lua模块<br />
`ngx_devel_kit-0.2.18.tar.gz` <br />
`v0.8.6.tar.gz（lua-nginx-module-0.8.6`<br />

5、安装Nginx<br />
`tar zxvf nginx-1.4.2.tar.gz`<br />
`cd nginx-1.4.2`<br />
`./configure —prefix=/usr/local/nginx —add-module=lua-nginx-module-0.8.6 —add-module=ngx_devel_kit-0.2.18`<br />
`make && make install`<br />

测试nginx扩展lua是否正常:
```ruby

location /test {
    default_type text/html;
    content_by_lua '
        ngx.say("hello world")
        ngx.log(ngx.ERR,"err err")
    ';
}
```

若返回`hello world`即说明正常<br />

6、启动Nginx<br />
若出错，如`/usr/local/nginx/sbin/nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory`<br />
请按如下操作:<br />
`find / -name libluajit-5.1.so.2`<br />

`ln -s /usr/local/lib/libluajit-5.1.so.2 /usr/lib64/libluajit-5.1.so.2`<br />

7、lua脚本ImageResizer.lua<br />

```ruby
local command = "/data/local/GraphicsMagick/bin/gm convert " .. ngx.var.request_filepath .. " -resize " .. ngx.var.width .. "x" .. ngx.var.height .. " +profile \"*\" " .. ngx.var.request_filepath .. "_" .. ngx.var.width .. "x" .. ngx.var.height .. "." ..                           
ngx.var.ext;os.execute(command);
ngx.exec(ngx.var.request_uri);

```

Nginx配置请见nginx.conf
