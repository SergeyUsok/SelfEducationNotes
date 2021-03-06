# CSharp and .NET

## Dynamic

- В С# 4 допускается встраивание нужных частей основной сборки взаимодействия (primaryinteropassembly - РІА) в вызывающий код, поэтому отпадает необходимость в отдельном развертывании сборки РІА. Вместо ссылки на сборку РІА подобно любой другой сборке можно связать ее (**Jon Skeet**)

- Распознавание, какую версию перегруженной метода вызвать происходит за счет статического типа объявленной переменной, даже если переменная содержит наследник. Пример:

Есть 2 метода:

**_1) Execute(dynamic a, object b)_**

**_2) Execute(dynamic a, string b)_**

Переменная **_object text = "string";_** объявлена как **object**, поэтому будет вызван **1)**. Если бы переменная **text** была бы объявлена как **_dynamic_**, то вызвался бы **2)**, поскольку идет выяснение **_действительного_** типа переменной.

- Нужен для **reflection**, **COM interop**, **interop with dynamic languages**

- Явная реализация интерфейсов и dynamic сочетаются плохо

- Жизненный цикл обработки динамического выражения:

 ![Схема работы Dynamic](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/DotNET/Dynamic.png)

## Struct

- Структуры выгодно использовать из-за их легковесности (на содержат лишней информации как объекты), но **структуры всегда копируются при передаче в метод или при возращении из него** , а значит, что тяжелая структура всегда копируется и возможно проще использовать класс. ( **Jon Skeet** )

## Miscellaneous

- Система типов C#1 является статической (с появлением **dynamic** есть изменения), явной ( **var** это изменил) и безопасной. ( **Jon Skeet** )

- public event EventHandler Click = delegate {}; // делегат по умолчанию

- namespaces hints:

     + **уточнитель пространства имен ::**    

     + псевдоним глобального пространства имен **global ::**    

     + внешние псевдонимы **extern alias**    

- буфер фиксированного размера в **unsafe :**    

     + **fixed byte data [20];** // массив из 20 байтов; это позволяет работать с полем data так, как будто оно имело тип byte\*

- Два инициализатора анонимных объектов с одинаковыми именами свойств, следующими в том же самом порядке, но имеющими отличающиеся типы, приводят к созданию двух разных типов, на самом деле они генерируются из **одного обобщенного типа**

- Join метод Linq загружает в память именно **inner** последовательность, а по **outer** идет потоком. Выгодней inner делать небольшую последовательность.

- SortedList<Key,Value> - list backed collection

- SortedDictionary<Key,Value> - tree backed collection (красно-черное дерево)

- **Any CPU 32-bit preferred –** всегда компилится под 32 бит, просто поддерживает ARM из коробки

- **SynchronizationContext** - это стратегия, которая позволяет маршаллить вызовы между потоками. Стратегия, потому что у каждого "специального" потока, которому нужен **SyncContext**, своя реализация этого класса (WPF, WinForms etc.) Соответственно, если есть неооходимость в создании спецального потока, операции которого должны выполняться только в нем - можно реализовать свой **SyncContext**. Как пример, StaSyncContext для работы с потоком в STA Apartments для работы с COM.

