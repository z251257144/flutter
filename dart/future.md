# Future及FutureBuilder

Dart代码运行在一个单线程。如果Dart代码阻塞了，整个程序就会卡死。比如：长时间的计算。

异步操作就是为了防止这一现象，可以允许程序在等待操作完成期间可以去完成其他工作。Dart使用`Future`对象（`futures`）来表示异步操作的结果。要想使用`futures`，你可以使用`async`和`await`，或者其他`Future API`。



#### 什么是Future？

 future是`Future`对象，它表示生成类型为T的异步操作。如果结果不是可用值，那么future的类型是`Future`。当函数返回的future被调用，会发生两件事：

1. 函数将要完成的工作进行排队，并且返回一个未完成的Future对象
2. 之后，当操作完成时，Future对象将以一个值或者一个错误来完成

当你需要完成Future的结果时，您有两个选择：

- 使用async和await

- 使用Future链式调用（then和catchError链式调用）

  

`async`和`await`关键字是`Dart`语言异步支持的一部分。它们允许你将 **异步代码** 转换为 **同步代码** 。async函数的主体前面有async关键字。await关键字只在异步函数中生效。代码示例如下：

```dart
  /*用户登录请求*/
  Future fetchLogin(phone, password) async {
    var param = Map<String, dynamic>();
    param["deviceId"] = await DeviceUtil.deviceID();
    param["deviceName"] = await DeviceUtil.deviceName();
    param["mobile"] = phone;
    param["pwd"] = password;
    debugPrint(param.toString());

    return super.requestPostData("user/m/login", param: param);
  }

	
  /*调用用户登录请求*/
  var result = await server.fetchLogin(phone, password);
  print(result.toString());

```



#### Future链式调用

- 使用future.then获取future值
- 使用cat捕获异常
- 使用whenComplete获取结束回调

```dart
main() {
  testFuture().then((value){
    print(value);
  },onError: (e) {
    print("onError: \$e");
  }).catchError((e){
    print("catchError: \$e");
  }).whenComplete(() {
    print("Done!!!")
  });
}

Future<String> testFuture() {
   return Future.value("Hello");
//  return Future.error("yxjie 创造个error");//onError会打印
//  throw "an Error";
}
```



#### Future 的常用函数

- **Future.then()** 任务执行完成会进入这里，能够获得返回的执行结果。
- **Future.catchError()** 有任务执行失败，可以在这里捕获异常。
- **Future.whenComplete()** 当任务停止时，最后会执行这里。
- **Future.wait() **可以等待多个异步任务执行完成后，再调用 then()。只要有一个执行失败，就会进入 catchError()。
- **Future.delayed()** 延迟执行一个延时任务。
- **Future.sync**同步执行
- **Future.microtask** 创建一个在微任务队列里运行的Future。dart下的异步任务队列有两个：`event queue`和`microtask queue`，`microtask queue`的优先级更高，而future的任务默认是属于`event queue`。Future.microtask就可以创建属于`microtask queue`的future。



### 什么是[FutureBuilder](https://flutter-academy.com/async-in-flutter-futurebuilder/)？

`FutureBuilder`是一个将异步操作和异步UI更新结合在一起的类，通过它我们可以将网络请求，数据库读取等的结果更新的页面上。

##### FutureBuilder的构造方法

```dart
FutureBuilder({Key key, Future<T> future, T initialData, @required AsyncWidgetBuilder<T> builder })
```

- `future`： Future对象表示此构建器当前连接的异步计算；
- `initialData`： 表示一个非空的Future完成前的初始化数据；
- `builder`： AsyncWidgetBuilder类型的回到函数，是一个基于异步交互构建widget的函数；

`builder`函数接受两个参数`BuildContext context`与 `AsyncSnapshot snapshot`，它返回一个widget。

```dart
///  * [FutureBuilder], which delegates to an [AsyncWidgetBuilder] to build
///    itself based on a snapshot from interacting with a [Future].
typedef AsyncWidgetBuilder<T> = Widget Function(BuildContext context, AsyncSnapshot<T> snapshot);
```

`AsyncSnapshot`包含异步计算的信息，它具有以下属性：

* `connectionState` - 枚举ConnectionState的值，表示与异步计算的连接状态，ConnectionState有四个值：none，waiting，active和done;
* `data` - 异步计算接收的最新数据；
* `error` - 异步计算接收的最新错误对象；
* `hasData`和`hasError`属性，检查它是否包含数据或错误。

现在我们可以看到使用`FutureBuilder`的基本模式。在创建新的FutureBuilder对象时，我们将Future对象作为要处理的异步计算传递。 在构建器函数中，我们检查connectionState的值，并使用AsyncSnapshot中的数据或错误返回不同的窗口小部件。

```dart
    //网络请求
	return FutureBuilder(
      future: future,
      builder: (BuildContext context, AsyncSnapshot snapshot){
        //返回数据
        if (snapshot.hasData) {
          return goodsDetailList();
        }
        
        //报错
        if (snapshot.hasError) return errorWidget(snapshot.error);
          
        //等待中  
        return Center(child: CircularProgressIndicator());
    });
```



使用FutureBuilder每次都需要创建等待界面、错误界面，这里我封装了**ZMFuture.builder**

```dart
import 'package:fire_shop/widgets/empty_view.dart';
import 'package:flutter/material.dart';

class ZMFuture<T> {
  static FutureBuilder builder({
    @required Future future,
    @required Widget view,
    Widget errorView,
    Widget loadingView
  }) {

    return FutureBuilder(
      future: future,
      builder: (BuildContext context, AsyncSnapshot snapshot) {
        if (snapshot.hasData) {
          return view;
        }

        // 错误界面
        if (snapshot.hasError) {
          if (errorView != null) {
            return errorView;
          }

          return ZMFuture.errorView(snapshot.error);
        };

        // 默认加载图
        if (loadingView != null) {
          return loadingView;
        }

        return Center(child: CircularProgressIndicator());
      }
    );
  }

  /*
  * 错误界面
  * */
  static Widget errorView(error) {
    return EmptyView(
    );
  }
}
```



使用方式如下

```dart
ZMFuture.builder(
  future: future,
  view: view
)
```































