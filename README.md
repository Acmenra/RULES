## Руководство по созданию классов в проекте

<details>

<summary>Смотреть руководство</summary>

В данном руководстве представлены рекомендации по созданию классов в рамках нашего проекта. Эти рекомендации помогут нам поддерживать высокий уровень чистоты и удобства сопровождения кода.

## Правила создания классов

При создании новых классов в проекте необходимо соблюдать следующие правила:

### 1. Импорт библиотек и модулей

```python
# 1. Standard library imports
import logging
import math
from typing import List, Dict, Optional, Any, Tuple, Union
from enum import Enum, EnumType
from abc import ABC, abstractmethod

# 2. Third-party imports
import numpy as np
import cv2
from ultralytics import YOLO

# 3. Local application imports
from acmenra_cv.instance.point import Point
from acmenra_cv.instance.enums import TaskType, DeviceType
from acmenra_cv.inference.backend import Backend
```

**Правила:**
- Сначала стандартные библиотеки Python
- Затем сторонние библиотеки (numpy, cv2, ultralytics)
- В конце — наши собственные модули из `acmenra_cv`
- Разделяйте категории импортов пустой строкой
- Используйте абсолютные импорты (`from acmenra_cv.instance.point import Point`)

### 2. Структура класса

```python
class MyClass:
    """Class-level docstring describing the purpose."""
    
    # 1. Class attributes (если есть)
    DEFAULT_VALUE = 0.5
    
    # 2. Constructor
    def __init__(self, param1: Type1, param2: Type2):
        """Constructor docstring."""
        self.param1 = param1  # Вызывает setter
        self.param2 = param2  # Вызывает setter
    
    # 3. String representations
    def __str__(self) -> str:
        """Human-readable representation."""
        pass
    
    def __repr__(self) -> str:
        """Debug representation."""
        pass
    
    # 4. Properties (getters)
    @property
    def param1(self) -> Type1:
        """Getter docstring."""
        return self._param1
    
    # 5. Property setters (с валидацией)
    @param1.setter
    def param1(self, value: Type1) -> None:
        """Setter docstring with validation."""
        if not isinstance(value, Type1):
            raise TypeError(f"'param1' must be '<Type1>', but got {type(value).__name__}")
        self._param1 = value
    
    # 6. Computed properties (только для вычисляемых значений)
    @property
    def computed_value(self) -> float:
        """Computed property docstring."""
        return self._param1 * 2.0
    
    # 7. Regular methods
    def some_method(self, arg: Type) -> ReturnType:
        """Method docstring."""
        pass
    
    # 8. Static methods
    @staticmethod
    def static_method() -> None:
        """Static method docstring."""
        pass
    
    # 9. Class methods (factory methods)
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'MyClass':
        """Factory method docstring."""
        pass
```

### 3. Типизация

**Обязательные аннотации:**
```python
def method(
    self,
    param1: float,
    param2: Optional[int] = None,
    param3: List[Tuple[int, int, int]]
) -> Dict[str, Any]:
    pass
```

**Используйте:**
- `Optional[Type]` для необязательных параметров
- `List`, `Dict`, `Tuple`, `Set` из `typing`
- Union[Type1, Type2] для множественных типов
- `Any` только когда тип действительно неизвестен

### 4. Документирование (наш стиль)

**Класс:**
```python
class Point:
    """
    Represents a point in 3D space with normalized coordinates.
    
    This class provides validated 3D coordinates with strict type checking
    and range enforcement [0.0, 1.0]. Designed for spatial calculations
    in computer vision pipelines.
    
    Attributes:
        _x (float): X coordinate in normalized space [0.0, 1.0].
        _y (float): Y coordinate in normalized space [0.0, 1.0].
        _z (float): Z coordinate in normalized space [0.0, 1.0].
    
    Note:
        - All coordinates use normalized space (0.0-1.0)
        - Z coordinate defaults to 0.0 for 2D images
        - Coordinates are validated on assignment
    """
```

