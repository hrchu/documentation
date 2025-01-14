---
title: Collecte de logs avec PHP
kind: documentation
aliases:
  - /fr/logs/languages/php
further_reading:
  - link: 'https://www.datadoghq.com/blog/php-logging-guide'
    tag: Blog
    text: 'Comment recueillir, personnaliser et analyser des logs PHP'
  - link: /logs/processing
    tag: Documentation
    text: Apprendre à traiter vos logs
  - link: /logs/processing/parsing
    tag: Documentation
    text: En savoir plus sur le parsing
  - link: /logs/explorer
    tag: Documentation
    text: Apprendre à explorer vos logs
  - link: logs/explorer/analytics
    tag: Documentation
    text: Effectuer des analyses de logs
  - link: /logs/faq/log-collection-troubleshooting-guide
    tag: FAQ
    text: Dépannage pour la collecte de logs
---
## Présentation

Enregistrez vos logs PHP dans un fichier, puis [utilisez l'Agent][1] pour les transmettre à Datadog. Vous avez la possibilité d'utiliser l'une des bibliothèques suivantes :

{{< tabs >}}
{{% tab "PHP Monolog" %}}

Utilisez Composer pour ajouter Monolog en tant que dépendance :

```
composer require "monolog/monolog"
```

Vous pouvez également l'installer manuellement :

1. Téléchargez Monolog depuis le référentiel et ajoutez-le aux bibliothèques.
2. Lancez l'instance dans le bootstrap de l'application.

    ```php
    <?php

    require __DIR__ . '/vendor/autoload.php';

    // load Monolog library
    use Monolog\Logger;
    use Monolog\Handler\StreamHandler;
    use Monolog\Formatter\JsonFormatter;    
    ```

## Configurer l'enregistrement de logs dans un fichier avec Monolog

Utilisez la configuration ci-dessous pour activer le format JSON et enregistrer les logs et les événements dans le fichier `application-json.log`. Juste après avoir lancé l'instance Monolog, modifiez votre code afin d'ajouter un nouveau gestionnaire :

```php
 <?php

require __DIR__ . '/vendor/autoload.php';

// Charger la bibliothèque Monolog
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\JsonFormatter;

// Créer un canal pour l'enregistrement des logs
$log = new Logger('nom_canal');

// Créer un formateur JSON
$formatter = new JsonFormatter();

// Créer un gestionnaire
$stream = new StreamHandler(__DIR__.'/application-json.log', Logger::DEBUG);
$stream->setFormatter($formatter);

// Connexion
$log->pushHandler($stream);

// Exemple
$log->info('Ajout d'un nouvel utilisateur', array('username' => 'Seldaek'));
```

## Configurer votre Agent Datadog

Créez un fichier `php.d/conf.yaml` dans votre dossier `conf.d/` avec le contenu suivant :

```yaml
init_config:

instances:

## Section Log
logs:

    ## - type (obligatoire) : type de fichier de la source d'entrée de log (tcp/udp/file).
    ##   port / path (obligatoire) : définit le type tcp ou udp du port. Choisit le chemin si le type est défini sur file.
    ##   service (obligatoire) : nom du service propriétaire du log.
    ##   source (obligatoire) : attribut qui définit l'intégration qui envoie les logs.
    ##   sourcecategory (facultatif) : attribut à valeur multiple. Il peut être utilisé pour préciser l'attribut source.
    ##   tags (facultatif) : ajoute des tags à chaque log recueilli.

  - type: file
    path: /chemin/vers/votre/php/application-json.log
    service: php
    source: php
    sourcecategory: sourcecode
```

## Ajouter des champs de métadonnées et du contexte

L'ajout de données de contexte à vos logs et événements est particulièrement utile.  
Monolog rend cette opération simple en proposant différents moyens de définir des données de contexte propres à chaque thread, qui sont ensuite automatiquement envoyées avec tous les événements. À tout moment, il vous est possible de loguer un événement accompagné de données de contexte :

```php
$logger->info('Ajout d'un nouvel utilisateur', array('username' => 'Seldaek'));
```

Monolog intègre un préprocesseur. Il s'agit d'un rappel simple qui enrichit vos événements en ajoutant les métadonnées de votre choix (ID de la session, ID de la requête, etc.) :

```php
 <?php 

$log->pushProcessor(function ($record) {

    // Enregistrer l'utilisateur actuel
    $user = Acme::getCurrentUser();
    $record['context']['user'] = array(
        'name' => $user->getName(),
        'username' => $user->getUsername(),
        'email' => $user->getEmail(),
    );

    // Ajouter différents tags
    $record['ddtags'] = array('key' => 'value');

    // Ajouter des données de contexte générales
    $record['extra']['key'] = 'value';

    return $record;
});
```

## Intégration aux frameworks

Monolog est intégré aux frameworks suivants :

* [Symfony2, Symfony3][1]
* [PPI][2]
* [Laravel 4/5][3]
* [Silex][4]
* [Lumen][5]
* [CakePHP][6]


Intégrez Monolog à votre framework, puis configurez votre enregistreur :

```php
 <?php
// Vérifier que la bibliothèque Monolog est bien chargée
//use Monolog\Logger;
//use Monolog\Handler\StreamHandler;
//use Monolog\Formatter\JsonFormatter;

// Avec l'instance Monolog suivante
$monolog = ...

///// Configuration du log shipper

$formatter = new JsonFormatter();
$stream = new StreamHandler(__DIR__.'/application-json.log', Logger::DEBUG);
$stream->setFormatter($formatter);

$monolog->pushHandler($stream);
return $r;
```

### Symfony (v2+, v3+)

Dans votre répertoire de configuration `/chemin/vers/répertoire/configuration/`, modifiez les fichiers `config_dev.yml` et `config_prod.yml` afin de configurer la gestion des logs en fonction de vos besoins sur vos environnements de développement et de production.

```yaml
# app/config/config.yml
monolog:

# Retirer la mise en commentaire de cette section si vous souhaitez utiliser un processeur 
#       Processor: 
#           session_processor:
#               class: Acme\Bundle\MonologBundle\Log\SessionRequestProcessor
#            arguments:  [ @session ]
#            tags:
#               - { name: monolog.processor, method: processRecord }


    json_formatter:
        class: Monolog\Formatter\JsonFormatter

    handlers:

        # Configuration du log shipper
        to_json_files:
            # Enregistrer les logs dans var/logs/(environment).log
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            # Inclure tous les canaux (doctrine, erreurs, etc.)
            channels: ~
            # Utiliser le formateur JSON
            formatter: monolog.json_formatter
            # Loguer tous les événements (des données de debugging aux erreurs fatales)
            level: debug
```

### PPI

Dans votre répertoire de configuration `/chemin/vers/répertoire/configuration/`, modifiez les fichiers `config_dev.yml` et `config_prod.yml` afin de configurer la gestion des logs en fonction de vos besoins sur vos environnements de développement et de production.

```yaml
monolog:
    handlers:

        # Configuration du log shipper
        to_json_files:
            # Enregistrer les logs dans var/logs/(environment).log
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            # Utiliser le formateur JSON
            formatter: monolog.json_formatter
            # Loguer tous les événements (des données de debugging aux erreurs fatales)
            level: debug
```

### Laravel

```php
//file: bootstrap/app.php
$app->configureMonologUsing(function($monolog) {
    $monolog->pushHandler(...);

  // Configurer votre enregistreur ci-dessous
});

return $app;
```

### Silex

```php
// file: bootstrap
$app->extend('monolog', function($monolog, $app) {
    $monolog->pushHandler(...);
    // Configurer votre enregistreur ci-dessous
    return $monolog;
});
```

### Lumen

```php
//file: bootstrap/app.php
$app->configureMonologUsing(function($monolog) {
    $monolog->pushHandler(...);
    // Configurer votre enregistreur ci-dessous
});

return $app;
```

### CakePHP

Commencez par ajouter la dépendance suivante au fichier `composer.json`,
puis exécutez `composer update`.

```json
{
    "require": {
        "cakephp/monolog": "*"
    }
}
```

Créez ensuite un fichier de configuration de la journalisation (p. ex., `app/Config/log.php`) et incluez-y votre `app/Config/bootstrap.php` :

```php
include 'log.php';
```

Voici un exemple de configuration basique permettant de reproduire les fonctionnalités de Cake avec Monolog :

```
CakePlugin::load('Monolog');
```

Enfin, enregistrez les logs dans un fichier :

```
CakeLog::config('debug', array(
  'engine' => 'Monolog.Monolog',
  'channel' => 'app',
  'handlers' => array(
    'Stream' => array(
      LOGS . 'application-json.log',
      'formatters' => array(
        'Json' => array("")
      )
    )
  )
));
```

[1]: /fr/logs/log_collection/php/#symfony-v2-v3
[2]: /fr/logs/log_collection/php/#ppi
[3]: /fr/logs/log_collection/php/#laravel
[4]: /fr/logs/log_collection/php/#silex
[5]: /fr/logs/log_collection/php/#lumen
[6]: /fr/logs/log_collection/php/#cakephp
{{% /tab %}}
{{% tab "PHP Symfony" %}}

Cette section couvre les aspects suivants :

* Comment envoyer vos logs à Datadog depuis un environnement Symfony 2
* Comment configurer Monolog pour envoyer des logs au format JSON
* Comment enrichir vos événements de log en y ajoutant des données de contexte propres à chaque session

## Configurer l'enregistrement des logs dans un fichier avec Monolog

1. Déclarez un formateur JSON Monolog en tant que service :

    ```yaml
    services:
        monolog.json_formatter:
            class: Monolog\Formatter\JsonFormatter
    ```

2. Configurez le formateur dans votre configuration Monolog en déclarant le champ formatter comme suit :

    ```yaml
     monolog:
        handlers:
            main:
                type:   stream
                path:   "%kernel.logs_dir%/%kernel.environment%.log"
                level:  debug
                formatter: monolog.json_formatter
    ```

## Configurer votre Agent Datadog

Créez un fichier `php.d/conf.yaml` dans votre dossier `conf.d/` avec le contenu suivant :

```yaml
init_config:

instances:

## Section Log
logs:

    ## - type (obligatoire) : type de fichier de la source d'entrée de log (tcp/udp/file).
    ##   port / path (obligatoire) : définit le type tcp ou udp du port. Choisit le chemin si le type est défini sur file.
    ##   service (obligatoire) : nom du service propriétaire du log.
    ##   source (obligatoire) : attribut qui définit l'intégration qui envoie les logs.
    ##   sourcecategory (facultatif) : attribut à valeur multiple. Il peut être utilisé pour préciser l'attribut source.
    ##   tags (facultatif) : ajoute des tags à chaque log recueilli.

  - type: file
    path: /chemin/vers/votre/php/application-json.log
    service: php
    source: php
    sourcecategory: sourcecode
```

## Ajouter un processeur de session pour inclure des données de contexte variables dans vos logs

1. Implémentez votre processeur de session :
  Voici un exemple de processeur. Ce dernier connaît la session actuelle et enrichit l'entrée de log en y ajoutant des informations utiles telles que les attributs `requestId`, `sessionId`, etc.

    ```php
    <?php

    namespace Acme\Bundle\MonologBundle\Log;

    use Symfony\Component\HttpFoundation\Session\Session;

    class SessionRequestProcessor {
      private $session;
      private $sessionId;
      private $requestId;
      private $_server;
      private $_get;
      private $_post;

      public function __construct(Session $session) {
        $this->session = $session;
      }

      public function processRecord(array $record) {
        if (null === $this->requestId) {
          if ('cli' === php_sapi_name()) {
            $this->sessionId = getmypid();
          } else {
            try {
              $this->session->start();
              $this->sessionId = $this->session->getId();
            } catch (\RuntimeException $e) {
              $this->sessionId = '????????';
            }
          }
          $this->requestId = substr(uniqid(), -8);
          $this->_server = array(
            'http.url' => (@$_SERVER['HTTP_HOST']).'/'.(@$_SERVER['REQUEST_URI']),
            'http.method' => @$_SERVER['REQUEST_METHOD'],
            'http.useragent' => @$_SERVER['HTTP_USER_AGENT'],
            'http.referer' => @$_SERVER['HTTP_REFERER'],
            'http.x_forwarded_for' => @$_SERVER['HTTP_X_FORWARDED_FOR']
          );
          $this->_post = $this->clean($_POST);
          $this->_get = $this->clean($_GET);
        }
        $record['http.request_id'] = $this->requestId;
        $record['http.session_id'] = $this->sessionId;
        $record['http.url'] = $this->_server['http.url'];
        $record['http.method'] = $this->_server['http.method'];
        $record['http.useragent'] = $this->_server['http.useragent'];
        $record['http.referer'] = $this->_server['http.referer'];
        $record['http.x_forwarded_for'] = $this->_server['http.x_forwarded_for'];

        return $record;
      }

      protected function clean($array) {
        $toReturn = array();
        foreach(array_keys($array) as $key) {
          if (false !== strpos($key, 'password')) {
            // Do not add
          } else if (false !== strpos($key, 'csrf_token')) {
            // Do not add
          } else {
            $toReturn[$key] = $array[$key];
          }
        }

        return $toReturn;
      }
    }
    ```

2. Connectez le processeur à Symfony :
  ```yaml
   services:
      monolog.processor.session_request:
          class: Acme\Bundle\MonologBundle\Log\SessionRequestProcessor
          arguments:  [ @session ]
          tags:
              - { name: monolog.processor, method: processRecord }
  ```

3. [Transférez le fichier JSON généré à Datadog.][1]


[1]: /fr/logs/log_collection
{{% /tab %}}
{{% tab "PHP Zend-Log" %}}

Zend-log fait partie du framework Zend. Utilisez [Composer][1] pour ajouter Zend-Log :

```
composer require "zendframework/zend-log"
```

Vous pouvez également l'installer manuellement :

1. Téléchargez la source depuis le référentiel et ajoutez-la aux bibliothèques.
2. Lancez l'instance dans le bootstrap de l'application.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Zend\Log\Logger;
use Zend\Log\Writer\Stream;
use Zend\Log\Formatter\JsonFormatter;
```

## Configurer l'enregistrement des logs dans un fichier avec Zend-log

Utilisez la configuration ci-dessous pour activer le format JSON et enregistrer les logs et les événements dans le fichier `application-json.log`. Juste après avoir lancé l'instance Zend-log, modifiez votre code afin d'ajouter un nouveau gestionnaire.

```php
<?php

use Zend\Log\Logger;
use Zend\Log\Writer\Stream;
use Zend\Log\Formatter\JsonFormatter;


// Créer un enregistreur
$logger = new Logger();

// Créer un service d'écriture
$writer = new Stream('file://' . __DIR__ . '/application-json.log');

// Créer un formateur JSON
$formatter = new JsonFormatter();
$writer->setFormatter($formatter);

// Connexion
$logger->addWriter($writer);
Zend\Log\Logger::registerErrorHandler($logger);
```

[transférez ensuite vos fichiers de log à Datadog.][2]

## Configurer votre Agent Datadog

Créez un fichier `php.d/conf.yaml` dans votre dossier `conf.d/` avec le contenu suivant :

```yaml
init_config:

instances:

## Section Log
logs:

    ## - type (obligatoire) : type de fichier de la source d'entrée de log (tcp/udp/file).
    ##   port / path (obligatoire) : définit le type tcp ou udp du port. Choisit le chemin si le type est défini sur file.
    ##   service (obligatoire) : nom du service propriétaire du log.
    ##   source (obligatoire) : attribut qui définit l'intégration qui envoie les logs.
    ##   sourcecategory (facultatif) : attribut à valeur multiple. Il peut être utilisé pour préciser l'attribut source.
    ##   tags (facultatif) : ajoute des tags à chaque log recueilli.

  - type: file
    path: /chemin/vers/votre/php/application-json.log
    service: php
    source: php
    sourcecategory: sourcecode
```


## Ajouter des champs de métadonnées et du contexte

La majorité des informations utiles proviennent des données de contexte supplémentaire que vous pouvez ajouter à vos logs et événements. Zend-Log rend cette opération simple en proposant différents moyens de définir des données de contexte propres à chaque thread, qui sont ensuite automatiquement envoyées avec chaque événement. À tout moment, il vous est possible de loguer un événement accompagné de données de contexte :

```php
$logger->info('Ajout d'un nouvel utilisateur', array('username' => 'Seldaek'));
```

Plus utile encore, la bibliothèque intègre également un processeur. Les processeurs vous permettent d'ajouter des informations supplémentaires à vos logs de façon automatisée. Ils sont appelés par l'enregistreur avant que l'événement ne soit transmis au service d'écriture ; ils reçoivent alors le tableau des événements, puis renvoient un tableau des événements une fois l'opération terminée.

Exemples de cas d'utilisation :

* Ajout d'informations pour le suivi des exceptions
* Injection de substitutions dans le message
* Injection d'un identifiant de requête (afin de pouvoir inspecter ultérieurement les logs associés à un identifiant spécifique)

Vous pouvez utiliser le code suivant si vous le souhaitez :

```php
$logger->addProcessor(new Zend\Log\Processor\Backtrace());
$logger->addProcessor(new Zend\Log\Processor\PsrPlaceholder());
$logger->addProcessor(new Zend\Log\Processor\ReferenceId());
$logger->addProcessor(new Zend\Log\Processor\RequestId());
```

Pour rédiger votre propre code, [consultez la documentation relative à Zend][3].


[1]: https://getcomposer.org
[2]: /fr/logs/log_collection
[3]: https://docs.zendframework.com/zend-log/processors
{{% /tab %}}
{{< /tabs >}}

## Pour aller plus loin

{{< partial name="whats-next/whats-next.html" >}}




[1]: /fr/logs