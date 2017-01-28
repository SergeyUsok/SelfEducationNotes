### MVC stands for model-view-controller.  
 
 MVC is a pattern for developing applications that are well architected, testable  and easy to maintain. MVC-based applications contain:

- **Models** : Classes that represent the data of the application  and that use validation logic to enforce business rules for that data.
- **Views**: Template files that your application uses to dynamically  generate HTML responses.
- **Controllers**: Classes that handle incoming browser requests,  retrieve model data, and then specify view templates that return a response  to the browser.

### Notes:
1. При создании констроллера создаем класс MyController, который должен содеражть префикс Controller и наследоваться от класса Controller.
2. При этом автоматически создается папка My (без префикса Controller) в папке Views. Там хранятся .cshtml файлы, то есть шаблоны для генерации
HTML контента под конкретный контроллер.
3. Имя контроллера служит часть URL адреса страницы, то есть MyController будет по адресу http://localhost:port/**My**
4. Дефолтное отображение контроллера подгружается через публичный метод Index() определенный в контроллере
```csharp
public string Index()
{
    return "This is my <b>default</b> action...";
}
```
5. В контроллере модно задавать другие public методы, которые будут отображаться как часть URL, то есть если определить метод Welcome,
то его адрес будет http://localhost:port/**My/Welcome**:
```csharp
public string Welcome(string name, string surname)
{
    return HttpUtility.HtmlEncode("Hello " + name + ", surname: " + surname);
}
```
6. Параметры следуют за знаком **?** и разделяются знаком **&**. Такой механизм называется **query strings**, пример:
http://localhost:port/My/Welcome**?name=John&surname=Doe**
7. Кроме того, параметры можно посылать как часть URL адреса, но для этого нужно настроить Routes. То есть пример выше может выглядеть так:
http://localhost:port/My/Welcome**/John/Doe**

> ASP.NET MVC invokes different controller classes (and different action methods within  them) depending on the incoming URL. 
> The default URL routing logic used by ASP.NET  MVC uses a format like this to determine what code to invoke:
> **/[Controller]/[ActionName]/[Parameters]**

**You set the format for routing in the App_Start/RouteConfig.cs  file.**

### Razor:
Подходы описанные выше возвращают напрямую HTML-строку, что нарушает принцип MVC и смешивает представление и контроллер, кроме того так тяжело менеджить View. Для этих целей создан движок **Razor**.

**Razor** - это движок, для создания шаблонов View, в которых можно писать html-код и C# код для динамической генерации страниц. Файл шаблона имеет расширение **.cshtml** и **должен** иметь такое же имя как и соответствующий метод (action method), для которого шаблон определен. 
В контроллере View подключается через создание объекта в соответствующем методе:
```csharp
public ActionResult Index() 
{ 
    return View(); 
}
```
Папка Views содержит папку Shared, где хранятся Shared Views, которые общие для всех View в проекте. Там есть Layout Template, содержащий контейнер, в который подставляется текущее View. Таким образом, можно иметь Header and Footer сайта всегда один и тот же, а контент будет меняться.  Строка с **RenderBody** является placeholder для всех специфичных View, обернутых в Layout страницу. Например, если выбрать About ссылку, Views\Home\About.cshtml view ренедерится внутри метода **RenderBody**:
```html
<div class="container body-content">
        @RenderBody()
        <hr />
        <footer>
            <p>&copy; @DateTime.Now.Year - My ASP.NET Application</p>
        </footer>
    </div>
```
Папка Views содержит файл \_ViewStart.cshtml, который опредеялет layout для всех View, использующихся в проекте. Он содержит ссылку на shared layout view, поэтому нет смысла подключать shared layout view к специфическому view, поскольку оно возьмет layout из этого общего view:
```html
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
```
Однако это можно переопределить с специфическом view или вообще убрать и layout'a не будет.

**ViewBag** - это свойство класса ControllerBase. Это dynamic object, в который можно определять любые свойства. н используется для передачи данных из контроллера во view:
```csharp
public ActionResult Welcome(string name, int numTimes = 1)
{
    ViewBag.Message = "Hello " + name;
    ViewBag.NumTimes = numTimes;

    return View();
}
```
```html
<h2>Welcome</h2>

<ul>
    @for (int i = 0; i < ViewBag.NumTimes; i++)
    {
        <li>@ViewBag.Message</li>
    }
</ul>
```


А также ViewBag позволяет передавать параметры между специфичным View и Shared Layout.

Файл Views\\SpecificView.cshtml:
```html
@{
    ViewBag.Title = "My Title";
}
```
Файл Views\\Shared\\\_Layout.cshtml :
```html
@{
    <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>@ViewBag.Title - My App</title>
    <head>
}
```
В итоге при переходе на сраницу SpecificView title будет **My Title - My App**

### Security Note: 
**HttpServerUtility.HtmlEncode**  is being used to protect the application from malicious input (namely JavaScript).
For more information see [How to: Protect Against Script Exploits in a Web Application by Applying HTML Encoding to Strings] (msdn.microsoft.com/en-us/library/a2a4yykt(v=vs.100).aspx)