**Метод:**
```python
def get_distance(self, point: 'Point') -> float:
    """
    Calculates the Euclidean distance to another point.
    
    This method computes the 3D Euclidean distance using the standard
    formula: sqrt((x2-x1)² + (y2-y1)² + (z2-z1)²)
    
    Args:
        point (Point): The other point to calculate distance to.
    
    Returns:
        float: The Euclidean distance in normalized units.
    
    Note:
        Distance is returned in normalized units. Multiply by image
        dimensions to obtain pixel distance for downstream tasks.
    """
    dx = point.X - self.X
    dy = point.Y - self.Y
    dz = point.Z - self.Z
    return math.sqrt(dx * dx + dy * dy + dz * dz)
```

**Property:**
```python
@property
def width(self) -> float:
    """
    float: Gets the width of the box (right - left).
    
    Returns:
        float: Width in normalized units (always non-negative).
    
    Note:
        Width is computed as absolute difference to handle any
        vertex ordering. Used for area and dimension calculations.
    """
    return abs(self.right - self.left)
```

### 5. Обработка ошибок (наш стиль)

**Type checking:**
```python
@x.setter
def x(self, x: float) -> None:
    """Sets the X coordinate with validation."""
    if not isinstance(x, float):
        raise TypeError(f"'x' must be '<float>', but got {type(x).__name__}")
    if not 0.0 <= x <= 1.0:
        raise ValueError(f"'x' must be in [0.0, 1.0], but got {x}")
    self._x = x
```

**Range validation:**
```python
@alpha.setter
def alpha(self, alpha: float) -> None:
    """Sets the alpha value with range validation."""
    if not isinstance(alpha, float):
        raise TypeError(f"'alpha' must be '<float>', but got {type(alpha).__name__}")
    if not 0.0 <= alpha <= 1.0:
        raise ValueError(f"'alpha' must be in [0.0, 1.0], got {alpha}")
    self._alpha = alpha
```

**Boolean rejection:**
```python
@thickness.setter
def thickness(self, thickness: int) -> None:
    """Sets the thickness with type validation."""
    if not isinstance(thickness, int) or isinstance(thickness, bool):
        raise TypeError(f"'thickness' must be '<int>', but got {type(thickness).__name__}")
    if thickness < 0:
        raise ValueError(f"'thickness' must be >= 0, got {thickness}")
    self._thickness = thickness
```

### 6. Фабричные методы

**safe() — с clamping:**
```python
@classmethod
def safe(cls, x: float, y: float, z: float = 0.0,
         min_val: float = 0.0, max_val: float = 1.0) -> 'Point':
    """
    Creates a Point with coordinates clamped to [min_val, max_val].
    
    This factory method safely constrains coordinates to specified range
    without raising exceptions. Bypasses __init__ validation to support
    custom coordinate systems.
    
    Args:
        x (float): X coordinate to clamp.
        y (float): Y coordinate to clamp.
        z (float, optional): Z coordinate to clamp. Defaults to 0.0.
        min_val (float, optional): Minimum allowed value. Defaults to 0.0.
        max_val (float, optional): Maximum allowed value. Defaults to 1.0.
    
    Returns:
        Point: New instance with clamped coordinates.
    
    Note:
        - Directly assigns to private attributes to bypass validation
        - No exceptions raised, guaranteed to return valid Point
        - Recommended for processing external/untrusted data
    """
    point = cls.__new__(cls)
    point._x = float(min_val if x < min_val else (max_val if x > max_val else x))
    point._y = float(min_val if y < min_val else (max_val if y > max_val else y))
    point._z = float(min_val if z < min_val else (max_val if z > max_val else z))
    return point
```

**from_dict() — десериализация:**
```python
@classmethod
def from_dict(cls, data: Dict[str, Any]) -> 'Fill':
    """
    Reconstructs a Fill instance from a serialized dictionary.
    
    Args:
        data (Dict[str, Any]): Dictionary with fill configuration.
            Required: alpha.
    
    Returns:
        Fill: Fully initialized and validated instance.
    
    Raises:
        KeyError: If alpha is missing from input dictionary.
        TypeError: If alpha is not a valid float.
        ValueError: If alpha is outside [0.0, 1.0].
    
    Note:
        - Delegates validation to property setters
        - Maintains perfect round-trip fidelity with to_dict()
    """
    required_fields = ['alpha']
    for field in required_fields:
        if field not in data:
            raise KeyError(f"Required field '{field}' is missing")
    
    return cls(alpha=float(data.get("alpha")))
```

