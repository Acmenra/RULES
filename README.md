
---


## Руководство по созданию классов в проекте

<details>

<summary>Смотреть руководство</summary>

В данном руководстве представлены рекомендации по созданию классов в рамках нашего проекта. Эти рекомендации помогут нам поддерживать высокий уровень чистоты и удобства сопровождения кода. Ниже приведён пример класса, который демонстрирует правильную структуру и оформление.

## Правила создания классов

При создании новых классов в проекте необходимо соблюдать следующие правила:

1. **Импорт библиотек и модулей**:
   - Сначала импортируются стандартные библиотеки, затем сторонние, и в последнюю очередь – наши собственные модули.
   - Разделяйте разные категории импортов пустой строкой.

2. **Структура класса**:
   - Начните с конструктора `__init__`.
   - После свойств размещайте сеттеры и геттеры (для КАЖДОГО переданного в `__init__` аргумента).
   - Затем определите свойства класса с использованием декоратора `@property`. Свойства допустимы ИСКЛЮЧИТЕЛЬНО для получения вспомогательных значений УЖЕ существующих полей. Например: ключи словаря.
   - Далее следуют методы класса, начиная с основных.
   - В конце добавьте статические методы с декоратором `@staticmethod`.

3. **Типизация**:
   - Обязательно используйте аннотации типов для параметров и возвращаемых значений методов.

4. **Документирование**:
   - Каждый класс, метод и свойство должны сопровождаться соответствующей документацией на английском, по шаблону представленном в примере.
   - Опишите параметры, типы параметров и возвращаемый результат.

5. **Обработка ошибок**:
   - Проверяйте типы данных и генерируйте подходящие исключения, если данные не соответствуют ожидаемым.

6. **Имя и стиль**:
   - Называйте классы и методы в соответствии с рекомендациями PEP8.
   - Используйте информативные названия переменных и функций.


<details>

<summary>Пример класса</summary>

## Пример класса

```python
import os  # Standard library
from typing import Union, List, Dict  # Typing
from custom import module  # Custom module

class MyClass:
    """
    A class for data processing.

    This class is designed to perform various operations on data,
    providing convenient methods for their transformation and analysis.
    """

    def __init__(self, param1: SomeAnotherClass, ..., paramN: int):
        """
        Constructor for the MyClass object.

        Parameters:
        -----------
        param1 : SomeAnotherClass
            An instance of `SomeAnotherClass` used to initialize the `param1` attribute.
        """
        self.param1 = param1
        ...
        self.paramN = paramN

    @property
    def param1(self) -> SomeAnotherClass:
        """Getter for the param1 attribute.

        Returns:
        --------
        SomeAnotherClass
            The current value of the `param1` attribute, which is an instance of `SomeAnotherClass`.
        """
        return self._param1

    @param1.setter
    def param1(self, value: SomeAnotherClass) -> None:
        """Setter for the param1 attribute.

        Parameters:
        -----------
        value : SomeAnotherClass
            The new instance of `SomeAnotherClass` to set for the `param1` attribute.

        Raises:
        -------
        TypeError
            If the provided value is not an instance of `SomeAnotherClass`.
        """
        if not isinstance(value, SomeAnotherClass):  # Always check the type of input data for the setter
            raise TypeError("The provided value must be an instance of 'SomeAnotherClass'.")
        self._param1 = value
        
    # Ather params setters and getters

    @property
    def items(self) -> List[str]:
        """Getter for the list of items from the `SomeAnotherClass` instance.

        Returns:
        --------
        List[str]
            A list of items retrieved from the `SomeAnotherClass` instance.
        """
        return [item for item in SomeAnotherClass.objects()]

    def some_method_with_simple_arg(self, value: str) -> bool:
        """Performs some operation with the given value.

        Parameters:
        -----------
        value : str
            The value to operate on. Must be a non-empty string.

        Returns:
        --------
        bool
            The result of the operation. Returns `True` if the operation is successful.

        Raises:
        -------
        TypeError
            If the provided value is not a string.
        ValueError
            If the provided string is empty.
        """
        if not isinstance(value, str):  # Always check the type of input data for each method argument
            raise TypeError("The provided value must be a string.")
        if len(value) == 0:  # Always validate the input data for each method argument
            raise ValueError("The length of 'value' should be greater than zero.")
        return True

    def some_method_with_intricate_arg(self, value: Dict[str, float]) -> bool:
        """Performs some operation with the given value.

        Parameters:
        -----------
        value : Dict[str, float]
            A dictionary where keys are strings and values are floats.
            This dictionary is used for the operation.

        Returns:
        --------
        bool
            The result of the operation. Returns `True` if the operation is successful.

        Raises:
        -------
        ValueError
            If the provided dictionary fails validation.
        """
        self._check_value_intricate_method(value)
        return True

    def _check_value_intricate_method(self, value: Dict[str, float]) -> None:
        """Validates the input value for the `some_method_with_intricate_arg` method.

        Parameters:
        -----------
        value : Dict[str, float]
            A dictionary where keys are strings and values are floats.

        Raises:
        -------
        ValueError
            If the provided dictionary fails validation.
        """
        # Value checking logic
        if not value:
            raise ValueError("The provided dictionary cannot be empty.")

    @staticmethod
    def static_method() -> None:
        """A static method that performs some common task.

        Returns:
        --------
        None
            This method does not return any value.
        """
        # Static method logic
        print("Static method called!")
```
</details>

