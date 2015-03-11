# Tests

## `plone.testing`

https://pypi.python.org/pypi/plone.testing

Contient la logique utilisé pour faire des tests sous Plone.

## Definitions

https://pypi.python.org/pypi/plone.testing#definitions

## Layers

### Les bases

https://pypi.python.org/pypi/plone.testing#layer-basics

Permet d'initialiser une série de tests. Chaque série de test peut être associé à un layer. Un layer met quatre méthodes à disposition :

* `setUp()` => Initialize le layer, appelée une fois par layer.
* `tearDown()` => Permet de nettoyer ce que le layer a fait, appelée une fois que le layer est fini d'être utilisé.
* `testSetUp()` => Initialise un test, appelée une fois par test.
* `testTearDown()` => Permet de nettoyer ce que le layer a fait, appelée après chaque test.

Un layer peut utiliser d'autres layers comme base, permettant de tous les initialisés. Example avec trois layers, les tests étant dépendant soit de A, soit de B. Ces deux layers ayant C comme base.

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

Le layer doit être instancié avant de pouvoir être utilisé. Son instance se trouve aussi dans le fichier `testing.py` par convention.

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

Afin d'être sur d'avoir toujours les bonne données, il faut penser à les supprimer dans le `tearDown()` ou le `testTearDown()`.

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

### `test_suite`

Pour `zope.testing` inférieur à la 3.8.0, une méthode doit être ajoutée aux fichiers de tests.

```Python
def test_suite():
    return unittest.defaultTestLoader.loadTestsFromName(__name__)
```

### Déclarations disponibles

https://docs.python.org/2/library/unittest.html#test-cases

## `plone.app.testing`

https://pypi.python.org/pypi/plone.app.testing/4.2.4

Contient des layers ainsi que des outils permettant de faciliter les tests Plone.

### Layers

https://pypi.python.org/pypi/plone.app.testing/4.2.4#layer-reference

`plone.app.testing.PLONE_FIXTURE` est le layer de base de `plone.app.testing`. Il permet de créer un site plone de base avec un utilisateur administrateur et un utilisateur de test. Le layer s'assure aussi que les données Zope (ZODB, configuration, ...) soit correctement nettoyées entre chaque tests.

Il ne faut pas utiliser ce layer directement car il ne gère pas (entre autre) les transactions. Deux autres ayant ce layer pour base sont mis à disposition :

* `plone.app.testing.PLONE_INTEGRATION_TESTING`, pour les tests unitaires ou d'intégrations
* `plone.app.testing.PLONE_FUNCTIONAL_TESTING`, pour les tests fonctionnels

Trois ressources sont accessibles sur ces layers : `app`, `portal` et `request`.

Pour créer un nouveau layer, il convient de lui faire utiliser `PLONE_FIXTURE` comme base et d'instancier un layer pour les tests d'intégration et un pour les tests fonctionnels :

```Python
from plone.testing import Layer
from plone.app.testing import PLONE_FIXTURE
from plone.app.testing import IntegrationTesting, FunctionalTesting

class MyFixture(Layer):
    defaultBases = (PLONE_FIXTURE,)

    ...

MY_FIXTURE = MyFixture()

MY_INTEGRATION_TESTING = IntegrationTesting(bases=(MY_FIXTURE,), name="MyFixture:Integration")
MY_FUNCTIONAL_TESTING = FunctionalTesting(bases=(MY_FIXTURE,), name="MyFixture:Functional")
```

### `PloneSandboxLayer`

https://pypi.python.org/pypi/plone.app.testing/4.2.4#layer-base-class

Un layer pour un module Plone doit en fait s'assurer que tout soit initialisé et nettoyé correctement. Il est donc un peu plus complexe qu'un simple layer basique. La classe `plone.app.testing.PloneSandboxLayer` s'occupe de tout cela et permet de simplifier l'écriture d'un layer pour Plone.

Cette classe met à disposition les méthodes suivantes :

* setUpZope(self, app, configurationContext)
* setUpPloneSite(self, portal)
* tearDownZope(self, app)
* tearDownPloneSite(self, portal)

