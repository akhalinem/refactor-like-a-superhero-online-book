# Рефакторинг как процесс

В прошлых главах мы уделяли основное внимание техническим деталям рефакторинга, но это только часть процесса улучшения кода в проекте.

В этой главе поговорим, как находить на рефакторинг время и заниматься им регулярно. Обсудим, как преодолевать трение в проекте и приступать к рефакторингу, даже если кодовая база большая и в ней много легаси.

## Рефакторить или переписывать

Если мы работаем с легаси-кодом, то первая мысль при взгляде на него — «да проще с нуля переписать». Иногда это действительно так, но адекватно оценить ситуацию с первого взгляда можно не всегда. Перед тем как решить, рефакторить или переписывать, нам стоит оценить 3 вещи: ресурсы команды, выгоды и риски нашего решения.

Эти параметры не дадут прямого ответа на вопрос «Что делать?», но дадут более объективную оценку состояния проекта. Иногда одной такой оценки бывает достаточно, чтобы принять решение. Но даже если её не достаточно, она подскажет, каких знаний о проекте нам не хватает для принятия окончательного решения.

## Ресурсы

Под _ресурсами_ мы будем понимать время, знания и опыт, которые есть в распоряжении команды. Это то, что можно «тратить» на рефакторинг или переписывание кода, чтобы улучшить его.

### Время

Оценивать свободное время разработчиков можно ретроспективно, смотря на прошлый опыт работы над проектом. Нам потребуется посчитать сколько времени команда уделяла улучшениям в прошлом, как часто появлялись свободные «окна» в расписании проекта.

Прошлый опыт покажет, как расходовалось время на разработку в прошлом и какие паттерны прослеживались во время неё. Это поможет спрогнозировать, сколько свободного времени у нас будет в ближайшем будущем.

Такой прогноз может показаться заниженным, потому что мы можем хотеть уделять улучшениям кода больше времени, но он отражает привычные паттерны разработки. Их полезно знать, потому что даже если мы договорились уделять больше времени рефакторингу и тех. долгу, без перемен в рабочих процессах разработка вернётся к привычным паттернам.

К тому же нам всегда стоит помнить о «непредвиденных проблемах», которые точно появятся при работе с легаси и будут отнимать на решение дополнительное время.

### Накопленные знания

Накопленные знания о проекте можно оценить по количеству документации, полезных комментариев в коде, качеству истории коммитов, доступности участников, которые писали код, и выразительности самого кода.

Знания сложно оценить количественно, поэтому можно использовать качественную оценку. Чем больше противоречий между разными источниками информации мы находим, тем хуже можем считать качество накопленных знаний. И наоборот, чем стройнее выглядит модель проекта и проще найти людей, которые занимались им с самого начала, тем качество знаний выше.

### Опыт

Под опытом будем иметь в виду знакомство разработчиков с текущим и _различными другими_ проектами, языками и парадигмами. Чем разностороннее опыт, чем больше разработчики «повидали», тем меньше времени мы будем тратить на разработку нерабочих решений.

| К слову 🧑‍🏫                                                                                                                                     |
| :--------------------------------------------------------------------------------------------------------------------------------------------- |
| Стоит также учитывать и опыт сторонних консультантов и экспертов, если мы допускаем возможность их участия, но это сложнее сделать объективно. |

## Выгоды и риски

Также нам стоит определить выгоды и риски рефакторинга и переписывания кода. Они будут напрямую зависеть от рабочих процессов и «кухни» проекта, поэтому понадобится какое-то внутреннее исследование. Удобнее всего оценивать выгоды и риски параллельно, потому что стремление к любой выгоде имеет под собой какой-то риск.

## Мета-информация о проекте

«Мета-информация» рассказывает о процессе работы над проектом и том, какие части кода в нём наиболее важные. Если команда использует систему контроля версий, то вся необходимая «мета-информация» уже есть там. Например, история коммитов может нам рассказать:

- что является ключевой частью проекта — в каких файлах дописывают фичи и правят баги чаще всего;
- какое неявное зацепление есть между частями проекта — какие файлы менялись вместе с другими файлами;
- в каких частях проекта было больше всего багов — какие коммиты относились к баг-фиксам и т.д.

Анализ мета-информации о проекте может рассказать, какие части кода самые сложные или самые полезные с точки зрения бизнеса.

| Подробнее 📚                                                                            |
| :-------------------------------------------------------------------------------------- |
| Хорошо об этом написал Адам Торнхил в своей книге “Your Code as a Crime Scene”.[^scene] |

Если системы контроля версий нет, то можно попробовать собрать косвенные показатели (релиз-ноуты, базу данных поддержки и т.д.), но делать выводы по ним будет сложнее.

## Эстимейты

Если мы всё же решили отрефакторить конкретный кусок кода, то следующий шаг — спланировать итерацию.

Чтобы понять, какое количество времени может уйти на рефакторинг куска кода, сперва стоит выделить в нём места, которые вызывают вопросы. Чтобы определиться, с какими проблемами мы имеем дело, мы можем разложить проблемные места в коде по такой таблице:

