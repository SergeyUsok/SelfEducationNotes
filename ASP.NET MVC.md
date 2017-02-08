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

### Razor
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

**ViewBag** - это свойство класса ControllerBase. Это dynamic object, в который можно добавлять любые свойства. Он используется для передачи данных из контроллера во view:
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

### Strongly Typed Model in View
Во View можно послать строго типизированную модель вместо использования ViewBag. Это даст возможность compile-time checks и включить  IntelliSense. Для этого в контроллере посылаем параметр во View:
```csharp
public ActionResult Details(int? id)
{
    if (id == null)
    {
        return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
    }
    Movie movie = db.Movies.Find(id);
    if (movie == null)
    {
        return HttpNotFound();
    }
    return View(movie);
}
```
А в самой View определяем тип модели через ключевое слово **@model** и работаем с ней:
```html
@model MvcMovie.Models.Movie

@{
    ViewBag.Title = "Details";
}

<h2>Details</h2>

<div>
    <h4>Movie</h4>
	<hr />
    <dl class="dl-horizontal">
        <dt>
            @Html.DisplayNameFor(model => model.Title)
        </dt>
         @*Markup omitted for clarity.*@        
    </dl>
</div>
<p>
    @Html.ActionLink("Edit", "Edit", new { id = Model.ID }) |
    @Html.ActionLink("Back to List", "Index")
</p>
```
### Data Annotations
Набор аттрибутов, как правило, применямый к моделям для определения различных метаданных и описания поведения членов модели в различных условиях. Пространство имен https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.aspx.

Пример:
```csharp
public class Movie
{
    public int ID { get; set; }
    public string Title { get; set; }

    [Display(Name = "Release Date")] // specifies how to display header for this property
    [DataType(DataType.Date)] // describes type of property
    [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)] // describes how to show property value
    public DateTime ReleaseDate { get; set; }
    
    public string Genre { get; set; }
    public decimal Price { get; set; }
}
```
Кроме того, здесь же содержатся атрибуты для различных видов валидации данных модели. При определении какого-либо атрибута валидации правила валидации распространяются и на клиент (jquery validation) и на сервер, т.о. мы защищаемся от случая, когда на клиенте отключен javascript.

Цитата Microsoft:

> What's really nice about this approach is that neither the controller nor the  Create view template knows anything about the actual validation rules being  enforced or about the specific error messages displayed. The validation rules  and the error strings are specified only in the Movie class. These  same validation rules are automatically applied to the Edit view and any other  views templates you might create that edit your model.

> If you want to change the validation logic later, you can do so in exactly  one place by adding validation attributes to the model (in this example, the movie class). You won't have to worry about different parts of the application  being inconsistent with how the rules are enforced — all validation logic will  be defined in one place and used everywhere. This keeps the code very clean, and  makes it easy to maintain and evolve. And it means that that you'll be fully  honoring the DRY principle.

Несколько из множества атрибутов:
 - StringLength - ограничивает длину строки
 - RegularExpression - собственное регулярное выражение для валидации
 - Required - обязательное поле
 - Range - задает разрешенный диапазон значений
 - DataType - задает тип для валидации, например Currency
 - и т.д.

Пример модели с DataAnnotations и валидацией:
```csharp
public class Movie
{
	public int ID { get; set; }

	[StringLength(60, MinimumLength = 3)]
	public string Title { get; set; }

	[Display(Name = "Release Date")]
	[DataType(DataType.Date)]
	[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
	public DateTime ReleaseDate { get; set; }

	[RegularExpression(@"^[A-Z]+[a-zA-Z''-'\s]*$")]
	[Required]
	[StringLength(30)]
	public string Genre { get; set; }

	[Range(1, 100)]
	[DataType(DataType.Currency)]
	public decimal Price { get; set; }

	[RegularExpression(@"^[A-Z]+[a-zA-Z''-'\s]*$")]
	[StringLength(5)]
	public string Rating { get; set; }
}
```