</details>



---

## Руководство по созданию тестирования в проекте

<details>
<summary>Смотреть руководство</summary>

В данном руководстве представлены рекомендации по созданию тестов для классов в рамках нашего проекта. Эти рекомендации помогут нам поддерживать высокий уровень качества кода, обеспечивая его корректность и устойчивость к изменениям. Ниже приведены правила и примеры написания тестов.

## Правила создания тестов для классов

При создании тестов для классов в проекте необходимо соблюдать следующие правила:

1. **Именование файлов и классов**:
   - Файл с тестами должен называться `test_[имя файла, в котором лежит тестируемый класс].py`. Например, если тестируемый класс находится в файле `my_class.py`, то файл с тестами должен называться `test_my_class.py`.
   - Тестирующий класс должен называться `Test[ИмяТестируемогоКласса]`. Например, если тестируемый класс называется `MyClass`, то тестирующий класс должен называться `TestMyClass`.

2. **Именование тестовых методов**:
   - Тестовые методы должны называться по шаблону `test_[имя класса]_[метод класса]_[на что тестируем]`. Например, если тестируется метод `some_method` класса `MyClass` на корректность обработки аргументов, то метод должен называться `test_my_class_some_method_wrong_type`.

3. **Импорт библиотек и модулей**:
   - Сначала импортируются стандартные библиотеки, затем сторонние, и в последнюю очередь – наши собственные модули.
   - Разделяйте разные категории импортов пустой строкой.

4. **Структура тестового класса**:
   - Начните с тестов инициализации класса (конструктора `__init__`).
   - Затем добавьте тесты для сеттеров и геттеров (для каждого переданного в `__init__` аргумента).
   - Далее следуют тесты для свойств класса (если они есть).
   - После этого добавьте тесты для методов класса, начиная с основных.
   - В конце добавьте тесты для статических методов (если они есть).

5. **Типизация**:
   - Обязательно используйте аннотации типов для параметров и возвращаемых значений тестовых методов.

6. **Документирование**:
   - Каждый тестовый класс и метод должны сопровождаться соответствующей документацией на английском языке.
   - Опишите, что именно тестируется, какие параметры используются и какие результаты ожидаются.

7. **Обработка ошибок**:
   - Проверяйте, что методы корректно обрабатывают невалидные входные данные и генерируют ожидаемые исключения.

8. **Использование `unittest` и `ddt`**:
   - Используйте модуль `unittest` для создания тестов.
   - Для параметризации тестов используйте библиотеку `ddt` (Data-Driven Tests), чтобы избежать дублирования кода.

9. **Имя и стиль**:
   - Называйте тестовые классы и методы в соответствии с рекомендациями PEP8.
   - Используйте информативные названия переменных и функций.

<details>

<summary>Пример тестирования</summary>