### 7. Пример полного класса (из нашего кода)

```python
import logging
from typing import Dict, Any

logger = logging.getLogger(__name__)


class Fill:
    """
    Configuration for shape fills.
    
    If `Style.fill` is set to `None`, no interior fill is drawn.
    When present, uses `palette[class_id]` for color combined with this parameter.
    
    Attributes:
        _alpha (float): Fill opacity [0.0, 1.0].
    
    Note:
        - Alpha value is multiplied with global Style.alpha during compositing
        - Controls how much of the underlying video frame shows through
    """
    
    def __init__(self, alpha: float = 0.35):
        """
        Initializes fill configuration with validation.
        
        Args:
            alpha (float, optional): Fill opacity [0.0, 1.0]. Defaults to 0.35.
        
        Raises:
            TypeError: If alpha is not a float.
            ValueError: If alpha is outside [0.0, 1.0].
        """
        self.alpha = alpha
    
    def __str__(self) -> str:
        """
        Returns a human-readable string representation.
        
        Returns:
            str: Concise summary with fill opacity level.
                 Format: "Fill(alpha=0.35)"
        """
        return f"Fill(alpha={self.alpha:.2f})"
    
    def __repr__(self) -> str:
        """
        Returns a detailed string representation for debugging.
        
        Returns:
            str: Exact representation including the canonical alpha parameter.
        """
        return f"Fill(alpha={self.alpha!r})"
    
    @property
    def alpha(self) -> float:
        """
        float: Gets the fill opacity level.
        
        Returns:
            float: Opacity value in range [0.0, 1.0].
        """
        return self._alpha
    
    @alpha.setter
    def alpha(self, alpha: float) -> None:
        """
        Sets the fill opacity with range validation.
        
        Args:
            alpha (float): Opacity value to set. Must be in [0.0, 1.0].
        
        Raises:
            TypeError: If alpha is not a float.
            ValueError: If alpha is outside [0.0, 1.0].
        """
        if not isinstance(alpha, float):
            raise TypeError(f"'alpha' must be '<float>', but got {type(alpha).__name__}")
        if not 0.0 <= alpha <= 1.0:
            raise ValueError(f"'alpha' must be in [0.0, 1.0], got {alpha}")
        self._alpha = alpha
    
    def to_dict(self) -> Dict[str, Any]:
        """
        Serializes the Fill configuration to a JSON-compatible dictionary.
        
        Returns:
            Dict[str, Any]: Dictionary containing the fill opacity parameter.
        
        Note:
            - Ready for direct DB insertion or API transmission
            - Maintains perfect round-trip fidelity with from_dict()
        """
        return {"alpha": self.alpha}
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'Fill':
        """
        Reconstructs a Fill instance from a serialized dictionary.
        
        Args:
            data: Dictionary with fill configuration. Required: alpha.
        
        Returns:
            Fill: Fully initialized and validated instance.
        
        Raises:
            KeyError: If alpha is missing from input dictionary.
            TypeError: If alpha is not a valid float.
            ValueError: If alpha is outside [0.0, 1.0].
        """
        required_fields = ['alpha']
        for field in required_fields:
            if field not in data:
                raise KeyError(f"Required field '{field}' is missing from input dictionary: {data}")
        
        return cls(alpha=float(data.get("alpha")))
```

</details>

---

## Руководство по созданию тестирования в проекте

<details>
<summary>Смотреть руководство</summary>

В данном руководстве представлены рекомендации по созданию тестов для классов в рамках нашего проекта.

## Правила создания тестов

### 1. Именование файлов и классов

