# ZCA

http://muthukadan.net/docs/zca.html

## Interfaces

http://muthukadan.net/docs/zca.html#interfaces

Une interfaces spécifie les caractéristiques d'un objet. Ce que l'objet peut faire. Pour voir comment, il faut regarder son implémentation.

```Python
from zope.interface import Interface, implements


class IHost(Interface):

    name = Attribute("""Name of host""")

    def goodmorning(guest):
        """Say good morning to guest"""


class Host(object):

    implements(IHost)

    name = u''

    def goodmorning(self, guest):
        """Say good morning to guest"""
        return "Good morning, %s!" % guest
```

Example qui nous servira pour la suite :

```Python
from zope.interface import Interface

class IDesk(Interface):
    """A frontdesk will register object's details"""

    def register():
        """Register object's details"""


from zope.interface import Interface
from zope.interface import Attribute

class IGuest(Interface):

    name = Attribute("Name of guest")
    place = Attribute("Place of guest")
```

### "Marker interfaces"

Les interfaces "marker" sont un peu speciales. Elles ne spécifient aucun attribut ou méthode et permettent simplement de marquer un objet comme appartenant à une catégorie spécial. Par example :

```Python
from zope.interface import Interface

class ISpecialGuest(Interface):
    """A special guest"""
```

Example avec les browser layers :

https://pypi.python.org/pypi/plone.browserlayer

## Adapters

Un adapter prend un objet implémentant une interface en entrée et fournis un objet implémentant une autre interface en sortie. Example :

```Python
from zope.interface import implements
from zope.component import adapts

class FrontDeskNG(object):

    implements(IDesk)
    adapts(IGuest)

    def __init__(self, guest):
        self.guest = guest

    def register(self):
        guest = self.guest
        next_id = get_next_id()
        bookings_db[next_id] = {
        'name': guest.name,
        'place': guest.place,
        'phone': guest.phone
        }
```

On a ici un adapter pour `IDesk` qui adapte `IGuest`. Une instance de cette adapter nous permettra donc d'avoir un `IDesk` à partir d'un `IGuest` :

```Python
class Guest(object):

    implements(IGuest)

    def __init__(self, name, place):
        self.name = name
        self.place = place
```

```Python
>>> jack = Guest("Jack", "Bangalore")
>>> jack_frontdesk = FrontDeskNG(jack)

>>> IDesk.providedBy(jack_frontdesk)
True
```

### Enregistrements

Les adapters sont enregistrés dans le "global site manager" :

```Python
>>> from zope.component import getGlobalSiteManager
>>> gsm = getGlobalSiteManager()
>>> gsm.registerAdapter(FrontDeskNG,
...                     (IGuest,), IDesk, 'ng')

```

Ou, comme les informations d'entrée et de sorties sont déjà sur l'adapter :

```Python
>>> gsm.registerAdapter(FrontDeskNG, name='ng')
```

