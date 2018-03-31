## 1.canvas 自动增加了display:none 属性--OK

增加了canvas-id就会展示了。

## 2. 拖拽延迟

https://developers.weixin.qq.com/blogdetail?action=get_post_info&docid=000c043a28c2e819bd36628d75b000&highline=旋转%20缩放&token=1894067750&lang=zh_CN

- 节流函数
- 移动量大于一定值得时候再setData()
- 使用动画API处理

原来这个原因：

```js
/**
   * 触摸小配件中
   */
  partTouchMove: function (e) {
    //获取当前小配件的partTouchData
    //let partTouchData = e.currentTarget.dataset.item.partTouchData;
    //获取当前小配件的id
    let id = e.currentTarget.dataset.item.id;
    let partTouchData = this.data.showDogs[id].partTouchData
```

把

```js
let partTouchData = e.currentTarget.dataset.item.partTouchData;
```

改为

```js
let partTouchData = this.data.showDogs[id].partTouchData
```

完美解决！！！

可能是通过data-绑定了复杂数据，不断循环获取的时候，会严重影响性能。



## 3. Canvas 画图之后会覆盖其他元素



## 4. Canvas 不能水平居中



## 5. movable-area 尝试一下这个布局