**Файлы:**
- `test_[имя_файла_с_классом].py`
- Пример: `test_point.py`, `test_box.py`, `test_fill.py`

**Классы:**
- `Test[ИмяКласса]`
- Пример: `TestPoint`, `TestBox`, `TestFill`

### 2. Именование тестовых методов

**Шаблон:** `test_[имя_класса]_[что_тестируем]_[результат]`

**Примеры из нашего кода:**
```python
# Инициализация
def test_point_initialization(self, point: Point) -> None:

# String representations
def test_point_str_human_readable(self, point: Point) -> None:
def test_point_repr_debuggable(self, point: Point) -> None:

# Setter - wrong type
def test_point_x_setter_wrong_type(self, value: Any) -> None:

# Setter - invalid values
def test_point_x_setter_invalid_values(self, value: float) -> None:

# Setter - valid values
def test_point_x_setter_valid_values(self, value: float) -> None:

# Round-trip fidelity
def test_fill_roundtrip_fidelity(self, fill: Fill) -> None:
```

### 3. Структура тестового класса

```python
import unittest
from ddt import ddt, data
from typing import Any

from acmenra_cv.instance.point import Point
from tests.instance.mocks.mock_point import *
from tests.instance.mocks.data_to_test_cases import *


@ddt
class TestPoint(unittest.TestCase):
    """
    Comprehensive test suite for the Point entity validation and lifecycle.
    
    This test class thoroughly validates all aspects of the Point class, including:
    - Proper initialization with normalized 3D coordinates (X, Y, Z)
    - Strict type validation for all coordinate properties
    - Accurate handling of valid, invalid, and boundary input values
    - Correct behavior of geometric operations
    """
    
    def setUp(self) -> None:
        """
        Sets up a baseline Point instance for property modification tests.
        
        Note:
            - Initializes with standard normalized coordinates (0.5, 0.5, 0.5)
            - Provides a mutable baseline for setter tests
        """
        self.point = Point(0.5, 0.5, 0.5)
    
    # =========================================================================
    # INITIALIZATION
    # =========================================================================
    @data(pt_0, pt_1, pt_2, ..., pt_49)
    def test_point_initialization(self, point: Point) -> None:
        """Test initialization with pre-configured mocks."""
        pass
    
    # =========================================================================
    # STRING REPRESENTATIONS
    # =========================================================================
    @data(pt_0, pt_1, pt_2, ..., pt_49)
    def test_point_str_human_readable(self, point: Point) -> None:
        """Test __str__ output format."""
        pass
    
    # =========================================================================
    # X COORDINATE SETTER
    # =========================================================================
    @data(1, "1", True, [], ...)
    def test_point_x_setter_wrong_type(self, value: Any) -> None:
        """Test X setter with invalid types."""
        pass
    
    @data(-1_000_000.0, ..., 1_000_000.0)
    def test_point_x_setter_invalid_values(self, value: float) -> None:
        """Test X setter with out-of-range values."""
        pass
    
    @data(0.0, 0.5, 1.0, ...)
    def test_point_x_setter_valid_values(self, value: float) -> None:
        """Test X setter with valid values."""
        pass
```

### 4. Использование DDT (Data-Driven Tests)

**Для параметризации тестов:**
```python
@data(0, 1, 10, 100, 500, 1_000, 5_000, 10_000)
def test_tracker_id_setter_valid_values(self, value: int) -> None:
    """Test ID setter with valid non-negative integers."""
    self.tracker.id = value
    self.assertIsInstance(self.tracker.id, int)
    self.assertEqual(self.tracker.id, value)
```

**Для тестирования boundary values:**
```python
@data(-1_000_000.0, -500_000.0, -100_000.0, -50_000.0, -10_000.0,
      -5_000.0, -1_000.0, -500.0, -100.0, -50.0, -25.0, -10.0,
      -5.0, -2.5, -1.0, -0.5, -0.25, -0.1, -0.05, -0.01, -0.001)
def test_fill_alpha_setter_invalid_values(self, value: float) -> None:
    """Test alpha setter with negative values."""
    with self.assertRaises(ValueError):
        self.fill.alpha = value
```

