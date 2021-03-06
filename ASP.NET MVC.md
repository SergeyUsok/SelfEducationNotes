### Table of Contents

- [MVC stands for model-view-controller](#mvc-stands-for-model-view-controller)
- [Notes](#notes)
- [Routing](#routing)
- [Razor](#razor)
	- [Strongly Typed Model in View](#strongly-typed-model-in-view)
	- [Partial Views](#partial-views)
	- [Html Helpers](#html-helpers)
- [Data Annotations](#data-annotations)
- [HTTP method attributes, BindAttribute (IModelBinder), ValidateAntiForgeryToken](#http-method-attributes-bindattribute-imodelbinder-validateantiforgerytoken)
- [Filters](#filters)
- [Code First Approach](#code-first-approach)
- [OWIN](#owin)
- [Авторизация и аутентификация](#Авторизация-и-аутентификация)
- [Entity Framework notes](#entity-framework-notes)
	- [Entity Framework handling concurrency](#entity-framework-handling-concurrency)
		- [Detecting Concurrency Conflicts](#detecting-concurrency-conflicts)
	- [Entity Framework handling inheritance](#entity-framework-handling-inheritance)
- [ASP.NET MVC Application Life Cycle](#aspnet-mvc-application-life-cycle)
- [Security Notes](#security-notes)
	- [XSRF/CSRF attacks](#xsrfcsrf-attacks)
	- [Open Redirection attacks](#open-redirection-attacks)
	- [Clickjacking](#clickjacking)
- [Web API](#web-api)
- [SignalR](#signalr)

### MVC stands for model-view-controller
 
 MVC is a pattern for developing applications that are well architected, testable  and easy to maintain. MVC-based applications contain:

- **Models** : Classes that represent the data of the application  and that use validation logic to enforce business rules for that data.
- **Views**: Template files that your application uses to dynamically  generate HTML responses.
- **Controllers**: Classes that handle incoming browser requests,  retrieve model data, and then specify view templates that return a response  to the browser.

### Notes
- При создании констроллера создаем класс MyController, который должен содеражть префикс Controller и наследоваться от класса Controller.
- При этом автоматически создается папка My (без префикса Controller) в папке Views. Там хранятся .cshtml файлы, то есть шаблоны для генерации HTML контента под конкретный контроллер.
- Имя контроллера служит часть URL адреса страницы, то есть MyController будет по адресу http://localhost:port/**My**
- Дефолтное отображение контроллера подгружается через публичный метод Index() определенный в контроллере
```csharp
public string Index()
{
    return "This is my <b>default</b> action...";
}
```

- В контроллере можно задавать другие public методы, которые будут отображаться как часть URL, то есть если определить метод Welcome,
то его адрес будет http://localhost:port/**My/Welcome**:

```csharp
public string Welcome(string name, string surname)
{
    return HttpUtility.HtmlEncode("Hello " + name + ", surname: " + surname);
}
```
- Параметры следуют за знаком **?** и разделяются знаком **&**. Такой механизм называется **query strings**, пример:
http://localhost:port/My/Welcome**?name=John&surname=Doe**
- Кроме того, параметры можно посылать как часть URL адреса, но для этого нужно настроить Routes. То есть пример выше может выглядеть так:
http://localhost:port/My/Welcome**/John/Doe**
- View подгружаются из папки Views -> подпапка с именем контроллера -> файл .cshtml с именем текущего метода контроллера. Это поведение по умолчанию, которое можно изменить и даже явно посылать путь к нужному View из метода контроллера.
- Объект **HttpContext** содержит инфо о браузере пользователя, url, ip-адресе и т.п. Доступ к немк можно получить через одноименное свойство HttpContext базового класса контроллера. Кроме того, в него можно напрямую записывать ответ в свойство Response, хотя это и не желательно делать. Также через этот объект можно работать с **cookies** через свойство **HttpContext.Request.Cookies**
- Базовый контроллер также содержит объект **Session**, с которым можно работать через одноименное свойство.

### Routing

> ASP.NET MVC invokes different controller classes (and different action methods within  them) depending on the incoming URL. 
> The default URL routing logic used by ASP.NET  MVC uses a format like this to determine what code to invoke:
> **/[Controller]/[ActionName]/[Parameters]**

**You set the format for routing in the App_Start/RouteConfig.cs  file.**
Порядок задания роутов **важен**. Поэтому более специфические роуты надо задавать перед более общими. 
Можно использовать псевдонимы для контроллера и его метода:
```csharp
routes.MapRoute( name: "Default2", url: "Store/Buy", defaults: new { controller = "Home", action="Index" } );
```
Здесь для контроллера HomeController используется псевдоним _Store_, а для действия Index - псевдоним _Buy_. В итоге данный маршрут будет сопоставляться с таким запросом, как _Store/Buy_.

Также, можно обозначить любое количество сегментов в запросе, чтобы не быть жестко привязанным к числу сегментов с помощью параметра **{\*catchall}**:
```csharp
routes.MapRoute(name: "Default", url: "{controller}/{action}/{id}/{*catchall}",defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional});
```
Обрабатывать все лишние сегменты URL должен разработчик в action'е контроллера. Получить их можно из объекта **RouteData**.

На роуты можно ставить ограничения **(constraint)**. Например, если поставить ограничение на контроллер, то контроллер вызовется, только если он не подпадает под ограничение. Ограничения погут быть RegEx, HttpMethod и т.д. Можно создавать свои ограничения. Подробнее здесь:
https://metanit.com/sharp/mvc5/6.3.php

Также существуют **area** (области). Их можно использовать как большие разделы сайта со своими контроллерами, представлениями и моделями. Для них нужно настраивать роутинг отдельно. Подробнее: https://metanit.com/sharp/mvc5/6.5.php

В MVC 5 появилась возможность устанавливать атрибуты маршрутизации для контроллеров и их методов **Route**. В атрибутах также сожно указывать placeholder'ы и ставить огрничения на параметры.

Существует возможность создания собственного обрабочтика маршрутов. Для этого надо реализовать **IRouteHandler** и зарегестрировать свой обработчик в **RouteConfig** для определенного роута (маршрута).

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
Папка Views содержит файл \_ViewStart.cshtml, который устанавливает layout для всех View, использующихся в проекте. Он содержит ссылку на shared layout view, поэтому нет смысла подключать shared layout view к специфическому view, поскольку оно возьмет layout из этого общего view:
```html
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
```
Код \_ViewStart.cshtml выполняется **до** кода любого View в проекте, т.о. он устанавливет layout для каждого view. Однако layout можно переопределить в специфическом view или вообще убрать и layout'a не будет.

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

#### Strongly Typed Model in View
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
#### Partial Views
Кроме обычных View, Controller Action может также возвращать **Partial View**. Их можно встраивать в обычне View. Это удобно, например, при показе результатов AJAX-запроса.

За рендеринг частичных представлений отвечает объект PartialViewResult, который возвращается методом PartialView.
В самом .cshtml файле Partial View можно рендерить с помощью двух вспомогательных методов:
- @Html.Partial("Имя\_Partial\_View")
- @{Html.RenderPartial("Имя\_Partial\_View");}

Отличие между ними в том, что второй надо вызывать в блоке кода, то есть обрамялть в {}. Кроме того, второй способ пишет напрямую в выходной поток, что может улучшитть производительность. 

Partial View также поддерживают строго типизированные модели.

#### Html Helpers
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
Можно определять свои extension методы для облегчения рендеринга HTML:
```csharp
// Render <img /> tag
public static MvcHtmlString Image(this HtmlHelper html, string src, string alt)
{
    TagBuilder img = new TagBuilder("img");
    img.MergeAttribute("src", src);
    img.MergeAttribute("alt", alt);
    return MvcHtmlString.Create(img.ToString(TagRenderMode.SelfClosing));
}
```
Кроме того, можно определять свои helper методы прямо в коде View c помощью директивы **@helper**.
```html
@helper BookList(IEnumerable<BookStore.Models.Book> books)
{
    <ul>
        @foreach (BookStore.Models.Book b in books)
        {
            <li>@b.Name</li>
        }
    </ul>
}
<h3>Books list</h3>
@BookList(ViewBag.Books)
<!-- for strongly typed Model -->
@BookList(Model)
```
Проблема данного способа в том, что оно смешивает коди представление. Чтобы как-то отделить подобные хелперы от view можно определить их в отдельном cshtml файле в папке **App_Code**. Это специально зарезервированное имя папки для HTML-хелперов. Внутри папки можно создать файл, например, Helpers.cshtml и там их определять.

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
 - Remote - находится в System.Web.Mvc. Позволяет выполнять валидацию на стороне клиента с обратными вызовами на сервер. Это удобно, например, для проверки незанятого логина в базе
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

### HTTP method attributes, BindAttribute (IModelBinder), ValidateAntiForgeryToken
В ASP.NET MVC существет несоклько аттрибутов, которые отражают Http методы (Put, Post, Delete, Head, Get). Они применяются к методам контроллера. Кроме того, HttpGet атрибут по-умолчанию всегда присутствует, поэтому он явно не пишется:
- \[HttpPost\]
- \[HttpPut\]
- \[HttpDelete\]
- \[HttpGet\]
- etc.

**BindAttribute** применяется для предоставления сведений о том, как должна осуществляться привязка модели к параметру формы и аргументу метода. Кроме того, этот атрибут является выжным механизмом безопасности, защищающим от overposting'а. Сам механизм привязки работает с помощью реализации интерфейса **IModelBinder**. Можно реализовать его самостоятельно, но в общем случае обычно достаточно класса **DefaultModelBinder**. Он работает за счет reflection и находит соответствия между именами свойств объекта, именами параметров формы и именами аргументов action метода. Пример использованиz атрибута Bind будет ниже.

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
    public class Startup
    {
        public void Configure(IAppBuilder app)
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
Пример выше содержит определение 2 модулей, цепочка вызовов которых будет следующей: <br />
UseCookieAuthentication -> UseExternalSignInCookie <br />
Оба метода являются методами расширения к стандартному методу Use и просто упрощают чтение кода.

### Авторизация и аутентификация
В ASP.NET MVC 5 появилась новая система авторизации и аутентификации **ASP.NET Identity**. Эта система пришла на смену провайдерам **Simple Membership**, которые были введены в ASP.NET MVC 4. Система ASP.NET Identity подключается в проект через **OWIN**.

Основные типы этой системы:

1. IUser - представляет пользователя. Интерфейс реализован классом IdentityUser для работы с EntityFramework. От него нужно наследоваться в собственном проекте.
2. IRole - представляет роль пользователя. Интерфейс реализован классом IdentityRole для работы с EntityFramework. От него нужно наследоваться в собственном проекте.
3. UserManager\<T\> - позволяет управлять пользователями, добавлять их, менять пароли, имейлы и т.п.
4. RoleManager\<T\> - позволяет управлять ролями в проекте.
5. IdentityDbContext - реализация DbContext для EntityFramework, которая уже содержит необходимые методы для работы с ролями, пользователями, авторизацией, аутентификацией и т.д. В своем проекте просто надо отнаследоваться от этого класса.
6. IUserStore и IRoleStore - представляют объекты для хранения и управления пользователями и ролями. Интерфейсы уже реализованы для EntityFramework. UserManager и RoleManager агрегируют эти объекты соответственно. Сами IUserStore и IRoleStore агрегируют IdentityDbContext для работы с базой данных.

### Entity Framework notes
1. EF позволяет перехватывать сгенерированные SQL команды, которые отправляются на SQL Server. Перехваченные команды можно логировать, менять и совершать какие-либо другие оперции с ними. Можно создавать несколько перезватчиков, которые вызываюся по очереди в порядке их добавления. Для создания перехватчика надо отнаследоваться от класса **DbCommandInterceptor** и переорпеделить необходимые методы, а также добавить экземпляр этого класса в список перехватчиков в _static_ метод _static_ класса **DbInterception.Add()**.
2. Property без публичного сеттера игнорируются Entity Framework.
3. Можно явно задать имя колонки с помощью атрибута Column. Также существуют атрибуты Key, ForeignKey.
4. A **navigation property** is an optional property on an entity type that allows for navigation from one end of an association to the other end
5. Если **navigation property** сделать как **virtual**, то EF автоматически включает **lazy loading**. Также существует **eagier loading** и **explicit loading**. Какой из трех применять, зависит от кнтекста задачи, у всх есть достоинства и недостатки.
6. EF поддерживает [Fluent API] (https://blogs.msdn.microsoft.com/aspnetue/2011/05/04/entity-framework-code-first-tutorial-supplement-what-is-going-on-in-a-fluent-api-call/). Fluent API может заменить собой атрибуты на моделях, плюс имеет дополнительные возможности. 
7. EF поддерживает использование транзакций.
8. EF поддерживает использование SQL запросов напрямую в базу. Подробнее [Performing Raw SQL queries] (https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/advanced-entity-framework-scenarios-for-an-mvc-web-application#rawsql)
9. По умолчанию EF трекает изменения в объектах по сравнению с данными в базе. Это поведение можно отключить: [No-Tracking Queries] (https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/advanced-entity-framework-scenarios-for-an-mvc-web-application#notracking)

#### Entity Framework handling concurrency
2 подхода по управлению параллелизмом:

1. **Pessimistic Concurrency (Locking)** - блокирование строки на update или readonly. В случае блокировки на update никто не может блокировать строку даже на update или readonly. В случае блокировки на readonly другие пользователи могут получить свою блокировку на readonly, но не на update. Упраление подобными блокировками сложное и не все DBMS поддерживают их. Поэтому EF не иммет встроенной поддержки подобного вида управления параллелизмом.
2. **Optimistic Concurrency** - позволяет случаться конфликтам связанным с параллельным измененениям данных, но реагирует и обрабатывает их соответственно. Несколько опций обработки конфликтов, используя Optimistic Concurrency:
  - Можно трекать какие property объекта подверглись изменениям, и если это разные свойства одного и того же объекта, то просто слить эти изменения вместе. Этот метод способен уменьшить количество конфликтов, но все равно не избавляет от них, если пользоваиели меняют одно и тоже свойство. Кроме того, он сложен в реализации и поддержке, особенно с увеличением количества сущностей и их свойств. Метод не практикуется в web applications.
  - Можно позволять последнему пользователю полностью изменять свойства объекта, игнорируя предыдущие изменения. Это назвается _Client Wins_ или _Last in Wins scenario_.
  - Можно не дать последнему изменяющему сделать изменения и показать ошибку вместе со списком измененных полей (как разницу между текущим состоянием и теми изменениями, которые вносит данный пользователь) и дать пользователю шанс выбрать - завершить изменения или отказаться от их. Это называется _Store Wins scenario_. Уже сохраненные данные имеют приоритет над изменениями.

##### Detecting Concurrency Conflicts
EF позволяет определять конфликты параллелизма 2-мя способами:

1) Добавить в таблицу, которую надо отслеживать на наличие изменений и конфликтов еще одну колонку. Тип колокни [rowversion] (https://msdn.microsoft.com/en-us/library/ms182776(v=sql.110).aspx). Затем надо сконфигурировать EF включать данную колонку в условие Where для  Update и Delete команд и отлавливать **OptimisticConcurrencyException** в коде в случае конфликта. Чтобы данный способ заработал, надо добавть в сущность свойство RowVersion и применить к нему атрибут **Timestamp**. Это сконфигурирует EF на включение данного свойства в условие Where.

```csharp
public class Entity
{
    public int ID { get; set; }

    [StringLength(50, MinimumLength = 3)]
    public string SomeProperty { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; }
}
```

2) Также можно сконфигурировать EF добавлять **каждую** колонку в условие Where для Update и Delete команд. Для таблиц и сущностей с большим количеством колонок (свойств) данный метод требует обработки большого количества состояний, ведет к большим запросам в Where и может просадиь производительность. Поэтому он не рекомендуется к использованию. Если все же необходимо применить данный подход, надо на все свойств, которые должны попасть в Where, применть атрибут **ConcurrencyCheck**

#### Entity Framework handling inheritance
Сущности для EF отображенные в коде в виде объектов могут иметь похожие свойства. Например, сущности Student и Teacher будет иметь свойства FirstName и LastName. Это приводит к повторениям в коде. Можно вывести базовый класс Person. EF поддерживает наследование для сущностей базы данных. Есть три способа отразить наследование в базе данных:

1. TPH (table-per-hierarchy) - одна общая таблица для всех унаследованных типов (то есть таблица Person), которая содержит колонки для всех свойств всех унаследованных типов. Обычно здесь используются _discriminator column_ для того, чтобы понять какому конкретному типу соответствует данныя колонка. Этот паттерн используется по умолчанию в EF.
2. TPT (table-per-type) - имитация наследования в БД, когда есть общая таблица Person с общими колонками, и 2 таблицы Student и Teacher c foreign key на таблицу Person. Данный подход может просадить производительность ввиду наличия join'ов, так как в коде будет идти работа с конкретным типом, а не абстрактным базовым, а значит поля из общей таблицы всегда будут нужны.
3. TPC (еable-per-сoncrete Class) - таблицы создаются только для конкретных типов и они содержат, в том числе, общие свойства. Это приводит к некоторой повторяемсоти колонок в разных таблицах, но для БД это не страшно. В данном примере будут только 2 таблицы Student и Teacher. 

### ASP.NET MVC Application Life Cycle
Точкой входа приложения является файл **Global.asax**. В нем определен класс-наследник *HttpApplication*, который содержит метод *Application_Start()*, в котором регистрируются маршруты, фильтры, бандлы, которые будут использоваться в приложении в дальнейшем.
Объект **HttpApplication** создается **дважды** - один раз при запуске самго приложения, именно здесь вызывается *Application_Start()*, а второй раз во время кождого запроса и здесь уже *Application_Start()* **не вызывается**.

Кроме того, существует набор событий, которые вызываются на протяжении обработки запроса (*BeginRequest*, *MapRequestHandler*, *AquireRequestState* и т.д.). На эти события можно пописаться в наследнике **HttpApplication**.

В приложении существуют модули, которые реализуют интерфейс **IHttpModule**. Их инициализация происходит после создания объекта приложения. С помощью модулей можно управлять обработкой запроса в приложении с помощью подписки на события, возникающие на протяжении всей обработки запроса. Кроме того, есть возможность определять собственные модули. Пример https://metanit.com/sharp/mvc5/19.3.php

Объект интерфейса **IHttpHandler** отвечает за вызов *ConrollersFactory*, которая возвращает нужный контроллер. Однако, можно определить собственный *HttpHandler*, который будет вызываться для определенного маршрута и реализовывать своб логику без обращения к *ConrollersFactory*.

Общий pipeline запроса выглядит примерно так:

Global.asax (Application_Start) -> Routing(HttpModules) -> HttpHandler -> 
ControllersFactory -> Controller -> ActionInvoker -> Action -> ViewEngine -> View

Пример из интернета:

![Image of ASP.NET MVC Pipeline](http://www.dotnettricks.com/img/mvc/ASP.NETMVC5Pipeline.png)

Еще примеры: http://saulius.sunauskas.com/2013/10/14/understanding-asp-net-mvc-request-processing-pipeline-visually/

### Security Notes
**HttpServerUtility.HtmlEncode**  is being used to protect the application from malicious input (namely JavaScript).
For more information see https://msdn.microsoft.com/en-us/library/a2a4yykt(v=vs.100).aspx (How to: Protect Against Script Exploits in a Web Application by Applying HTML Encoding to Strings)

#### XSRF/CSRF attacks
XSRF/CSRF (Cross-site request forgery) - атака, которая приводит к тому, что хакер может выполнить на неподготовленном сайте массу различных действий от имени других, зарегистрированных посетителей. Алгоритм атаки следующий:

1. Пользователь логиниться на сайт _some-site.com_
2. После удачного логина у пользователя сохранена сессия в куках
3. После работы на первом сайте пользователь заходит на второй "злой" сайт
4. На злом сайте JavaScript отрабатывает скрытую форму и посылет запрос на _some-site.com_ используя сохраненные куки пользователя
5. _some-site.com_ валидирует запрос и выполняет его. Т.о, если _some-site.com_, например, является сайтом для работы с банковски аккаунтом, "злой" сайт может перевести часть денег с пользовательского счета на свой счет с помощью кук пользователя.

Для того, чтобы предотвратить эту атаку сервер дополнительно генерирует **токен**, который проверяется вместе с куками и только при правильном токене и куках авторизации сервер позволит выполнить нужную операцию пользователю. Токе генерируется новый под каждую сесссию, не хранится в куках и "злой" сайт не сможет его узнать и подделать.

В ASP.NET MVC для генерации токена используются следующие механизмы:

1. Во View используется helper **Html.AntiForgeryToken()**, который генерирует hidden form element с токеном
2. В Контроллере к action использующему Token применяется атрибут **ValidateAntiForgeryToken** для валидации токена.
3. **AntiForgeryAdditionalDataProvider** позволяет разработчикам расширить стандартный механизм anti-XSRF системы

#### Open Redirection attacks
Атаки Open Redirection основываются на подмене адреса перенаправления пользователя на вредоносную страницу, которая внешне копирует оригинал. Алгоритм атаки:

1. Как правило, сайт после успешного log on'a перенаправляет пользователя на главную или какую-либо еще свою страницу.
2. Адрес перенаправления может быть указан в URL: http://site.com/Account/LogOn?**returnUrl=http://site.com/Home**
3. Используя фишинг злоумышленник подменяет в своем письме пользователю returnUrl, например, на http://sit**a**.com/Account/LogOn, то есть не на оригинальный сайт, а на подделку, которая выглядит как оригинал
4. Подделка сообщает пользователю, что login не удался и нужно ввести данные еще раз
5. Пользователь вводит логин и пароль и его данные теперь доступны злоумышленнику.

Для того, чтобы уберечься от этих атак достаточно проверять происходит ли redirect на локальный адрес (то есть тот же домен) или нет. Если нет, но редиректить куда-нибудь по умолчанию. В ASP.NET MVC начиная с версии 3 проверка **Url.IsLocalUrl(returnUrl)** присутствует сгенерированных Account контроллерах. В версиях MVC 1 и MVC 2 этой проверки нет.

#### Clickjacking
Атака Clickjacking представляет собой внедрение на страницу ресурса невидимого фрейма и подмена жлементами этого фрейма элементы реальной страницы. Таким образом пользователь жмет на кнопку, считая, что эта кнопка находится на этой же странице, но на самом деле он нажимает кнопку на внешнем фрейме, который не пренадлежит данному сайту. Так же можно подменить поля ввода логина и пароля. Подробнее по ссылке https://www.owasp.org/index.php/Clickjacking.

### Web API
Технология Web API позволяет создавать REST сервисы. Платформа Web API 2 не является частью фреймворка ASP.NET MVC и может быть задействована как в связке с MVC, так и в соединении с Web Forms. 

Основным классом платформы является **ApiController**, который никак не связан с классом **Controller** из ASP.NET MVC. При создании проекта Web API Visual Studio создает **ValuesController**, который является наследником **ApiController**. 

Есть 2 способа для того, чтобы методы в **ValuesController** соответствовали методам протокола HTTP:

1. Имена методов по умолчанию должны начинаться с имени предназначенного для них метода HTTP (Get, Post, Put, Delete)
2. Применить к методам контроллера атрибут для того, чтобы показать какому методу HTTP соответствует данный метод.

```csharp
public class ValuesController : ApiController
{
    public IEnumerable<string> GetAllItems()
    {
        return new string[] { "value1", "value2" };
    }
 
    public string GetItem(int id)
    {
        return "value";
    }
 
    [HttpPost]
    public void CreateItem([FromBody]string value)
    {}
 
    [HttpPut]
    public void EditItem(int id, [FromBody]string value) 
    {}
 
    [HttpDelete]
    public void RemoveItem(int id)
    {}
}
```
Методы контроллеров в Web API 2 могут возвращать различные типы:

- **void**: просто возвращает клиенту статусный код HTTP 204
- **HttpResponseMessage**: если метод возвращает объект HttpResponseMessage, то Web API преобразует возвращаемое значение в текст ответа HTTP
- **IHttpActionResult**: реализация этого интерфейса позволяет составить объекты, которые реализуют различные сценарии при обработке запроса
- любой другой тип: объекты любого другого типа сериализуются и добавляются в тело ответа, который также несет статусный код 200 (ОК)

При взаимодействии с клиентом между ним и сервером происходят два процесса: **content negotiation** (согласование содержимого) и **media formatting** (форматирование или сериализация содержимого). На первом этапе сервер узнает, в каком формате клиенту предпочтительнее отправлять ответ. На втором этапе происходит сериализация данных в этот формат **(можно добавииь собственный formatter для сериализации в кастоиный формат)**. Данные о формате клиент передает с помощью заголовка **Accept**. Web API имеет встроенную поддержку для следующих форматов: JSON, BSON, XML. Если заголовок Accept не передается, то по умолчанию используеся JSON. Пример явного возврщания ответа в JSON формате с помощью вспомогательного метода **Json()**:

```csharp
public IHttpActionResult Get()
{
    return Json(new List<Book>
    {
        new Book {Id=1, Name="Война и мир", Author="Л. Толстой" },
        new Book {Id=2, Name="Отцы и дети", Author="И. Тургенев" },
        //......................................
    });
}
```

В Web API определены все те же виды **фильтров**, что и в MVC 5 с аналогичным действием, но фильтры в Web API отличаются тем, что реализуют интерфейс **IFilter** из пространства имен *System.Web.Http.Filters*

В Web API доступны три вида аутентификации: 
- стандартная через куки
- через внешние сервисы
- аутентификация с помощью токена

Аутентификация с помощью токена имеет следующий pipeline:
1. Клиент (браузаер, мобильное приложение и т.д.) обращается в сервису указывая логин и пароль
2. Сервис проверяет логин и пароль и, если все в порядке, генерирует токен и отправляет его клиенту
3. Клиент получает токен и при каждом обращении к ресурсам сервиса добавляет токен в запрос
4. Сервис валидирует токен и отдает запрошенные данные клиенты, если токен прошел валидацию

Web API применяет спецификацию **Oath2**, которая описывает механизм аутентификации на основе токенов. Конкретную реализацию этой спецификации представляют компоненты **OWIN**.

### SignalR

SignalR - это библиотека для создания приложений осуществляющих коммуникации в реальном времени. SignalR предоставляет простой API для создания функционала, который позволяет вызывать функции JavaScript на стороне клиента из серверного кода, написанного с помощью языков платформы .NET. SignalR значительно упрощает работу с коммуникациями реального времени. Библиотека обрабатывает все подключения и автоматически рассылает сообщения всем подключенным клиентам, либо каким-нибудь специфическим клиентам. Фактически библиотека SignalR состоит из API серверной стороны, который применяется в коде на C#, и из клиентских библиотек JavaScript.

SignalR предоставляет разработчикам две модели: **Persistent Connection** (низкоуровневый протокл коммуникации) и **Hubs** (позволяют клиенту и серверу вызывать методы друг друга напрямую).

SignalR предоставляет несколько типов технологий для взаимодействия сервера и клиента. Если клиент или сервер не поддурживает какую-либо технологию, то выбирается та, которая поддерживается. Список технологий (от наиболее оптимальной к наименее):
- WebSockets
- Server-sent events
- Forever Frames
- Long polling
