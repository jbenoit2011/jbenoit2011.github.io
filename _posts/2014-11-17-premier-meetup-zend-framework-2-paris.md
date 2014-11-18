---
layout: post
title:  "Premier Meetup Paris Zend Framework"
date:   2014-11-17
tags: ZF2
---

__Si vous êtes développeur ZF1/ZF2/Apigility, venez [assister aux meetups](http://www.meetup.com/Paris-Zend-Framework-Meetup/) avec nous !__

Afin de fédérer une communauté de développeurs passionnés autour de Zend Framework, le groupe "Paris Zend Framework Meetup" a été créé. Il organisait ce lundi 17 novembre, son premier meetup. En tant que développeur Zend Framework 1 et 2, je me suis rendu à cet événement. Il s'est déroulé dans les locaux d'[Inovia](https://twitter.com/inoviateam) qui nous a accueilli chaleureusement.

Au programme de cette rencontre, deux sujets. Le premier, l'évolution de ZF2. Et le second, une technique plutôt pratique pour débugger vos applis ZF2 rapidement. Allez je vous raconte tout ça !

# Avenir de ZF2

Présenté par [Sophie Beaupuis](https://twitter.com/so_php_ie).

__Disclaimer:__ Les infos données ci-dessous n'ont pas été encore officialisées. Même s’il y a de fortes chances qu'elles se réalisent. Il est possible que l'avenir n'en soit pas exactement ainsi. Une annonce devrait être faite dans les semaines à venir par Zend.

## ZF3
Au risque de décevoir (beaucoup) de gens, il n'y aura pas de ZF3 pour tout de suite ! Tout le monde sait que Symfony 3.0 devrait être publié en novembre 2015, cependant ZF ne subira pas le même saut de versions. Il y aura plutôt un ZF2++.

## Un ZF2 quoi ?
Vous avez bien entendu, un ZF2++. Une version à mi-chemin entre le changement de version majeur et mineur. L'explication est très simple. Beaucoup d'applications sont encore sous ZF1 et sortir ZF3 aurait pour effet de ralentir la migration. Qui plus est ZF2 a du potentiel en l'état et peut encore évoluer sans qu'une refonte soit nécessaire. Toutefois, Zend pense à ZF3 et sa représentante a insisté sur le fait que les erreurs commises lors de la sortie de ZF2 (changement de paradigme ...) ne se reproduiront pas lors de la sortie de ZF3. La transition sera "douce" pour reprendre ses mots.

## ZF2++
Le ZF2 actuel (version 2.3 à l'heure où j'écris ces lignes) embarque de nombreux composants créés spécifiquement pour lui. Bien que le framework favorise grandement le découplage, le processus actuel ne permet pas de faire évoluer de manière indépendante l'un de ses composants. Par exemple, si la brique Zend Log est mise à jour cela induit une évolution complète du framework, d'un point de vue versioning. Par conséquent, les utilisateurs devront attendre la prochaine release de ZF2 pour utiliser cette nouvelle mouture de Zend Log.

Il est évident que fonctionnement n'est pas agile. ZF2++ répond à ce problème en changeant l'architecture. Alors que les frameworks actuels qu'il soit full-stack (ZF2, Symfony2 ...) ou Micro (Silex, Slim ...) sont construit à partir d'un core sur lequel se greffe des composants, ZF2++ change radicalement d'approche en proposant une architecture orientée middleware (workflow ou chain of responsibility) :

![Ceci est un schéma très simplifié](/assets/full-stack-micro-vs-zf2plusplus.png)

La nouvelle architecture rendra tous les composants indépendants l'un vis-à-vis de l'autre. Il n'y a plus de core. La requête rentre par un composant, elle est utilisée et est renvoyée au composant suivant. Cette architecture sera rendue possible par Composer, et par la définition d'une interface commune entre les composants, prévue par le [PSR-7](https://github.com/php-fig/fig-standards/blob/master/proposed/http-message.md). En outre, le MVC tel qu'on le connaît est amené à disparaître. À noter que cette idée avait déjà été pensée par Andi Gutmans pour ZF1. On peut dire que ZF2++ tend plus à se rapprocher d'une librairie que d'un microframework ou d'un full-stack.

# 6 breakpoints
Présenté par [Corentin Larose](https://twitter.com/corentinlarose)

6 breakpoints est une méthode de débogage facile et rapide que nous a présenté Corentin Larose. Mais tout d'abord, faisons quelques rappels sur ZF2.

## Rappels sur ZF2
ZF2 est event-driven. Ce qui signifie qu'entre le moment où une requête survient et le moment ou le document est rendu, plusieurs events sont émis.
Par exemple:
* Le ZF2 Skeleton utilise ~35 events pour s'afficher
* L'Apigility Skeleton utilise ~76 events

Rappel sur le fonctionnement de ZF2. Il y a 6 étapes à partir de la réception d'une requête jusqu'au rendu d'une page :

1. Bootstrap: Injection de la configuration, découverte des modules, enregistrement des écouteurs d'événements
2. Route: Fait correspondre l'URL ou les arguments du CLI avec une classe Dispatchable
3. Dispatch: Appel une méthode sur la classe Dispatchable correspondante
4. Render: Génère un rendu de la vue
5. Finish: Envoie la réponse

Si vous avez la curiosité de lire le code à l'intérieur de `Zend\MVC\Application` vous verrez que tous les events correspondants aux étapes ci-dessus, y sont tous lancés.

Les events en question :

* Zend\MVC\MVCEvent::EVENT_BOOTSTRAP
* Zend\MVC\MVCEvent::EVENT_ROUTE
* Zend\MVC\MVCEvent::EVENT_DISPATCH
* Zend\MVC\MVCEvent::EVENT_RENDER
* Zend\MVC\MVCEvent::EVENT_FINISH

## La méthode de débogage
Cette méthode de débogage consiste à localiser les appels à `$events->trigger(MvcEvent::EVENT_*)` dans `Zend\MVC\Application` et à y poser des breakpoints en fonction des informations que l'on souhaite obtenir.
Vous avez un problème avec la découverte de vos modules, posez un breakpoint sur `Zend\MVC\MVCEvent::EVENT_BOOTSTRAP`. Vous avez obtenez une page 404 alors que vous attendiez autre chose, posez un breakpoint sur `Zend\MVC\MVCEvent::EVENT_ROUTE`. Et ainsi de suite.
À chaque breakpoint, vous aurez la possibilité de voir les variables du contexte, pour vous permettre de résoudre plus facilement votre bug.

Vous demanderez où est le 6e breakpoint ?
Et bien il se trouve dans `Zend\EventManager\EventManager`. Il s'agit de la fonction `triggerListeners` où se situe  la ligne `$responses->push(call_user_func($listenerCallback, $e));`.
Placez-y un breakpoint et vous pourrez ainsi obtenir le contexte autour du déclenchement des écouteurs d'évènements.
