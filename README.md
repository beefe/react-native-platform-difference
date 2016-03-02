#React Native 双平台差异及解决方法(0.20)

### 标签及属性差异
--  

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


### API差异
--  

**1. `api` NativeMethodsMixin `static measure(callback: MeasureOnSuccessCallback)`**
- iOS下正常  
- android下，回调函数获取不到视图的尺寸 `x`、`y`、`width`、`heigth`、`pageX`、`pageY` 等值。可以考虑用 `onLayout` 属性来代替，需要注意的是，需要计算 layout 的视图，载入之后，防止内部不必要的重绘引起的尺寸变化，基本上保证 state 改变后，视图的尺寸即可预见。

**2. `api` `Dimensions.get('window').height`**
- 两个平台都是整个屏幕的高度(包含statusBar)  
- iOS平台的布局是从statusBar的顶端开始  
- android平台的布局是从statusBar的底端开始  
- 如果设置view的高度是`Dimensions.get('window').height`，然后设置`position: 'absolute', top: 0`，会发现android平台view的底端被遮住了一小部分，这一小部分正好就是android平台statusBar的高度。解决方法：设置view的高度是整屏的高度减去statusBar的高度，或者设置`top`值为负的statusBar的高度

**3. `组件` Picker**
- iOS下跟PickerIOS的UI一致(类似iOS平台的时间选择组件)，有较好的体验  
- android下则是由类似web端`select`标签的UI构成，体验较差  
- 可以使用组件 [react-native-picker](https://github.com/beefe/react-native-picker) 来兼容UI或实现多列picker

**4. `组件` Toast**
- iOS下暂时没有该组件  
- android下有`ToastAndroid`  
- 可以使用组件 [react-native-toast](https://github.com/remobile/react-native-toast) 来兼容双平台

**5. `组件` 轮播图**
- 可以使用组件 [react-native-slide](https://github.com/beefe/react-native-slide) 来兼容双平台

**6. `组件` 微信SDK**
- iOS下使用组件 [react-native-wechat-ios](https://github.com/beefe/react-native-wechat-ios)  
- android下使用组件 [react-native-wechat-android](https://github.com/beefe/react-native-wechat-android)


### 其他差异
--  

**1. 返回事件的处理**
- iOS如果使用`NavigatorIOS`，并且设置了`navigationBarHidden={true}`(隐藏NavigationBar)，将会导致右滑返回手势失效，解决方法：使用`Navigator`代替，但需要自己设置切场动画。  
- android物理返回键，需要在每个`Navigator`的源头使用`BackAndroid`做监听，如果`this.navigator.state.routeStack.length === 1`，则可以认为当前处于根view，可以退出app。否则将navigator往回退1即可。
```javascript
    //android
    import React, {Navigator, BackAndroid} from 'react-native';
    //拓展类似于NavigatorIOS的popN
    Navigator.prototype.popN = function(n) {
        let routeStack = this.state.routeStack;
        let index = routeStack.length - n - 1;
        if (routeStack[index]) {
            this.popToRoute(routeStack[index]);
        }
    };
    class MyApp extends React.Component {
        ...
        componentDidMount(){
            BackAndroid.addEventListener('hardwareBackPress', () => {
                //不在根view, 往回退1，阻止退出
                if(this._navigator.state.routeStack.length > 1){
                    this._navigator.popN(1);
                    return true;
                }
                //在根view，退出(或再次点击退出)
                else{
                    return false;
                }
            });
        }
        render(){
            <Navigator
                ref={(navigator) => {
                    if(navigator !== this._navigator){
                        this._navigator = navigator;
                    }
                }}
            />
        }
    }
```