**Для тестирования wrong types:**
```python
@data(1, "1", True, False, None, [], ["hi"], (), ("hi",), set(), {"hi"},
      1 + 2j, b"", b"hi", bytearray(b""), range(0))
def test_stroke_thickness_setter_wrong_type(self, value: Any) -> None:
    """Test thickness setter with invalid types."""
    with self.assertRaises(TypeError):
        self.stroke.thickness = value
```

### 5. Mock объекты

**Создание mock данных:**
```python
# tests/instance/mocks/mock_point.py
from acmenra_cv.instance.point import Point

# Boundary values
pt_0 = Point(0.0, 0.0, 0.0)  # Minimum values
pt_1 = Point(1.0, 1.0, 1.0)  # Maximum values
pt_2 = Point(0.5, 0.5, 0.5)  # Middle values

# Edge cases
pt_3 = Point(0.000_000_01, 0.000_000_01, 0.000_000_01)  # Epsilon
pt_4 = Point(0.999_999_99, 0.999_999_99, 0.999_999_99)  # Near max

# 2D points (Z=0)
pt_5 = Point(0.3, 0.7, 0.0)
```

**Использование mock в тестах:**
```python
from tests.instance.mocks.mock_point import *
from tests.instance.mocks.data_to_test_cases import *

@data(pt_0, pt_1, pt_2, ..., pt_49)
def test_point_initialization(self, point: Point) -> None:
    """Test initialization with pre-configured mocks."""
    self.assertIsInstance(point.X, float)
    self.assertTrue(0.0 <= point.X <= 1.0)
```

### 6. Документирование тестов (наш стиль)

**Тестовый метод:**
```python
def test_point_x_setter_invalid_values(self, value: float) -> None:
    """
    Verifies that setting the X coordinate with out-of-range values raises ValueError.
    
    This test ensures that:
    - Only values within the normalized range [0.0, 1.0] are accepted
    - Boundary violations (< 0.0 or > 1.0) are strictly rejected
    - Clear ValueError messages indicate the constraint violation
    
    The test validates the range safety of the X property setter,
    confirming robust protection against invalid coordinate values
    which could cause errors in spatial calculations.
    
    Args:
        value (float): Out-of-bound float values to test range validation
    
    Note:
        Normalized coordinates prevent pixel overflow and ensure consistent
        behavior across different image resolutions.
    """
    with self.assertRaises(ValueError):
        self.point.X = value
```

### 7. Пример полного теста (из нашего кода)