```Python
from plone.app.testing import PloneSandboxLayer
from plone.app.testing import PLONE_FIXTURE
from plone.app.testing import IntegrationTesting

from plone.testing import z2

class MyProduct(PloneSandboxLayer):

    defaultBases = (PLONE_FIXTURE,)

    def setUpZope(self, app, configurationContext):
        # Load ZCML
        import my.product
        self.loadZCML(package=my.product)

        # Install product and call its initialize() function
        z2.installProduct(app, 'my.product')

        # Note: you can skip this if my.product is not a Zope 2-style
        # product, i.e. it is not in the Products.* namespace and it
        # does not have a <five:registerPackage /> directive in its
        # configure.zcml.

    def setUpPloneSite(self, portal):
        # Install into Plone site using portal_setup
        self.applyProfile(portal, 'my.product:default')

    def tearDownZope(self, app):
        # Uninstall product
        z2.uninstallProduct(app, 'my.product')

        # Note: Again, you can skip this if my.product is not a Zope 2-
        # style product

MY_PRODUCT_FIXTURE = MyProduct()
MY_PRODUCT_INTEGRATION_TESTING = IntegrationTesting(bases=(MY_PRODUCT_FIXTURE,), name="MyProduct:Integration")
MY_PRODUCT_FUNCTIONAL_TESTING = FunctionalTesting(bases=(MY_PRODUCT_FIXTURE,), name="MyProduct:Functional")
```

L'écriture des tests reste identique :

```Python
import unittest
from my.product.testing import MY_PRODUCT_INTEGRATION_TESTING

class IntegrationTest(unittest.TestCase):

    layer = MY_PRODUCT_INTEGRATION_TESTING

    def test_page_dublin_core_title(self):
        portal = self.layer['portal']

        page1 = portal['page-1']
        page1.title = u"Some title"

        self.assertEqual(page1.Title(), u"Some title")
```

### Helpers

https://pypi.python.org/pypi/plone.app.testing/4.2.4#helper-functions

Un certain nombre de méthodes sont disponibles pour faciliter les tests, entre autres :

* `ploneSite`, pour récuperer le site plone depuis un layer.
* `login`, `logout` et `setRoles` pour la gestion des utilisateurs.

## Installation et utilisation

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

## Tests fonctionnels

https://pypi.python.org/pypi/plone.app.testing/4.2.4#simulating-browser-interaction

`zope.testbrowser` peut être utilisé pour simuler des interactions de l'utilisateurs.

```Python
import unittest
from plone.testing.z2 import Browser
from plone.app.testing import TEST_USER_NAME, TEST_USER_PASSWORD
from my.product.testing import MY_PRODUCT_FUNCTIONAL_TESTING

class FunctionalTest(unittest.TestCase):

    layer = MY_PRODUCT_FUNCTIONAL_TESTING

    def test_home_welcome(self):
        portal = self.layer['portal']
        app = self.layer['app']

        browser = Browser(app)
        browser.open(portal.absolute_url())

        self.assertTrue(u"Welcome" in browser.contents)

    def test_login(self):
        portal = self.layer['portal']
        app = self.layer['app']
        browser = Browser(app)

        browser.open(portal.absolute_url() + '/login_form')
        browser.getControl(name='__ac_name').value = TEST_USER_NAME
        browser.getControl(name='__ac_password').value = TEST_USER_PASSWORD
        browser.getControl(name='submit').click()

        self.assertTrue(TEST_USER_NAME in browser.contents)
```

Plus d'informations sur l'utilisation du browser sur la documentation de `zope.testbrowser` :

https://pypi.python.org/pypi/zope.testbrowser#page-contents

## Autres notions

### Doctest

Permet la création de tests dans les doctstrings Python.

https://pypi.python.org/pypi/plone.testing#doctests

### Coverage

Permet de determiner quelle partie du module est testé ou non.

https://pypi.python.org/pypi/plone.testing#coverage-reporting

### Mock

Permet de créer de faux objets/méthodes.

https://pypi.python.org/pypi/plone.testing#mock-requests
https://pypi.python.org/pypi/mock

### Selenium

Autre bibliothèque gérant les tests fonctionnels en utilisant directement le navigateur.

https://github.com/plone/plone.app.testing/blob/master/plone/app/testing/selenium.rst
https://selenium-python.readthedocs.org
