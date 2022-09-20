# Статическая типизация

В прошлых главах, говоря о читабельности, мы в основном обсуждали переменные, функции и модули. Мы иногда уделяли внимание особенностям статически типизированных языков, но не погружались в подробности.

Однако, статическая типизация тоже может быть инструментом для написания более выразительного кода. Мы можем использовать типы и интерфейсы для передачи дополнительной информации читателю или для наглядного проектирования приложения.

В этой главе мы обсудим, как передавать больше знаний о предметной области через типы и «запрещать» невалидные преобразования данных. Рассмотрим, как использовать повсеместный язык в сигнатурах функций и выслеживать с помощью типов ошибки в дизайне API.

| Перед началом 💬                                                                                                                                                                   |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| В тексте мы не будем обсуждать, _нужна ли_ статическая типизация. Вместо этого мы сосредоточимся на том, как использовать типы в качестве ещё одного инструмента для рефакторинга. |
| Статическая типизация вызывает много споров и нравится не всем, это нормально. Если вашему проекту типизация не подходит, эту главу можно пропустить.                              |

## Повсеместный язык

В “Domain Modeling Made Functional” Скотт Влашин предостерегает читателей от одержимости примитивами.[^dmmf][^primitiveobsession] Чтобы избежать её, он предлагает использовать типы для описания предметной области.

| Осторожно 🚨                                                                                                                          |
| :------------------------------------------------------------------------------------------------------------------------------------ |
| Идея замены примитивов на доменные типы может вызывать споры в команде. Перед применением обсудите эту идею с другими разработчиками. |

Типы данных и сигнатуры функций могут нести дополнительную информацию о задаче, которую код решает. Они могут отражать контекст задачи, взаимодействие сущностей и даже модель работы бизнес-процессов. При вдумчивом использовании типы могут стать альтернативой документации:

```ts
// Примитивные типы не передают контекста задачи,
// в них не достаёт деталей предметной области:

type Account = {
  date: string;
  user: number;
  value: number;
};

// Описание доменных типов помогает передать
// отношения между сущностями и принципы работы домена:

type Account = {
  date: DateTimeIso;
  user: UserId;
  value: MoneyAmount;
};
```

Когда названия типов используют термины из предметной области, они составляют часть _повсеместного языка (ubiquitous language)_[^ddd][^ubiquitouslanguage]. Этим языком выражаются владельцы бизнеса и люди, которые непосредственно связаны с бизнес-процессами.

| Подробнее 👀                                                                                |
| :------------------------------------------------------------------------------------------ |
| Более детально о повсеместном языке мы говорили в главах об именах сущностей и архитектуре. |

Польза повсеместного языка в его _однозначности_. Если вся команда, включая «неразработчиков» выражается одинаковыми терминами, вероятность потерь «при переводе с языка бизнеса на программистский» ниже. Баги и неправильную работу доменной модели на ранних этапах заметить проще.

## Моделирование домена

Во многих статически типизированных языках преобразования данных удобно выражать через функциональные типы.[^functionaltype] Совокупность таких типов может описывать процессы предметной области — моделировать домен. Польза такой модели в том, что цена ошибки в ней ниже, чем при реализации бизнес-процессов в коде.

Типы помогают составить «верхнеуровневое» понимание работы системы. В такой модели видно взаимодействие её частей, контракты модулей и используемые данные. Но также в ней видно ошибки проектирования и несоответствия модели реальному миру.

Ошибки в типах исправить проще, чем ошибки в реализации. Так мы можем проектировать работу приложения _до_ того, как приступить к реализации. Пишем черновик, смотрим на недостатки модели, переделываем и повторяем проверку:

