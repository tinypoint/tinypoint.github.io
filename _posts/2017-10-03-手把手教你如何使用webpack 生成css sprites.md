---
layout:     post
title:      手把手教你如何使用webpack 生成css sprites
subtitle:   使用webpack 生成css sprites
date:       2017-10-03
author:     tinypoint
header-img: img/css-sprite.jpg
catalog: true
tags:
    - 前端
    - sass
    - css
    - 性能优化
    - 面试
    - 图片处理
---

## 前言 

我们在开发网站的时候，通常会把常用的图标合并成css sprite（雪碧图），可以有效的减少站点的http请求数量，从而提高网站性能。

下面让我们一起来学习一下如何使用webpack合并css sprite图。

### 准备

- `webpack`
- `webpack-spritesmith`插件
- `file-loader`
- `sass-loader`(因为`webpack-spritesmith`除了生成雪碧图之外，还会生成相应的`mixin`，使用起来很方便，所以要用`sass`)
- 这里我还使用了`ExtractTextPlugin`来提取css文件，这不是必须的你也可以不用，用的话需要安装`extract-text-webpack-plugin`这个插件


使用npm安装好上面的东西

### 下面请开始我们的表演


#### step1:将我们要合并的图标准备好，放在src下的ico文件夹下
给大家分享一个好用的icon下载网站，里面的图标风格我很喜欢，而且是免费的哦，大家也可以到里面去下载自己喜欢的icon用于本次练习

