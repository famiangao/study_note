### 点击事件

* qt 中移动端的点击事件用 onTapped 表示 ,例如
  TapHandler 是一个所有和点击有关的容器
* 网页端的鼠标点击事件 onClick 用了 MouseArea 这个容器 这个容器是为了检测
  所有的鼠标点击事件的容器

```qml
    Rectangle {
        id: rectangle
        x: 40
        y: 20
        width: 120
        height: 120
        color: "red"

        TapHandler {
            onTapped: rectangle.width += 10
        }
    }

```


```qml
Rectangle {
    width: 100
    height: 100
    color: "blue"

    MouseArea {
        anchors.fill: parent
        onClicked: {
            console.log("鼠标点击了我")
        }
    }
}

```

### 布局属性

**anchor**属性可以控制此块元素的布局,例如相对父元素居中,靠左等属性

```qml
Rectangle {
    width: 100
    height: 100
    color: "blue"
    anchors {
        bottom: parent.bottom
        bottomMargin: 20
        fill: parent.horizontalCenter
    }
}
```

#### 别的布局组件

除了 anchor 设置自己的布局之外,qml 还提供了

> using the RowLayout, ColumnLayout, and GridLayout QML types.

### 在代码中运行 js

可以通过函数?的方式运行 js

```
Image {
    source: {
        if (Screen.PixelDensity < 40)
        "image_low_dpi.png"
        else if (Screen.PixelDensity > 300)
        "image_high_dpi.png"
        else
        "image.png"
        }
    }
```

```qml
import QtQuick 2.12

Item {
    id: container
    width: 320
    height: 480

    function randomNumber() {
        return Math.random() * 360;
    }

    function getNumber() {
        return container.randomNumber();
    }

    TapHandler {
        // This line uses the JS function from the item
        onTapped: rectangle.rotation = container.getNumber();
    }

    Rectangle {
        color: "#272822"
        width: 320
        height: 480
    }

    Rectangle {
        id: rectangle
        anchors.centerIn: parent
        width: 160
        height: 160
        color: "green"
        Behavior on rotation { RotationAnimation { direction: RotationAnimation.Clockwise } }
    }

}
```

还可以直接运行 js 文件

```js
// myscript.js
function getRandom(previousValue) {
  return Math.floor(previousValue + Math.random() * 90) % 360;
}
```

```qml
import QtQuick 2.12
import "myscript.js" as Logic

Item {
    width: 320
    height: 480

    Rectangle {
        color: "#272822"
        width: 320
        height: 480
    }

    TapHandler {
        // This line uses the JS function from the separate JS file
        onTapped: rectangle.rotation = Logic.getRandom(rectangle.rotation);
    }

    Rectangle {
        id: rectangle
        anchors.centerIn: parent
        width: 160
        height: 160
        color: "green"
        Behavior on rotation { RotationAnimation { direction: RotationAnimation.Clockwise } }
    }

}
```

### 动态创建和添加组件
#### Qt.createComponent() 
Qt.createComponent() 是一个函数，用于创建一个 Component 实例。使用该函数，可以动态地创建组件并在需要时使用它们。

Qt.createComponent() 函数的语法如下：

```qml
function Qt.createComponent(source, parent, mode)
```

其中：

source：要创建的组件的源文件。可以是本地文件路径、网络 URL 或 QML 字符串。
parent：可选参数，表示新组件的父对象。
mode：可选参数，表示组件的创建模式，取值可以是 Component.NonRecursive 或 Component.Asynchronous。
使用 Qt.createComponent() 函数创建组件时，会返回一个 Component 实例。然后，可以使用该组件的 createObject() 方法，根据需要动态地创建实例。

以下是一个示例：

```qml
import QtQuick 2.0

Item {
    id: myItem
    
    Component {
        id: myComponent
        Rectangle {
            width: 100
            height: 100
            color: "red"
            border.width: 2
            border.color: "black"
        }
    }
    
    function createNewComponent() {
        var component = Qt.createComponent(myComponent)
        var object = component.createObject(myItem)
        //但更推荐下面这种方式，上面的方式如果销毁了myItem则组件跟着销毁（即使组件移到别的地方了）
        //下面这种方式组件独立存在
          var object = component.createObject()
          //或者(添加组件传参的方式）
          var object = component.createObject(null,{color:"pink"})
          object.parent=myItem
        //补充完毕
        
        
        object.x = Math.random() * myItem.width
        object.y = Math.random() * myItem.height
    }
}
```
在这个示例中，我们定义了一个 Item，其中包含一个名为 myComponent 的 Component。然后，我们在 createNewComponent() 函数中使用 Qt.createComponent() 创建了一个新的 Component 实例，并使用 createObject() 方法创建了一个新的 Rectangle 实例。这个新的 Rectangle 实例被添加到了 myItem 中，并随机地放置在 myItem 内部。

