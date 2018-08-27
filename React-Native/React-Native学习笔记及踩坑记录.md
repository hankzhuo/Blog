## 组件

- `FlatList`：高性能列表组件
  - `refreshing`：loading 效果
  - `onRefresh`：下拉刷新功能
- `SectionList`：分组列表组件，与 FlatList 不同的是，这是可以分组的。
  - `renderSectionHeader`：当下一个section把它的前一个section的可视区推离屏幕的时候，让这个`section`的header粘连在屏幕的顶端（IOS 适用）
- `VirtualizedList`：只渲染视口的列表组件，窗口之外用空白渲染，是 `FlatList` 和 `SectionList` 底层的实现，具有更好的性能，但一般使用 `FlatList` 和 `SectionList`。
- `ScrollView`：滚动视图组件
- `swipeableFlatList`：列表可以从右侧滑出
- `Fetch`：获取网络数据，不过一般建议使用第三方库 `axios`
- `Linking`：通过调用相应方法 `Linking.canOpenURL(url)` 跳转手机浏览器

## 第三方库

- [`react-navigation`](https://reactnavigation.org/docs/en/getting-started.html) 导航器
  - `StackNavigator`：顶部导航组件
  - `BottomTabNavigator`：底部导航组件
  - `DrawNavigator`：侧滑导航组件
- [`react-native-device-info`](https://github.com/pusherman/react-native-network-info) 获取设备网络相关信息
- [`react-native-network-info`](https://github.com/pusherman/react-native-network-info)：获取网络`SSID`、`Password`、`IP`、`Broadcast` 等关于网络信息
- [`react-native-swipeable`](https://github.com/jshanson7/react-native-swipeable) 列表每项可以左右滑动的组件
- [`axios`](https://github.com/axios/axios)：用于 Node 和浏览器的基于 `promise` 的 HTTP 客户端
- [`react-native-i18n`](https://github.com/AlexanderZaytsev/react-native-i18n)：用于切换多国文字的库，默认是使用本地语言
- [react-native-qrcode-scanner](https://github.com/moaazsidat/react-native-qrcode-scanner)：二维码扫描库
- [`moment.js`](https://momentjs.com/) 跨时区、解决时间问题，必备库

## Bug

1、手机真机开启远程 JS 调试，`192.168.1.104:8081/debugger-ui/`（`192.168.1.104` 换成你的 ip 地址）；模拟器 JS 调试，浏览器打开`http://localhost:8081/debugger-ui/`。当然，我推荐使用 `React Native Debugger` 插件桌面版，可以需要切换，自动连接。

2、`TextInput` 组件无法输入中文，可以通过修改 `RCTBaseTextInputView.m` 文件。[参考解决方案](https://github.com/facebook/react-native/commit/892212bad2daadd373f4be241e4cd9889b0a1005)

3、使用 `TextInput` 组件，键盘底部文字默认是 `next` ，点击不能收起键盘，可以设置属性 `returnKeyType={'done'}` ，这样就可以收起键盘。

4、`Image` 组件在 `release` 版本上一定要加上 `resizeMode` 属性，否则图片会出现超出视图，`debugger` 倒是正常。

5、键盘升起时，会挡住视图，可以使用 `KeyboardAvoidingView ` 组件（`react-native` 组件），使得视图上移，键盘收起时，视图回归原始位置.

6、使用 `<TextInput keyboardType='numeric'/>` 组件时，键盘失去焦点，无法自动收回问题，[参考解决方案](https://stackoverflow.com/questions/29685421/hide-keyboard-in-react-native?rq=1)

7、系统字体变化后，`Text` 组件也会跟着变化，如何阻止解决不变化？[参考解决方案](https://stackoverflow.com/questions/44369600/how-to-handle-a-change-in-system-level-font-size-on-ios-with-react-native)

8、注册登录输入框，当用户以前登录过，再次进入时，焦点需要在密码输入框上。解决步骤是：
  - 把 `TextInput` 变为非受控组件 `ref = 'input`
  - 然后根据 `props` 判断 `this.refs.input.focus()`

9、`React-Navigation` 有个问题，连续点击两次，会出现黑屏，解决方案：设置 200 ms（根据实际业务而定）延迟，少于这个时间不执行跳转。