### HTTP method attributes, BindAttribute, ValidateAntiForgeryToken
В ASP.NET MVC существет несоклько аттрибутов, которые отражают Http методы (Put, Post, Delete, Head, Get). Они применяются к методам контроллера. Кроме того, HttpGet атрибут по-умолчанию всегда присутствует, поэтому он явно не пишется:
- \[HttpPost\]
- \[HttpPut\]
- \[HttpDelete\]
- \[HttpGet\]
- etc.

**BindAttribute** применяется для предоставления сведений о том, как должна осуществляться привязка модели к параметру формы и аргументу метода. Кроме того, этот атрибут является выжным механизмом безопасности, защищающим от overposting'а. Пример будет ниже.

**ValidateAntiForgeryToken** - это атрибут, который применяется в паре с вызовом **@Html.AntiForgeryToken()** во View. Атрибут является **фильтром** (см. ниже) и предусматривает ситуацию, когда пользователь ввел данные форму, отправил ее, но она не прошла валидацию и вернулсь назад пользователю. Чтобы пользователь заново не вводил данные атрибут вклюяает механизм заполнения формы уже ранее введенными данными.
@Html.AntiForgeryToken()  генерирует скрытый токен формы который должен совпадать с соответствующим параеметром метода в контроллере. Данный механизм позволяет защититься от атак Cross-site request forgery (also known as XSRF or CSRF).

Controller:
```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Edit([Bind(Include="ID,Title,ReleaseDate,Genre,Price")] Movie movie)
{
    if (ModelState.IsValid)
    {
        db.Entry(movie).State = EntityState.Modified;
        db.SaveChanges();
        return RedirectToAction("Index");
    }
    return View(movie);
}
```
View:
```html
@model MvcMovie.Models.Movie

@{
    ViewBag.Title = "Edit";
}
<h2>Edit</h2>
@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()    
    <div class="form-horizontal">
        <h4>Movie</h4>
        <hr />
        @Html.ValidationSummary(true)
        @Html.HiddenFor(model => model.ID)

        <div class="form-group">
            @Html.LabelFor(model => model.Title, new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Title)
                @Html.ValidationMessageFor(model => model.Title)
            </div>
        </div>
.......
```

### HtmlHelper
Класс **HtmlHelper** используется для облегчения работы с HTML контролами и их ренедеринга. Имеет множество extension методов, на которые стоит обратить внимание. Объект HtmlHelper находится в свойстве **Html** базового класса System.Web.Mvc.WebViewPage. Пример:
```html
<div>
    <h4>Movie</h4>
	<hr />
    <dl class="dl-horizontal">
        <dt>
            @Html.DisplayNameFor(model => model.Title)
        </dt>
         @*Markup omitted for clarity.*@        
    </dl>
</div>
<p>
    @Html.ActionLink("Edit", "Edit", new { id = Model.ID }) |
    @Html.ActionLink("Back to List", "Index")
</p>
```
### Filters
Фильтры позволяют добавить некоторую логику перед вызовом action'a или после него, то есть некий пре- и постпроцессинг. Фильтры реализованы как атрибуты, благодаря чему позволяют уменьшить объем кода в контроллере, т.е. фильтр вызывается автоматически и код его вызова не надо писать в каждом методе. Данные атрибуты могут применяться как ко всему классу, так и к отдельным его методам, свойствам и полям

В MVC реализовано 4 типа фильтров:
- Authorization filters. These implement **IAuthorizationFilter** and make security decisions about whether to execute an action method, such as performing authentication or validating properties of the request. The **AuthorizeAttribute** class and the **RequireHttpsAttribute** class are examples of an authorization filter. Authorization filters run before any other filter.

- Action filters. These implement **IActionFilter** and wrap the action method execution. The IActionFilter interface declares two methods: OnActionExecuting and OnActionExecuted. OnActionExecuting runs before the action method. OnActionExecuted runs after the action method and can perform additional processing, such as providing extra data to the action method, inspecting the return value, or canceling execution of the action method.

- Result filters. These implement **IResultFilter** and wrap execution of the ActionResult object. IResultFilter declares two methods: OnResultExecuting and OnResultExecuted. OnResultExecuting runs before the ActionResult object is executed. OnResultExecuted runs after the result and can perform additional processing of the result, such as modifying the HTTP response. The **OutputCacheAttribute** class is one example of a result filter.

