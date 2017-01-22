### MVC stands for model-view-controller.  
 
 MVC is a pattern for developing applications that are well architected, testable  and easy to maintain. MVC-based applications contain:

- **Models** : Classes that represent the data of the application  and that use validation logic to enforce business rules for that data.
- **Views**: Template files that your application uses to dynamically  generate HTML responses.
- **Controllers**: Classes that handle incoming browser requests,  retrieve model data, and then specify view templates that return a response  to the browser.

### Notes:
1. При создании констроллера создаем класс MyController, который должен содеражть префикс Controller и наследоваться от класса Controller.
2. При этом автоматически создается папка My (без префикса Controller) в папке Views. Там хранятся .cshtml файлы, то есть шаблоны для генерации
HTML контента под конкретный контроллер.
3. Имя контроллера служит часть URL адреса страницы, то есть MyController будет по адресу http://localhost:54898/**My**
4. Дефолтное отображение контроллера подгружается через публичный метод Index() определенный в контроллере
5. В контроллере модно задавать другие public методы, которые будут отображаться как часть URL, то есть если определить метод Welcome,
то его адрес будет http://localhost:54898/**My/Welcome**:

> ASP.NET MVC invokes different controller classes (and different action methods within  them) depending on the incoming URL. 
> The default URL routing logic used by ASP.NET  MVC uses a format like this to determine what code to invoke:
> **/[Controller]/[ActionName]/[Parameters]**
