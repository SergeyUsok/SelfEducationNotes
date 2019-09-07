### Xamarin разработку можно разбить на следующие пересекающиеся множества стратегий:

Application project - здесь, проект для конкретной платформы (Android, IOS, Windows Phone)

#### UI разработка
  1. Native development
      - UI для каждой платформы свой и **не может быть** переиспользован
      - Бизнес логика выделяется в отдельный проект и **может быть** переиспользована
      - Удобно для разработчиков знающих нативную разработку на определенной платформе
      - UI можно делать более платформозависимым, но при этом он может использовать все преимущества платформы
  2. Xamarin.Forms
      - абстракция поверх инфраструктуры Xamarin для разработки единого UI для всех платформ
      - из пункта 1 следует, что теряются преимущества каждой платформы и какие-то специфические контролы
      - удобно для .NET разработчиков не знающих нативной разработки под платформы
      - UI описывается некой абстракцией Page, которая затем компилирется в Activity (android) или AppDelegate (ios)
      - [согласно CEO Xamarin](https://news.ycombinator.com/item?id=9322702), Xamarin.Forms разрабатывались, чтобы помочь людям быстро написать очень простые приложения для ввода данных. Они не предназначены для требовательных к производительности приложений.
  3. Xaml проекты
      - тоже самое, что и Xamarin.Forms, только UI описывается на Xaml, что ближе для .NET разработчиков, но опять же теряются преимущества Native
      
#### Переиспользование кода (code sharing)
  1. Shared Project
      - позволяет переиспользовать код между множеством проектов
      - внутри кода можно использовать директивы компилятора #if __IOS__ для написания платформо-специфического кода
      - Application проекты (Android, IOS) могут содержать references на платформо-специфический код, который общий код может использовать
      - shared project не имеет 'output' assembly, то есть весь код компилируется в прямо в application сборку Android, IOS, WinPhone 
      - **не может иметь referenced assemblies из-за предыдущего пункта (нет сборки как выхлопа компиляции)**
      - даннное решение подходит для случаев, когда Common project будет исользоваться только подключенными в solution application проектами и не предназанчен для дистрибьюции как библиотека (через nuget например)
      - при написании Unit тестов надо линкать application projects, потому что shared project интегрирутеся в них
  2. Portable Class Libraries (PCL)
      - позволяет переиспользовать код между множеством проектов
      - нет возможности использовать директивы компилятора для написания платформо-специфичного кода
      - только подмножество классов .NET Framework доступно (определяется по выбранному профилю), а не весь .NET
      - платформо-специфичный код реальзиется через интерфейсы (я так понимаю методом обратного вызова, когда реализация интерфейса находится в application project, который знает для какой платформы он предназначен)
      - при написании Unit тестов достачно залинкать саму PCL библиотеку
      - хорошее решение для написания независимых библиотек для дистрибьюции (например через nuget)
  3. .NET Standard Libraries
      - позволяет переиспользовать код между множеством проектов
      - большее количество .NET классов (Base Class Library - BCL) доступно чем в PCL
      - нет возможности использовать директивы компилятора для написания платформо-специфичного кода
      - .NET Standard похоже на PCL, но с более простой моделью поддержки платформ и больших количество классов из BCL
      
Стратегии разработки описанные выше пересекются друг с другом (каждая с каждой), поэтому в Visual Studio существуют Project Templates, которые комбинирует эти подходы (Shared Xamarin.Xaml, PCL Xamarin.Forms, PCL Native etc.)

### Полезные ссылки

- Объясненение разницы между Shared и PCL с примерами приложений и их deployment'a в магазины: [https://jj09.net/getting-started-with-xamarin-in-2016/]
- Объяснение опций переиспользования кода с официального сайта: [https://developer.xamarin.com/guides/cross-platform/application_fundamentals/code-sharing/]

### Approaches of Android game dev
- Canvas (All drawing logic written by dev)
- WebView
    - Hybrid App (allows calls from JS to C#)
    - All logic written on JS
- Game Egnine
    - CocosSharp
    - UrhoSharp
- Native UI Controls
    - Predefined SDK controls
    - Custom Views

### Написание проложений и игр для разных платформ
1. Чистый JS (HTML, CSS + optionally Phaser or another Game Engine)
    - **Android:** завернуть web проложение в WebView
    - **Windows:** завернуть web проложение в exe с помощью NW.js или Electron
2. Corona is 2D Game engine based on Lua
    - **Android, iOS, Windows:** https://ru.coronalabs.com/
3. PhoneGap и/или Cordova
    - **Android, iOS, Windows:** https://www.phonegap.com/about/license/, https://cordova.apache.org/
4. NativeScript (JS + HTML + CSS)
    - **Android, iOS:** https://www.nativescript.org/
5. Cocos2D is Game Engine which has JS port
    - **Android, iOS, Windows:** https://github.com/cocos2d/cocos2d-x
6. Love2D is Game framework based on Lua (есть игра на Steam)
    - **Android, iOS, Windows:** https://love2d.org/
7. Flutter is SDK based on Dart lang developed by Google (это скорее для приложений, а не игр)
    - **Android, iOS, Windows:** https://flutter.dev/


Technology | Language | Android | IOS | Windows | Game
------------ | -------------
Vanilla JS + WebView | JavaScript | &#10004; | &#10004; | &#10004; | &#10004;