|           | Просто                                                          | Сложно                                                       |
| --------- | --------------------------------------------------------------- | ------------------------------------------------------------ |
| Понятно   | Ресёрч не нужен, ясно как решать, не займёт много времени       | Ясно как решать, но точно замёт много времени                |
| Непонятно | Задача маленькая и изолированная, но может потребоваться ресёрч | Точно нужен ресёрч, много скрытых связей, незнакомая область |

Количество требуемого времени будет напрямую зависеть от пропорции кода в этой таблице. Чем больше сложного и непонятного, тем сложнее адекватно спланировать итерацию.

Задачи из последней ячейки спланировать сложнее всего. Для таких задач можно предложить команде использовать метод «Итерация-гипотеза». В этом методе каждое предположение о проблеме будет отдельной итерацией разработки. Опровержение или подтверждение этого предположения — цель итерации.

Проверка одной гипотезы занимает не так много времени, как исследование всей проблемы целиком. Итерации с такими проверками проще планировать и проводить. При этом с каждой проверенной гипотезой мы понимаем о проблеме больше и рано или поздно она перейдёт из правой нижней ячейки в какую-то другую — тогда мы сможем точнее оценить масштаб предстоящей работы и требуемое время.

| К слову 👀                                                                                                                                                                                                                                               |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| План итерации и оценка масштаба задачи могут быть полезны, даже если команда следует философии “No Estimates.”[^noestimates] Например, с ними легче отследить, когда пора менять стратегию работы, не успев при этом потратить лишние ресурсы на задачу. |

## Рефакторинг больших кусков кода

Если кусок кода не отрефакторить разом, то полезно следовать методологии «Рядом, а не вместо».

Суть подхода в том, чтобы _не заменять_ кусок кода под рефакторингом, а создавать аналогичную функциональность _рядом_ с этим кодом. Когда аналог достаточно проработан, чтобы заменить собой проблемную часть, — мы заменяем старый код на свежесозданный.

| К слову 🌳                                                                                                                                                                                                                                                                                                |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Мне нравится аналогия этого подхода с фикусом-душителем.[^strangler] Фикус обвевает дерево, твердеет, а когда дерево умирает, фикус остаётся в форме этого дерева. Рефакторинг «рядом» — это такой вот фикус: мы сперва «оплетаем снаружи» фичу новым кодом, и когда он готов, убираем старый код внутри. |

При работе по подходу «Рядом, а не вместо» важно часто сливать результаты в главную ветку проекта. Это поможет предотвратить конфликты в системе контроля версий и не даст процессу рефакторинга затянуться на долгое время.

## Частота и гигиена

Рефакторить лучше как можно чаще. Удобнее всего рефакторить код сразу после внесения в него изменений. Идеально, если мы можем вернуться к этому коду ещё раз после отдыха, чтобы глянуть на него свежей головой. Такое ревью помогает раньше выявить слабые решения.

При работе с легаси стоит рефакторить код _до_ того, как фиксить в нём баги или добавлять новые фичи. После рефакторинга код будет более понятен, покрыт тестами и в целом приятнее для работы. Так мы уменьшим необходимое время на починку бага или новую фичу.

Также при разработке стоит помнить о правиле бойскаута:[^opportunistic][^codethatfits]

---

**❗️ Оставлять после себя код чище, чем он был до**

---

## Метрики

Хоть рефакторинг и опирается на субъективные показатели типа красоты и читаемости, мы всё же можем использовать метрики для оценки качества внесённых изменений.

За основу мы возьмём метрики из доклада “Where does bad code come from?” и немного расширим список.[^wherefrom] У нас получится список из 7 метрик, каждая из которых отвечает на вопрос «Сколько времени нужно, чтобы сделать XXX?»:

- **S**earch — сколько времени нужно, чтобы найти требуемое место в коде.
- **W**rite — чтобы написать новую фичу и покрыть её тестами.
- **A**gree — чтобы разработчики согласились, что «код хороший».
- **R**ead — чтобы прочесть и понять, что кусок кода делает.
- **M**odify — чтобы изменить код под новые требования.
- **E**xecute — чтобы выполнить/собрать/задеплоить кусок кода.
- **D**ebug — чтобы найти и отладить баг.

Если мы проведём замеры эти метрик до рефакторинга и после, то сможем сравнить качество внесённых изменений. Понятно, что некоторые метрики всё ещё довольно субъективны, но их тем не менее можно выразить в цифрах. Количественное описание характеристик поможет нам заметить тренд на улучшение или ухудшение при оценке качества кода.

[^scene]: “Your Code As a Crime Scene” by Adam Tornhill, https://www.goodreads.com/book/show/23627482-your-code-as-a-crime-scene
[^strangler]: “Strangler Fig Application” by Martin Fowler https://martinfowler.com/bliki/StranglerFigApplication.html
[^opportunistic]: “Opportunistic Refactoring” by Martin Fowler https://martinfowler.com/bliki/OpportunisticRefactoring.html
[^codethatfits]: “Code That Fits in Your Head” by Mark Seemann, https://www.goodreads.com/book/show/57345272-code-that-fits-in-your-head
[^wherefrom]: “Where Does Bad Code Come From?” https://youtu.be/7YpFGkG-u1w
[^noestimates]: “No Estimates” by Allen Holub, https://youtu.be/QVBlnCTu9Ms