```ts
// Опишем данные, с которыми работает приложение.
// Укажем различные состояния, через которые они проходят
// на разных стадиях жизненного цикла приложения:

type CreatedOrder = {
  createdAt: TimeStamp;
  user: UserId;
  items: ProductList;
};

type ValidatedOrder = {
  /*...*/
};

type DiscountedOrder = {
  /*...*/
};

type Order = CreatedOrder | ValidatedOrder | DiscountedOrder;

// Спроектируем процессы предметной области,
// которые преобразуют эти данные:

type CreateOrder = (user: UserId, items: ProductList) => CreatedOrder;
type ValidateOrder = (order: CreatedOrder) => ValidatedOrder;
type ApplyDiscount = (order: ValidatedOrder, value: Price) => DiscountedOrder;

// Если мы заметим, что в каком-то из типов есть ошибка,
// (например, процесс отличается от того, что происходит в реальности)
// мы можем быстро и относительно дёшево исправить модель:

type ApplyDiscount = (
  order: ValidatedOrder,
  coupon: DiscountCoupon
) => DiscountedOrder;
```

Во время рефакторинга такие проверки помогают обнаружить бизнес-процессы, которые не отвечают требованиям проекта или не учитывают ограничений предметной области:

```ts
type Divider = (a: number, b: number) => number;
const divide: Divider = (a, b) => a / b;

// Принимает ли тип `divide` вторым аргументом ноль?
// Стоит ли это разрешить?
// Если да, как обработать деление на ноль?
// Нужно ли возвращать контейнер из этой функции?

// Мы можем обогатить доменную модель
// дополнительными данными и ограничениями:

type RealNumber = // ...Допускает любое число.
type NaturalNumber = // ...Допускает число больше нуля.

type Divider =  (a: NaturalNumber, b: NaturalNumber) => RealNumber;
const divide: Divider = (a, b) => a / b;

// Теперь из типа `Divider` ясно, что деление на ноль
// уже должно быть обработано в вызывающем коде.
```

### Типизация в TypeScript

В TypeScript для моделирования домена и создания доменных типов мы можем использовать несколько способов:[^typealias][^typebranding][^factorymethod]

- Использовать тайп-алиасы;
- Использовать классы в качестве типов;
- «Брендировать» типы.

Самый простой, но при этом самый ненадёжный способ создания доменных типов — использовать тайп-алиасы.[^typealias] С их помощью удобно давать примитивным типам информативные имена, но сложно передавать _ограничения_ предметной области. Например, такой код синтаксически вполне валиден, а с точки зрения домена — нет:

```ts
// Тайп-алиас может дать примитиву полезное имя,
// которое будет отражать смысл типа согласно домену:
type NaturalNumber = number;

// Но валидации значений в нём не будет:
const x: NaturalNumber = -1;
```

Тайп-алиасы по умолчанию не валидируют присвоенные значения, поэтому нет гарантий, что в `NaturalNumber` окажется именно натуральное число:

```ts
function divide(a: NaturalNumber, b: NaturalNumber): RealNumber {
  return a / b;
}

// Компилятор доволен, а в рантайме ошибка:
divide(1, 0);
```

Поэтому если нам нужна настоящая проверка на несопоставимость разных типов, придётся использовать классы или брендированные типы:[^typebranding][^factorymethod]

```ts
// При использовании классов
// мы можем добавить валидацию значений в конструктор:

class NaturalNumber {
  constructor(value) {
    if (value <= 0 || Math.floor(value) !== value) {
      throw new Error("The value must be a positive integer.");
    }

    this.value = value;
  }
}

// Тогда создать «неправильное» значение не получится:

new NaturalNumber(-1); // Error!
new NaturalNumber(42); // NaturalNumber

// Но классы довольно многословные
// и ими не удобно пользоваться, например,
// если нужно сделать обёртку над примитивом.

// Создавать «числа» через new NaturalNumber(42),
// а потом как-то реализовать арифметические операции
// с такими значениями — накладно и требует много кода.

// Второй вариант — использовать брендирование типов:

type Tagged<T, S> = T & { __tag: S };
type NaturalNumber = Tagged<number, "natural">;

// А значения создавать только через специальные фабрики:

function naturalFrom(x: number): NaturalNumber {
  // Всю валидацию значений можно будет реализовать
  // внутри такой функции-фабрики:

  if (value <= 0 || Math.floor(value) !== value) {
    throw new Error("The value must be a positive integer.");
  }

  return x as NaturalNumber;
}

naturalFrom(-1); // Error!
naturalFrom(42); // NaturalNumber
```