En Plone, l'enregistrement de l'adapter ce fera en général en ZCML (http://muthukadan.net/docs/zca.html#zca-usage-in-zope) :

```XML
<adapter
    for=".interfaces.IGuest"
    provides=".interfaces.IDesk"
    factory=".adapter.FrontDeskNG"
    name="ng"
    />
```

`for` et `provides` sont optionnel si ils sont fournis par la class :

```XML
<adapter
    factory=".adapter.FrontDeskNG"
    name="ng"
    />
```

Le nom est une chaine vide ("") par défaut.

### Récuperation

Deux méthodes sont disponible pour récuperer des adapters : `getAdapter` et `queryAdapter`. Elles prennent les même arguments, mais `getAdapter` soulevera une exception tandis que `queryAdapter` retournera None si aucun adapter n'est trouvé.

```Python
>>> from zope.component import getAdapter
>>> from zope.component import queryAdapter

>>> getAdapter(jack, IDesk, 'ng') #doctest: +ELLIPSIS
<FrontDeskNG object at ...>
>>> queryAdapter(jack, IDesk, 'ng') #doctest: +ELLIPSIS
<FrontDeskNG object at ...>
```

Pour les adapter sans nom, il est possible de les récuperer en utilisant l'interface directement :

```Python
>>> gsm.registerAdapter(FrontDeskNG)
>>> IDesk(jack)
<FrontDeskNG object at ...>
```

`IDesk(jack)` => Je veux un objet de type `IDesk` à partir de `jack` (qui est un objet `IGuest`).

### Multi adapter

Un multi adapter est un adapter prenant plusieurs objets en entrée. Le principe et le fonctionnement reste quasiment le même :


```Python
>>> from zope.interface import Interface
>>> from zope.interface import implements
>>> from zope.component import adapts

>>> class IAdapteeOne(Interface):
...     pass

>>> class IAdapteeTwo(Interface):
...     pass

>>> class IFunctionality(Interface):
...     pass

>>> class MyFunctionality(object):
...     implements(IFunctionality)
...     adapts(IAdapteeOne, IAdapteeTwo)
...
...     def __init__(self, one, two):
...         self.one = one
...         self.two = two

>>> from zope.component import getGlobalSiteManager
>>> gsm = getGlobalSiteManager()

>>> gsm.registerAdapter(MyFunctionality)

>>> class One(object):
...     implements(IAdapteeOne)

>>> class Two(object):
...     implements(IAdapteeTwo)

>>> one = One()
>>> two = Two()

>>> from zope.component import getMultiAdapter

>>> getMultiAdapter((one,two), IFunctionality)
<MyFunctionality object at ...>

>>> myfunctionality = getMultiAdapter((one,two), IFunctionality)
>>> myfunctionality.one
<One object at ...>
>>> myfunctionality.two
<Two object at ...>
```

Il est possible de les enregistrer en ZCML :

```XML
<adapter
    for=".IAdapteeOne
         .IAdapteeTwo"
    provides=".IFunctionality"
    factory=".MyFunctionality"
    />
```

## Utility

Un utilitaire est comme un adapter, sauf qu'il ne prend rien en entrée ! On a donc juste un objet implémentant une interface en sortie.

```Python
>>> from zope.interface import Interface
>>> from zope.interface import implements

>>> class IGreeter(Interface):
...
...     def greet(name):
...         """Say hello"""

>>> class Greeter(object):
...
...     implements(IGreeter)
...
...     def greet(self, name):
...         return "Hello " + name

>>> from zope.component import getGlobalSiteManager
>>> gsm = getGlobalSiteManager()

>>> greet = Greeter()
>>> gsm.registerUtility(greet, IGreeter)
```

En ZCML :

```XML
<utility
    component=".database.connection"
    provides=".interfaces.IConnection"
    name="fakedb"
    />
```

Il est également possible d'utiliser une factory :

```XML
<utility
    factory=".database.Connection"
    provides=".interfaces.IConnection"
    name="fakedb"
    />
```

La récupération est similaire à celle des adapters :

```
>>> from zope.component import queryUtility
>>> from zope.component import getUtility

>>> queryUtility(IGreeter).greet('Jack')
'Hello Jack'

>>> getUtility(IGreeter).greet('Jack')
'Hello Jack'
```

Attention à bien enregistrer une instance de l'utilitaire et non sa classe.

Il est possible de nommer un utilitaire :

```Python
>>> greet = Greeter()
>>> gsm.registerUtility(greet, IGreeter, 'new')

>>> from zope.component import queryUtility
>>> from zope.component import getUtility

>>> queryUtility(IGreeter, 'new').greet('Jill')
'Hello Jill'

>>> getUtility(IGreeter, 'new').greet('Jill')
'Hello Jill'
```

### Les factories (Exemple d'utilisation concrete)

Les factories sont un cas particulier d'utilitaires. Ils implementent tous IFactory.

```Python
>>> from zope.interface import Attribute
>>> from zope.interface import Interface
>>> from zope.interface import implements

>>> class IDatabase(Interface):
...
...     def getConnection():
...         """Return connection object"""

>>> class FakeDb(object):
...
...     implements(IDatabase)
...
...     def getConnection(self):
...         return "connection"

>>> from zope.component.factory import Factory

>>> factory = Factory(FakeDb, 'FakeDb')

>>> from zope.component import getGlobalSiteManager
>>> gsm = getGlobalSiteManager()

>>> from zope.component.interfaces import IFactory
>>> gsm.registerUtility(factory, IFactory, 'fakedb')

>>> from zope.component import queryUtility
>>> queryUtility(IFactory, 'fakedb')()
<FakeDb object at ...>
```

Dans le cas des factories, il est également possible de faire :

```Python
>>> from zope.component import createObject
>>> createObject('fakedb')
<FakeDb object at ...>
```
