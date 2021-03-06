# System Design Guide
https://github.com/donnemartin/system-design-primer

# Domain Driven Design

## Ubiquitous Language

**Ubiquitous Language** - это Единый язык, который должен постепенно и итеративно вырабатываться и развиваться во время такого же итеративного и постепенного процесса проектирования и реализации системы. Этот язык позволяет выработать единую терминологию предметной области, на которой могут говорить как бизнес (эксперты в этой области), так и разработчики ПО для данной области. Единый язык ускоряет процесс понимания друг друга, позволяет писать код и моделировать систему в теримнах предметной области, так что сам код может быть понятен бизнесу. Пример:

```csharp
  // Ubiquitous Language. Это может понять даже не технический специалист. 
  var ticket = cashdesk.Sale(seat, count); 

  // Обычный код
  ticket.State = TicketState.Sold; // А это нет
  ticketRepository.Update(ticket);
```

Некоторые тезисы из книги Эванса касательно Ubiquitous Language

  - Используйте **модель** в качестве **основы** Ubiquitous Language.
  - Команда должна неуклонно применять Ubiquitous Language **при общении внутри команды и в коде**.
  - Ubiquitous Language должен использоваться в **диаграммах, письме и особенно в речи**. 
  - В случае возникновения трудностей с выраженим каких-то моделей или терминов в выработанном языке, нужно экспериментировать с альтернативными выражениями, которые отражают альтернативные модели, а после внедрения данных модеоей в язык, необходимо выполнить рефакторинг кода, переименовав классы, методы и модули, чтобы соответствовать новой модели. **В терминах языка не должно быть путаницы**, все, кто его использует должны однозначно понимать о чем речь
  - **Изменение в UBIQUITOUS LANGUAGE является изменением модели**. 
  - Специалисты по предметной области должны возражать против терминов или структур, которые неудобны или неадекватны для понимания предметной области; разработчики должны следить за двусмысленностью или непоследовательностью, которые могут привести к путанице в дизайне.
  - С UBIQUITOUS LANGUAGE модель не просто артефакт проектирования. Она становится неотъемлемой частью всего, что делают разработчики и эксперты в предметной области вместе. UBIQUITOUS LANGUAGE несет знания в динамической форме. Обсуждения на ЯЗЫКЕ "оживляет" диаграммы и код
  - Проектная документация тоже должна быть написана в терминах UBIQUITOUS LANGUAGE, кроме того она должна дополнять код, то есть не описывать реализацию и детали, а описывать проект на более высоком уровне, а также описывать **намерения и мотивацию, почему сделано именно так**. Документация должна обновляться вместе с эволюцией UBIQUITOUS LANGUAGE на проекте.

## Model-Driven Design

- Чистый анализ предметной области бизнес-анаитиками без участия девов часто ведет к тому, что сами девы не понимают домен, а модель выведенная аналитиками слишком абстрактна и плохо лодится на дизайн и его реализацию. Кроме того, теряется возможность самим девам _*вывести правильную модель*_ за счет итеративного процесса дизайна/имплементации системы для предметной области. Есть риск того, что такая чисто аналитическая модель в итоге будет заюрошена и не нужна.
- Если дизайн или какая-то его центральная часть не соответствуют модели предметной области, эта модель имеет небольшую ценность, и правильность спроектированной/написанной системы под сомнением. В то же время, сложные отношения между моделями и их проектами/реализациями в коде тоже трудно понять, и на практике их невозможно поддерживать при изменении проекта. Возникает разрыв меджу анализом и разработкой, так что понимание, полученное в каждом из этих видов деятельности, не влияет друг на друга.
- **Анализ должен охватывать фундаментальные понятия** из предметной области в понятной, выразительной форме. **Дизайн должен спроектировать набор компонентов**, которые могут быть созданы с использованием инструментов программирования, используемых в проекте, которые будут эффективно работать в целевой среде развертывания и будут правильно решать задачи, поставленные для приложения.
- MODEL-DRIVEN DESIGN отбрасывает дихотомию аналитической модели и дизайна для поиска **единой модели**, которая удовлетворяет обеим целям. Если оставить в стороне чисто технические проблемы, каждый объект в дизайне играет концептуальную роль, описанную в модели. Этот подход требует _*модель, которая хорошо работает как для анализа, так и для проектирования*_. Когда модель не кажется практичной для реализации, мы должны искать новую. Когда модель не совсем точно выражает ключевые концепции домена, мы должны искать новую. **Процесс моделирования и проектирования становится единым итерационным циклом**
- Спроектируйте часть системы, чтобы она буквально _*отражала модель предметной области*_, чтобы сопоставление было очевидным. Пересматривайте модель и изменяйте ее, чтобы она более естественно смотрелась в коде. Требуется **одна модель**, которая хорошо подходит для обеих целей, в дополнение к поддержке надежного UBIQUITOUS LANGUAGE.
- Извлеките из модели терминологию, используемую в проекте, и базовое распределение обязанностей. **Код становится выражением модели, поэтому изменение кода может быть изменением модели**. Его эффект должен распространяться на все активности проекта соответственно.
- **Разработка становится итеративным процессом уточнения модели, дизайна и кода как единая активность**