## TPL, Async and PLINQ
- **Асинхронность дает отзывчивость на UI (UI поток не блокируется) и масштабируемость на сервере (потоки из пула не ждут окончания операции, а могут обрбатывать новые запросы)**
- В библиотечных методах лучше не использовать **Task.Run**. Пусть пользователь сам решит, надо ли ему многопоточность. Для имитации асинхронности тоже его не надо использовать, либо **async/await**, либо синхронный код
- **void** в async методах использовать только для обработчиков событий, поскольку это fire-and-forget, то можно попасть в ситуацию пропущенного исключения или чего-то подобного.
- При использовании async/await не гарантии, что метод будет выполняться асинхронно, поскольку в целях оптимизации CLR вполне может выполнить его синхронно. **Task.Yield()** позволяет явно сказать, что Continuation должен выполняться в другом потоке, т.е. он **forcibly fork method control flow**
- **Task.Delay()** можно использовать как **timeout**. Например, var task = Task.WhenAny(BusinessTask, Task.Delay(3000)), где BusinessTask - это нужная там таска, второй параметр - это **timeout**. Таким образом, если первой закончится таска **Task.Delay(3000)**, то это можно обрабатывать как **timeout expiration**.
- Метод PLINQ **ForAll()** отличается от обычного **foreach** тем, что он поребляет результаты в разных потоках, в то время как **foreach** обрабатывет все в одном потоке, в котором он собственнно и объявлен, поэтому предварительно необходимо объединить все результаты в один поток.
Разница работы в схеме: https://i-msdn.sec.s-msft.com/dynimg/IC387953.jpeg
- Платформа PLINQ не помещает одно исключение **OperationCanceledException** в исключение System.AggregateException. Исключение OperationCanceledException должно обрабатываться в отдельном блоке catch. Если один или несколько пользовательских делегатов создают исключение **OperationCanceledException()** и не создают другого исключения, а запрос был определен как AsParallel().WithCancellation(externalCT), PLINQ сгенерирует одно исключение **OperationCanceledException(externalCT)** вместо System.AggregateException. Однако если один пользовательский делегат создает исключение OperationCanceledException, а другой делегат создает другой тип исключения, оба исключения помещаются в AggregateException.
- Чтобы обеспечить отклик системы, рекомендуется проверять **CancellationToken** из пользоватльского кода приблизительно каждую **миллисекунду**. Однако допускается любой период **до 10 миллисекунд**.Эта частота не должна оказывать отрицательное влияние на производительность кода
- Когда цикл **Parallel.For** имеет небольшое тело (быстро выполняется), сам цикл может выполняться медленне чем обычный **for**. Снижение производительности происходит из-за нагрузки, связанной с секционированием данных **(partitioning)** и стоимостью вызова делегата в каждой итерации цикла. Для разрешения таких сценариев можно юзать метод Partitioner.Create, который позволяет выполнять последовательный цикл для тела делегата, чтобы делегат вызывался только один раз для каждой секции, а не по разу в каждой итерации. То есть имеется в виду, что мы заранее партицируем данные. Пример кода: https://msdn.microsoft.com/ru-ru/library/dd560853(v=vs.110).aspx
- **Task.Factory.StartNew()** имеет перегрузку, которая принимает **Object**, который можно назначить как состояние задачи. Это может быть любой объект. Доступ к нему во время инициализации задачи осуществляется из **Action(Object)**, а во время работы с задачей его можно взять из свойства **Task.AsyncState**. Такая же перегрузка есть для **Task.ContinueWith()**, то есть можно иметь **state** даже для продолжения.
- Если продолжение является присоединенной дочерней задачей, созданной с использованием параметра **TaskContinuationOptions.AttachedToParent**, его исключения будут распространяться родительской задачей обратно в вызывающий поток, как и в случае любой другой присоединенной дочерней задачи

|  Category | Attached Tasks  | Detached Tasks |
|---|---|---|
| Task Graph Lifecycle  | Parent Started -> Child Started -> Child Finished -> Parent Finished  | Parent Started -> Child Started -> Parent Finished -> Child Finished (or could be as in Attached)  |
| Исключения в дочерних задачах распространяются в вызывающий поток  | **Yes**  | **No**  |
| Состояние родительского объекта зависит от состояния дочернего объекта  | **Yes**  | **No**  |
| Отмена Child приводит к отмене Parent, TaskCanceledException распространяется в ожидающий Parent задачу поток | **Yes**  | **No**  |
- **UnobservedTaskException** происходит когда Task с необработанным исключением наичнает собираться **GC**. Таким образом мы можем подписаться на событие **TaskScheduler.UnobservedTaskException** и обработать его в "последний момент" и не валить процессс. Начиная с **.NET 4.5** UnobservedTaskException не валит процесс. Изменение в app.config даст поведение как в **.NET 4.0**
- Политика планирования задач: существует **GlobalQueue** (FIFO) для всех потоков ThreadPool'a и **LocalQueue**(LIFO) для каждого потока. В Global попадают Parent задачи, а в Local - Attached and Detached Child tasks. Если в Local очереди потока нет задач, они ищутся в Global, а если и там нет, то в Local очереди другого потока. **LongRunning** опция позволяет сказать, что задаче нужен выделенный поток и она точно не попадет в Local queue. 
- С помощью Task.Delay можно реализовать **асинхронный retry**, а с помощью CancellationTokenSource - **timeout** для асинхронной операции, которая не принимает на вход собственно сам CancellationToken:
```csharp
  async Task<string> DownloadStringWithTimeout(HttpClient client, string uri)
  {
     using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
     Task<string> downloadTask = client.GetStringAsync(uri);
     Task timeoutTask = Task.Delay(Timeout.InfiniteTimeSpan, cts.Token);

     Task completedTask = await Task.WhenAny(downloadTask, timeoutTask);
     if (completedTask == timeoutTask)
       return null;
     return await downloadTask;
  }
```
- **CancellationTokenSource** можно линковать с другим **CancellationTokenSource** используя метод _**CancellationTokenSource
      .CreateLinkedTokenSource(cancellationToken)**_. В итоге скомбинированный CancellationToken может раегировать на отмену при разных случаях, в зависимости от того какой из оригинальных токенов (из которых он скомбинирован) был вызван для отмены задачи. **Главное не забывать Dispose'ить комбинированные CancellationTokenSource**.
