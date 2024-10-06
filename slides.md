[comment]: # (width: 1600)
[comment]: # (height: 900)
[comment]: # (THEME = white)
[comment]: # (CODE_THEME = routeros)


Enrique Soria

# django-lifecycle

Descubriendo cómo gestionar en Django el ciclo de vida de un modelo, _sin pelos ni señales_

[comment]: # (!!!)

### Nuestro proyecto

Vamos a suponer que estamos creando un blog.

Nuestro blog tiene publicaciones.

[comment]: # (!!! )

### Nuestro proyecto

```python [4-7]
from django.db import models


class Post(models.Model):
    title = models.CharField(max_length=32)
    slug = models.SlugField(max_length=32)
    body = models.TextField()
```

[comment]: # (!!!)

### Objetivo

Setear el _slug_ de nuestro flamante blog automáticamente a partir del título.


[comment]: # (!!!)

### Usando django-lifecycle

Añadimos el mixin de lifecycle a nuestro modelo

```python [3, 6]
from django.db import models
from django.utils.text import slugify
from django_lifecycle import LifecycleModelMixin


class Post(LifecycleModelMixin, models.Model):
    title = models.CharField(max_length=32)
    slug = models.SlugField(max_length=32)
    body = models.TextField()
```

[comment]: # (!!!)

### Usando django-lifecycle

Creamos una función que calcule y setee el slug

```python [6-7]
class Post(LifecycleModelMixin, models.Model):
    title = models.CharField(max_length=32)
    slug = models.SlugField(max_length=32)
    body = models.TextField()

    def set_slug(self):
        self.slug = slugify(self.title)
```


[comment]: # (!!!)

### Usando django-lifecycle

Ahora le indicamos [en qué momentos](https://rsinger86.github.io/django-lifecycle/hooks_and_conditions/#lifecycle-moments) se quieren realizar estas acciones:

```python [3-4]
class Post(LifecycleModelMixin, models.Model):

    @hook(BEFORE_CREATE)
    @hook(BEFORE_UPDATE)
    def set_slug(self):
        self.slug = slugify(self.title)
```

[comment]: # (!!!)

### Usando django-lifecycle

También podemos especificar [en qué condiciones se ejecutan los hooks](https://rsinger86.github.io/django-lifecycle/hooks_and_conditions/#conditions).

```python [3-6]
class Post(LifecycleModelMixin, models.Model):

    @hook(BEFORE_CREATE)
    @hook(
        BEFORE_SAVE, condition=WhenFieldHasChanged("title") | WhenFieldValueIs("slug", value=None)
    )
    def set_slug(self):
        self.slug = slugify(self.title)
```


[comment]: # (!!!)

# Nuevos requerimientos

 - Impedir publicaciones sobre javascript en nuestro blog

[comment]: # (!!!)

## Custom conditions

Si las condiciones que existen no nos convencen podemos crear nuestras propias condiciones:

```python [9-11]
from django_lifecycle.conditions.base import ChainableCondition


class WhenFieldContains(ChainableCondition):
    def __init__(self, field_name, contains_value):
        self.field_name = field_name
        self.contains_value = contains_value
    
    def __call__(self, instance, update_fields=None):
        field_value = getattr(instance, self.field_name)
        return self.contains_value in field_value
```

[comment]: # (!!!)

## Custom conditions

Si las condiciones que existen no nos convencen podemos crear nuestras propias condiciones:

```python [3]
class Post(LifecycleModelMixin, models.Model):
    
    @hook(BEFORE_SAVE, condition=WhenFieldContains("title", "javascript"))   
    def avoid_artificial_intelligence_posts(self):
        raise Exception("¡Los artículos sobre javascript están prohibidos en esta casa!")
```

[comment]: # (!!!)

## Conclusión

La librería django-lifecycle nos permite, de forma declarativa, crear una serie de acciones asociadas al
ciclo de vida de un modelo Django.

https://rsinger86.github.io/django-lifecycle/

![](https://www.codigos-qr.com/qr/php/qr_img.php?d=https%3A%2F%2Fgithub.com%2Frsinger86%2Fdjango-lifecycle&s=8&e=m)


[comment]: # (!!!)

# Enrique Soria

- Github: [@EnriqueSoria](https://github.com/EnriqueSoria)
- Twitter: [@esoria_dev](https://twitter.com/esoria_dev)
- Mastodon: [@esoria_dev@mastodon.social](https://mastodon.social/@esoria_dev)
- [Enrique Soria Magraner](https://www.linkedin.com/in/enriquesoriamagraner/) en Linkedin
