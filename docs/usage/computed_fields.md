## Field with computed value based on other fields

Computed fields allow `property` and `cached_property` to be included when serializing models or dataclasses.

```py
from pydantic import BaseModel, computed_field


class Rectangle(BaseModel):
    width: int
    length: int

    @computed_field
    @property
    def area(self) -> int:
        return self.width * self.length


print(Rectangle(width=3, length=2).model_dump())
#> {'width': 3, 'length': 2, 'area': 6}
```

If the `computed_field` decorator is applied to a bare function
(e.g. a function with the `@property` or `@cached_property` decorator)
it will wrap the function in `property` itself. Although this is more concise, you will lose IntelliSense in your IDE,
and confuse static type checkers, thus explicit use of `@property` is recommended.

!!! warning "Mypy Warning"
    Even with the `@property` or `@cached_property` applied to your function before `@computed_field`,
    mypy won't be happy because of [this issue](https://github.com/python/mypy/issues/1362),
    you'll need to add `# type: ignore[misc]` to the `@computed_field` line.

In contract, [pyright](https://github.com/microsoft/pyright) works nicely with `@computed_field`.

```py requires="3.8"
import random
from functools import cached_property

from pydantic import BaseModel, computed_field


class Square(BaseModel):
    width: float

    @computed_field
    def area(self) -> float:  # converted to a `property` by `computed_field`
        return round(self.width**2, 2)

    @area.setter
    def area(self, new_area: float) -> None:
        self.width = new_area**0.5

    @computed_field(alias='the magic number', repr=False)
    @cached_property
    def random_number(self) -> int:
        return random.randint(0, 1_000)


square = Square(width=1.3)

# `random_number` does not appear in representation
print(repr(square))
#> Square(width=1.3, area=1.69)

print(square.random_number)
#> 3

# same random number as before (cached as expected)
print(square.model_dump())
#> {'width': 1.3, 'area': 1.69, 'random_number': 3}

square.area = 4

print(square.model_dump_json(by_alias=True))
#> {"width":2.0,"area":4.0,"the magic number":3}
```
