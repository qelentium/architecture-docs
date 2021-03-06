.. _rhovm:

************************************************** ****************
Модель исполнения
************************************************** ****************

обзор
================================================== ================

Каждый экземпляр **Rho Virtual Machine** (RhoVM) поддерживает среду, которая неоднократно применяет правило сокращения низкоуровневого rho-calculus, выраженное на языке подряда высокого уровня Rholang, к элементам постоянных данных о значении ключа хранилище данных  [#]_. «Состояние» RhoVM аналогично состоянию блочной цепи.


.. figure:: ../img/execution_storage.png
    :width: 965
    :scale: 50
    :align: center
    
    *Рисунок - RhoVM в качестве резервного хранилища ключей и выполнения движка*

Выполнение контракта влияет на * среду * и * состояние * экземпляра RhoVM. В этом случае использование термина «среда» не относится исключительно к среде исполнения, а к конфигурации структуры ключа-значения. Окружающая среда и состояние - это сопоставление имен с ячейками в памяти и местами в памяти для значений, соответственно. Переменные непосредственно указывают местоположения, что означает, что среда эквивалентно отображению имен в переменной. Программа обычно изменяет одну или обе эти ассоциации во время выполнения. Изменения окружающей среды происходят с правилами лексической оценки Rholang, и значения могут быть простыми или сложными, т.е. примитивными значениями или полными программами.



  :align: center
    :scale: 40
    :width: 1017
    
    *Рисунок - Двухэтапное связывание от имен до значений*

RhoVM работает против хранилища данных с ключом. **Изменение состояния RhoVM осуществляется операцией, которая модифицирует, какой ключ сопоставляется с каким значением.** Поскольку, как и Rholang, RhoVM получен из модели вычисления rho-calculus, эта операция является низкоуровневым rho-исчислением ,правилом сокращения. Эффективно правило сокращения, известное как правило «COMM», представляет собой подстановку, которая определяет вычисление :code:`P`, если будет выполняться новое значение на ключе. Ключ аналогичен имени в том, что он ссылается на местоположение в памяти, которое удерживает замещаемое значение. В следующем примере :code:`key` - это ключ и :code:`val` - замещаемое значение:



::


    for ( val <- key )P | key! ( @Q ) -> P { @Q/val }

Запрет на консенсус - это вычислительная модель параллельного протокола, в котором заключен договор о блокчейне. В каком-то потоке процесс вывода :code:`key!` Хранит код контракта :code:`@ Q` в месте, обозначенном :code:`key`. В другом потоке, выполняющемся одновременно, входной процесс :code:`for (val <- key) P` ждет нового значения: code:` val` появится в :code:`key`. Когда некоторые :code:`val` появляется в :code:`key`, в этом случае :code:`@ Q`, :code:`P` выполняется в среде, где :code:`@ Q` заменяется каждое событие :code:`val`. Эта операция изменяет значение, которое :code:`key`, т.е. :code:`key`, ранее сопоставленный с общим :code:`val`, но теперь он сопоставляется коду контракта :code:`@ Q`, который квалифицирует сокращение как переход состояния RhoVM.



.. figure:: ../img/io_binding.png
    :align: center
    :width: 1650
	:scale: 80
    * Рисунок - Уменьшение, влияющее на хранилище данных с ключом *

Синхронизация процесса ввода и вывода по адресу :code:`key` - это событие, которое запускает переход транзакции в RhoVM. На первый взгляд процесс вывода, в котором хранится контракт :code:`@ Q` на место, обозначаемое :code:`key`, представляется как сам переход состояния. Однако семантика сокращения rho-calculus имеет требование *наблюдаемости*. Для любых будущих вычислений :code:`P`, правило сокращения требует, чтобы процесс ввода :code:`for (val <- key) P` *наблюдает* присваивание по адресу :code:`key`. Это связано с тем, что только входной термин определяет будущие вычисления, а это означает, что только выходной терминал является вычислительно незначительным. Следовательно, не происходит *наблюдаемого* перехода состояния до тех пор, пока входные и выходные условия не синхронизируются по адресу :code:`key`. Это требование об обеспечении работоспособности применяется во время компиляции для предотвращения DDoS-атак повторным выходом :code:`key! (@ Q)` invocation.

Было продемонстрировано, что применение правило сокращения rho-calculus к элементу  хранилища данных с ключом представляет собой переход состояния экземпляра RhoVM. Цель, однако, состоит в том, чтобы проверять и поддерживать каждый переход состояния, который указан любым контрактом, который когда-либо выполнялся на экземпляре RhoVM. Это означает, что история конфигурации хранилища данных с ключом должна поддерживаться посредством модификации, поэтому она является * постоянной структурой данных. Поэтому каждый ключ должен сопоставляться с проверенной историей сокращений, которые должны произойти в этом месте:


.. figure:: ../img/transaction_history.png
    :align: center
    :width: 2175
    :scale: 30
    
    *Рисунок - Уменьшение / история транзакций местоположения в памяти*

Каждый ключ сопоставляет список сокращений, который является, по сути, «историей транзакций» адреса. История транзакций: code: `{for (val1 <- keyn) .P1 | keyn! (@ Q1), ... , for(valn <- keyn).Pn | keyn!(@Qn) } -> { P1{@Q1/val1}, ... , Pn{@Qn/valn} }` ` обозначает изменения, которые были внесены в контракт :code:` @ Q`, где :code:`@ Qn` - самая последняя версия в магазине. Важно признать, что эта схема является транзакцией верхнего уровня на платформе RChain. Передаваемые сообщения - это сами контракты, которые чаще всего встречаются в клиентской системе или взаимодействиях системной системы. Однако каждый контракт :code:`@ Q` может сам выполнять многие транзакции более низкого уровня по более простым значениям.

После применения транзакции / сокращения он подвергается консенсусу. Консенсус подтверждает, что история транзакций :code:`{ for(val1 <- keyn).P1 | keyn!(@Q1) … for(valn <- keyn).Pn | keyn!(@Qn) }` из :code:`keyn`, последовательно реплицируется во всех узлах, запускающих этот экземпляр RhoVM. После проверки истории транзакций последняя транзакция добавляется в историю транзакций. Один и тот же консенсусный протокол применяется к диапазону ключей :code:`{ key1 -> val1 … keyn -> valn }` поскольку транзакции передаются в эти местоположения.

По объему транзакционные блоки представляют собой группы сокращений, которые были применены к элементам хранилища постоянных ключей, а истории транзакций представляют собой проверенные снимки конфигураций состояний и переходов экземпляра виртуальной машины Rho. Обратите внимание, что консенсусный алгоритм применяется, если и только если операторы узла предлагают сделать конфликт с меньшей историей сокращения.

Обобщить:

1. RhoVM - это композиция семантики редукции rho-calculus, выраженная в Rholang, и постоянное хранилище данных с ключом.
2. Правило сокращения rho-calculus заменяет значение на ключ для другого значения, где именованный канал соответствует ключу, а значения могут быть простыми или сложными.
3. Замены - это транзакции, которые проявляются в виде различий в байтекоде, хранящемся у ключа. Точная репликация этих различий между байт-кодом во всех узлах, работающих с этим экземпляром RhoVM, проверяется с помощью согласованного алгоритма.

.. [#]. «Окружающая среда» RhoVM будет позже представлена как «Rosette VM». Выбор использования Rosette VM зависел от двух факторов. Во-первых, система Розетта находится в коммерческом производстве более 20 лет. Во-вторых, модель памяти Rosette VM, модель вычислений и системы времени исполнения обеспечивают поддержку параллелизма, который требует RhoVM. RChain пообещал провести модернизированную повторную реализацию Rosette VM в Scala, чтобы служить исходной средой исполнения RhoVM.

Краткая информация о масштабности
-------------------------------------------------- -----------------

С точки зрения традиционной программной платформы понятие «параллельных» экземпляров VM избыточно. Предполагается, что экземпляры виртуальной машины работают независимо друг от друга. Соответственно, нет «глобального» RhoVM. Вместо этого в любой момент времени существует мультиплекс независимо работающих экземпляров RhoVM, работающих на узлах по сети, каждый из которых выполняет и проверяет транзакции для связанных с ними осколкам или, как мы уже говорили, их пространства имен.

Этот выбор дизайна представляет собой системный уровень параллелизма на платформе RChain, где параллельность на уровне команд дается Rholang. Следовательно, когда эта публикация относится к одному экземпляру RhoVM, предполагается, что существует мультиплекс экземпляров RhoVM, одновременно выполняющий другой набор контрактов для другого пространства имен.

Условия выполнения
================================================

Что такое Розетта?
------------------------------------------------

Розеттаа - это отражающий, объектно-ориентированный язык, который обеспечивает параллелизм через семантику оператора. Система Rosette (включая виртуальную машину Rosette) находится в коммерческом производстве с 1994 года в автоматизированных кассовых машинах. Благодаря доказанной надежности Rosette, RChain Cooperative взяла на себя обязательство завершить переориентацию чистой  Rosette VM в Scala (ориентированную на JVM). Это два основных преимущества. Во-первых, язык Розетты удовлетворяет семантике параллелизма на уровне инструкций, выраженной в Rholang. Во-вторых, Rosette VM была намеренно разработана для поддержки многокомпьютерной (распределенной) системы, работающей на произвольном количестве процессоров. Для получения дополнительной информации см. `Mobile Process Calculi for Programming the Blockchain`_

.. _Мобильные вычисления процесса для программирования блочной цепи: http://mobile-process-calculi-for-programming-the-new-blockchain.readthedocs.io/en/latest/

Проверка модели и утверждение теоремы
-------------------------------------------------- -

В RhoVM и, возможно, на догоняющих языках вверх, существует множество методов и проверок, которые будут применяться во время компиляции и  выполнения. Они помогают удовлетворить требования, например, как разработчик и сама система могут знать априори, что контракты, которые хорошо типизированы, прекратятся. Формальная проверка будет гарантировать сквозную правильность с помощью проверки модели (например, в SLMC) и доказательства теоремы (например, в Pro Verif). Кроме того, эти же проверки могут применяться во время выполнения, когда оцениваются новые предлагаемые сборки контрактов.

Служба обнаружения
-------------------------------------------------- -

Расширенная функция обнаружения, которая в конечном итоге будет реализована, позволяет искать совместимые контракты и собирать новый составной контракт из других контрактов. При формальных методах проверки автору нового контракта можно гарантировать, что при подключении рабочих контрактов они будут работать, а также один контракт.

Компиляция
================================================

Чтобы клиенты могли выполнять контракты на RhoVM, RChain разработал конвейер компилятора, который начинается с исходного кода Rholang. Первоначальный исходный код Rholang подвергается транскопиляции в исходный код Rosette. После анализа исходный код Rosette скомпилирован в промежуточное представление Rosette (IRs), которое подвергается оптимизации. Из Rosette IR байт-код Rosette синтезируется и передается виртуальной машине для локального выполнения. Каждый шаг перевода в конвейере компиляции является либо оправданным, либо коммерчески протестированным в производственных системах, либо и тем, и другим. Этот трубопровод показан на рисунке ниже:


.. figure:: ../img/compilation_strategy.png
    :width: 1109
    :align: center
    :scale: 40
    
    
    *Рисунок - стратегия компиляции RChain*

1. **Анализ**: Исходный код Rholang или другой язык умного-контракта, который компилируется в Rholang, включает в себя:

    а) анализ вычислительной сложности
    b) ввод кода для механизма ограничения скорости
    в) формальная проверка семантики транзакций
    d) десурагирование синтаксиса
    e) упрощение функциональных эквивалентов
2. **Транскомпиляция**: Из исходного кода Rholang компилятор:

    а) выполняет перевод источника с источника из исходного кода Rholang в Rosette.

3. **Анализ**: из исходного кода Rosette компилятор выполняет:
  
    а) лексический, синтаксический и семантический анализ синтаксиса Розетты, построение АСТ; а также
    б) синтезирует промежуточное представление Розетты

4. **Оптимизация**: из Rosette IR компилятор:

    а) оптимизирует IR путем устранения избыточности, устранения подвыражения, устранения мертвого кода, постоянной сгибания, идентификации индукционной переменной и упрощения прочности
    б) синтезирует байт-код, который должен выполняться виртуальной машиной Rosette
    
Механизм ограничения скорости
-------------------------------------------------- -

В конвейере компиляции будет реализован механизм ограничения скорости, связанный с некоторым вычислением ресурсов обработки, памяти, хранения и полосы пропускания. Поскольку правило восстановления rho-calculus является атомной единицей вычисления на платформе RChain, расчет сложности вычислений обязательно коррелирует с количеством сокращений, выполняемых на контракт. Этот механизм необходим для восстановления затрат на оборудование и связанные с ним операции. Хотя Ethereum (Gas) имеет схожие потребности, механизмы разные. В частности, измерение не будет выполняться на уровне VM, но будет введено в код контракта на этапе анализа компиляции.
    
Подробнее см. Здесь `join`_ канал` # rhovm`_ на RChain Slack. Работу компилятора можно увидеть на `GitHub`_.

.. _GitHub: https://github.com/rchain/Rholang/tree/master/src/main/scala/rholang/rosette
.. _ # rhovm: https://ourchain.slack.com/messages/rhovm/
.. _join: http://slack.rchain.coop/


