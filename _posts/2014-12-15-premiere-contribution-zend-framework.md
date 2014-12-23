---
layout: post
title:  "Première contribution Zend Framework"
date:   2014-12-15
tags: Zend Framework 2, Contribution
---

Ce weekend, j'ai pour la première fois contribué à Zend Framework 2 et je voulais partager cela avec vous.


## tl;dr

Bien que votre première contribution puisse être fastidieuse, n'hésitez pas un seul instant. Cela demande juste de la rigueur et un peu de connaissance. C'est la meilleure opportunité pour <u>vraiment</u> gagner en compétence et rendre ce que les projets open-source vous donne chaque jour.


## Once upon a time ...

~~Tout commence dans un château, où vivait une jolie princesse emprisonnée par un affreux monstre ...~~

Tout commence, il y a 1 mois lors du premier meetup parisien de Zend Framework 2,
[que je vous avais conté]({% post_url 2014-11-17-premier-meetup-zend-framework-2-paris %}).
Notre hôte, [Corentin Larose](https://twitter.com/corentinlarose), avait demandé qui parmi l'audience avait déjà contribué à ZF2.
Beaucoup de mains restèrent baissées, y compris la mienne.

Je me sentais un peu honteux. Utiliser un logiciel de qualité industrielle, entièrement gratuit, supporté par une communauté bénévole, cela me mettait mal à l'aise. J'avais comme l'impression de ne pas rendre ce que l'on me donne. Un peu comme lorsque l'on vous fait un cadeau et que vous n'offrez rien en retour.

J'utilise de nombreux logiciels open-source, bien évidemment il est humainement impossible de contribuer à tous, mais pour ZF2, qui est certainement celui que j'utilise le plus actuellement, je devais m'investir.

Affectionnant l'open-source depuis quelques années, je connaissais de loin la contribution,
mais je n'avais aucune expérience.
J'entrepris alors de me renseigner. Pour cela, Zend a tout prévu, ils ont mis a disposition quelques documents rassemblés sur leur [repo GitHub](https://github.com/zendframework/zf2/blob/master/CONTRIBUTING.md) qui m'ont permis de me faire une idée de leur processus de contribution.


## Le déclic

Toutefois, je ne m'y suis pas mis tout de suite. Deux semaines ont passées, jusqu'à ce jour de décembre. J'étais de passage sur le dépôt Zend Framework 2, et là je vis que le commit d'un [collègue de travail](http://php-underground.blogspot.com/) avait été mergé par [Ocramius](https://twitter.com/ocramius).

J'ai instantanément envoyé un mail à cette connaissance, en lui posant quelques questions sur la contribution. Son retour d'expérience m'a beaucoup rassuré. Je me sentais moins seul, j'avais enfin quelqu'un avec qui je pouvais partager mes commits sur ZF2.


## Premier ticket

J'ai donc commencé par chercher un ticket. Je pensais naïvement au début que l'on m'assignerait une tâche, j'envoie donc un tweet à Ocramius pour m'enquérir du processus d'attribution des issues. Il me répond que c'est un projet open-source et que je prends le ticket qui me plaît, comme un grand garçon en somme. Je me suis donc rendu sur le dépôt. Après avoir compris comment étaient classées les issues, j'ai élaboré [une requête](https://github.com/zendframework/zf2/issues?q=is%3Aopen+is%3Aissue+label%3Abug+comments%3A0+no%3Aassignee) pour récupérer celles qui pourraient m'intéresser. Mon dévolu s'est finalement porté sur [celui-ci](https://github.com/zendframework/zf2/issues/6414). Il concernait un composant non critique et il semblait simple, ce qui est mieux pour débuter et appréhender le workflow de contribution.

Je prépare mon espace de travail : fork du repo ZF2, clone du skeleton et du framework, ... j'affûte mes outils : Configuration de Netbeans, Xdebug. J'étais prêt à me mettre au travail.

Je commence par constater que le bug est toujours d'actualité. Je génère d'abord un formulaire avec un captcha SANS label. Pas d'erreur de validation HTML. Je recommence l'opération mais cette fois-ci avec un label. Bingo, j'ai mon erreur de validation. Cela était dû à la présence de plusieurs balises à l'intérieur de '<label>'. Ce qui n'est pas permis par la [spécification HTML](http://www.w3.org/TR/html5/forms.html#the-label-element).

Maintenant, reste à savoir à quel endroit je vais bien pouvoir introduire mon fix sans toutefois rompre l'architecture en place. C'est là qu'interviens Xdebug, j'exécute pas à pas la génération du formulaire et je finis par tomber sur la classe qui a pour rôle le rendu des `<label>` dans les éléments de formulaire.

Je réfléchissa longuement avant d'appliquer mon patch. La solution semblait évidente et c'est justement ce qui me faisait douter. Allais-je briser la structure de la classe ? N'était-ce pas une solution de facilité ? Répondait-il aux exigences de l'équipe de Zend Framework ? Autant de questions qui m'ont fait douter et puis je me suis lancé. J'ai ajouté mon correctif, j'ai testé à plusieurs reprises : le code était désormais valide avec un `<label>` sur le captcha !


## Commit, push ... repeat

Fier de moi je commit. Je ne rédige pas de test unitaire me disant stupidement que c'est une erreur de validation HTML et qu'il n'y a pas besoin de tests unitaire. J'effectue ma [pull-request](https://github.com/zendframework/zf2/pull/7030). Et là le couperet tombe, Ocramius me dit ["nothing gets merged without tests :-\"](https://github.com/zendframework/zf2/pull/7030#issuecomment-66879015). Je réalise mon erreur, et j'entreprends immédiatement d'écrire un test.

Là encore, j'essaye de me conformer au code en place, j'observe comment ont été rédigés les tests précédents. J'implémente [le mien](https://github.com/jbenoit2011/zf2/commit/edf3f0becb6806b3294a3734cf885393d0de0a6b), je teste, ça fonctionne. Houra ! Re-belote, je commit mon test. Je me rends sur Travis pour surveiller mon test. Je trépigne d'impatience, et là ça échoue. Je parcours les logs, et j'aperçois que le code HTML rendu sur Travis est différent de celui rendu sur mon ordinateur.

Après quelques recherches, je finis par comprendre que c'est une question de doctype. Sur ma machine, les balises étaient rendues en HTML alors que sur Travis elles étaient en XHTML. C'est une erreur que j'aurais pu éviter si j'avais examiné un peu plus les tests existants dans la classe. Je corrige cette erreur et au passage, j'en profite pour rectifier les quelques erreurs d'indentation que j'avais laissées par méconnaissance des standards ZF2.

Je commit, pousse, attends avec impatience le résultat du test et [cela réussi finalement](https://github.com/zendframework/zf2/pull/7030#issuecomment-66924898). Quelques heures plus tard, [Ocramius a mergé mon correctif](https://github.com/zendframework/zf2/pull/7030#issuecomment-67152739). Youpi, je venais de contribuer pour la première fois sur ZF2.


## Conclusion

Pour terminer, je dirai que c'était une expérience très formatrice. Cela demande beaucoup de rigueur, le processus est strict, les standards de code aussi. C'est aussi l'occasion de faire du reverse-engineering sur des composants et de mieux comprendre leur fonctionnement interne.
Je recommande vivement aux développeurs qui hésite à se lancer, il n'y a que de bonnes choses à en retirer. C'est de loin la meilleure manière d'améliorer ses compétences en développement et sa connaissance d'un framework. L'équipe de Zend ne vous clouera pas en place publique pour avoir mis une virgule au mauvais endroit, ils essaieront plutôt de vous remettre dans le droit chemin avec des conseils avisés. Alors n'hésitez plus, contribuez !
