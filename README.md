# GoPay
一个兼容易支付的 Nodejs 版支付系统

Nodejs + Mysql

##### 1、如果当前网站对接的是易支付，运行本程序后其余完全不需要改动，直接修改域名即可。

> 做这个小程序的初衷是因为易支付代码太垃圾了，业务逻辑不严谨，经常掉单，售后苦不堪言。自己业务替换成 GoPay后基本是0掉单！

配置也不复杂，只需要配置下网站的配置文件即可，因为是个人使用，不搞什么面板什么的了。

#### 环境当然是要先安装好 Nodejs、Mysql

```shell script
cd gopay

npm install

node app.js
```
* 守护启动
```shell script

npm install -g pm2

pm2 start app.js --name=gopay

```

配置网站和支付 **configure.js**

```

const fs = require('fs')
module.exports = {

    web: {
        // 支付域名，用于回调通知，结尾不加 /
        payUrl: 'https://pay.xxxx.com'
    },
    
    // 易支付的用户 key
    user: {
        '10001': {
            key: 'XM9b0ce7BE6R9NQ897B0wW0LW031B182'
        }
    },
    alipay: [
        {
            appId: '',
            // 应用私钥
            privateKey: '',
            // 支付宝公钥
            alipayPublicKey: ''
        }
    ],
    wxpay: [

        {
            appId: '',
            mchid: '',
            privateKey: fs.readFileSync('./cert/wxpay/apiclient_key.pem'), // 微信官网通过工具生成的证书 私钥
            serial: '4034C472D3C81CCA157C5837B7186E366886EA75', // 官网生成证书后，再点击即可显示 证书序列号
            secret: '', // v2版 32位密钥 ,官网自己设置的
            certs: {
                /*
                    参数说明
                    cd gopay
                    npm install yargs
                    cd ./node_modules/.bin
                    wxpay crt -m {mchid} -s {serial} -f {privateKey.pem} -k {secret} -o
                    输出：
                    $ The WeChatPay Platform Certificate#0
                    $ serial=36736A4D01F0056B033693E40B6DB995D2A9B6D1
                    $ notBefore=Mon, 29 Mar 2021 10:04:33 GMT
                    $ notAfter=Sat, 28 Mar 2026 10:04:33 GMT
                    $ Saved to: C:\Users\ADMINI~1\AppData\Local\Temp\wechatpay_36736A4D01F0056B033693E40B6DB995D2A9B6DB.pem

                    将输出的 serial=36736A4D01F0056B033693E40B6DB995D2A9B6D1 的值作为 key
                    将输出的 Saved to: ....\wechatpay_36736A4D01F0056B033693E40B6DB995D2A9B6D1.pem 证书用作 value
                 */
                '36736A4D01F0056B033693E40B6DB995D2A9B6D1': fs.readFileSync('./cert/wxpay/wechatpay_36736A4D01F0056B033693E40B6DB995D2A9B6D1.pem'),
            },
            only_native: false // 是否开启仅扫码 , 如果是在手机上就需要另一个手机扫码，保存相册的native二维码是无法支付的
        }
    ],

    /**
     * 替换源网站至支付宝\微信的订单名称
     */
    form: {
        subject: {
            rewrite: true,
            text: ['个性手机壳', '蓝牙耳机', 'OPPO原装数据线', '通用1A充电器', '品胜充电宝', '安卓通用耳机']
        },
        body: {
            rewrite: true,
            text: '投诉请联系QQ123456'
        }
    },
    
    // 配置好数据库连接后自动创建表
    db: {
        mysql: {
            host: '127.0.0.1',
            database: 'gopay',
            username: '',
            password: '',
            pool: {
                max: 5,
                min: 1,
                acquire: 30000,
                idle: 10000
            },
            logging: false
        }
    }


}

```

#### 配置文件里面基本已经解释的很详细了，因为微信使用的是v3的支付版本，所以配置稍微要多一点点。

微信支付权限

- h5支付
- native支付

支付宝

- PC支付
- 手机网站支付

##### 支付配置可以配置多个，这样就会自动负载均衡到多个支付主体上，目前手机端\PC是靠浏览器UA识别的，自动调转到各自的支付页面。


[![g3eMq0.png](https://z3.ax1x.com/2021/05/07/g3eMq0.png)](https://imgtu.com/i/g3eMq0)


#### 最后使用Nginx进行反代

1、先安装个宝塔吧，毕竟这样Nginx,Mysql就都有了

2、创建网站 -> 反向代理[127.0.0.1:3000] 



