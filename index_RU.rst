.. Главный файл документации по архитектуре RChain, созданный
   sphinx-quickstart в Пт Дек 23 09:00:37 2016.
   Вы можете полностью адаптировать этот файл по своему вкусу, но он должен по крайней мере
   содержат корневую директиву `toctree`.

###############################################
Архитектура платформы RChain
###############################################

: Авторы: Эд Эйхолт, Люциус Мередит, Джозеф Денман
: Дата: 2017-07-22
: Организация: RChain Cooperative
: Авторское право: этот документ лицензируется в соответствии с лицензией Creative Commons Attribution 4.0 International (CC BY 4.0) License`_

.. _Creative Commons Attribution 4.0 International (CC BY 4.0) Лицензия: https://creativecommons.org/licenses/by/4.0/


Абстрактные
===============================================

Описание архитектуры платформы RChain обеспечивает высокоуровневую схему децентрализованной, экономически устойчивой публичной вычислительной инфраструктуры RChain. Хотя дизайн RChain вдохновлен дизайном предыдущих блоков, он также проводит десятилетия исследований в разных областях параллельного и распределенного вычислений, математики и программирования языка программирования. Платформа включает в себя модульную сквозную конструкцию, которая обязывает к разработке программного обеспечения и промышленной расширяемости.

**Предполагаемая аудитория:** Настоящий документ написан для разработчиков программного обеспечения и новаторов, заинтересованных в децентрализованных системах.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   introduction/motivation.rst
   introduction/introduction.rst
   introduction/comparison-of-blockchains.rst
   introduction/architecture-overview.rst
   contracts/contract-design.rst
   contracts/namespaces.rst
   execution_model/rhovm.rst
   execution_model/storage_and_query.rst
   execution_model/consensus_protocol.rst
   execution_model/applications.rst
   references.rst

