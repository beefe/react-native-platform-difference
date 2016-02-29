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

**3. Text**
- iOS下最主要的`padding`、`lineHeight`的 style 属性都正常支持  
- android下`padding`、`lineHeight`无效，包括单独支持 android 的`textAlignVertical enum('auto', 'top', 'bottom', 'center')` 也不支持，解决办法就是 `padding` 改用 `margin`，或 `padding` 改用父节点 `padding`；`lineHeight` 改用 `marginTop` 负值
```javascript
    //双平台

    let isAndroid = Platform.OS === 'android';

    let styleTextLine1 = {
    	fontSize: 12,
    	marginBottom: 20,
	};
	let styleTextLine2 = {
		fontSize: 50,
		height: 50,
		lineHeight: 50, // ios
		marginTop: isAndroid ? -15 : 0,
	};

    <View style={{height: 100, justifyContent: 'center', alignItems: 'center'}}>
    	<Text style={styleTextLine1}>第一行，要求与第二行有间隔 20，字体大小 12</Text>
    	<Text style={styleTextLine2}>第二行，要求字体大小 50，字体上下居中，高度 50</Text>
    </View>
```

**4. style `position: 'absolute'`**
- iOS下正常  
- android下，`position: 'absolute'` 超过父节点高宽部分，会隐藏掉。解决办法：当需要用`position: 'absolute'`的时候，恰巧要求：子节点定位超出父节点高或宽，放弃使用，改用别的布局，或者将子节点放到于父节点同级，再定位。

**5. api `NativeMethodsMixin`**
- iOS下正常  
- android下，回调函数获取不到视图的尺寸 `x`、`y`、`width`、`heigth`、`pageX`、`pageY` 等值。可以考虑用 `onLayout` 属性来代替，需要注意的是，需要计算 layout 的视图，载入之后，防止内部不必要的重绘引起的尺寸变化，基本上保证 state 改变后，视图的尺寸即可预见。