## Layered (Onion) architecture

Layer | Description
------------ | -------------
**User Interface (or Presentation Layer)** | Responsible for showing information to the user and interpreting the user'scommands. The external actor might sometimes be another computers ystem rather than a human user.
**Application Layer** | Defines the jobs the software is supposed to do and directs the expressivedomain objects to work out problems. The tasks this layer is responsible for are meaningful to the business or necessary for interaction with the application layers of other systems. This layer is kept thin. It does not contain business rules or knowledge, but only coordinates tasks and delegates work to collaborations of domaino bjects in the next layer down. It does not have state reflecting the business situation, but it can have state that reflects the progress of a task for the user or the program.
**Domain Layer (or Model Layer)** | Responsible for representing concepts of the business, information aboutthe business situation, and business rules. State that reflects the business situation is controlled and used here, even though the technical details of storing it are delegated to the infrastructure. This layer is the heart of business software.
**Infrastructure Layer** | Provides generic technical capabilities that support the higher layers: message sending for the application, persistence for the domain, drawing widgets for the UI, and so on. The infrastructure layer may also support the pattern of interactions between the four layers through an architectural framework.

The essential principle is that any element of a **layer depends only on other elements in the same layer or on elements of the layers "beneath" it**. Communication upward must pass through some indirect mechanism, like callbacks or observers. Concentrate all the code related to the domain model in one layer and isolate it from the user interface, application, and infrastructure code. **The domain objects, free of the responsibility of displaying themselves, storing themselves, managing application tasks, and so forth, can be focused on expressing the domain model**. This allows a model to evolve to be rich enough and clear enough to capture essential business knowledge and put it to work.

## Entities, Value Objects and Services

**Entity** - single, **identifiable** item of domain design.

**Value Object** - single, **immutable** item which does not have concept of identity in domain design. Value object describes characteristics of a thing.

**Services** - When a significant process or transformation in the domain is not a natural responsibility of an **ENTITY** or **VALUE OBJECT**, add an operation to the model as a standalone interface declared as a SERVICE. Define the interface in terms of the language of the model and make sure the operation name is part of the UBIQUITOUS LANGUAGE. Make the SERVICE stateless.

## Aggregate Root

Aggregate Root является шаблоном в DDD. Он представляет собой кластер объектов предметной области, которые можно рассматривать 
как единое целое. Примером может быть заказ и его элементы (items), которые могут представлять собой отдельные объекты, но все это
может быть полезно трактовать как единый **агрегат**.

Один из объектов данного агрегата будет являться **Aggregate Root**. Любые ссылки из-за пределов агрегата должны идти только 
к **Aggregate Root**. Таким образом, **Aggregate Root** может обеспечить целостность агрегата в целом.

Агрегаты являются основным элементом передачи и хранения данных. При данной концепции агрегаты загружаются и сохраняются целиком. 
Транзакции не должны пересекать границы агрегатов. 

**Aggregate Root** можно рассматривать как entry point в иерархию объектов предметной области. 

## Aggregate Root, Event Sourcing and CQRS

**Aggregate Root** может быть абстрактным классом, содеражщим все необходимые методы для **Command, Query and Event Handling**. 
Каждый потомок данного класса может представлять собой отдельный кластер из предметной области и иметь методы, специфичные 
для данного контекста, но при использовании CQRS обращаться к методам базового класса для выполнения комманд (command),
запросов (query), хранения Events и отката до определенного состояния по этим Events.