[57,900 个免费的平面图标](https://zh.icons8.com/)

这是我下载的png图片

![facebook](http://ox4mn0gf1.bkt.clouddn.com/17-10-3/10535276.jpg)![twitter](http://ox4mn0gf1.bkt.clouddn.com/17-10-3/99027912.jpg)![google](http://ox4mn0gf1.bkt.clouddn.com/17-10-3/33750402.jpg)

![](http://ox4mn0gf1.bkt.clouddn.com/17-10-3/41330403.jpg)

#### step2 在你的webpack.config.js中按下面这样编写

```javascript
const path = require('path');
const SpritesmithPlugin = require('webpack-spritesmith');

module.export = {
    // ...
    module: {
        rules: [
            {
                test: /png$/
                loader:[
                    'file-loader?name=i/[hash].[ext]'
                ]
            },
            {
                test: /\.(css|scss)$/,
                use: ExtractTextPlugin.extract({
                    fallback: 'style-loader',
                    use: ['css-loader', 'postcss-loader', 'sass-loader']
                })
            }
        ]
    },
    
    resolve: {
        modules: [
            'node_modules',
            'assets/sprite' //css在哪里能找到sprite图
        ]
    },
    
    plugins: [
        new SpritesmithPlugin({
            src: {
                cwd: path.resolve(__dirname, 'src/ico'),  //准备合并成sprit的图片存放文件夹
                glob: '*.png'  //哪类图片
            },
            target: {
                image: path.resolve(__dirname, 'src/assets/sprites.png'),  // sprite图片保存路径
                css: path.resolve(__dirname, 'src/assets/_sprites.scss')  // 生成的sass保存在哪里
            },
            apiOptions: {
                cssImageRef: "~sprite.png" //css根据该指引找到sprite图
            }
        })
    ]
}
```

#### step3 在你的`scss`文件中`@import`导入生成的文件

```css
@import '../../../assets/sprite/_sprite.scss'; //路径请自行更改

.facebook{ 
    @include sprite($facebook); 
}

.twitter{ 
    @include sprite($twitter); 
}

.google{ 
    @include sprite($google); 
}

```

#### step4 运行webpack，看到我们的sprite图已经被用在我们的站点上了

/assets/sprite/sprite.png就是我们生成的sprite图了

![](http://ox4mn0gf1.bkt.clouddn.com/17-10-3/57992283.jpg)

看看同时生成的sass文件

```sass
// SCSS variables are information about icon's compiled state, stored under its original file name
//
// .icon-home {
//   width: $icon-home-width;
// }
//
// The large array-like variables contain all information about a single icon
// $icon-home: x y offset_x offset_y width height total_width total_height image_path;
//
// At the bottom of this section, we provide information about the spritesheet itself
// $spritesheet: width height image $spritesheet-sprites;
$facebook-name: 'facebook';
$facebook-x: 0px;
$facebook-y: 0px;
$facebook-offset-x: 0px;
$facebook-offset-y: 0px;
$facebook-width: 64px;
$facebook-height: 64px;
$facebook-total-width: 128px;
$facebook-total-height: 128px;
$facebook-image: '~sprite.png';
$facebook: (0px, 0px, 0px, 0px, 64px, 64px, 128px, 128px, '~sprite.png', 'facebook', );
$google-name: 'google';
$google-x: 64px;
$google-y: 0px;
$google-offset-x: -64px;
$google-offset-y: 0px;
$google-width: 64px;
$google-height: 64px;
$google-total-width: 128px;
$google-total-height: 128px;
$google-image: '~sprite.png';
$google: (64px, 0px, -64px, 0px, 64px, 64px, 128px, 128px, '~sprite.png', 'google', );
$twitter-name: 'twitter';
$twitter-x: 0px;
$twitter-y: 64px;
$twitter-offset-x: 0px;
$twitter-offset-y: -64px;
$twitter-width: 64px;
$twitter-height: 64px;
$twitter-total-width: 128px;
$twitter-total-height: 128px;
$twitter-image: '~sprite.png';
$twitter: (0px, 64px, 0px, -64px, 64px, 64px, 128px, 128px, '~sprite.png', 'twitter', );
$spritesheet-width: 128px;
$spritesheet-height: 128px;
$spritesheet-image: '~sprite.png';
$spritesheet-sprites: ($facebook, $google, $twitter, );
$spritesheet: (128px, 128px, '~sprite.png', $spritesheet-sprites, );

// The provided mixins are intended to be used with the array-like variables
//
// .icon-home {
//   @include sprite-width($icon-home);
// }
//
// .icon-email {
//   @include sprite($icon-email);
// }
//
// Example usage in HTML:
//
// `display: block` sprite:
// <div class="icon-home"></div>
//
// To change `display` (e.g. `display: inline-block;`), we suggest using a common CSS class:
//
// // CSS
// .icon {
//   display: inline-block;
// }
//
// // HTML
// <i class="icon icon-home"></i>
@mixin sprite-width($sprite) {
  width: nth($sprite, 5);
}

@mixin sprite-height($sprite) {
  height: nth($sprite, 6);
}

@mixin sprite-position($sprite) {
  $sprite-offset-x: nth($sprite, 3);
  $sprite-offset-y: nth($sprite, 4);
  background-position: $sprite-offset-x  $sprite-offset-y;
}

@mixin sprite-image($sprite) {
  $sprite-image: nth($sprite, 9);
  background-image: url(#{$sprite-image});
}

@mixin sprite($sprite) {
  @include sprite-image($sprite);
  @include sprite-position($sprite);
  @include sprite-width($sprite);
  @include sprite-height($sprite);
}

// The `sprites` mixin generates identical output to the CSS template
//   but can be overridden inside of SCSS
//
// @include sprites($spritesheet-sprites);
@mixin sprites($sprites) {
  @each $sprite in $sprites {
    $sprite-name: nth($sprite, 10);
    .#{$sprite-name} {
      @include sprite($sprite);
    }
  }
}

```

最后看看应用在网站上的效果

![](http://ox4mn0gf1.bkt.clouddn.com/17-10-3/23902519.jpg)


### 总结

好了，以上就是webpack生成css sprite的办法了，是不是觉着很简单呢，如果本文对您有帮组，请记得点个赞哦

> 参考
> - [webpack-spritesmith](https://www.npmjs.com/package/webpack-spritesmith)
