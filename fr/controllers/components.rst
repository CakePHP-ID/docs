Components (Composants)
#######################

Les Components (Composants) sont des regroupements de logique applicative
qui sont partagés entre les controllers. Si vous vous surprenez à vouloir
copier et coller des choses entre vos controllers, alors vous devriez envisager
de regrouper plusieurs fonctionnalités dans un Component.

CakePHP est également livré avec un fantastique ensemble de components,
que vous pouvez utiliser pour vous aider :

- Sécurité
- Sessions
- Listes de contrôle d'accès (ACL)
- Emails
- Cookies
- Authentification
- Traitement de requêtes
- Pagination

Chacun de ces components d’origine est détaillé dans des chapitres
spécifiques. Pour l’heure, nous allons vous montrer comment créer
vos propres components. La création de components vous permet de garder
le code de vos controllers propres et vous permet de réutiliser
du code entre vos projets.

.. _configuring-components:

Configuration des Components
============================

De nombreux components du cœur nécessitent une configuration. Quelques
exemples :
:doc:`/core-libraries/components/authentication`,
:doc:`/core-libraries/components/cookie`
et :doc:`/core-libraries/components/email`.
Toute configuration pour ces components, et pour les components en général,
se fait dans le tableau des ``$components`` de la méthode ``beforeFilter()``
de vos controllers::

    class PostsController extends AppController {
        public $components = array(
            'Auth' => array(
                'authorize' => array('controller'),
                'loginAction' => array('controller' => 'users', 'action' => 'login')
            ),
            'Cookie' => array('name' => 'CookieMonster')
        );

Serait un exemple de configuration d'un component avec le tableau
``$components``. Tous les components du coeur permettent aux paramètres
d'être configurés dans la méthode de votre controller ``beforeFilter()``.
C'est utile quand vous avez besoin d'assigner les résultats d'une fonction
à la propriété d'un component. Ceci peut aussi être exprimé comme ceci::

    public function beforeFilter() {
        $this->Auth->authorize = array('controller');
        $this->Auth->loginAction = array('controller' => 'users', 'action' => 'login');

        $this->Cookie->name = 'CookieMonster';
    }

C'est possible, cependant, que le component nécessite certaines options de
configuration avant que le controller ``beforeFilter()`` soit lancé.
Pour cela, certains components permettent aux options de configuration
d'être définies dans le tableau ``$components``::

    public $components = array(
        'DebugKit.Toolbar' => array('panels' => array('history', 'session'))
    );

Consultez la documentation appropriée pour connaître les options de
configuration que chaque component fournit.

Un paramètre commun à utiliser est l'option ``className``, qui vous autorise
les alias des components. Cette fonctionnalité est utile quand vous voulez
remplacer ``$this->Auth`` ou une autre référence de Component commun avec
une implémentation sur mesure::

    // app/Controller/PostsController.php
    class PostsController extends AppController {
        public $components = array(
            'Auth' => array(
                'className' => 'MyAuth'
            )
        );
    }

    // app/Controller/Component/MyAuthComponent.php
    App::uses('AuthComponent', 'Controller/Component');
    class MyAuthComponent extends AuthComponent {
        // Ajouter votre code pour surcharger le AuthComponent du coeur
    }

Ce qu'il y a au-dessous donnerait un *alias* ``MyAuthComponent`` à
``$this->Auth`` dans vos controllers.

.. note::

    Faire un alias à un component remplace cette instance n'importe où où le
    component est utilisé, en incluant l'intérieur des autres Components.

Utiliser les Components
=======================

Une fois que vous avez inclu quelques components dans votre controller,
les utiliser est très simple. Chaque component que vous utilisez est enregistré
comme propriété dans votre controller. Si vous avez chargé la
:php:class:`SessionComponent` et le :php:class:`CookieComponent` dans votre
controller, vous pouvez y accéder comme ceci::

    class PostsController extends AppController {
        public $components = array('Session', 'Cookie');
        
        public function delete() {
            if ($this->Post->delete($this->request->data('Post.id')) {
                $this->Session->setFlash('Post deleted.');
                return $this->redirect(array('action' => 'index'));
            }
        }

.. note::

    Puisque les Models et les Components sont tous deux ajoutés aux
    controllers en tant que propriété, ils partagent le même 'espace de noms'.
    Assurez vous de ne pas donner le même nom à un component et à un model.

Charger les components à la volée
---------------------------------

Vous n'avez parfois pas besoin de rendre le component accessible sur chaque
action. Dans ce cas là, vous pouvez charger à la volée en utilisant la
:doc:`Component Collection </core-libraries/collections>`. A partir de
l'intérieur d'un controller, vous pouvez faire comme ce qui suit::
    
    $this->OneTimer = $this->Components->load('OneTimer');
    $this->OneTimer->getTime();

.. note::

    Gardez à l'esprit que le chargement d'un component à la volée ne va pas
    appeler la méthode initialize. Si le component que vous appelez a cette
    méthode, vous devrez l'appeler manuellement après le chargement.

Callbacks des Components
========================

Les components vous offrent aussi quelques callbacks durant leur cycle de vie
qui vous permettent d'augmenter le cycle de la requête. Allez voir l'api
:ref:`component-api` pour plus d'informations sur les callbacks possibles
des components.

Créer un Component
==================

Supposons que notre application en ligne ait besoin de réaliser une opération
mathématique complexe dans plusieurs sections différentes de l'application.
Nous pourrions créer un component pour héberger cette logique partagée afin
de l'utiliser dans plusieurs controllers différents.

La première étape consiste à créer un nouveau fichier et une classe pour
le component. Créez le fichier dans
``/app/Controller/Component/MathComponent.php``. La structure de base pour
le component ressemblerait à quelque chose comme ça ::

    class MathComponent extends Component {
        public function faireDesOperationsComplexes($montant1, $montant2) {
            return $montant1 + $montant2;
        }
    }

.. note::

    Tous les components comme Math doivent étendre :php:class:`Component`.
    Ne pas le faire vous enverra une exception.

Inclure votre component dans vos controllers
--------------------------------------------

Une fois notre component terminé, nous pouvons l’utiliser au sein
des controllers de l’application en plaçant son nom
(sans la partie "Component") dans le tableau ``$components`` du controller.
Le controller sera automatiquement pourvu d'un nouvel attribut nommé
d'après le component, à travers lequel nous pouvons accéder à une instance
de celui-ci::

    /* Rend le nouveau component disponible par $this->Math
    ainsi que le component standard $this->Session */
    public $components = array('Math', 'Session');

Les Components déclarés dans ``AppController`` seront fusionnés avec ceux
déclarés dans vos autres controllers. Donc il n'y a pas besoin de re-déclarer
le même component deux fois.

Quand vous incluez des Components dans un Controller, vous pouvez
aussi déclarer un ensemble de paramètres qui seront passés à la
méthode initialize() du Component. Ces paramètres peuvent alors être
pris en charge par le Component::

    public $components = array(
        'Math' => array(
            'precision' => 2,
            'generateurAleatoire' => 'srand'
        ),
        'Session', 'Auth'
    );

L'exemple ci-dessus passerait le tableau contenant "precision"
et "generateurAleatoire" comme second paramètre au
``MathComponent::__construct()``. Par convention, tout paramètre passé
qui est aussi une propriété publique sur votre component aura
la valeur basée sur ces paramètres.

Utiliser d'autres Components dans votre Component
-------------------------------------------------

Parfois un de vos components a besoin d'utiliser un autre component.
Dans ce cas, vous pouvez inclure d'autres components dans votre component
exactement de la même manière que dans vos controllers - en utilisant la
variable ``$components``::

    // app/Controller/Component/CustomComponent.php
    class CustomComponent extends Component {
        // l'autre component que votre component utilise
        public $components = array('Existing'); 

        public function initialize($controller) {
            $this->Existing->foo();
        }

        public function bar() {
            // ...
       }
    }

    // app/Controller/Component/ExistingComponent.php
    class ExistingComponent extends Component {

        public function initialize($controller) {
            $this->Parent->bar();
        }

        public function foo() {
            // ...
        }
    }

Notez qu'au contraire d'un component inclu dans un controller, aucun callback
ne sera attrapé pour un component inclu dans un component.

.. _component-api:

API de Component
================

.. php:class:: Component

    La classe de base de Component vous offre quelques méthodes pour le
    chargement facile des autres Components à travers
    :php:class:`ComponentCollection` comme nous l'avons traité avec la gestion
    habituelle des paramètres. Elle fournit aussi des prototypes pour tous
    les callbacks des components.

.. php:method:: __construct(ComponentCollection $collection, $settings = array())

    Les Constructeurs pour la classe de base du component. Tous les
    ``$parametres`` qui sont aussi des propriétés publiques, vont avoir leurs
    valeurs changées pour matcher avec les valeurs de ``$settings``.

Les Callbacks
-------------

.. php:method:: initialize(Controller $controller)

    La méthode initialize est appelée avant la méthode du controller
    beforeFilter.

.. php:method:: startup(Controller $controller)

    La méthode startup est appelée après la méthode du controller
    beforeFilter mais avant que le controller n'exécute l'action prévue.

.. php:method:: beforeRender(Controller $controller)

    La méthode beforeRender est appelée après que le controller exécute la
    logique de l'action requêtée, mais avant le rendu de la vue et le
    layout du controller.

.. php:method:: shutdown(Controller $controller)

    La méthode shutdown est appelée avant que la sortie soit envoyée au
    navigateur.

.. php:method:: beforeRedirect(Controller $controller, $url, $status=null, $exit=true)

    La méthode beforeRedirect est invoquée quand la méthode de redirection
    du controller est appelée, mais avant toute action qui suit. Si cette
    méthode retourne false, le controller ne continuera pas de rediriger la
    requête. Les variables $url, $status et $exit ont la même signification
    que pour la méthode du controller. Vous pouvez aussi retourner une chaîne
    de caractère qui sera interpretée comme une url pour rediriger ou retourner
    un array associatif avec la clé 'url' et éventuellement 'status' et 'exit'.


.. meta::
    :title lang=fr: Components (Composants)
    :keywords lang=fr: tableau controller,librairies du coeur,authentification requêtes,tableau de nom,Liste contrôle accès,public components,controller code,components du coeur,cookiemonster,cookie de connexion,paramètres de configuration,fonctionalité,logic,sessions,cakephp,doc
