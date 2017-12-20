#### 当前分支 assistance  微信分享助力活动 12.12 和 圣诞节。

#### 总体说明
1. 目录基本结构基于ThinkPHP3.2
2. PHP后台部分仅仅是作为路由使用。页面切换，参数传递，数据绑定
3. 开发是使用目录Src，线上最终部署是Home.使用r.js的自带打包压缩工具。打包命令 node r.js -o build.js

##### 前端部分：
1. Content
    1. images 是图片
    2. style 是样式。样式使用less语法，并最终转换成index.css，统一在页面中使用，没有区分某个页面使用，全部统一使用index.css
2. JavaScript。
    1. app。App主体 js
         1. 使用的是requirejs，最后通过r.js进行压缩打包到Application/Home目录。打包规则在build.js
            2. app/controller ：具体的业务逻辑。对应每个功能模块（页面），全部都继承自app/model/base.js
            3. app/main： 定义每个页面所需的js模块，也就是require 的配置文件
            4. app/model：公用模块。
                - super.js 最基础类，定义部分最基础的js方法，共用方法，和项目业务没有关系。
                - base.js 继承super.js。外加一些和业务有关的功用方法，如http请求，折扣券通用信息等。
    2. libs 。第三方js库
    3. operations 。 运营部门的页面js

3. View
    1. Index所有功能模块的页面。
    2. Public 共用页面，只有menu.html
    2. Operations 运营页面
    3. layout.html。这个是除了菜单栏对应页面以外，所有页面的母版页。

4. Controller
    1. IndexController 对应App总体功能
    2. OperationsController 对应运营的功能


### 开发调试说明

##### 问题和说明
1. 公众号调试查看需要在微信web开发工具中进行。由于微信授权回调只能设置为外网可以访问到的方法，所以需要把回调方法问题部署到外网。
2. 由于授权的问题，本地修改添加的页面效果，只有提交部署到服务器之后才能看到效果。
3. 当多人开发时，大家都使用同一个测试号，都提交代码到测试服务器，这就导致了每次提交前，都要求先更新服务器代码。即使是修改了一点点代码也必须这么做，这大大地增加了我们web的开发难度，拉低工作效率。

     ##### 大家都很希望能像开发普通web页面一样，修改了自己的代码，然后在浏览器中刷新即可看到新修改的效果。这个愿望是可以实现，但是比较复杂。
     
##### 解决办法
1. 如果想实现本地开发调试，就必须要突破解决一个问题——授权回调。实现让微信可以访问到我们本地代码的回调方法。实现这个目标，我们要是使用一个黑科技技术——内网穿透。
2. 我们使用[natapp](https://natapp.cn/)。注册账号并进行实名制后，可以开通一个免费的隧道。如图：

    ![image](http://7xqnxu.com1.z0.glb.clouddn.com/natapp.png)
    
3. 下载[natapp客户端](链接：http://pan.baidu.com/s/1slQatZn 密码：35mf) ,修改其中config.ini的authtoken 的值，修改为自己的 authtoken 。保存后双击 natapp.exe。（本地要先安装apache，并开启,默认端口为80）。
4. 成功运行后natapp.exe 后会提示一个地址，类似 http://3msfaiidfa.natappfree.cc 这个地址就是客户端为我们生成的外网地址。在浏览器里输入这个地址，可以看到我们自己的apache。通过这个地址，别人可以访问到我们自己的本地apache。这就是开启了内网穿透。所以微信也会可以访问到我们的apache。这个地址每次开启都是随机的，为了方便后面的开发，我建议大家开通一下会员，5元每个月的套餐就可以。把地址固定下来。

5. 申请自己的微信公众号，并配置相应的信息。如图：

    ![image](http://7xqnxu.com1.z0.glb.clouddn.com/weichat1.png)
    
    ![image](http://7xqnxu.com1.z0.glb.clouddn.com/weichat2.png)
    
    ![image](http://7xqnxu.com1.z0.glb.clouddn.com/weichat3.png)
    
    ![image](http://7xqnxu.com1.z0.glb.clouddn.com/weichat4.png)
    
6. 配置好外界的环境后，我们修改代码的配置信息。先修改web 代码配置。在Application/Src/Conf中添加 config.php文件。如下：
    

     <?php
     $tempHost = 'http://3msfaiidfa.natappfree.cc';
     $host = $tempHost.'/jiayouzan-web/';

     $ctrl='/index.php/src/index';

     $ctrl_operations='/index.php/Src/operations';

     return array(

        'TMPL_PARSE_STRING' => array(
            '__IMG__'            => __ROOT__ . '/Application/Src/Content/images',
            '__STYLE__'          => __ROOT__ . '/Application/Src/Content/style',
            '__JS__'             => __ROOT__ . '/Application/Src/Javascript',
            '__JSON__'           => __ROOT__ . '/Application/data/',
            '__CTRL__'           => __ROOT__ . $ctrl,
            '__URL__'            => __ROOT__ . '/index.php',
            '__API__'            => $tempHost.'/japi/public/index.php',
            '__WX_AppID__'       => '*********',
            '__Host__'           => $host,
            '__HOME__'           => $host.$ctrl,
            '__WX_RedirectUrl__' => $host.$ctrl.'/home',
            '__WX_OperationsShareBaseUrl__' => $host.$ctrl_operations,
            '__Version__'        =>'201711251132'
        ),
        'LAYOUT_ON'         => true,
        'LAYOUT_NAME'       => 'layout'
        );
    ?> 
    
    其中的 WX_AppID 修改 为自己的 微信测试号appid。
    
7. 配置本地api。下载 api 代码，搭建本地 api 服务。git地址： git@git.coding.net:jiayouzan/JAPI.git。添加三个配置文件。文件较为私密，请找潘潇要。并做好相应的配置。

8. 配置好后，在开发工具中使用  http://3msfaiidfa.natappfree.cc/jiayouzan-web/index.php/src/index 进行开发测试。本地修改后，直接刷新页面即可。

9. 每天下班前必须要把今天开发的东西，保证没有大问题后提交到代码仓库。有重大修改，如果会影响其他组员开发时，要及时通知提醒。