```python
import unittest
from ddt import ddt, data
from typing import Any

from acmenra_cv.render.style.fill import Fill
from tests.render.mocks.mock_fill import *


@ddt
class TestFill(unittest.TestCase):
    """
    Comprehensive test suite for the Fill entity validation and lifecycle.
    
    This test class thoroughly validates all aspects of the Fill class, including:
    - Proper initialization with alpha opacity [0.0, 1.0]
    - Strict type validation for the alpha property
    - Accurate handling of valid, invalid, and boundary alpha values
    - Correct behavior of string representations
    - Robust error handling for type mismatches
    """
    
    def setUp(self) -> None:
        """
        Sets up a baseline Fill instance for property modification tests.
        
        Note:
            - Initializes with standard alpha value (0.5)
            - Provides a mutable baseline for setter tests
        """
        self.fill = Fill(alpha=0.5)
    
    # =========================================================================
    # INITIALIZATION
    # =========================================================================
    @data(fi_0, fi_1, fi_2, ..., fi_20)
    def test_fill_initialization(self, fill: Fill) -> None:
        """
        Verifies successful initialization of Fill instances using pre-configured mocks.
        
        This test ensures that:
        - All Fill instances have correctly typed alpha values
        - Boundary values (0.0, 1.0, epsilon) are handled properly
        - Core constraint (alpha in [0.0, 1.0]) is strictly enforced
        
        Args:
            fill (Fill): Pre-configured Fill mock instance provided by DDT.
        
        Note:
            DDT automatically runs this test once per mock (21 total executions).
        """
        self.assertIsInstance(fill.alpha, float)
        self.assertGreaterEqual(fill.alpha, 0.0)
        self.assertLessEqual(fill.alpha, 1.0)
    
    # =========================================================================
    # ALPHA SETTER
    # =========================================================================
    @data(1, "1", True, False, None, [], ["hi"], (), 1 + 2j, b"")
    def test_fill_alpha_setter_wrong_type(self, value: Any) -> None:
        """
        Verifies that setting alpha with invalid types raises TypeError.
        
        This test ensures that:
        - Only float values are accepted for the alpha property
        - Integers, strings, booleans, containers are explicitly rejected
        - Clear TypeError messages are provided
        
        The test validates the type safety of the alpha property setter,
        confirming robust protection against invalid style representations.
        
        Args:
            value (Any): Various non-float values to test type validation
        
        Note:
            Strict float typing prevents silent precision loss.
        """
        with self.assertRaises(TypeError):
            self.fill.alpha = value
    
    @data(-1_000_000.0, -500_000.0, -100_000.0, -1.0, -0.5, -0.1, -0.01,
          1.000_000_1, 1.000_01, 1.001, 1.01, 1.1, 1.5, 2.0, 10.0, 100.0)
    def test_fill_alpha_setter_invalid_values(self, value: float) -> None:
        """
        Verifies that setting alpha with out-of-range values raises ValueError.
        
        This test ensures that:
        - Only values within [0.0, 1.0] are accepted
        - Boundary violations are strictly rejected
        - Clear ValueError messages indicate the constraint violation
        
        Args:
            value (float): Out-of-bound float values to test range validation
        
        Note:
            Normalized alpha prevents OpenCV assertion errors.
        """
        with self.assertRaises(ValueError):
            self.fill.alpha = value
    
    @data(0.0, 0.05, 0.15, 0.25, 0.35, 0.5, 0.65, 0.75, 0.9, 0.99, 1.0)
    def test_fill_alpha_setter_valid_values(self, value: float) -> None:
        """
        Verifies accurate updating of the alpha property with valid normalized floats.
        
        This test ensures that:
        - The alpha property accepts all valid normalized floats
        - Boundary conditions (0.0 and 1.0) are handled correctly
        - Type and value integrity are preserved
        
        Args:
            value (float): Valid normalized alpha values to test
        
        Note:
            0.0 = fully transparent, 1.0 = fully opaque.
        """
        self.fill.alpha = value
        self.assertIsInstance(self.fill.alpha, float)
        self.assertAlmostEqual(self.fill.alpha, value)
    
    # =========================================================================
    # ROUND-TRIP FIDELITY
    # =========================================================================
    @data(fi_0, fi_1, fi_2, ..., fi_20)
    def test_fill_roundtrip_fidelity(self, fill: Fill) -> None:
        """
        Verifies perfect round-trip serialization/deserialization fidelity.
        
        This test ensures that:
        - to_dict() followed by from_dict() recreates identical instance
        - Alpha value is preserved exactly (no precision loss)
        - All boundary values survive round-trip
        
        Args:
            fill (Fill): Pre-configured Fill mock instance provided by DDT.
        
        Note:
            Round-trip fidelity is essential for configuration persistence
            and API transmission.
        """
        serialized = fill.to_dict()
        reconstructed = Fill.from_dict(serialized)
        
        self.assertIsInstance(reconstructed, Fill)
        self.assertAlmostEqual(reconstructed.alpha, fill.alpha, places=9)
        
        re_serialized = reconstructed.to_dict()
        self.assertEqual(serialized, re_serialized)
```

### 8. Запуск тестов

```bash
# Запуск всех тестов
pytest tests/

# Запуск тестов конкретного модуля
pytest tests/instance/
pytest tests/inference/
pytest tests/tracker/
pytest tests/render/

# Запуск с подробным выводом
pytest -v tests/render/test_fill.py

# Запуск с покрытием кода
pytest --cov=acmenra_cv tests/
```

</details>