#### Qt.createQmlObject()
Qt.createQmlObject()是一个用于在运行时动态创建QML对象的函数。它通常用于创建小的、轻量级的组件，例如弹出窗口或动态添加的小部件。

它接受两个参数：QML代码和父级组件。QML代码可以是字符串形式的代码，也可以是指向QML文件的URL。父级组件是可选的，如果指定，则新创建的对象将成为该组件的子级。

下面是一个简单的例子，展示了如何使用Qt.createQmlObject()动态创建一个红色矩形：

```qml
Rectangle {
    id: container
    width: 200
    height: 200
    color: "gray"

    Component.onCompleted: {
        var rect = Qt.createQmlObject('import QtQuick 2.0; Rectangle { color: "red"; width: 100; height: 100 }', container)
        rect.x = container.width / 2 - rect.width / 2
        rect.y = container.height / 2 - rect.height / 2
    }
}
```
这个例子在完成组件初始化时创建了一个红色矩形，并将其添加到了container的子级。Qt.createQmlObject()函数接受一个QML字符串作为第一个参数，该字符串定义了要创建的矩形对象的属性。在这个例子中，我们定义了一个红色矩形，它的宽度和高度都是100像素。

第二个参数是父级组件，我们将它设置为container，这样新创建的矩形就成为了container的子级。最后，我们将矩形的x和y位置设置为容器的中心点，使其居中显示。

需要注意的是，Qt.createQmlObject()函数创建的对象是动态创建的，并没有被定义在QML文件中。因此，它没有自己的ID，也没有办法被其他组件引用。如果需要在其他地方引用该对象，可以将其保存在一个变量中，然后在需要的地方使用。


####区别
感觉上面的这个方法等同于 ：
```
var component = Qt.createComponent('import QtQuick 2.0;Rectangle { color: "red"; width: 100; height: 100 }')
var rect = component.createObject(container)
```
的结合,但Qt.createComponent()更加灵活，可以动态判断如果创建成功在添加到付组件中，例如:
```qml
var component = Qt.createComponent("MyComponent.qml");
if (component.status === Component.Ready) {
    var myObject = component.createObject(parent, {"x": 100, "y": 100});
} else {
    console.log("Error creating component");
}

```
> Component.status属性是一个指示Component对象是否已准备好创建对象的值。当Component对象已经成功加载，它的status属性为Component.Ready，这时可以通过它的createObject()方法来创建该组件对应的QML对象。而在Component对象的加载过程中出现错误时，其status属性为Component.Error。在使用Component对象之前，通常需要检查其status属性是否为Component.Ready，以确保该组件已经成功加载并准备好使用。


### 动态组件（动态添加点击事件）

```
import QtQuick 2.0

Rectangle {
    id: myRectangle
    width: 200
    height: 200
    color: "green"

    Component.onCompleted: {
        var mouseArea = mouseAreaComponent.createObject(myRectangle);
        mouseArea.clicked.connect(clickedHandler);
    }

    Component {
        id: mouseAreaComponent

        MouseArea {
            anchors.fill: parent
            hoverEnabled: true
        }
    }
    function clickedHandler() {
        console.log("Rectangle clicked");
    }
}

```
* Component.onCompleted:当组件渲染完成时调用
* Component：表示此布局为组件


### c++中的指针和引用
&和*的区别：
https://www.runoob.com/cplusplus/cpp-pointer-operators.html
#### 取地址运算符 &
& 是一元运算符，返回操作数的内存地址。例如，如果 var 是一个整型变量，则 &var 是它的地址。该运算符与其他一元运算符具有相同的优先级，在运算时它是从右向左顺序进行的。

您可以把 & 运算符读作"取地址运算符"，这意味着，&var 读作"var 的地址"。

#### 间接寻址运算符 *
第二个运算符是间接寻址运算符 *，它是 & 运算符的补充。* 是一元运算符，返回操作数所指定地址的变量的值。

请看下面的实例，理解这两种运算符的用法。

