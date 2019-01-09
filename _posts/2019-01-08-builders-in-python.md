**Abs**: In this post I would like to present a couple of solutions for builder pattern in python.

Builder pattern comes in handy when we need to set a lot of fields of an object and we do not want to use many methods with multiple parameters.

All implementations are found in this repo: [https://github.com/torokmark/builders_in_python](https://github.com/torokmark/builders_in_python)

## The Simple One


**Task**: Let us implement a `Person` with the following fields like `name`, `age`, `phone`.   
**Approach**: Make a `PersonBuilder` with methods according to the fields in `Person`, plus a `build` which returns an object of `Person`.


```python
class Person:
    def __init__(self, name, age, phone):
        self._name = name
        self._age = age
        self._phone = phone

    def name(self):
        return self._name

    def age(self):
        return self._age
    
    def phone(self):
        return self._phone

    def __str__(self):
        return '[name={}; age={}; phone={}]'.format(self._name, self._age, self._phone)
```

```python
class PersonBuilder:
    def __init__(self):
        self._name = 'John Doe'
        self._age = 99
        self._phone = '000'

    def name(self, name):
        self._name = name
        return self

    def age(self, age):
        self._age = age
        return self

    def phone(self, phone):
        self._phone = phone
        return self

    def build(self):
        return Person(self._name, self._age, self._phone)
```

```python
if __name__ == '__main__':
    print(PersonBuilder()
        .name('jancsi')
        .age(12)
        .phone('11223344')
        .age(18)
        .build())            # => [name=jancsi; age=18; phone=11223344]
    print(PersonBuilder()
        .age(12)
        .phone('11223344')
        .age(18)
        .build())            # => [name=John Doe; age=18; phone=11223344]
    print(PersonBuilder()
        .build())            # => [name=John Doe; age=99; phone=000]
```


## The One with Named Parameters


Though builder is very flexible, it is inevitable to have some default values for those fields that are not set during the building.


**Task**: How to do this without builder?   
**Approach**: Let us use default values in constructor parameter list.


```python
class Person:
    def __init__(self, name='John Doe', age=99, phone='000'):
        self._name = name
        self._age = age
        self._phone = phone

    def name(self):
        return self._name

    def age(self):
        return self._age
    
    def phone(self):
        return self._phone

    def __str__(self):
        return '[name={}; age={}; phone={}]'.format(self._name, self._age, self._phone)
```

```python
if __name__ == '__main__':
    print(Person())                        # => [name=John Doe; age=99; phone=000]
    print(Person(phone='123456', age=45))  # => [name=John Doe; age=45; phone=123456]
    print(Person('jancsi', 54, '312333'))  # => [name=jancsi; age=54; phone=312333]
```

## The Classic

**Task**: Let us implement a classic car builder.   
**Solution**: Take the example from *Gang of Four*!

```python
class Car:
    def __init__(self, wheel=4, seat=4, color='red'):
        self.wheel = wheel
        self.seat = seat
        self.color = color

    def __str__(self):
        return '[wheels: {}, seats: {}, color: {}]'.format(self.wheel, self.seat, self.color)
```

```python
class Builder:
    def set_wheels(self, wheel): pass
    def set_seats(self, seat): pass
    def set_color(self, color): pass
```

```python
class CarBuilder(Builder):

    def __init__(self):
        self.car = Car()

    def set_wheels(self, wheel):
        self.car.wheel = wheel

    def set_seats(self, seat):
        self.car.seat = seat

    def set_color(self, color):
        self.car.color = color

    def get_result(self):
        return self.car
```

```python
class CarBuilderDirector:
    def build(self):
        builder = CarBuilder()
        builder.set_wheels(8)
        builder.set_seats(4)
        builder.set_color("Red")
        return builder.get_result()
```

```python
if __name__ == '__main__':
    car = CarBuilderDirector().build()
    print(car)      # => [wheels: 8, seats: 4, color: Red]
```

## The Builder with Builders

**Task**: Implement a *Response object* builder. It has a status code, a header and a body part. Create a builder that can build multiple part of an object.   
**Approach**: Let us implement so called subbuilders and pass the created objects to an outer builder, which combines them together.

> Using just `HeaderBuilder#add(key, value)` instead of using header specific setters.

```python
class Builder:
    def build(self): pass
```

```python
class HeaderBuilder(Builder):
    def __init__(self):
        self._header = {}

    def add(self, key, value):
        self._header[key] = value
        return self

    def build(self):
        return self._header
```

```python
class BodyBuilder(Builder):
    def __init__(self):
        self._body = []

    def add(self, content):
        self._body.append(content)
        return self

    def build(self):
        return self._body
```

```python
class ResponseBuilder(Builder):
    def __init__(self):
        self._header = None
        self._body = None
        self._status = None

    def header(self, header):
        self._header = header
        return self

    def body(self, body):
        self._body = body
        return self

    def status(self, status):
        self._status = status
        return self

    def build(self):
        ret = {}
        ret['statusCode'] = self._status
        ret['header'] = self._header
        ret['body'] = self._body
        return ret

```

```python
if __name__ == '__main__':
    print(ResponseBuilder()
            .header(
                HeaderBuilder()
                    .add('Accept', '*')
                    .add('Age', 12)
                    .build()
            )
            .body(
                BodyBuilder()
                    .add('some message comes here')
                    .add('another message here...')
                    .build()
            )
            .status(200)
            .build()
    )                   # => {'statusCode': 200, 'header': {'Accept': '*', 'Age': 12}, 'body': ['some message comes here', 'another message here...']}
```


## The Multibuilder

**Task**: Create a builder that can build multiple part of an object at one time whitout using so called subbuilders.   
**Approach**: Let us chain everything up and memorize the already created objects.

```python
class HeaderBuilder:
    def __init__(self):
        self._header = {}

    def add(self, key, value):
        self._header[key] = value
        return self

    def body(self):
        return self._body

    def build(self):
        return self.status(200)

    def status(self, code):
        ret = {}
        ret['status'] = code
        ret['header'] = self._header
        ret['body'] = self._body.__dict__.get('_body')
        return ret

    def set_body(self, body):
        self._body = body
```

```python
class BodyBuilder:
    def __init__(self):
        self._body = [] 

    def add(self, value):
        self._body.append(value)
        return self

    def header(self):
        return self._header

    def build(self):
        return self.status(200)

    def status(self, code):
        ret = {}
        ret['status'] = code
        ret['header'] = self._header.__dict__.get('_header')
        ret['body'] = self._body
        return ret

    def set_header(self, header):
        self._header = header
```

```python
class ResponseBuilder:

    def __init__(self):
        self.__header = HeaderBuilder()
        self.__body = BodyBuilder()
        self.__body.set_header(self.__header)
        self.__header.set_body(self.__body)

    def header(self):
        return self.__header

    def body(self):
        return self.__body
```

```python
if __name__ == "__main__":
    print(ResponseBuilder()
            .header()
                .add('Access-Control-Allow-Origin', '*')
                .add('Accept', '*')
            .body()
                .add('body', json.dumps('hello', default=str))
            .status(204))

    print(ResponseBuilder()
            .header()
                .add('Age', '12')
                .add('Accept', '*')
            .body()
                .add('body', 'message comes here!')
            .header()
                .add('Access-Control-Allow-Origin', '*')
            .build()) # status is default to 200
```

All these approaches are found in [https://github.com/torokmark/builders_in_python](https://github.com/torokmark/builders_in_python)