- Exception filters. These implement **IExceptionFilter** and execute if there is an unhandled exception thrown during the execution of the ASP.NET MVC pipeline. Exception filters can be used for tasks such as logging or displaying an error page. The **HandleErrorAttribute** class is one example of an exception filter.

Класс Controller реализует все эти интерфейсы, т.о. методы этих интерфейсов могут быть переопределены в унаследованном контроллере.

Filters run in the following order:

1. Authorization filters
2. Action filters
3. Response filters
4. Exception filters

Можно создавать свои собственные фильтры. Кроме того, выполнение фильтров можно отменять. Подробнее здесь https://msdn.microsoft.com/en-us/library/gg416513(VS.98).aspx

### Code First Approach
Entity Framework дает возможность создать базу данных с таблицами автоматически, основываясь на сущностях в коде и их отношениях между собой (один-ко-многим, один-к-одному и т.д.)

Кроме того, Entity Framework предоставляет механизм миграций, когда надо поменять что-то в таблицах базы данных мы меням сущности в коде и таким образом эти изменения мапятся и отображаются в базе данных. Например, если надо добавить еще одно свойство в объект (т.е. колонку в таблицу) мы добавляем это свойство, включаем возможность миграции для нашего проекта, создаем файл миграции через nuget консоль и обновляем таблицу. Тоже самое можно делать, если необходимо поменять тип свойства (поля таблицы) или добавить какие-то ограничения для валидации. Подробнее здесь: https://www.asp.net/mvc/overview/getting-started/introduction/adding-a-new-field

### OWIN
OWIN - Open Web Interface for .NET. OWIN - это стандарт, а **Katana** - это его реализация от Microsoft. До появления OWIN ASP.NET можно было хостить только на IIS. Теперь же OWIN позволяет создать даже простой хост для веб-сервера в консоли. С помощью библиотек можно даже имитировать работу ASP.NET, но сам по себе OWIN сделан light-weight насколько это возможно. То есть он низкоуровневый.

Класс **Startup** в проекте MVC является точкой входа в приложение со стороны хостинга (IIS, Console etc.). Класс должен содержать метод **Configuration**, который принимает интерфейс **IAppBuilder** как аргумент.

Для работы OWIN использует модули, которые могут писаться сторонним разработчиком. Эти модули соединяются в pipeline, цепоска вызовов которого как раз настраивается **Configure** с помощью вызовов **IAppBuilder.Use()** с нужным модулем. Поскольку OWIN это спецификация, она задает интерфейс взаимодействия между модулями, основной которого является **Func\<IDictionary\<string, object\>,Task\> (так называемый AppFunc)**. IDictionary\<string, object> содержит всю envorinment информацию (url, request, response etc.). 
Порядок определения модулей (обработчиков) в методе Configure - **важен**.

Сами модули можно определять как классы, которые обязаны:
- иметь конструктор принимающий AppFunc
- иметь метод Task _Invoke(IDictionary\<string, object\> environment)_, в котором должна содержаться вся логика модуля
Также модули модно определять как делегаты в перегрузе метода Use(). 

```csharp
    public partial class Startup
    {
        public void ConfigureAuth(IAppBuilder app)
        {
            app.UseCookieAuthentication(new CookieAuthenticationOptions
            {
                AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
                LoginPath = new PathString("/Account/Login")
            });
	    
            app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);
        }
    }
```
Пример выше содержит определение 2 модулей, цепочка вызовов которых будет следующей:
UseCookieAuthentication -/> UseExternalSignInCookie
Оба метода являются методами расширения к стандартному методу Use и просто упрощают чтение кода.

### Security Note: 
**HttpServerUtility.HtmlEncode**  is being used to protect the application from malicious input (namely JavaScript).
For more information see [How to: Protect Against Script Exploits in a Web Application by Applying HTML Encoding to Strings] (msdn.microsoft.com/en-us/library/a2a4yykt(v=vs.100).aspx)
