# Получение оптимизационного файла, минимизирующего структуру подгружаемых каскадных таблиц в HTML
(с) 2020 Лебедев Евгений

> практика по программированию

# Постановка задачи
Каскадные таблицы стилей или CSS — это язык, который используется для определения визуального представления веб-сайта на основе содержимого, содержащегося в документе языка разметки. Файлы стилей с расширением .css повсеместно применяются при создании различных веб-страниц. Они используются для задания шаблонов отображения текстовых и графических блоков той или иной веб-страницы. В процессе того, как мы заходим на веб-страницу, если браузер обнаруживает ссылки на таблицы стилей, он немедленно отправляет запрос на сервер и загружает файлы. Соответственно скорость загрузки напрямую зависит от оптимизации CSS кода. В данной работе главной целью как раз и является минимизация структуры подгружаемых каскадных таблиц стилей CSS в HTML.
# План создания программы

![План создания программы](plan.png)

## 1 блок

> Файл с разрешением .html проверяется на существование. Если такой файл существует, он загружается для обработки; в случае несуществования файла, выдается соответствующее сообщение об ошибке.

## 2 блок

> По существующему html файлу, формируются списки тегов, из которых выделяется информация, включающая: наименование тега; классы описывающие тег (может быть несколько); идентификатор, описывающий тег (единственный).

## 3 блок

> Из файла каскадных таблиц выделяются все строки, относящиеся к именам классов и идентификаторов из списка используемых в html документе. Открывается css файл, строится словарь по типу: ключ - имя класса (идентификатора); значение - содержимое, между началом и концом описания класса (идентификатора).

## 4 блок

> Создается минимизированный файл разметки min_html(имя).html и минимизированные файлы каскадных таблиц min_css(имя).css. Файл html модифицируется (меняются имена подключаемых каскадных таблиц на имена, сформированные в 3 блоке), добавлением символа подчеркивания ('_').

## 5 блок

> Формируется статистика о проделанной работе. Пользователю выводятся результаты работы (количество оптимизированных каскадных таблиц, статистические данные относительно файлов html и css).

# Функциональная структуры программы

## 1 блок
>проверка существования файла

![Блок-схема 1](img001.jpg)

## 2 блок
>составление словаря, являющегося основанием для выборки css правил

![Блок-схема 1](img001.jpg)

## 3 блок
>выбор необходимых css правил

![Блок-схема 3](img003.png)

## 4 блок
>создание минимизированных css и html файлов

![Блок-схема 4](img004.png)


# Описание функций программы htmltags.py:

> file_existance:

Функция проверки существования файла. Возвращает путь к файлу, если искомый файл существует, а также сообщение об ошибке в случае несуществования файла (блок схема 1).

> html_tags:

Функция, возвращающая список HTML тегов по документу .html. Примерный алгоритм работы: считывает каждую строчку документа .html, определяет наличие html тега в текущей строке; в случае, если тег в строке найден и он уникален в пределах документа, добавляет его в список.

> css_styles:

Функция, возвращающая список подгружаемых локальных css документов к html документу. Примерный алгоритм работы: считывает каждую строку документа .html, определяет наличие тега link в строке; в случае, если путь к связываемому файлу не содержит подстроку "https", добавляет путь в список.

> class_list:

Функция, возвращающая список всех используемых классов в html документе. Примерный алгоритм работы: считывает каждую строку документа .html, при обнаружении подстроки 'class="' выделяет название класса из текущей строки (добавляет название класса в список).

> id_list:

Функция, возвращающая список всех используемых идентификаторов в html документе. Примерный алгоритм работы: считывает каждую строку документа .html, при обнаружении подстроки 'id="' добавляет название идентификатора в список.

> css_work:

Функция, возвращающая словарь по типу: ключ - название селектора; значение - блок описания данного селектора. Возвращает лишь те селекторы, которые используются в html документе. Примерный алгоритм работы представлен на схеме алгоритма к 3 блоку.

Регулярные выражения, используемые для проверки селектора:
- на вхождение хотя бы одного используемого тега
reg_exp = `'([^\w\.]|^)' + cur_tag + r'($|:{1,2}.+$| *\,.+$| ?\..+$|\[.+$| *\~.+$| *>.+$| *\+.+$| +.+$)'`
- на вхождение хотя бы одного используемого класса
reg_exp = `'\.' + cur_class + r'($|:{1,2}.+$| *\,.+$| *\..+$|\[.+$| *\~.+$| *>.+$| *\+.+$)'`
- на вхождение хотя бы одного используемого идентификатора
reg_exp = `'\#' + cur_id + r'($|:{1,2}.+$| *\,.+$|\[.+$| *\~.+$| *>.+$| *\+.+$| +.+$)'`
#### Описание алгоритма разбора выражения проверки селектора:
Алгоритм не допускает вхождение буквы или точки, если тег находится не в начале селектора; после тега/класса/идентификатора возможна одна из следующих комбинаций: конец строки; ':' или '::' (описание псевдокласса); запятая для перечисления следующих тегов/классов; точка для описания вложенности элементов (недоступная для идентификаторов); \[attribute] для описания селектора атрибута; знак ~ для описания одноуровневых элементов; знак '>' для описания дочерних элементов; знак + для описания смежных селекторов. Комбинация завершается концом строки. В случае вхождения хотя бы одного используемого тега/класса/идентификатора, проверка считается успешной и возвращается True, иначе - False.
#### Тестирование проверки селекторов:
|     Используемые   теги/классы/id в HTML    |                             Селектор                            |     Ожидаемый   результат    |     Полученный   результат    |
|:-------------------------------------------:|:---------------------------------------------------------------:|:----------------------------:|:-----------------------------:|
|                      h1                     |                                h1                               |              True            |              True             |
|                  h1, h2, h6                 |                      h1, h2, h3, h4, h5, h6                     |              True            |              True             |
|                     body                    |                                div                              |             False            |              False            |
|                       a                     |                              a:hover                            |              True            |              True             |
|                    class1                   |                              .class1                            |              True            |              True             |
|                class1, class2               |                           class2.class1                         |             False            |              False            |
|                class1, class2               |                          .class2.class1                         |              True            |              True             |
|                      id1                    |                               #id1                              |              True            |              True             |
|                       a                     |                               li a                              |              True            |              True             |
|                    class2                   |                          class2::pseudo2                        |              True            |              True             |
|                     x, y                    |                               x + y                             |              True            |              True             |
|                     x, y                    |                               x > y                             |              True            |              True             |
|               custom-file-label             |     :lang(en) .custom-file-input ~ .custom-file-label::after    |              True            |              True             |
|                   .nav-link                 |        .navbar-nav.mainUser > li.nav-item .nav-link:hover       |              True            |              True             |
|                   WellItem                  |     main .WndRegist .WndPanel   .WellItem[data-isactive="1"]    |              True            |              True             |

> new_css:

Функция, создающая оптимизированные css файлы. Примерный алгоритм работы представлен на схеме алгоритма к 4 блоку.

> new_html:

Функция, создающая оптимизированный html файл. Изменяет ссылки на стили, добавлением символа "\_" в начало стиля. Примерный алгоритм работы представлен на схеме алгоритма к 4 блоку.
