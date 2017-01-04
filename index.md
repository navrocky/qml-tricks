# QML/QtQuick Tips&Tricks

## Полезные ссылки

http://qmlbook.org - хорошая книжка по QML
http://developer.ubuntu.com/apps/qml/ - мануал по QML/QtQuick из Ubuntu SDK

## Язык

Объявление констант и глобальных переменных

Существует два способа:
1. Создаем компонент от  QtObject, объявляем константы в виде свойств:

    MyConsts.qml
    ```qml
    import QtQuick 2.0
    QtObject {
       property string myTextConst: ”My text”
       property int myIntConst: 100
    }
    ```
    
    