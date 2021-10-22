# nginx-auth-cas-lua
> 静态的web站点或者第三方的web站点，通过反向代理（nginx）接入sso

##  使用说明

- 首先在nginx.conf指定lua库文件的地址，并且开启一个10M的内存空间用于存储cas的session信息：

```
lua_package_path '/etc/nginx/lua/?.lua;;';
lua_shared_dict cas_store 10M;
```
- 然后就可以在具体的虚拟主机（server）配置里添加响应的信息开启SSO了
```
location / {
    access_by_lua_block { require('cas').forceAuthentication() }  // 开启SSO
    add_header Cache-Control no-cache;
    add_header Cache-Control no-store;
    root   /data/test;
    index  index.html index.htm;
}
```

-  如果nginx反向代理的第三方站点（非本地静态文件），access_by_lua_block一定要在proxy_pass 之前
```
location / {
    access_by_lua_block { require('cas').forceAuthentication() }  // 要在proxy_pass之前
    add_header Cache-Control no-cache;
    add_header Cache-Control no-store;

    proxy_set_header Host www.autohome.com.cn;
    proxy_pass http://192.168.0.1;
}
```

- 当访问到图片、js、css等静态资源时，无需通过SSO

```
location ~* \.(gif|jpg|png|js|css|ico|ttf|woff|woff2)$ {
    root   /data/test;
    index  index.html index.htm;
}
```

- 当登录成功后，跳转到/cas-logout 即可退出登录
```
location /cas-logout {
    access_by_lua_block { require('cas').logout("http://your-sso-server-domain.com/login?service=http://your-website-domain.com/") }
    root   html;
    index  index.html index.htm;
}
```
- 我们其实是可以在access_by_lua_block里可以用lua写一些逻辑了，比如判断什么情况下可以不用登录（超级IP、包含某个参数等）
