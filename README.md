# weex-practice-plan
经过一个月半的填坑，总结下weex变成可上线项目(app)中间的坑，水平有限，只能写js的部分，原生部分不是我弄的

##版本与项目
npm install weex-toolkit -g
➜  weex-practice-plan git:(master) ✗ weex -v
   v1.3.11
 - weexpack : v1.2.7
 - weex-builder : v0.4.0
 - weex-previewer : v1.5.1

weex create app

? Project name app
? Project description A weex project
? Select weex web render latest
? Babel compiler (https://babeljs.io/docs/plugins/#stage-x-experimental-presets) stage-0
? Use vue-router to manage your view router? (not recommended) No
? Use ESLint to lint your code? No
? Set up unit tests No
? Should we run `npm install` for you after the project has been created? (recommended) npm

weex platform add ios
weex platform add android
生成ios和android这部分要原生来看


## webpack 与 vuex
###webpack 分析
...javascript
"scripts": {
    "start": "npm run serve",
    "build": "webpack --env.NODE_ENV=common",
    "build:prod": "webpack --env.NODE_ENV=production",
    "build:prod:web": "webpack --env.NODE_ENV=release",
    "build:plugin": "webpack --env.NODE_ENV=plugin",
    "clean:web": "rimraf ./release/web",
    "clean:ios": "rimraf ./release/ios",
    "clean:android": "rimraf ./release/android",
    "dev": "webpack --env.NODE_ENV=common --progress --watch",
    "serve": "webpack-dev-server --env.NODE_ENV=development --progress",
    "ios": "weex run ios",
    "web": "npm run serve",
    "android": "weex run android",
    "pack:ios": "npm run clean:ios && weex build ios",
    "pack:android": "npm run clean:android && weex build android",
    "pack:web": "npm run clean:web && npm run build:prod:web"
  },
...



## 编写页面
### 传参
### css 

![](img/batch-another.png)

![面对疾风吧](http://upload-images.jianshu.io/upload_images/3876828-a4346506018aa44f.gif?imageMogr2/auto-orient/strip "哈赛给 啊痛")