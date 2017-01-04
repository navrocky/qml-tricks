# QML/QtQuick Tips&Tricks

## Полезные ссылки

http://qmlbook.org - хорошая книжка по QML

http://developer.ubuntu.com/apps/qml/ - мануал по QML/QtQuick из Ubuntu SDK

## Язык

### Объявление констант и глобальных переменных

Существует два способа:

1. Создаем компонент от  QtObject, объявляем константы в виде свойств:

`MyConsts.qml`
```qml
import QtQuick 2.0
QtObject {
   property string myTextConst: ”My text”
   property int myIntConst: 100
}
```
    
Используем:
```qml
import QtQuick 2.0
Rectangle {
   MyConsts { id: consts }
   width: consts.myIntConsts
}
```

**Недостаток:** предположительно не эффективно расходуются ресурсы.

**Достоинства:** константы типизированы и исполнение более эффективное, чем с нетипизированными значениями.

2. С помощью отдельного JS файла:

**Constants.js**
```js
.pragma library
var colorBackground = "#DFDFDF"; // gray
var colorForeground = "#FFFFFF"; // white
```

Используем:
```qml
import "js/Constants.js" as Consts
Item {
   property color itemColor: Consts.colorAccent
}
```
Достоинства и недостатки второго метода прямо противоположны таковым для первого метода.

### Скрытие деталей реализации (идиома PIMPL)

```qml
import QtQuick 2.0
Rectangle {
   // public section
   property int myProp: 100
 
   function action() {
       d.doSomeAction(); // use private
   }
     
   width: d.width // use private
   height: d.height
   // private section
   QtObject {
       id: d
       property int width: 100
       property int height: 100
       function doSomeAction() {
       }
   }
}
```

### Обращение к переопределенному родительскому методу (call overriden method)

Нет такой возможности. Известны только обходные пути:
* различное именование методов в родителе и дочернем компоненте
* можно в `Component.onCompleted` подменить функцию на новую, запомнить указатель на старую (недостаток - кододополнялка показывает метод предка)

### Объявление default property list\<Item\> (workaround)

По нормальному эта конструкция не парсится. Есть workaround - https://bugreports.qt.io/browse/QTBUG-26810

```qml
Item {
   id: root
   property list<Item> pages
   default property alias defaultPages: root.pages
```

### Двухсторонние биндинги

Используются в тех случаях, когда простого `property alias` не достаточно. Например, надо значение преобразовывать.
```qml
Item {
   id: root
 
   property int value: parseInt(textInput.text)
   Binding { target: textInput; property: "text"; value: root.value.toString() }
     
   TextInput {
       id: textInput
   }
}
```

### Мощь биндингов

Биндить можно не только формулы на JS, но и функции любой сложности, причем при изменении тех значений, от которых так или иначе зависит результат функции, происходит корректный пересчет.

Пример:
```qml
Item {
   property bool amIVisible: isItemVisibleToUser(this)
   onAmIVisibleChanged: console.log(amIVisible ? "Visible" : "Hidden")
   function isItemVisibleToUser(item) {
       while (item) {
           if (!item.visible)
               return false;
           item = item.parent
       }
       return true;
   }
}
```

## Компоненты

### Создание компонента

Компонент объявляется в отдельном файле `.qml`. Имя файла будет являться именем компонента.

Объявляем `MyCoolRect.qml`:
```qml
import QtQuick 2.0
Rectangle {
   width: 100
   height: 50
}
```

Используем `Main.qml`:
```qml
import QtQuick 2.0
MyCoolRect {
   width: 100
   height: 50
}
```

### Создание и использование делегата

Задача: необходимо передать внутрь своего компонента компонент, который будет объявлять пользователь.

Объявляем `MyCoolCtl.qml`:

```qml
import QtQuick 2.0
Rectangle {
   property Component delegate
   Loader {
       sourceComponent: delegate
   }
}
```

Используем `Main.qml`:
```qml
MyCoolCtl {
	delegate: Rectangle {
    	width: 100
        height: 50
    }
}
```

### Проброс свойств из компонента в делегат

Задача: в делегате нам надо видеть и использовать некоторые свойства из компонента, в котором мы определены.

Объявляем `MyCoolCtl.qml`:
```qml
import QtQuick 2.0
Rectangle {
   property Component delegate
   property string myProperty
   Loader {
       sourceComponent: delegate
   }
}
```

Используем в делегате свойство из компонента `MyCoolCtl`:
```qml
MyCoolCtl {
    delegate: Text {
        text: myProperty
    }
}
```
Еще есть вариант доступа к свойствам родительского компонента, когда имя свойства из модели совпадает с собственным свойством, `model.<имя свойства>`

### Использование функций перевода в ListModel (qsTr)

Использование функций перевода и вообще любых функций в `ListModel` запрещено по непонятным мне причинам. Есть тикет - https://bugreports.qt-project.org/browse/QTBUG-11403

Вместо `ListModel` можно использовать обычный список JS-объектов:
```qml
   id: root
 
   // define plain JS object list
   property var model: [
       { title: qsTr("Airplane"), descr: qsTr("Some descr 1") },
       { title: qsTr("Car"), descr: qsTr("Some descr 2") },
       { title: qsTr("Credit Card"), descr: qsTr("Some descr 3") }
   ]
   ListView {
       model: root.model
       delegate: Rectangle {
           height: 20
         
           // cross operability with ListModel and plain JS object list
           property var item: model.modelData ? model.modelData : model
         
           Text { text: item.title + " " + item.descr }
       }
   }
```

## Форматирование строки и преобразования типов

### Форматирование строки

```qml
"%1 - %2".arg(Number(10).toFixed(2)).arg(Qt.formatDate(
    "2014-01-25"))
// 10.00 - 1/25/14
```

### Число в строку

```qml
var val = 1234.56789;
console.log( val.toFixed(2) ); // 1234.57
console.log( val.toPrecision(3) ); // 1.2e+3
console.log( val.toLocaleString(Qt.locale(), 'f', 3) ); // 1,234.568
console.log( val.toLocaleCurrencyString(Qt.locale()) ); // $1,234.57
```