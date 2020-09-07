---
title: ionic添加Android指纹验证页面
date: 2018-08-31 00:49:18
categories: ionic
tags:
- ionic
---
#### 安装插件
```bash
ionic cordova plugin add cordova-plugin-android-fingerprint-auth
npm install --save @ionic-native/android-fingerprint-auth
```

#### 创建页面
在项目中下运行,添加 fingerprint 页面
```bash
ionic generate fingerprint
```
创建文件夹fingerprint,包含以下文件
```bash
fingerprint.html ---页面模板
fingerprint.module.ts ---页面入口
fingerprint.scss ---样式
fingerprint.ts ---配置选择器和模板文件，写js代码
```

<!--more-->

#### 在文件主入口添加声明和实例
app.module.ts部分代码
```js
import { FingerprintPage } from '../pages/fingerprint/fingerprint';

@NgModule({
  declarations: [
    MyApp,
    HelloIonicPage,
    ItemDetailsPage,
    ListPage,
    FingerprintPage
  ],
  imports: [
    BrowserModule,
    IonicModule.forRoot(MyApp),
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp,
    HelloIonicPage,
    ItemDetailsPage,
    ListPage,
    FingerprintPage
  ],
  providers: [
    StatusBar,
    SplashScreen,
    AndroidFingerprintAuth,
    {provide: ErrorHandler, useClass: IonicErrorHandler}
  ]
})
```

#### 添加跳转代码
list.ts 源码修改部分
```js
items: Array<{ title: string, note: string, icon: string, id: Number }>;


this.items.push({
      title: '指纹识别',
      note: '指纹识别页面',
      icon: this.icons[Math.floor(Math.random() * this.icons.length)],
      id: 11
    });
    
itemTapped(event, item) {
    if (item.id == 11) {
      this.navCtrl.push(FingerprintPage);
    } else {
      this.navCtrl.push(ItemDetailsPage, {
        item: item
      });
    }
  }    
```

#### 添加指纹功能
fingerprint.ts 源码添加指纹功能
```js
import { AndroidFingerprintAuth } from '@ionic-native/android-fingerprint-auth';

constructor(private androidFingerprintAuth: AndroidFingerprintAuth) {
    this.androidFingerprintAuth.isAvailable()
      .then((result) => {
        if (result.isAvailable) {
          this.androidFingerprintAuth.encrypt({ clientId: 'myAppName', username: 'myUsername', password: 'myPassword' })
            .then(result => {
              if (result.withFingerprint) {
                console.log('Successfully encrypted credentials.');
                console.log('Encrypted credentials: ' + result.token);
              } else if (result.withBackup) {
                console.log('Successfully authenticated with backup password!');
              } else console.log('Didn\'t authenticate!');
            })
            .catch(error => {
              if (error === this.androidFingerprintAuth.ERRORS.FINGERPRINT_CANCELLED) {
                console.log('Fingerprint authentication cancelled');
              } else console.error(error)
            });

        } else {
          console.log('不支持');
        }
      })
      .catch(error => console.error(error));
  }
```

完成