- **CancellationToken** имеет метод **Register**, в который можно передать callback для отмены операции. Это полезно для взаимодействия с кодом, который имеет логику отмены, но не основанную на CancellationToken. Т.о., если высокоуровневый код вызовет token.Cancel(), тот в свою очередь вызовет переданный в Register() callback и отменит операцию. Пример с классом Ping, который не поддерживает CancellationToken в своем API:
```csharp
  async Task<PingReply> PingAsync(string hostNameOrAddress,  CancellationToken cancellationToken)
  {
    using var ping = new Ping();
    Task<PingReply> task = ping.SendPingAsync(hostNameOrAddress);
    
    // Register callback which cancel Ping request when tokenSource.Cancel() invoked
    using CancellationTokenRegistration _ = cancellationToken.Register(() => ping.SendAsyncCancel());	  
    return await task;
  }
```
- **ValueTask\<T\>** может использоваться для тюнинга производительности, поскольку является значимым типом, но имеет ряд ограничений:
    - A ValueTask or ValueTask\<T\> may only be **awaited once**
    - ValueTask можно сконвертировать в Task с помощью метода AsTask, но только **один раз на одном экземпляре**
    - **Нельзя вызвать await и AsTask** на одном и том же экземпляре ValueTask
    - Synchronously getting results from a ValueTask or ValueTask\<T\> **may only be done once**, after the ValueTask has completed, and that same ValueTask **cannot be awaited or converted to a task**, но так лучше не делать вообще так как **Synchronously retrieving a result from Task\<T\> blocks the calling thread until the task completes; ValueTask\<T\> makes no such guarantees**

## Rx
- Помимо стандартных методов как в Linq'е, Rx имеет еще такие методы как:
	- Buffer - буферизирует входящие события
	- Window - группирует поток событий по N штук в каждой группе, но возвращает не List of List, а IObserveable<IObserveable<T>>, т.е. на каждую группу нужно пописаться отдельно, чтобы получить события 
	- Throttle - используется для того, чтобы не забить потребителя потоком сообщений. Метод принимает на вход интервал, по истечении которого отдает сообщение потребителю, но только в том случае, если другое сообщение не приходило, иначе интервал сбрасывается снова.
	- Sample - аналогичен Throttle, но метод не сбрасывает интервал, а просто отдает последний на данный момент элемент, когда интервал проходит, затем заново отсчитвает время.
	- Timeout - используется, чтобы задать время за которое каждый последующий эелемент должен прийти, иначе выбрасывет исключение.
- Observable.LastAsync/FirstAsync/ToList() можно **await**'ать, чтобы получить последний/первый/все эелементы асинхронно.
- Для конвертации Task'и или метода, который возвращает Task, в IObservable можно использовать:
	- ToObvservable() extension method on Task
	- Observable.StartAsync() - он поддерживает CancellationToken
	- Observable.FromAsync() - он поддерживает CancellationToken, но возвращает _*cold*_ IObservable, то есть он начнет работать, только при появлении подписчика, и каждый новый подписчик будет вызывать новый запуск метода переданного в FromAsync.


## C# 6.0 features:
https://github.com/dotnet/roslyn/wiki/New-Language-Features-in-C%23-6
