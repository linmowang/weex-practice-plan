# weex-practice-plan
经过一个月半的填坑，总结下weex变成可上线项目(app)中间的坑，水平有限，只能写写简单的js部分，原生部分不是我弄的。

## 版本与项目

### 版本
```
npm install weex-toolkit -g
weex -v
   v1.3.11
 - weexpack : v1.2.7
 - weex-builder : v0.4.0
 - weex-previewer : v1.5.1
```

### 项目
```
weex create app
? Project name app
? Project description A weex project
? Select weex web render latest
? Babel compiler (https://babeljs.io/docs/plugins/#stage-x-experimental-presets) stage-0
? Use vue-router to manage your view router? (not recommended) No
? Use ESLint to lint your code? No
? Set up unit tests No
? Should we run `npm install` for you after the project has been created? (recommended) npm
```

weex platform add ios<br/>
weex platform add android<br/>
生成ios和android这部分要原生来看

如果ios运行不下去，有可能是因为：
1. ios 需要到ios目录里面输入：
```pod  install```
这个就跟执行npm install，下载ios需要的依赖包
2. ios-deploy@1.9.4可以放入到package.json中
3. 微信登录命名和Weex依赖包命名冲突了 WXLogLevel => 改成了WXXLogLevel<br/>
解决方案： https://github.com/apache/incubator-weex/issues/1887

### Devtools(这个目前我没办法等我学了原生会试试这个)

## webpack 与 vuex

### webpack 分析
```
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
```

我就用到了

```

"build": "webpack --env.NODE_ENV=common",
"build:prod": "webpack --env.NODE_ENV=production",
"ios": "weex run ios",
"android": "weex run android",

```

在webpack.config.js文件中选择common 执行的是webpackConfig = require('./configs/webpack.common.conf');<br/>
webpack.common.conf.js 要分两部分看一个是生成web的配置，一个是生成native的配置，我不需要web所以把他们可以全忽略了。<br/>

webpack下的weexConfig:

* entry
* output
* resolve
* module
* plugins
* node: config.nodeConfiguration (这个现在没看懂)

#### entry
运行npm run ios<br/>
getEntryFile()生成weexEntry<br/>
通过getNativeEntryFileContent生成具体内容<br/>
由于entryFilter: '**/*.vue',所以所有的.vue都会打包成.js(包括component)<br/>
生成的内容 vue初始化在.temp文件夹下，所有的完整内容在dist下，原生页一个js代表一个页面(我的项目是这样做的), 所以需要把component过滤掉

```
// Retrieve entry file mappings by function recursion
const getEntryFile = (dir) => {
  dir = dir || config.sourceDir;
  const entries = glob.sync(`${dir}/${config.entryFilter}`, config.entryFilterOptions);
  console.log('in commom webpack')
  console.log('entries', entries)

  entries.forEach(entry => {
    // 如果路径包含 components 那就是组件，不是页面入口, 过滤掉
    if (entry.indexOf('components') > -1) {
      return
    }

    const extname = path.extname(entry);
    const basename = entry.replace(`${dir}/`, '').replace(extname, '');
    const templatePathForWeb = path.join(vueWebTemp, basename + '.web.js');
    const templatePathForNative = path.join(vueWebTemp, basename + '.js');
    fs.outputFileSync(templatePathForWeb, getWebEntryFileContent(templatePathForWeb, entry));
    fs.outputFileSync(templatePathForNative, getNativeEntryFileContent(templatePathForNative, entry));
    webEntry[basename] = templatePathForWeb;
    weexEntry[basename] = templatePathForNative;
  })
}
```

#### output(略过)

#### resolve
这可以定义快捷方式
```import HelloWorld from '@/components/HelloWorld'```
@就是这里定义的

#### module
npm i url-loader file-loader -D
```
      {
        test: /\.(png|jp(e*)g|svg)$/,  
        use: [{
            loader: 'url-loader',
            options: { 
                limit: 10000, // Convert images < 10kb to base64 strings
                name: 'images/[name].[ext]'
            }
        }]
      }
```
这可能把小图片打包成base64<br/>
但是在vue文件中引用require('图片路径')会报错<br/>
在mixins这样使用 会导致images2所有小图片都打到js文件里面，并不能达到理想中的效果，可以先放放<br/>
// loadImage(imgPath) {
//   const imagePath = require(`../images2/${imgPath}`) // eslint-disable-line
//   console.log('imagePath: ', imagePath)
//   return imagePath
// }

#### plugins
要发线上包的时候
npm run build:prod
在webpack.prod.conf.js使用了UglifyJsparallelPlugin 会压缩文件体积

### vuex
需要看完上面的entry才能懂下面的内容
由于getNativeEntryFileContent可以生成具体内容 
.temp下的文件都是通过getNativeEntryFileContent写的
所有vuex需要在这里添加

npm i vuex
```
// Wraping the entry file for native.
const getNativeEntryFileContent = (entryPath, vueFilePath) => {
  let relativeVuePath = path.relative(path.join(entryPath, '../'), vueFilePath);
  let contents = '';
  if (isWin) {
    relativeVuePath = relativeVuePath.replace(/\\/g, '\\\\');
  }

  const relativeStorePath = path.join(relativeVuePath, '../store.js')
  console.log(relativeStorePath)
  contents += `import App from '${relativeVuePath}'
import store from '${relativeStorePath}'

new Vue(Vue.util.extend({el: '#root', store}, App));
`;
  
  return contents;
}
```

.temp index.js
```
import App from '../src/index.vue'
import store from '../src/store.js'

new Vue(Vue.util.extend({el: '#root', store}, App));
```

## 编写页面
### 传参
### css

![](img/batch-another.png)

![面对疾风吧](http://upload-images.jianshu.io/upload_images/3876828-a4346506018aa44f.gif?imageMogr2/auto-orient/strip "哈赛给 啊痛")
