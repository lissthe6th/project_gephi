# project_gephi
Учимся составлять базы данных для gephi при помощи python и анализируем результат.

## Giphi — средство визуализации графиков и сетей.

Основным инструментом для своего исследования я выбрала программу Gephi.

Разработчики заявляют, что программа подходит для

- исследовательского анализа данных
- анализа связей
- анализа социальных сетей
- анализа биологических сетей
- создание высококачественных печатных карт-плакатов.

Чтобы быстро изучить особенности программы я окунулась в удивительный мир видео-объяснений. Однако они в основном были посвящены тому, как обрабатывать данные непосредственно в самой программе. А вот с тем, как добывать данные, нам предстоит разобраться.

Для работы программа просит импортировать два файла в формате .csv. В них данные хранятся в формате строк, записанных через запятую.

1. Данные об узлах: id, label
2. Данные о ребрах: source(откуда), target (куда), type (directed / undirected), weight (вес ребра)

Разобравшись в том, что необходимо программе для работы, я решил создать базу данных, хоть немного отвечающую моим исследовательским задачам, чтобы освоить этот инструмент.

## Исследование

Изначально я использовала англоязычный [топ-100 нон-фикшн книг](https://www.theguardian.com/books/2017/dec/31/the-100-best-nonfiction-books-of-all-time-the-full-list), в формате первичного эксперимента. Передо мной стояла цель написать костяк кода, который будет в состоянии обработать входные данные и создать на выходе два файла в необходимом нам формате. Посмотреть код можно [здесь](https://colab.research.google.com/drive/1JjrA72Fj31dJL_SIm0srCdfWS0dx7QQM?usp=sharing), а файл для проверки его работы лежит в репозитории под названием topcentury.txt.

Результат своей работы я покажу на другом материале.

### Материал для работы.

Это будет уже русскоязычный список нон-фикшн произведений. В отличии от первого эксперимента я обработаю материал отдельно от основного кода.

Я взяла список список популярных нон-фикшн книг на русском языке с [этого](https://www.livelib.ru/selection/21026-nonfikshn/listview/smalllist/~8) сайта. Выбрала вариант отображения, при котором название книги выводится полностью, а затем через кнопку показать еще вывела больше 40 предложений книг. Просто скопировала текст вместе со всем мусором, который также был на странице.

Полученный текст необходимо, конечно, почистить. До знакомства с питоном мне много раз приходилось чистить подобные списки руками. Ну, или с использованием найти-заменить в ворде. Но это очень долго и муторно.

Как говорят в тиктоке: 

> если стоит выбор между тем, чтобы сделать работу за час или автоматизировать её 6 часов — выбор очевиден, докажи миру, что ты программист
> 

Выделив основной паттерн в тексте, я убрала всю лишнюю информацию из документа при помощи кода ниже.

```python
ifile = open(topcent.txt, encoding=utf-8, mode=r)
proba = open(proba.txt, encoding=utf-8, mode=w)
array_books = []
books = []
names = []
all_words = []
dictionary = {}
num = -1
for line in ifile.read().lower().split(добавить)
    if line
        line = str(line).replace(общая оценка, ).replace( №, ).replace( (, ).split('')
        proba.write(str(line[1]) + '/n/n')
        print(line)
proba.close()
```

В результате был записан новый документ, где из лишнего для наших целей остались только имена авторов и принадлежность к определенному жанру.

Оставшиеся данные я подчистила руками, поскольку информация об авторах представлена в различных форматах и быстро найти способ автоматизировать этот процесс я не нашла. В результате перед нами остались только названия книг.

### Преобразование в базу данных

Для преобразования данных у нас есть отдельный код. Полностью он лежит [здесь](https://colab.research.google.com/drive/1fltR3mUu5cKwynu9fRpar6QcyFPLithu?usp=sharing). Файлы для него добавлены в репозиторий.

Задачи: нормализовать текст, разбить названия на слова, составить схему отношений между словами в каждом названии, а также выделить закономерность употребления отдельных словосочетаний во всем списке, который мы рассматриваем.

Импортируем морфологический анализатор русского языка от Яндекса, а затем прогоняем через него наш текст. Дальше будем работать уже с обработанным содержанием.

```python
from pymystem3 import Mystem

def lemmatize_text(text):
    m = Mystem()
    lemmas = m.lemmatize(text)
    lemmatized_text = ''.join(lemmas)
    return lemmatized_text

s = lemmatize_text(ifile.read())
```

Я вручную прописала некоторые знаки препинания, чтобы их убрать, а затем поделила материал на отдельные названия, а их — на слова. Дальше мы будем работать с ними.

```python
for line in s.lower().replace('.', '').replace(',', '').replace(':', '').replace('?', '').split("\n"):
    if line:
        books.append(line)

for i in books:
    temp = str(i).split(' ')
    names.append(temp)
```

**Подготовка базы данных. Nodes**

Теперь нам нужно присвоить каждому слову уникальный id, чтобы затем создать связи для базы данных.

Эта часть кода призвана присваивать айди уникальным значениям слов. В этом варианте кода мы не делаем нормализацию, только тестируем общую логику.  В словарь добавляется слово, если программа не находит существующего аналога. После выполнения макс айди увеличивается на 1 и так, пока не закончатся слова.

```python
max_id = 0
for name in names:
    for word in name:
        if dictionary.get(word, 'not found') == 'not found':
            dictionary[word] = max_id
            max_id += 1
```

**Подготовка базы данных. Edges**

Теперь сложнее. Нам нужно учесть отношения слов друг с другом, а также количество повторов комбинаций, которые будут составлять строку weight. Создаем другой словарь под ребра. Он будет выглядеть так: {source, {target:weight, ...}}. 

Когда мы рассматриваем слово нам надо учитывать два фактора. Во первых - что это не слово из которого мы строим ребро, за это у нас отвечает проверка на неравенство индексов i и j. Так же мы не рассматриваем связи между равными словами, за это отвечает проверка line[i] != line[j]

```python
dictionary_edges = {}
for line in names:
    for i in range(len(line)):
        dictionary_edges.setdefault(dictionary[line[i]], {})
        for j in range(len(line)):
            if j != i and line[i] != line[j]:
                dictionary_edges[dictionary[line[i]]].setdefault(dictionary[line[j]], 0)
                dictionary_edges[dictionary[line[i]]][dictionary[line[j]]] += 1
```

**Вывод данных в документы.**

Эта часть кода создает текстовый документ с перечисленными через запятую айди и их значениями. 

```python
sostav = []
sostav = list(dictionary.items())

for word, n_id in sostav:
    idfile.write(str(n_id) + "," + word + "\n")
idfile.close()
print(dictionary_edges)
```

А этот — собирает в нужном порядке информацию о ребрах. 

```python
for i in dictionary_edges.items():
    print(i)
    source, dat = i
    for j in dat.items():
        print(j)
        target, weight = j
        edgesfile.write(str(source) + "," + str(target) + "," + "undirected" + "," + str(weight) + "\n")
```

Теперь у нас есть файлы с входными данными. Их нужно немного подкорректировать. В каждый файл первой строкой вписываю название столбца, чтобы Gephi корректно их распознал, а затем сохраняю в .csv

## Исследование. Gephi

Здесь мы переходим непосредственно в интерфейс gephi. Для начала нужно загрузить наши cvs таблицы в новый проект. Внутри возникает наш необработанный и некрасивый граф, а также можно работать непосредственно с таблицей индексов и связей.

Чтобы начать работать я прохожусь вручную по списку и дочищаю артефакты. Это оставшиеся знаки препинания, а также удаляю союзы и предлоги, которые создают нам не релевантные связи.
![Альтернативный текст](http://images.vfl.ru/ii/1634820559/2b591c95/36356236.png)

А также чищу образовавшиеся повторы средствами самой программы.

![Альтернативный текст](http://images.vfl.ru/ii/1634820640/d075ffc4/36356253.png)

Вот такая штука образовалась.

![Альтернативный текст](http://images.vfl.ru/ii/1634820668/e66cb97d/36356277.png)

Поигравшись с настройками отображения, я смогла отделить основной клубок связанных значений от единичных графов. Разделила их по рабочим областям. 

На первой получилось много вот таких кластеров. По ним можно сделать вывод, связки каких объектов фигурируют в названиях работ вместе, а также как часто определенные словосочетания встречались в отдельных названиях. Ширина линий — тот самый «вес», который мы указывали при обработке текста, если программа находила дубли.

![Альтернативный текст](http://images.vfl.ru/ii/1634820745/4b3bd63e/36356305.png)

На втором рабочем пространстве нам надо организовать наш «клубок» связанных слов. Я применила встроенную укладку.

![Альтернативный текст](http://images.vfl.ru/ii/1634820802/98c948e9/36356312.png)

Теперь их нужно разобрать для лучшей визуализации, удалить лишние узлы (если таковые будут), а также сделать выводы по тому, что мы видим. А теперь галерея наглядного до-после.

Было:

![Альтернативный текст](http://images.vfl.ru/ii/1634821294/d092a237/36356490.png)

Стало:
![Альтернативный текст](http://images.vfl.ru/ii/1634820882/2159664d/36356326.png)

![Альтернативный текст](http://images.vfl.ru/ii/1634820901/7917d900/36356351.png)

![Альтернативный текст](http://images.vfl.ru/ii/1634820916/cca4ae5a/36356383.png)

![Альтернативный текст](http://images.vfl.ru/ii/1634820930/2bb39b12/36356386.png)

![Альтернативный текст](http://images.vfl.ru/ii/1634820947/504f4f81/36356388.png)

![Альтернативный текст](http://images.vfl.ru/ii/1634820966/718b9dbe/36356392.png)

## Выводы

При помощи небольшого кода в python и программы визуализации данных gephi можно создавать наглядные связи между разрабатываемыми в нон-фикшн сфере темами.

Исходя из представленной схемы (файл для программы добавлен в репозиторий), можно без усилий сделать следующие заключения:

- Интерес к Корее, России и Русско-Американским отношениям.
- Много связей между химическими и биологическими вопросами: биохимия, функции и возможности головного мозга, влияние генов на человека.
- Здоровое общество.
- Интересны связи между поступками человека (человек-делать-глупость)
- Вопросы о существовании других (и параллельных миров)
- Вопросы научного характера: поиск гипотез, строение мира, психология (психоанализ, личная и общая психология, психология власти)
- Устройство общества, общество и бизнес, общество и разум.

Впоследствии к подобному анализу можно подключать не только названия, но и аннотации, краткие описания работ — что позволит быстро установить контекстуальные связи, основные термины и т.д. Кроме того, можно создавать выборку по отраслям и годам публикаций.