```c++
#include <iostream>
 
using namespace std;
 
int main ()
{
   int  var;
   int  *ptr;
   int  val;

   var = 3000;

   // 获取 var 的地址
   ptr = &var;

   // 获取 ptr 的值
   val = *ptr;
   cout << "Value of var :" << var << endl;
   cout << "Value of ptr :" << ptr << endl;
   cout << "Value of val :" << val << endl;

   return 0;
}
```
输出
```
Value of var :3000
Value of ptr :0xbff64494
Value of val :3000
```


```c++
& 变量名 //获取地址
* 指针名 //获取地址内的值
 指针  //获取地址(不加*)
** 指针名 //指向指针的指针
 //指针指向null将清除指针
```

普通数据获取变量的地址

```
#include <iostream>

using namespace std;

int main ()
{
   int  var1;
   char var2[10];

   cout << "var1 变量的地址： ";
   cout << &var1 << endl;

   cout << "var2 变量的地址： ";
   cout << &var2 << endl;

   return 0;
}
```

指针类型的定义

```
int    *ip;    /* 一个整型的指针 */
double *dp;    /* 一个 double 型的指针 */
float  *fp;    /* 一个浮点型的指针 */
char   *ch;    /* 一个字符型的指针 */
```

使用

```
#include <iostream>

using namespace std;

int main ()
{
   int  var = 20;   // 实际变量的声明
   int  *ip;        // 指针变量的声明

   ip = &var;       // 在指针变量中存储 var 的地址

   cout << "Value of var variable: ";
   cout << var << endl;

   // 输出在指针变量中存储的地址
   cout << "Address stored in ip variable: ";
   cout << ip << endl;

   // 访问指针中地址的值
   cout << "Value of *ip variable: ";
   cout << *ip << endl;

   return 0;
}
```

### 引用和指针的区别

C++ 引用 vs 指针
引用很容易与指针混淆，它们之间有三个主要的不同：

- 不存在空引用。引用必须连接到一块合法的内存。
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
- 引用必须在创建时被初始化。指针可以在任何时间被初始化。

使用例子

```
#include <iostream>

using namespace std;

int main ()
{
   // 声明简单的变量
   int    i;
   double d;

   // 声明引用变量
   int&    r = i;
   double& s = d;

   i = 5;
   cout << "Value of i : " << i << endl;
   cout << "Value of i reference : " << r  << endl;

   d = 11.7;
   cout << "Value of d : " << d << endl;
   cout << "Value of d reference : " << s  << endl;

   return 0;
}
```


### 类的指针
```c++
#include <iostream>
 
using namespace std;

class Box
{
   public:
      // 构造函数定义
      Box(double l=2.0, double b=2.0, double h=2.0)
      {
         cout <<"Constructor called." << endl;
         length = l;
         breadth = b;
         height = h;
      }
      double Volume()
      {
         return length * breadth * height;
      }
   private:
      double length;     // Length of a box
      double breadth;    // Breadth of a box
      double height;     // Height of a box
};

int main(void)
{
   Box Box1(3.3, 1.2, 1.5);    // Declare box1
   Box Box2(8.5, 6.0, 2.0);    // Declare box2
   Box *ptrBox;                // Declare pointer to a class.
   //上面这个等同于
   Box* ptrBox;

   // 保存第一个对象的地址
   ptrBox = &Box1;

   // 现在尝试使用成员访问运算符来访问成员
   cout << "Volume of Box1: " << ptrBox->Volume() << endl;

   // 保存第二个对象的地址
   ptrBox = &Box2;

   // 现在尝试使用成员访问运算符来访问成员
   cout << "Volume of Box2: " << ptrBox->Volume() << endl;
  
   return 0;
}


```

### c++中的动态内存分配，栈内存和堆内存
了解动态内存在 C++ 中是如何工作的是成为一名合格的 C++ 程序员必不可少的。C++ 程序中的内存分为两个部分：

栈：在函数内部声明的所有变量都将占用栈内存。
堆：这是程序中未使用的内存，在程序运行时可用于动态分配内存。
很多时候，您无法提前预知需要多少内存来存储某个定义变量中的特定信息，所需内存的大小需要在运行时才能确定。

在 C++ 中，您可以使用特殊的运算符为给定类型的变量在运行时分配堆内的内存，这会返回所分配的空间地址。这种运算符即 new 运算符。

如果您不再需要动态分配的内存空间，可以使用 delete 运算符，删除之前由 new 运算符分配的内存。