Проблема классов и «брендирования» в том, что за правильностью их использования необходимо следить. То есть понадобится писать правила для линтера или искать ошибки во время код-ревью. Это ненадёжно.

Здесь сложно порекомендовать конкретный способ, всё зависит от проекта и нужд команды. Однако, можем отметить, что для _чисто описательных_ целей даже тайп-алиасы вполне подходят. Часто доменной модели, построенной на тайп-алиасах, уже достаточно, чтобы найти необнаруженные ранее ошибки в проектировании.

## Сверка модели с реальностью

Бизнес-процессы преобразуют данные из одного состояния в другое. Типы могут помочь синтаксически зафиксировать эти состояния и дать им названия. Когда каждый шаг преобразования как-то назван, нам проще рассуждать о процессе целиком и находить ошибки в его логике.

В примере ниже функция `sendRecoverLink` работает с объектом типа `User`. У этого типа есть флаг `verified`, но нет никаких правил, объяснявших бы, _когда и почему_ этот флаг принимает значение `true`:

```ts
type User = {
  id: string;
  verified?: boolean;
};

async function sendRecoverLink(user: User) {
  if (!user.verified) return false;
  await api.recoverPassword(user.id);
}
```

Из-за текущей реализации `User` функция `sendRecoverLink` принимает данные, которые в половине случаев для неё невалидны. Мы можем перестраховаться от невалидной передачи данных, затруднив её на уровне типов.

Верификация пользователя — вероятно, отдельный бизнес-процесс, _в результате которого_ объект пользователя становится верифицированным. Эту причинно-следственную связь можно выразить прямо в типах, если разделить типы верифицированных и неверифицированных пользователей:

```ts
// Описываем типы верифицированных и неверифицированных пользователей,
// как состояния, через которые проходят данные:

type CreatedUser = { name: string };
type VerifiedUser = { name: string; verified: true };

type User = CreatedUser | VerifiedUser;

// Показываем причинно-следственную связь, как тип процесса верификации.
// Рассказываем, через какие шаги проходят данные, и в результате чего
// пользователь становится верифицированным:

type VerifyUser = (user: CreatedUser) => Promise<VerifiedUser>;

// Фиксируем ограничения на восстановление пароля —
// восстановить пароль может только верифицированный пользователь:

type RecoverPassword = (user: VerifiedUser) => Promise<void>;

// Теперь выслать ссылку на восстановление пароля
// неверифицированному пользователю будет нельзя.
// Невалидную операцию выразить в коде станет сложнее:

const sendRecoverLink: RecoverPassword = async (user) => {
  await api.recoverPassword(user.id);
};

sendRecoverLink(unverifiedUser); // Error!
```

Опять же, в TypeScript добиться бронебойной невыразимости невалидных процессов сложно, в других типизированных языках это может быть проще. Но даже просто фиксация разных состояний данных в типах помогает замечать ошибки в логике бизнес-процессов на этапе проектирования или ревью.

В типах подобные ошибки замечать проще, потому что выражать условия в типах сложнее, чем в реализации. Это заставляет описывать процессы декларативно и прямолинейно. И если в логике преобразования заметна неоднозначность, это может стать сигналом проблем.

### Нарушения договорённостей

Благодаря явным типам, мы можем выявлять нарушения договорённостей или принятых в проекте правил. Например, функция `sendRecoverLink` из предпредыдущего примера нарушает CQS:

```ts
async function sendRecoverLink(user: User) {
  if (!user.verified) return false;
  await api.recoverPassword(user.id);
}

type RecoverPassword = (user: User) => Promise<false | void>;

// `false` — это признак запроса,
// `void` — признак команды.
```

Типы обращают внимание на подобные противоречия. Мы далее можем улучшить функцию, например, использовав в имени паттерн `try*`:

```ts
async function trySendRecoverLink(user: User) {
  if (!user.verified) return false;
  await api.recoverPassword(user.id);
}

type TryRecoverPassword = (user: User) => Promise<false | void>;

// Паттерн `try*` в имени явно говорит, что перед нами всё же команда,
// но она может в каком-то случае вернуть `false`.
```

