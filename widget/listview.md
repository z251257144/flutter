# ListView

ListView 是Flutter最常用的滑动组件，用于线性展示widgets 列表。列表有四种方式：默认方式、ListView.builder、ListView.separated、 ListView.custom。



#### 默认方式

```dart
ListView(
	children: <Widget>[
          ListTile(title: Text("姓名")),
          ListTile(title: Text("电话")),
          ListTile(title: Text("年龄")),
          ListTile(title: Text("手机号")),
          ListTile(title: Text("邮箱"))
	],
)
```



#### ListView.builder

```dart
ListView.builder(
	itemCount: 10,
	itemBuilder: (context, index){
		return ListTile(title: Text("姓名"));
	}
)
```



#### ListView.separated

`ListView.separated`方法与`ListView.builder`类似，可以直接通过设置分隔线，更加方便。

```dart
ListView.separated(
	itemCount: 10,
	itemBuilder: (context, index){
		return ListTile(title: Text("姓名"));
	},
    separatorBuilder: (context, index) => Divider(),  // 分割线
)
```



> ListView.custom待完善

