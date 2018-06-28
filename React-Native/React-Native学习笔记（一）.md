## 组件

- ```FlatList```：高性能列表组件
  - ```refreshing```：loading 效果
  - ```onRefresh```：下拉刷新功能
- ```SectionList```：分组列表组件，与 FlatList 不同的是，这是可以分组的。
  - ```renderSectionHeader```：当下一个section把它的前一个section的可视区推离屏幕的时候，让这个```section```的header粘连在屏幕的顶端（IOS 适用）
- ```VirtualizedList```：只渲染视口的列表组件，窗口之外用空白渲染，是 ```FlatList``` 和 ```SectionList``` 底层的实现，具有更好的性能，但一般使用 ```FlatList``` 和 ```SectionList```。
- ```swipeableFlatList```：列表可以从右侧滑出
- ```DeviceEventEmitter```
- ```Fetch```：获取网络数据，不过一般建议适用第三方库 ```axios```

## 第三方库

- [react-navigation](https://reactnavigation.org/docs/en/getting-started.html) 导航器
  - ```StackNavigator```：顶部导航组件
  - ```BottomTabNavigator```：底部导航组件
  - ```DrawNavigator```：侧滑导航组件
- [react-native-device-info](https://github.com/pusherman/react-native-network-info) 获取设备网络相关信息
- [react-native-swipeable](https://github.com/jshanson7/react-native-swipeable) 列表每项可以左右滑动的组
件
- [axios](https://github.com/axios/axios)：用于 Node 和浏览器的基于 promise 的 HTTP 客户端
