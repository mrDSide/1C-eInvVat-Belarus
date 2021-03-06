<p><big>C 1-го июля 2016 года для субъектов хозяйствования РБ вводится обязательный обмен электронными счетами-фактурами по НДС. Данная подсистема предназначена для автоматизации всех операций с документами из учётной системы. Возможно выставление всех видов документов, получение и учёт входящих документов, получение и учёт квитанций. Механизм хранения документов с портала предполагает возможность хранения файлов *.xml в базе, и на диске в хранилище.</big></p>

<p><big>С помощью данной подсистемы внедрён учёт входящих документов поставщиков и исходящих документов предприятий. В состав подсистемы включены документы формирования данных за произвольный период данными типовых операций на основании бухгалтерских проводок, а так же единичных произвольных документов (реализация без получателя и прочее).</big></p>

<p>В каталоге <a href="/pics">pics</a> расположены снимки экрана работы с конфигурацией</p>

<p><big>Перед начало работы следует произвести настройки. Для этого можно воспользоваться общей формой "ФормаНастроекЭСЧФ" или объединить форму "НастройкаПрограммы" с формой из вашей конфигурации (это добавить страницу "Электронный НДС" в форму настроек).</big></p>

<p><big>Блок "Криптография" не используется, т.к. все операции по подписанию и обмену производятся с помощью внешней компоненты. Обязательным является только указание каталога хранения схем ("Путь каталога xsd"). Но для оптимальной работы на данный момент рекомендую использовать следующие настройки: сохранение документов - всегда, расположение входящих файлов - в каталоге на диске, режим работы по-умолчанию - отложенный (если планируется выполнять обмен с помощью задания планировщика) или на клиенте (т.к. компонента не корректно работает в 64-битном серверном процессе 1с). <strong>Если выбрано расположение входящих файлов на диске, то следует указать каталог! </strong>Начиная с версии 1.2.15.48 компонента поддерживает работу с прокси сервером, параметры можно так же задать в форме настройки. Так же можно вручную задать параметры обмена (на данный момент поддерживается только параметр "connection.readTimeout").</big></p>

<p><big>Автоматический обмен данными с порталом реализован с помощью планировщика Windows, который запускает экземпляр клиентского приложения 1с с командной строкой следующего вида:</big></p>

<pre>
<code class="language-bash">ENTERPRISE /S1c-server.contoso.com\MyBase /RunModeOrdinaryApplication /DisableStartupMessages /Nuser /Puser /ExecuteC:\eInvVat\task\Exchange.epf</code></pre>

<p><big>Предварительно нужно настроить компьютер для работы по инструкциям <a href="http://vat.gov.by/mainPage/manual/" target="_blank">http://vat.gov.by/mainPage/manual/</a> или скачать и запустить сценарий автоматической настройки, приложенный к публикации (не забудьте выложить в нужный каталог файлы сертификатов для импорта).</big></p>

<p><big><strong>ВАЖНО</strong> все действия требуется выполнять от имени пользователя (учётной записи Windows), с которой будет запускаться задание планировщика.</big></p>

<p><big>Для простоты использования рекомендую использовать короткие пути, например "C:\eInvVat\task" для файлов обработок задания, "C:\eInvVat\task\crl" - для задания обновления СОС (списков отзыва сертификатов). Сценарий обновленя списка отзыва сертификатов так же приложен к публикации.</big></p>

<p><big>После интеграции подсистему в конфигурацию (или запуска на демонстрационной базе), если всё было выполнено верно, в базу начнут загружаться документы поставщиков и статусы по ним в автоматическом режиме. Если нет желания делать автоматический обмен т.о. или просто хотите посмотреть как это работает, можно воспользоватья одной из обработок подсистемы: "Обмен с порталом ИМНС" (не использует регистры автоматического обмена) или "Автоматический обмен (ЭСЧФ)" (её можно запускать вместо задания планировщика, а так же настроить параметры формирования очереди организаций на обмен). См. изображения №№35-38.</big></p>

<p><big>Для работы с порталом предусмотрено три режима: На клиенте (операции выполняются на стороне клиента), На сервере (вызов компоненты на стороне сервера 1с), Отложенный (данные только отражаются по регистрам автоматического обмена, далее нужно вызвать процедуру или выполнить обмен из обработки). На данный момент лучше всего себя зарекомендовала следующая схема работы:</big></p>

<div><big>1. </big><big>Пользователи отражают данные в документах "Пакет исходящих ЭСЧФ" и "Электронный счет-фактура выданны"</big></div>

<div><big>2. Документы из внешних учётных систем загружаются в документ "Электронный счет-фактура выданный"</big></div>

<div><big>3. Пользователи сверяют входящие документы поставщиков внутренними механизмами конфигурации</big></div>

<div><big>4. Планировщик заданий выполняет обмен данными с порталом по расписанию. Это включает в себя: загрузку документов, подписание нужных документов поставщиков, выгрузку исходящих документов, загрузку статусов по всем документам</big></div>

<div><big>5. Отдельно по расписанию запускается задание на архивирование данных</big></div>

<div>
<pre>
<code>ENTERPRISE /S1c-server.contoso.com\MyBase /RunModeOrdinaryApplication /DisableStartupMessages /Nuser /Puser /ExecuteC:\eInvVat\task\Archive.epf</code></pre>

<p><big>Для просмотра информации о состоянии обмена предусмотрен отчёт "СостояниеАвтоОбменаДаннымиСИМНС", а для анализа данных поставщиков (только 18 счёт) - </big>"<big>ДанныеПоКонтрагентамЭСЧФ" (можно посмотреть по каким поставщикам были входящие документы и операции по счёту 18).</big></p>

<p><big>Подсистема является автономном, в том смысле, что её наличия достаточно для полноценной работы с порталом в части обмена документами (получение, отправка, подтверждение).</big></p>

<p><big>Подсистема успешно работает на конфигурации БП 1.6, все запросы написаны под неё и для внедрения в другие конфигурации требуется изменять тексты в общих модулях подсистемы. <strong>Так же следует учитывать, что разработка ведётся под давно уже не типовую конфигурацию и изменения вносить придётся в любом случае.</strong> <strong>Подсистема полностью функциональна, т.е. можно получать/отправлять документы. Для интеграции в любую типовую конфигурацию требуется доработка общих модулей в части получения данных из базы.</strong></big></p>

<p><big>Что не реализовано:</big></p>

<p><big>- не все проверки возможно реализовать из-за ошибки в официальной документации (не критично)</big></p>

<p><big>- нет возможность автоматического формирования дополнительного ЭСЧФ (только исправленный), можно только вручную</big></p>

<p><big>Код разработки полностью открыт (не считая самой компоненты ИМНС). Данная разработка распространяется, основываясь на следующих основных принципах, что любой пользователь продукта имеет право: (0) выполнять программу, (1) изучать и править программу в виде исходного текста, (2) перераспространять точные копии и (3) распространять измененные версии. Если программа модифицируется, то и модифицированная программа должна сохранять все выше перечисленные свободы (чтобы все последующие измененные и дополненные версии программы тоже оставались свободными).</big></p>
</div>

<p></p>