Пример тут: http://danielwhittaker.me/2014/11/15/aggregate-root-cqrs-event-sourcing/

# Microservices (из книги Сэма Ньюмена Building Microservices)

## Некоторые мысли по поводу подхода к разработке архитектуры

### A Principled Approach
Архитектурные решения всегда представляют собой **компромиссы**, в том числе и микросервисы. Мы можем выбирать между знакомыми технологиями, не совсем подходящими под решение задачи и незнакомыми - идеальныно подходящими под решение. Некоторые решения могут быть приняты на основе информации о системе, которая нам уже полностью доступна, но другие решение должны быть приняты на основе **неполной** информации  системе, инормации, которая нам еще недуступна. Для принятия подобных решений и оформления их в некий каркас можно определить набор **принципов** и **практик** основанных на **целях**, которые мы пытаемся достичь.

#### Стратегические цели
Стратегические цели - это цели компании, то есть высокоцровневые цели, которые могут не включать технологий вообще. Например, цель "Расширить свое присутствие в Азии". То есть архитектору надо понимать куда движется его компания, а с примером выше, он также понимает, что в связи с расширением, потенциально увеличится нагрузка на серверы компании, и, возможно, стоит закладывать архитектуру с учетом этой информации.

#### Принципы
Принципы - это правила, которые созданы для достижения конкретной **стратегической цели**. Принципы могут иногда меняться, в зависимости от обстоятельств или изменения целей. Например, если целью является уменьшение времени поствки новых фич на production, то прицнипом может стать организация самостоятельных команд ответственных за опредеденную часть системы, которую можно выпускать независимо от остальных. Или, в случае с примером захвата рынка Азии, принципом может стать создание максимально легко переносимой(portable) и развертываемой системы, для наиболее быстрого захвата нового рынка.

Принципов не должно быть много, не больше 10, иначе их тяжело будет запомнить и следовать им.

Некоторые прицнипы на самом деле могут представлять собой **ограничения**, которые тяжело обойти или изменить. Для собственного удобства и разделения вы можете их так и называть - ограничения, то есть у вас будут существовать определенные **прицнипы** и **ограничения**. Однако автор предлагает **не разделать** их и держать их в одном списке, чтобы время от времени сталкиваться со сложными ограничениями и видеть, действительно ли их нельзя обойти.

#### Практики
Практики - это средства выполнения и реалицации принципов. Это набор подробных практических рекомендаций для выполнения задач. Они часто будут специфичными для конкретной технологии и должны быть достаточно низкоуровневыми, чтобы любой разработчик мог их понять. Практика может включать в себя рекомендации по кодированию, тот факт, что все логи должны хранитьтся централизованно или что HTTP/REST является стандартом взаимодествия различных сервисов. Из-за их технического характера, практики часто будут меняться чаще, чем принципы.

Как и в случае с принципами, иногда практика отражает ограничения в вашей организации или системе. Например, если вы поддерживаете только CentOS, это необходимо будет отразить в вашей практике.

**Практика должна основываться на наших принципах**. Принцип, согласно которому команды разработчиков контролирует полный жизненный цикл своих подсистем, может означать, что у вас есть практика, указывающая, что все службы развернуты в изолированные аккаунты AWS, обеспечивая самообслуживание, управление ресурсами и изоляцию от других команд.

Разные команды в пределах одной организации могут исользовать разный набор практик, который однако мапится на один набор принципов.

A real-world example of principles and practices

![A real-world example of principles and practices](https://pbs.twimg.com/media/Dvi9qXoUcAAkAJ3.jpg)

### Evolurionary architect

1. **Vision**. Ensure there is a clearly communicated technical vision for the system that will help your system meet the requirements of your customers and organization
2. **Empathy**. Understand the impact of your decisions on your customers and colleagues
3. **Collaboration**. Engage with as many of your peers and colleagues as possible to help define, refine, and execute the vision
4. **Adaptability**. Make sure that the technical vision changes as your customers or organization requires it
5. **Autonomy**. Find the right balance between standardizing and enabling autonomy for your teams
6. **Governance**. Ensure that the system being implemented fits the technical vision

