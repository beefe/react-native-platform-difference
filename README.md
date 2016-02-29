# 标签差异

**1. TextInput**
- iOS下的`textAlign`取值 `auto` `left` `right` `center` `justify`  
- android下的`textAlign`取值为 `start` `center` `end`  
安卓平台无清除按钮，可以使用 [react-native-textinput](https://github.com/beefe/react-native-textinput) 来兼容双平台  

**2. Image**
- iOS下`resizeMode`可以写在行间属性，也可以写在style，后者覆盖前者  
- android下`resizeMode`只可以写在行间，写在style无效  
```javascript
    //android 
    <Image resizeMode={'stretch'}></Image>
    //ios, style中的cover会覆盖掉stretch
    <Image resizeMode={'stretch'} style={{'resizeMode': 'cover'}}></Image>
```