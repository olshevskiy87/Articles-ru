Новые интересные структуры данных в Python 3
============================================

Python 3 уже давно не новинка.

Если быть точным, недавно праздновали_ его 3000-й день :). Сейчас, после некоторого затишья,
Python 3 стал очень активно осваиваться, и я думаю, что пришло время рассмотреть некоторые новые
структуры данных, которых не было во второй версии.

Мы рассмотрим ``type.MappingProxyType``, ``typing.NamedTuple`` и ``types.SimpleNamespace``,
которые появились только в Python 3.

``types.MappingProxyType``
--------------------------

``types.MappingProxyType`` используется как словарь с доступом только для чтения (был добавлен в
Python 3.3). Более подробную информацию можно найти в документации_.

"Только для чтения" в ``types.MappingProxyType`` означает, что нельзя напрямую модифицировать его
содержимое, и если пользователь хочет его изменить, то ему нужно явно создать копию и работать уже
с ней. Это замечательно работает, если вы передаете ``dict`` -подобные структуры некоему
обработчику и хотите быть уверены, что он нечаянно не изменит исходные данные. На практике это
бывает чрезвычайно полезно, так как изменение передаваемых данных влечет за собой появление
незаметных и, как следствие, трудно отслеживаемых ошибок в вашем коде.

Примеры использования ``types.MappingProxyType``:

.. code-block :: python
    
    >>> from  types import MappingProxyType
    >>> data = {'a': 1, 'b':2}
    >>> read_only = MappingProxyType(data)
    >>> del read_only['a']
    TypeError: 'mappingproxy' object does not support item deletion
    >>> read_only['a'] = 3
    TypeError: 'mappingproxy' object does not support item assignment

Заметьте, что в примере объект ``read_only`` нельзя изменить напрямую.

Итак, если вы хотите передать словари с данными в другую функцию или поток и хотите быть уверены,
что вызываемая функция не изменит данные, которые также используются другой функцией, можете
просто отправить объект ``MappingProxyType`` вместо привычного ``dict``. Следующий пример
показывает, как можно использовать ``MappingProxyType`` на практике:

.. code-block :: python
    
    >>> def my_threaded_func(in_dict):
    >>>    ...
    >>>    in_dict['a'] *= 10  # Упс, ошибка. Изменение переданного словаря
    
    ...
    # в какой-нибудь функции или в другом потоке
    >>> my_threaded_func(data)
    >>> data
    data = {'a': 10, 'b':2}  # заметьте, что элемент словаря data['a'] изменился из-за my_threaded_func

Если вы вместо этого отправите ``mappingproxy`` в ``my_threaded_func``, попытка изменить словарь
приведёт к ошибке.

.. code-block :: python
    
    >>> my_threaded_func(MappingProxyType(data))
    TypeError: 'mappingproxy' object does not support item deletion

Теперь видно, что нужно исправить код в функции ``my_threaded_func``, чтобы сначала создать копию
объекта ``in_dict``, а затем изменять уже её и таким образом избежать ошибок. Это просто
замечательная особенность нового типа.

Заметьте всё же, что хоть ``read_only`` и только для чтения, но не неизменяемый, поэтому, если вы
модифицируете объект ``data``, ``read_only`` также изменится.

.. code-block :: python
    
    >>> data['a'] = 3
    >>> data['c'] = 4
    >>> read_only  # изменился!
    mappingproxy({'a': 3, 'b': 2, 'c': 4})

Этого стоит по возможности избегать.

``typing.NamedTuple``
---------------------

``typing.NamedTuple`` - это перегруженная версия старого типа ``collections.namedtuple``, и хоть
он и был добавлен еще в версии Python 3.5, но по-настоящему самим собой он стал только в Python 3.6.

По сравнению с ``collections.namedtuple``, ``typing.NamedTuple`` даёт вам (Python >= 3.6):

- более удобный синтаксис
- наследрвание
- аннотации типов
- значения по умолчанию (python >= 3.6.1)

Пример использования ``typing.NamedTuple``:

.. code-block :: python
    
    >>> from typing import NamedTuple
    >>> class Student(NamedTuple):
    >>>    name: str
    >>>    address: str
    >>>    age: int
    >>>    sex: str
    
    >>> tommy = Student(name='Tommy Johnson', address='Main street', age=22, sex='M')
    >>> tommy
    Student(name='Tommy Johnson', address='Main street', age=22, sex='M')

В отличие от старого функционального синтаксиса мне нравится новый, основанный на классах. Мне он
кажется наиболее легким для чтения.

Имейте ввиду, что на самом деле у нас здесь получился кортеж, а не экземпляр класса:

.. code-block :: python
    
    >>> isinstance(tommy, tuple)
    True
    >>> tommy[0]
    'Tommy Johnson'

Более сложный пример, использующий наследование класса ``Student`` и значения по умолчанию
(последние работают только в Python >= **3.6.1**):

.. code-block :: python
    
    >>> class MaleStudent(Student):
    >>>    sex: str = 'M'  # значение по умолчанию, требуется Python >= 3.6.1
    
    >>> MaleStudent(name='Tommy Johnson', address='Main street', age=22)
    MaleStudent(name='Tommy Johnson', address='Main street', age=22, sex='M')  # пол по умолчанию 'M'

Короче говоря, эта новомодная версия ``namedtuples`` просто впечатляет и в будущем она, несомненно,
станет стандартной.

``types.SimpleNamespace``
-------------------------

``types.SimpleNamespace`` - это простой класс, который предоставляет атрибутный доступ к своему
пространству имен и выразительный ``repr``. Был добавлен в Python 3.3.

.. code-block :: python
    
    >>> from types import SimpleNamespace
    >>> data = SimpleNamespace(a=1, b=2)
    >>> data
    namespace(a=1, b=2)
    >>> data.c = 3
    >>> data
    namespace(a=1, b=2, c=3)

Вкратце, ``types.SimpleNamespace`` - это просто очень простой класс, позволяющий вам установить,
изменить и удалить атрибуты, в то же время позволяя использовать ``repr`` для строкового
представления. Иногда я использую его как более простую альтернативу ``dict`` или для удобного
наследования и "бесплатного" использования ``repr``:

.. code-block :: python
    
    >>> import random
    >>> class DataBag(SimpleNamespace):
    >>>    def choice(self):
    >>>        items = self.__dict__.items()
    >>>        return random.choice(tuple(items))
    
    >>> data_bag = DataBag(a=1, b=2)
    >>> data_bag
    DataBag(a=1, b=2)
    >>> data_bag.choice()
    (b, 2)

Такое наследование ``types.SimpleNamespace``, возможно, и не революционно само по себе, но оно
может помочь избавиться от нескольких лишних строк кода в некоторых случаях, что уже неплохо.

.. _праздновали: https://www.reddit.com/r/Python/comments/5v0tt6/python_3_created_via_pep_3000_is_exactly_3000/
.. _документации: https://docs.python.org/3/library/types.html#types.MappingProxyType