Как минимум, так мы можем сделать ожидания от функции более явными. Но лучше, конечно, отделить эффект от принятия решения и отрефакторить функцию согласно CQS.

| Подобнее 💡                                                                              |
| :--------------------------------------------------------------------------------------- |
| Более детально о CQS и отделении логики от эффектов мы говорили в главе о сайд-эффектах. |

## Дизайн API

Во время рефакторинга статическая типизация может помочь проверить, насколько понятно спроектировано API модуля или функции. Например, по сигнатуре функции мы можем проверить очевидность API, «вычеркнув» _названия_ функций и аргументов:

```ts
function getPostContents(user: number, post: string): Promise<string> {}
// ->
function xxx(xxx: number, xxx: string): Promise<string> {}
```

Если сигнатура функции несёт мало смысла, мы можем улучшать её до тех пор, пока не начнём видеть в ней цель функции:

```ts
function xxx(xxx: number, xxx: string): Promise<string> {}
// ->
function xxx(xxx: UserId, xxx: PostSlug): Promise<string> {}
// ->
function xxx(xxx: UserId, xxx: PostSlug): Promise<PostContents> {}

// number -> UserId: первый аргумент — ID пользователя;
// string -> PostId: второй аргумент — URL публикации;
// string -> PostContents: результат — содержимое публикации.

// Из сигнатуры становится понятна механика работы функции:
// запрашиваем содержимое конкретного поста по составному ключу
// из ID пользователя и URL публикации.
```

Если смысл напрашивается сам собой, значит сигнатура функции стала «говорящей». Говорящие сигнатуры несут часть контекста задачи, поэтому мы можем передать читателю в названиях функции и аргументов дополнительные знания:

```ts
function fetchPost(authorId: UserId, post: PostSlug): Promise<PostContents> {}

// getPostContents -> fetchPost: узнаём, что данные будут запрошены по сети;
// userId -> authorId: узнаём, как именно пользователь связан с этими постами.
```

| К слову 📚                                                                                                          |
| :------------------------------------------------------------------------------------------------------------------ |
| Более подробно техника «вычёркивания» описана у Марка Симанна в книге “Code That Fits in Your Head”.[^codethatfits] |

Этим же правилом можно проверять, следует ли код CQS:

```ts
class PostReader {
  constructor(private postSource: PostStorage) {}

  getPost(id) {
    this.contents = this.postSource.fetchPost(id);
  }
}

// По сигнатуре метод `getPost` больше похож на команду, чем запрос:

type GetPost = (id: PostId) => void;

// Возможно, стоит переработать API или выбрать другое название метода:

class PostReader {
  // ...

  readPost(id: PostId): void {
    this.contents = this.postSource.fetchPost(id);
  }
}
```

| К слову ❌                                                                                                                                                        |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Техника «вычёркивания» более полезна для проектирования публичного API, чтобы оно было более информативным. Непубличным функциям это может быть чуть менее важно. |

[^dmmf]: “Domain Modeling Made Functional” by Scott Wlaschin, https://www.goodreads.com/book/show/34921689-domain-modeling-made-functional
[^ddd]: “Domain-Driven Design” by Eric Evans, https://www.goodreads.com/book/show/179133.Domain_Driven_Design
[^ubiquitouslanguage]: Ubiquitous Language, https://martinfowler.com/bliki/UbiquitousLanguage.html
[^typealias]: Type Aliases, TypeScript Handbook, https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
[^typebranding]: “Branding and Type-Tagging” by Kevin B. Greene, https://medium.com/@KevinBGreene/surviving-the-typescript-ecosystem-branding-and-type-tagging-6cf6e516523d
[^factorymethod]: Factory Method, Refactoring Guru, https://refactoring.guru/design-patterns/factory-method/typescript/example
[^codethatfits]: “Code That Fits in Your Head” by Mark Seemann, https://www.goodreads.com/book/show/57345272-code-that-fits-in-your-head
[^primitiveobsession]: Одержимость элементарными типами, Refactoring Guru, https://refactoring.guru/ru/smells/primitive-obsession
[^functionaltype]: More on Functions, TypeScript Documentation, https://www.typescriptlang.org/docs/handbook/2/functions.html