```python
import unittest
from ddt import ddt, data, unpack
from my_class import MyClass


@ddt
class TestMyClass(unittest.TestCase):
    """
    A test class for `MyClass`.

    This class contains unit tests for the `MyClass` class, covering its initialization,
    properties, methods, and error handling.
    """

    # INIT
    @data(SomeAnotherClass(1, 2), SomeAnotherClass(2, 3), SomeAnotherClass(3, 4), SomeAnotherClass(4, 5))
    def test_my_class_initialisation_param1_success(self, value: SomeAnotherClass) -> None:
        """
        Test successful initialization of `MyClass` with valid `param1`.

        Parameters:
        -----------
        value : SomeAnotherClass
            A valid instance of `SomeAnotherClass` to initialize `MyClass`.

        Asserts:
        --------
        - `param1` is an instance of `SomeAnotherClass`.
        - `param1.some_field` matches the provided value.
        """
        my_class = MyClass(param1=value, ..., paramN=**default**)
        self.assertIsInstance(my_class.param1, SomeAnotherClass)
        self.assertEqual(my_class.param1.some_field, value.some_field)

    ...

    @data(0, 1, 10, 100, 500, 1_000, 5_000, 10_000, 50_000, 100_000, 500_000, 1_000_000)
    def test_my_class_initialisation_paramN_success(self, value: int) -> None:
        """
        Test successful initialization of `MyClass` with valid `paramN`.

        Parameters:
        -----------
        value : int
            A valid integer value to initialize `paramN`.

        Asserts:
        --------
        - `paramN` is an instance of `int`.
        """
        my_class = MyClass(param1=**default**, ..., paramN=value)
        self.assertIsInstance(my_class.paramN, int)
        self.assertEqual(my_class.paramN, value)

    # PARAM1-SETTER-GETTER
    @data(-1.0, 1, True, False, None, [], ["hi"], (), ("hi",), set(), {"hi"}, 1 + 2j, b"", b"hi", bytearray(b""), 
          bytearray(b"hi"), range(0), TestDataClass(1, 2), TestClass(1), TestEnum.VALUE_ONE)
    def test_my_class_param1_setter_wrong_type(self, value: Any) -> None:
        """
        Test `param1` setter with invalid types.

        Parameters:
        -----------
        value : Any
            An invalid value to set for `param1`.

        Asserts:
        --------
        - `TypeError` is raised when an invalid type is provided.
        """
        my_class = MyClass(param1=**default**, ..., paramN=**default**)
        with self.assertRaises(TypeError):
            my_class.param1 = value

    @data(SomeAnotherClass(1, 2), SomeAnotherClass(2, 3), SomeAnotherClass(3, 4), SomeAnotherClass(4, 5))
    def test_my_class_param1_setter_success(self, value: SomeAnotherClass) -> None:
        """
        Test `param1` setter with valid types.

        Parameters:
        -----------
        value : SomeAnotherClass
            A valid instance of `SomeAnotherClass` to set for `param1`.

        Asserts:
        --------
        - `param1` is successfully set to the provided value.
        """
        my_class = MyClass(param1=**default**, ..., paramN=**default**)
        my_class.param1 = value
        self.assertIsInstance(my_class.param1, SomeAnotherClass)
        self.assertEqual(my_class.param1, value)

    # ATHER PARAMS
    ...
    
    # ITEMS
    @data(SomeAnotherClass(1, 2), SomeAnotherClass(2, 3), SomeAnotherClass(3, 4), SomeAnotherClass(4, 5))
    def test_my_class_items_success(self, value: SomeAnotherClass) -> None:
        """
        Test the `items` property of `MyClass`.

        Parameters:
        -----------
        value : SomeAnotherClass
            A valid instance of `SomeAnotherClass` to initialize `MyClass`.

        Asserts:
        --------
        - `items` returns a list of items from the `SomeAnotherClass` instance.
        """
        my_class = MyClass(param1=value, ..., paramN=**default**)
        self.assertIsInstance(my_class.items, list)
        self.assertEqual(my_class.items, [...])

    # ATHER PROPERTIES
    ...
    
    
    # SOME_METHOD_WITH_SIMPLE_ARG
    @data(-1.0, 1, True, False, None, [], ["hi"], (), ("hi",), set(), {"hi"}, 1 + 2j, b"",
          b"hi", bytearray(b""), bytearray(b"hi"), range(0), TestDataClass(1, 2), TestClass(1), TestEnum.VALUE_ONE)
    def test_my_class_some_method_with_simple_arg_wrong_type(self, value: Any):
        """
        Test `some_method_with_simple_arg` with invalid types.

        Parameters:
        -----------
        value : Any
            An invalid value to pass to `some_method_with_simple_arg`.

        Asserts:
        --------
        - `TypeError` is raised when an invalid type is provided.
        """
        my_class = MyClass(param1=**default**, ..., paramN=**default**)
        with self.assertRaises(TypeError):
            my_class.some_method_with_simple_arg(value=value)

    @data("", " ", "invalid")
    def test_my_class_some_method_with_simple_arg_wrong_value(self, value: str):
        """
        Test `some_method_with_simple_arg` with invalid values.

        Parameters:
        -----------
        value : str
            An invalid string value to pass to `some_method_with_simple_arg`.

        Asserts:
        --------
        - `ValueError` is raised when an invalid value is provided.
        """
        my_class = MyClass(param1=**default**, ..., paramN=**default**)
        with self.assertRaises(ValueError):
            my_class.some_method_with_simple_arg(value=value)

    @data(("valid", True), ("another_valid", True))
    @unpack
    def test_my_class_some_method_with_simple_arg_success(self, value: str, answer: bool):
        """
        Test `some_method_with_simple_arg` with valid values.

        Parameters:
        -----------
        value : str
            A valid string value to pass to `some_method_with_simple_arg`.
        answer : bool
            The expected result of the method.

        Asserts:
        --------
        - The method returns the expected boolean result.
        """
        my_class = MyClass(param1=**default**, ..., paramN=**default**)
        result = my_class.some_method_with_simple_arg(value=value)
        self.assertIsInstance(result, bool)
        self.assertEqual(result, answer)
```
</details>

</details>

---
