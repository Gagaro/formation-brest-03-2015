# Tests

* https://pypi.python.org/pypi/plone.testing
* https://pypi.python.org/pypi/plone.app.testing/4.2.4

## Definitions

https://pypi.python.org/pypi/plone.testing#definitions

## Layers


### Les bases

https://pypi.python.org/pypi/plone.testing#layer-basics

Permet d'initialiser une série de tests. Chaque série de test peut être associé à un layer. Un layer met quatre méthodes à disposition :

* `setUp()` => Initialize le layer, appelée une fois par layer.
* `tearDown()` => Permet de nettoyer ce que le layer à fait, appelée une fois que le layer est fini d'être utilisé.
* `testSetUp()` => Initialize un test, appelée une fois par test.
* `testTearDown()` => Permet de nettoyer ce que le layer à fait, appelée après chaque test.

Un layer peut dépendre d'autres layers, permettant de tous les initialisé. Example avec trois layers, les tests étant dépendant soit de A, soit de B. Chacun de ses layers étant dépendant de C.

```
1. C.setUp()
1.1. A.setUp()

1.1.1. C.testSetUp()
1.1.2. A.testSetUp()
1.1.3. [One test using layer A]
1.1.4. A.testTearDown()
1.1.5. C.testTearDown()

1.1.6. C.testSetUp()
1.1.7. A.testSetUp()
1.1.8. [Another test using layer A]
1.1.9. A.testTearDown()
1.1.10. C.testTearDown()

1.2. A.tearDown()
1.3. B.setUp()

1.3.1. C.testSetUp()
1.3.2. B.testSetUp()
1.3.3. [One test using layer B]
1.3.4. B.testTearDown()
1.3.5. C.testTearDown()

1.3.6. C.testSetUp()
1.3.7. B.testSetUp()
1.3.8. [Another test using layer B]
1.3.9. B.testTearDown()
1.3.10. C.testTearDown()

1.4. B.tearDown()
2. C.tearDown()
```

### Écrire un layer 

https://pypi.python.org/pypi/plone.testing#writing-layers

Les layers sont créés dans un fichier `testing.py` par convention. Un layer est simplement un objet python :

```Python
from plone.testing import Layer
class SpaceShip(Layer):

    def setUp(self):
        print "Assembling space ship"

    def tearDown(self):
        print "Disasembling space ship"

    def testSetUp(self):
        print "Fuelling space ship in preparation for test"

    def testTearDown(self):
        print "Emptying the fuel tank"
```

Le layer doit être instancié avant de pouvoir être utilisé. Son instance se trouve aussi dans le `testing.py` par convention.

```Python
SPACE_SHIP = SpaceShip()
```

Un layer utilisant d'autre layers :

```Python
class ZIGSpaceShip(Layer):
     defaultBases = (SPACE_SHIP,)

    def setUp(self):
        print "Installing main canon"
ZIG = ZIGSpaceShip()
```

Il est possible de changer les bases d'un layer lors de son instanciation :

```Python
class CATSMessage(Layer):

    def setUp(self):
        print "All your base are belong to us"

    def tearDown(self):
        print "For great justice"
CATS_MESSAGE = CATSMessage()
ZERO_WING = ZIGSpaceShip(bases=(SPACE_SHIP, CATS_MESSAGE,), name="ZIGSpaceShip:CATSMessage")
```

Il est également possible de combiner des layers :

```Python
COMBI_LAYER = Layer(bases=(CATS_MESSAGE, SPACE_SHIP,), name="Combi")
```

Un layer peut stocker des objets et etre utilisé comme un dictionnaire :

```Python
class WarpDrive(object):
    """A shared resource"""

    def __init__(self, maxSpeed):
        self.maxSpeed = maxSpeed
        self.running = False

    def start(self, speed):
        if speed > self.maxSpeed:
            print "We need more power!"
        else:
            print "Going to warp at speed", speed
            self.running = True

    def stop(self):
        self.running = False

class ConstitutionClassSpaceShip(Layer):
    defaultBases = (SPACE_SHIP,)

    def setUp(self):
        self['warpDrive'] = WarpDrive(8.0)

    def tearDown(self):
        del self['warpDrive']
```

Afin d'être sur d'avoir toujours les bonne données, il faut penser à les supprimer dans le `tearDown()` ou le `testTerarDown()`.

Les instances de l'application Zope et du site Plone seront par example stocké de cette manière.

## Tests

https://pypi.python.org/pypi/plone.testing#writing-tests

Les tests devront être soit écrits dans un fichier `tests.py`, soit être écrit dans des fichiers `test_*.py` dans un dossier `tests` (ne pas oublier le `__init__.py` dans ce dernier cas).

Le module `unittest` inclus dans Python est utilisé pour écrire les tests. `unittest2` est requis pour les version de Python inférieures à la 2.7.

Chaque série de tests est en fait une classe héritant de `unittest.TestCase`. Chaque test est une méthode commencant par `test_`. Deux méthodes sont disponible pour initialiser les tests : `setUp()`  et `tearDown()`.

(Note : les méthodes `setUp()`  et `tearDown()` de `TestCase` sont l'équivalent `testSetUp()` et `testTearDown()` du layer.)

```Python
try:
    import unittest2 as unittest
except ImportError: # Python 2.7
    import unittest
from my.product.testing import GALAXY_CLASS_SPACE_SHIP
	
class TestFasterThanLightTravel(unittest.TestCase):
    layer = GALAXY_CLASS_SPACE_SHIP  # Instance du layer !

    def setUp(self):
        self.warpDrive = self.layer['warpDrive']
        self.warpDrive.stop()
		
    def tearDown(self):
        self.warpDrive.stop()

    def test_warp8(self):
        self.warpDrive.start(8)
        self.assertEqual(self.warpDrive.running, True)

    def test_max_speed(self):
        tooFast = self.warpDrive.maxSpeed + 0.1
        self.warpDrive.start(tooFast)
        self.assertEqual(self.warpDrive.running, False)
```

### test_suite

Pour `zope.testing` inférieur à la 3.8.0, une méthode doit être ajoutée au fichiers de tests.

```Python
def test_suite():
    return unittest.defaultTestLoader.loadTestsFromName(__name__)
```

### Déclarations disponibles

https://docs.python.org/2/library/unittest.html#test-cases

## `plone.app.testing`

### Layers

TODO

### Installation et utilisation

Dans le `setup.py` :
```Python
extras_require = {
    'test': [
            'plone.app.testing',
        ]
},
```

Dans le buildout :

```ini
[test]
recipe = zc.recipe.testrunner
eggs =
    my.package [test]
defaults = ['--auto-color', '--auto-progress']
```

Ne pas oublier les dépendances dans les eggs. Ne pas oublier non plus de rajouter cette section dans le buildout :

```ini
[buildout]
parts =
    ...
    test
```

Puis faire tourner les tests en utilisant le binaire généré :

```shell
$ bin/test
$ bin/test -s my.package
$ bin/test --help
```

## Autres notions

### Doctest

https://pypi.python.org/pypi/plone.testing#doctests

### Coverage

https://pypi.python.org/pypi/plone.testing#coverage-reporting

### Mock

https://pypi.python.org/pypi/plone.testing#mock-requests