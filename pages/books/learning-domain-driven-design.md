# Learning Domain Driven Design: Aligning Software Architecture and Business Strategy

### Introduction

- Le problème principal qui met en échec la plupart des projets de dev c'est la **communication**. Et c'est le cœur de ce qu'est le DDD.
- Le but du DDD c'est d'aligner le **D**esign logiciel avec le **D**omaine business.
- Le DDD se décompose en 2 parties :
  - Le **design stratégique** : créer une compréhension commune du domaine, et prendre des décisions haut niveau sur le projet.
    - C'est les questions “Quel logiciel on crée ?” et “Pourquoi on le crée ?”
  - Le **design tactique** : écrire du code qui épouse le domaine.
    - C'est la question “Comment on crée chaque partie du logiciel ?”

## Part I : Strategic Design

### 1 - Analyzing Business Domains

- Pour développer un bon logiciel qui réponde au besoin de notre entreprise, il faut que **nous les devs connaissions la stratégie du business** : différencier les parties importantes des parties moins importantes pour adapter nos techniques.
- Le **business domain** est le domaine d'activité principal de l'entreprise.
  - Exemple : FedEx fait de la livraison.
  - Pour une grande entreprise qui fait plusieurs choses très différentes (comme Google ou Amazon), on peut parler de plusieurs domains.
- Un **subdomain** est une activité que l'entreprise fait pour mener à bien son business.
  - Exemple : Starbucks fait des cafés, mais il doit aussi recruter, gérer la tréso etc.
  - Il existe 3 types de subdomains :
    - Les **core subdomains** sont les seules activités qui donnent à l'entreprise un **avantage concurrentiel** par rapport à ses concurrents. Soit par une innovation produit, soit par une optimisation qui permet de produire à moindre coût.
      - C'est une activité par nature **complexe** étant donné qu'il faut qu'elle ne soit pas facilement reproductible par les concurrents.
      - Le statut de “core” est aussi par nature **temporaire** : dès que les concurrents rattrappent ce n'est plus core.
      - A noter aussi qu'un core subdomain peut très bien ne pas résider dans du logiciel. Par ex une bijouterie va vendre mieux que les concurrents grâce au style donné par les artisans bijoutiers (qui donne l'avantage compétitif et qui est donc le core subdomain), la boutique en ligne dans ce cas est un generic subdomain.
    - Les **generic subdomains** sont des **problèmes communs** déjà résolus et largement disponibles et utilisés par les concurrents (soit en open source, soit sous forme de service payant).
      - Ils sont **complexes** comme les core subdomains, mais ne donnent juste pas d'avantage compétitif.
      - Exemple : un mécanisme d'authentification est complexe, mais des solutions open source et payantes existent, et personne ne recode son système à soi.
    - Les **supporting subdomains** sont des activités simples mais nécessaires et dont on n'a pas de solution générique à réutiliser (ou alors ça coûte moins cher de le faire soi-même) : c'est du ETL (extract, load, transform), ou encore CRUD.
      - Vu la simplicité, ils ne peuvent pas procurer d'avantage compétitif.
      - Vu qu'il n'y a pas d'avantage compétitif, on préfère appliquer l'effort sur les core subdomains qui apporteront plus de valeur business.
  - Côté stratégie :
    - Les core subdomains doivent être **développés au sein de l'entreprise**, par les développeurs les plus expérimentés, et en appliquant le **maximum de qualité** et les techniques d'ingénierie les plus avancées.
      - On ne peut pas les acheter sinon on perd la notion d'avantage compétitif, et il ne serait pas très malin de les sous-traiter.
    - Les generic subdomains étant difficiles mais déjà résolus, il est plus rentable de **les acheter** (ou de faire appel aux services d'un consultant spécialisé), ou d'**utiliser une solution open source**.
    - Les supporting subdomains sont simples et changent peu, donc ils peuvent être implémentés **avec moins de qualité**, ou **moins de techniques sophistiquées** (design patterns, techniques d'architecture etc.). Un simple framework rapide suffit.
      - On peut laisser les débutants se charger de ça, ou alors on peut le sous-traiter.
- Pour **trouver les subdomains**, il va falloir faire l'analyse nous-mêmes.
  - On peut déjà partir des départements qui composent l'entreprise, mais ça nous donne des subdomains grossiers.
  - On peut alors “distiller” les subdomains en plus petits subdomains : on prend un département et on liste les sous-activités, puis on analyse pour chacune d'entre elles si elles sont core, generic ou support.
    - On peut distiller au maximum jusqu'à arriver à “un subdomain comme un ensemble cohérent de cas d'usages” : un même acteur qui fait plusieurs tâches précises.
    - Il faut faire la distillation maximale pour les subdomains core, pour pouvoir éliminer les petits bouts génériques ou support à l'intérieur et se concentrer uniquement sur ce qui a le plus de valeur.
    - Pour les autres on s'arrête de distiller à partir du moment où une distillation donne des activités du même type (generic ou support), aller au-delà ne nous permettra de toute façon pas de prendre des décisions plus stratégiques.
- Les devs vont devoir collaborer avec les **domain experts**, mais qui sont-ils ?
  - Ce ne sont pas les analystes qui recueillent le besoin, ni les ingénieurs qui créent le système. Ces derniers transforment le modèle mental des domain experts pour en faire du logiciel.
  - Ce sont **soit ceux qui arrivent avec les besoins** (qui sont là depuis le début, qui ont créé l'activité etc.), **soit les utilisateurs finaux** dont on résout le problème.
  - A noter que les domain experts peuvent très bien n'être experts que d'un sous-domaine seulement.

### 2 - Discovering Domain Knowledge

- Généralement, les gens du business (les domain experts) communiquent les besoins à des intermédiaires (system/business analysts, product owners, project managers), qui vont ensuite communiquer ça aux ingénieurs logiciel qui créent le logiciel.
  `domain experts -> gens au milieu -> software engineers`
  - On assiste aussi à une ou plusieurs transformations :
    `domain knowledge -> analysis model -> software design model`
- Le DDD propose d'arrêter les transformations, et que tous les acteurs utilisent le même langage pour se parler : l'**Ubiquitous Language**.
  - Il doit pouvoir être compris par les domain experts, donc il va se baser sur les termes qu'ils utilisent déjà.
  - Chaque terme doit être **précis** et sans ambiguïté, et il ne doit **pas y avoir de synonymes**.
    - La raison est que si dans les conversations courantes on arrive à se comprendre même avec des ambiguïtés en fonction du contexte, dans le cadre du logiciel ça marche moins bien.
  - Le but de l'ubiquitous language est d'être un modèle du business domain, qui va représenter le modèle mental des domain experts.
    - Un modèle n'a pas à représenter toute la réalité mais seulement une partie de manière à être utile. Et donc ici aussi il ne s'agit pas de représenter tout le business domain, ni de faire de tout le monde un domain expert.
  - Côté outils :
    - Il faut un **glossaire sous forme de wiki**, que **tous** font évoluer au fur et à mesure. On y met les termes de l'ubiquitous language.
    - Pour la logique il faut d'autres outils comme des **description de cas d'usage**, ou des tests Gherkin.
      Exemple Gherkin :
      ```
      Scenario: Notifier l'agent d'un nouveau cas
          Given: Jules qui crée un nouveau cas "Nouveau cas"
          When: Le ticket est assigné à M. Wolf
          Then: L'agent reçoit une notification
      ```
    - Ceci dit, le plus important est l'usage quotidien de l'ubiquitous language par les acteurs. Les outils ne sont qu'un plus.
- Pour que la compréhension du business domain soit bonne, il faut une **communication régulière** entre les domain experts et ceux qui sont en charge de réaliser le logiciel.
  - Cette communication se fait bien sûr dans l'ubiquitous language.
  - Ça implique de poser beaucoup de questions aux domain experts.
  - A force, on va se rendre compte qu'on pourra aider les domain experts à mieux comprendre leur champ d'expertise sur certains points, par exemple en pensant à des edge cases et pas seulement aux “happy paths”.
    - On peut alors aller vers une **co-construction de la compréhension du domaine** (même si les experts en sauront toujours plus).
- A propos de la langue de l'ubiquitous language, le conseil de Vlad est d'utiliser l'anglais au moins pour les noms d'entités du business domain.

### 3 - Managing Domain Complexity

- On se rend parfois compte qu'un même mot est utilisé par plusieurs domain experts avec une signification différente.
  - Exemple : le mot _lead_ peut désigner un simple prospect intéressé dans le département marketing, alors que ça peut désigner une entité plus complexe avec des histoires de process de vente dans le département sales.
- En général on a naturellement tendance à agrandir notre modèle pour que chaque version du mot puisse être représentée, mais ça finit par donner un modèle très complexe et difficile à maintenir.
  - La solution DDD est de créer deux **bounded contexts** qui auront chacun leur ubiquitous language.
- Un bounded context doit avoir des **limites** (boundaries).
  - Une manière de les trouver est de repérer les incohérences de terminologies utilisées par les domain experts : un bounded context ne pourra pas être plus grand que ce que ces incohérences permettent.
  - Mais on peut diviser ces ensembles en bounded contexts plus petits. Il s'agit là d'une **décision stratégique**, qui dépend du contexte.
    - Avoir des bounded context trop grands impliquera une plus grande complexité du modèle, alors que s'ils sont trop petits on aura un plus grand effort d'intégration à faire.
    - Les raisons qui peuvent pousser à découper en bounded contexts plus petits peuvent être :
      - d'ordre organisationnels : par exemple une nouvelle équipe qui va prendre en charge un morceau du logiciel qui va se développer sur un rythme différent du reste.
      - d'ordre non fonctionnels : par exemple le fait de devoir scaler certaines parties du logiciel indépendamment.
    - De manière générale, on va essayer de trouver des fonctionnalités cohérentes qui opèrent sur un même jeu de données, et les laisser dans un même bounded context.
- La différence entre subdomains et bounded contexts c'est que les subdomains sont là de fait et ressortent avec l'analyse. Ils sont de la responsabilité du business. Alors que les bounded contexts sont le résultat d'un choix stratégique, et sont de la responsabilité des ingénieurs logiciel.
- Chaque bounded context a une séparation physique : il devra être implémenté avec sa codebase, en tant que **projet autonome**, et éventuellement service autonome.
  - Dans le cas où il s'agit d'un service autonome, on peut aussi choisir le langage, la stack, indépendants des autres.
- Un bounded context ne doit être géré que par **une seule équipe de dev**.
  - Une équipe peut par contre en avoir plusieurs en charge.
- Si on se tourne vers la vie de tous les jours, on peut remarquer plein d'exemples où des mots ont des significations différentes selon les contextes, et où un modèle différent serait utile pour chaque :
  - Un carton rectangulaire qu'on met au sol permet de représenter un réfrigérateur pour savoir s'il va bien s'insérer. Ce modèle est partiel mais il est utile pour ce qu'on veut faire, et c'est tout ce qui compte.
  - Et si on a besoin de vérifier aussi la hauteur, on peut utiliser un autre modèle du réfrigérateur qui suffit : un simple mètre. => On a là une vision de l'approche DDD des modèles.

### 4 - Integrating Bounded Contexts

- Chaque bounded context permet de représenter un modèle au sens DDD, c'est-à-dire une abstraction utile pour résoudre un problème particulier.
- Les bounded contexts doivent quand même avoir des points de contact pour que le logiciel forme un tout, ces points sont appelés **contracts**.
  - On parle ici de points de contact au niveau du modèle (terminologie, concepts), mais le modèle se traduit à chaque fois en implémentation aussi.
- La nature de ces contracts dépend de la nature de la relation entre les équipes qui gèrent les bounded contexts, il y en a de 3 types :
  - **Cooperation** : il s'agit soit de la même équipe, soit d'équipes qui ont une bonne communication et collaboration (par ex le succès de l'une dépend du succès de l'autre).
    - **Partnership** : aucune équipe ne dicte sa loi à l'autre. Elles collaborent pour faire des changements dès qu'il y en a besoin d'un côté ou de l'autre.
      - Il faut une intégration continue pour que les problèmes soient vite résolus.
      - Il faut une communication vraiment au top entre équipes, et pas d'histoires de rivalités.
    - **Shared Kernel** : il s'agit d'un cas à part où les boundaries des bounded contexts sont violés : deux bounded contexts partagent une partie de leur modèle.
      - Par exemple, ça peut être la partie d'authentification maison qu'ils auront en commun.
      - La partie partagée peut être changée par chaque bounded context, et affectera l'autre immédiatement. Il faut donc que des tests d'intégration soient déclenchés à chaque changement.
      - Bien réfléchir avant de l'utiliser : on l'utilise quand le coût pour appliquer les changements du modèle sous forme d'implémentation dépasse le coût de coordination entre équipes. C'est donc pertinent pour des modèles qui changent beaucoup.
      - Cas d'usage courants :
        - Quand deux équipes n'ont pas une assez bonne communication, et que le modèle partnership donnerait lieu à des problèmes d'intégration.
        - Quand on a un système legacy qu'on essaye de découper en bounded contexts, la partie pas encore découpée peut être temporairement partagée par les autres bounded contexts.
        - Quand une même équipe a deux bounded contexts en charge, il est possible qu'avec le temps les frontières de ces deux bounded contexts s'estompent. Pour éviter ça on peut introduire une partie partagée qui va bien définir les frontières des deux autres.
  - **Customer-Supplier** : contrairement au modèle de type cooperation, la réussite de chaque équipe en charge de chaque bounded context n'est pas liée. On a donc un rapport de force qui va pencher vers l'une ou l'autre partie (le customer qui consomme ou le supplier qui fournit).
    - **Conformist** : la provider n'a pas vraiment de motivation à satisfaire le customer. Soit parce que c'est un provider externe, soit il est interne et c'est la politique de l'entreprise qui fait ça. Le customer va donc se conformer au modèle et aux changements de modèle du provider.
      - Le supplier le fait soit parce que c'est un standard bien établi, soit simplement parce que ça lui convient.
    - **Anticorruption Layer** : le provider a toujours le pouvoir, mais cette fois le consumer ne va pas se conformer, il va vouloir se protéger par une couche d'abstraction entre ce qui lui arrive et son propre modèle.
      - Le consumer peut vouloir cette couche quand :
        - Il contient un core subdomain, et que le modèle du provider pourrait être gênant.
        - Le modèle du provider est bordélique, et on veut s'en protéger.
        - Le modèle du provider change souvent, et on veut s'en protéger.
      - Grâce à cette couche filtrante, le modèle du consumer n'aura pas à être pollué par des concepts dont il n'a pas besoin.
    - **Open-Host Service** : cette fois c'est le consumer qui a l'avantage parce que le provider a une motivation à satisfaire le consumer. Le provider va donc mettre en place une couche d'abstraction de son côté pour découpler son modèle interne du modèle public qu'il expose.
      - C'est comme l'anticorruption layer, mais côté provider.
      - Le provider peut éventuellement maintenir plusieurs OHS le temps que les customers migrent au nouveau modèle public.
  - **Separate ways** : les deux bounded contexts n'interagissent pas du tout. Si elles ont des fonctionnalités communes, elles les dupliquent de leur côté.
    - Ça peut être quand la communication entre équipes est impossible.
    - Ou encore quand ça coûte plus cher de partager une fonctionnalité que de la refaire chacun de son côté. Parce que la fonctionnalité est trop simple, ou que les deux modèles sont trop différents.
- Le **context map** est une représentation graphique de chaque bounded context, avec les relations que chacun entretient avec les autres.
  - Ça peut permettre de faire des choix stratégiques haut niveau.
  - Ça peut faire apparaître des problèmes organisationnels :
    - Par exemple si une équipe provider a tous ses consumers qui mettent en place un anticorruption layer. Ou encore si une équipe n'a que des relations separate ways avec les autres.
  - Le context map doit être maintenu par l'ensemble des équipes.
    - On peut utiliser des outils comme [Context Mapper](https://contextmapper.org), qui consiste à avoir un fichier source dans un dépôt de code, et permet de visualiser le graphique résultant avec une extension VSCode.

## Part II : Tactical Design

### 5 - Implementing Simple Business Logic

- Ce chapitre présente deux patterns tactiques simples. Ce sont des patterns connus, décrits notamment dans _Patterns of Enterprise Application Architecture_ de Martin Fowler.
- **1 - Transaction Script** : il s'agit pour le provider de fournir une interface publique avec des endpoints qui ont un **comportement transactionnel** : soit une requête réussit entièrement, soit elle est entièrement annulée.
  - Martin Fowler écrit à propos de ce pattern : Organizes business logic by procedures where each procedure handles a single request from the presentation.
  - Notre fonction peut accéder à la base de données pour faire des manipulations, soit à travers une mince couche d'abstraction, soit directement. (Donc **forte dépendance avec la BDD**).
  - Dans le cas simple où on a une BDD relationnelle, il s'agit simplement d'utiliser la fonctionnalité native de transaction de sa BDD : on crée la transaction au début de la fonction, et on commit ou rollback à la fin.
    - La chose se complique quand notre BDD ne supporte pas les transactions, qu'on doit mettre à jour des données dans plusieurs BDD, ou encore qu'on est dans un contexte distribué où les transactions ne sont pas effectives immédiatement (eventual consistency).
  - Potentiels problèmes qu'on rencontre souvent :
    - Oubli de transaction : parfois on oublie tout simplement d'utiliser le mécanisme de transaction au début de notre fonction. En exécutant plusieurs écritures en BDD on peut avoir la 1ère qui réussit et la 2ème qui échoue. On se retrouve alors potentiellement dans un système incohérent.
    - Système distribué : souvent dans les systèmes distribués on met à jour une donnée, et on prévient d'autres serveurs en leur envoyant un message. Il suffit que l'envoi de message ne marche pas pour que le système devienne incohérent parce qu'une partie du travail ne sera pas faite.
      - Il n'y a pas de solution simple à ce problème, mais de nombreuses solutions qui font chacune du mieux qu'elles peuvent, en fonction du contexte.
      - (pas dit dans le livre, mais c'est l'objet de _Designing Data-Intensive Applications_ de Martin Kleppmann)
  - Transaction Script est le pattern le plus simple, et convient pour les opérations simples de type ETL (extract, transform, load) où la logique business est elle-même simple et change peu souvent.
    - Il convient bien :
      - Dans les supporting subdomains.
      - Comme couche d'adapter pour intégrer un generic subdomain.
      - Dans un anticorruption layer vis-à-vis d'un autre bounded context.
    - Par contre sa simplicité fait que plus la logique business est complexe, plus il y a de la duplication de logique entre les diverses fonctions, et donc un risque d'incohérence au fil du temps. Donc il ne conviendra pas aux core subdomains.
  - Tous les autres patterns plus avancés, sont d'une manière ou d'une autre basés sur Transaction Script.
- **2 - Active Record** : on garde les caractéristiques transactionnelles du Transaction Script, mais on va ajouter des objets (les fameux Active Records) qui vont être en charge de **transformer les données** depuis la BDD vers des structures en mémoire qui conviennent mieux à ce qu'on veut faire dans nos fonctions.
  - Martin Fowler écrit à propos de ce pattern : An object that wraps a row in a database table or view, encapsulates the database access, and the domain adds domain logic on that data.
  - La conséquence est que les fonctions appelées par les endpoints ne vont plus faire d'appel à la BDD, mais vont passer uniquement par les Active Records pour manipuler les données.
    - L'Active Record va fournir des méthodes pour accéder aux données et les modifier, et va faire la traduction à chaque fois.
    - On encapsule la complexité de la transformation des données pour éviter de dupliquer cette logique-là un peu partout.
  - Active Record est utile pour les cas d'usage simples, mais qui impliquent des transformations de données potentiellement complexes.
    - Il convient donc lui aussi seulement aux supporting subdomains, ou à des couches d'adapter pour intégrer un generic subdomain ou un autre bounded context.
    - Quand il est utilisé pour les logiques business complexes comme pour les core subdomains, on parle parfois d'un anemic domain model antipattern (et c'est déconseillé).
- Il s'agit cependant de **rester pragmatique** : bien qu'il soit souhaitable de protéger les données, il faut se demander à chaque fois quelles seraient les implications de la corruption de données dans tel ou tel projet.
  - Par exemple, si on ingère des millions d'entrées venant de devices IOT dans notre base, risquer d'en corrompre 0.001% peut ne pas forcément être très grave.

### 6 - Tackling Complex Business Logic

- Ce chapitre présente le pattern du DDD pour les logiques business compliquées (à utiliser dans les core subdomains donc) : le **domain model pattern**.
- Ce pattern est aussi dans le bouquin de Fowler, mais il a été développé plus en détail dans celui d'Eric Evans : _Domain Driven Design: Tackling Complexity in the Heart of Software_, sorti un an après (2003).
- Au niveau de l'implémentation :
  - Le code du domain model est là pour représenter le domaine qui est déjà complexe, donc il **ne doit pas introduire de complexité supplémentaire liée aux technos ou à l'infrastructure**. => on utilisera donc du code pur (par exemple du TypeScript pur), pas de framework, d'ORM ou autre chose d'extérieur dans ce code.
  - Débarrassé de tout ce qui est lié aux technos, le code peut **utiliser l'ubiquitous language** et épouser le modèle mental des domain experts.
  - A noter que ce livre présente les choses d'un point de vue orienté objet, mais une approche fonctionnelle serait tout aussi valide, d'autres ressources dans la communauté du DDD vont dans ce sens.
- Les blocs qui composent le domain model sont :
  - **1- Value objects** :
    - Ce sont des objets **immutables** identifiés par les valeurs de chacune de leurs propriétés. Si on veut le même objet avec une valeur différente pour une de ses propriétés, on crée de fait un autre objet.
      - Par exemple, dans Python ou JavaScript, le string est un value object : on ne peut pas modifier une valeur string en elle-même, mais on peut créer de nouveaux strings avec des valeurs différentes (et qui auront bien une référence en mémoire différente).
      - Les value objects n'ont **pas d'attribut ID**, puisque leur identification se fait par la composition des valeurs de leurs propriétés qui les rend uniques.
        - En revanche, il faut implémenter un opérateur d'égalité pour pouvoir comparer deux value objects entre eux.
    - Le value object sera **utilisé comme type pour les données** à la place des types primitifs (comme string, number etc).
      - L'utilisation systématique de types primitifs est connue comme “[the primitive obsession](https://wiki.c2.com/?PrimitiveObsession)”.
      - Par exemple, plutôt que d'avoir `email: string`, on peut avoir `email: Email`, avec le value object `Email` qui va **faire la validation** et refuser toute valeur qui ne correspond pas.
        - Ça évite de dupliquer la logique de validation partout, mais aussi d'oublier de faire cette validation à certains endroits.
      - En plus de la simple validation de données, le value object peut **encapsuler en un endroit unique du code business** lié au concept précis qu'il décrit. (Et c'est là qu'il “brille” vraiment)
        - Concept qui est normalement dans le lexique de l'ubiquitous language de notre bounded context.
        - Par exemple, un value object représentant une couleur peut avoir une méthode pour mixer la couleur avec une autre, et en obtenir une 3ème, selon des règles business bien particulières.
      - Et ça **augmente la clarté du code** en rendant plus explicite l'intention.
    - On **utilise les value objects dès que possible**. Typiquement comme type des attributs des entities.
  - **2.1- Entities** :
    - Ce sont des objets dont les propriétés **peuvent changer** (de type value object ou de type primitif), identifiés par leur **ID qui lui ne change pas**.
    - Exemple d'entity : une personne représentée par son nom, son email etc.
    - Les entities sont importants, mais ils n'ont vraiment d'existence que dans le cadre d'un aggregate.
  - **2.2 - Aggregates** :
    - Un aggregate est une entity qui en contient d'autres.
      - On choisit un **aggregate root** qui est l'entity principale, et contient les autres.
      - L'idée c'est de regrouper des **entities qui sont amenées à changer ensemble**.
    - L'aggregate est responsable de **garantir la consistance des données** en son sein.
      - Les objets extérieurs pourront donc seulement lire ses propriétés, mais pour les changer il faudra toujours passer par l'interface publique de l'aggregate.
        - Les fonctions de cette interface publique s'appellent les **commands**.
        - Il y a 2 manière d'implémenter ces commands :
          - Soit comme méthodes publiques de l'aggregate root.
          - Soit la logique se trouve dans des objets de commande spécifiques à l'aggregate et contenant la logique à appliquer. On instancie un tel objet command avec les infos qu'on veut changer, et on le passe à une méthode “execute” de l'aggregate root chargée d'appliquer la commande.
        - Le fait que l'aggregate soit le seul responsable de modifier ses données fait que la logique business le concernant ne peut qu'être en son sein, et pas disséminée ailleurs.
      - La couche applicative (aussi appelée souche “service” : celle qui forward les actions de l'API publique au domain model) n'a donc plus qu'à :
        - Charger l'état actuel de l'aggregate.
        - Exécuter la commande requise.
        - Enregistrer le nouvel état en BDD.
        - Renvoyer le résultat à l'appelant.
      - Attention à la concurrence. Pour éviter les problèmes, la couche applicative devra s'assurer que l'état lu au début n'a pas changé entre-temps avant de valider le nouvel état de l'aggregate en BDD.
        - Pour ce faire, le moyen le plus classique est que l'aggregate ait une propriété “version” incrémentée à chaque modification.
        - Si la version a changé entre-temps par une autre action concurrente, on pourra choisir d'annuler notre action, ou d'appliquer un mécanisme de résolution approprié à notre logique business (si on en est capable).
    - L'aggregate doit avoir un **comportement transactionnel** : toute commande devra soit être intégralement exécutée, soit annulée.
      - En revanche, **aucune opération ne pourra modifier plusieurs aggregates de manière transactionnelle** : c'est une transaction par action sur un seul aggregate.
      - Si on se retrouve à avoir besoin de modifier plusieurs aggregates en une transaction, c'est que les limites de nos aggregates ont été mal pensées : il faut les revoir.
      - Exemple d'aggregate : un ticket de support, qui peut contenir un ou plusieurs messages, chacun pouvant contenir une ou plusieurs pièces jointes.
    - La règle de base pour délimiter son aggregate c'est de le choisir **le plus petit possible**, et d'y inclure toutes les données qui doivent rester **fortement consistantes** (strong consistency) avec l'aggregate.
      - Donc si en prenant en compte la logique business, certaines données liées à l'aggregate peuvent être un peu désynchronisées (et se mettre à jour un peu plus tard) sans causer de problèmes ( => eventual consistency), alors elles doivent être hors de l'aggregate.
      - Dans ce cas la bonne pratique est de placer dans l'aggregate une propriété représentant leur ID, là où pour une entity faisant partie de l'aggregate on placera en propriété une instance de celle-ci :
        ```typescript
        // le customer ne fait pas partie de l'aggregate
        private customer: UserID;
        // les messages en font partie
        private messages: Message[];
        ```
      - Exemple d'aggregate qu'on délimite : imaginons qu'on a notre ticket de support, avec une règle business qui implique de le réassigner à un autre agent s'il n'y a pas eu de nouveau messages alors qu'il reste moins de 50% du temps restant pour le traiter.
        - Si les messages mettent du temps à être à jour par rapport au ticket (c'est-à-dire qu'il y a une eventual consistency entre les messages et leur ticket), on aura du mal à respecter la règle business parce qu'on ne saura pas si il y a eu un message récemment ou pas. => Donc les entities “messages” vont dans le même aggregate.
        - En revanche, si les données des agents ou des produits sont légèrement en retard, ça ne nous empêchera pas de respecter notre règle business. => Donc ceux-là ne vont pas dans le même aggregate.
  - **2.3 - Domain Events** :
    - Les domain events sont des événements importants que chaque aggregate peut publier, et auquel d'autres aggregates peuvent souscrire.
    - Leur nom est important, et il doit être formulé au passé.
      - Par ex : `Ticket assigned` ou `Ticket escaleted`
    - Leur contenu doit décrire précisément ce qui s'est passé :
      - Exemple :
        ```json
        {
          "ticketId": "7bfaaf0c-128a-46f8-99b3-63f0851eb",
          "eventId": 146,
          "eventType": "ticket-escalation",
          "escalationReason": "missed-sla",
          "escalationTime": 65463113312
        }
        ```
  - **3 - Domain services** :
    - Quand on a de la logique business qui **n'appartient à aucun aggregate** ou qui semble être **pertinente pour plusieurs aggregates**, on peut la placer dans un domain service.
    - Le mot service ici ne fait pas référence à autre chose qu'on appellerait habituellement des services, c'est simplement des objets sans état qui contiennent de la logique business.
    - Dans les domain services on peut toucher à plusieurs aggregates, mais **seulement pour de la lecture** : la contrainte de ne modifier qu'un aggregate par transaction tient toujours.
    - Exemple de domain service : le temps pour répondre aux tickets de support doit être calculé en fonction de plusieurs paramètres, dépendant de l'agent en charge, d'éventuelles escalations, des shifts horaires etc. On peut donc avoir une fonction qui va lire dans plusieurs aggregates des informations, calcule le temps et le renvoie.
- Au fond, l'idée des value objects et des aggregates c'est de **réduire le degré de liberté du système** en encapsulant des morceaux de logique dans des endroits uniques, pour en réduire la complexité.

### 7 - Modeling the Dimension of Time

- On peut appliquer l'**event-sourcing** au domain model pattern pour créer un **event-sourced domain model pattern**.
  - Avec l'event sourcing on introduit la **dimension du temps** dans notre modèle.
  - Il s'agit de reprendre les mêmes blocs que le domain model pattern, mais avec une différence au niveau de la manière de stocker les informations en base de données (et c'est ça qui caractérise l'event sourcing) : on va **stocker les domain events** des aggregates comme **source de vérité**, au lieu de l'état actuel de l'aggregate.
    - On appelle cette BDD d'événements l'event store.
    - La BDD doit au minimum permettre de :
      - récupérer tous les événements d'un même aggregate
      - ajouter des événements à un aggregate
- Il s'agit de **rejouer l'ensemble des domain events** d'un aggregate à chaque fois qu'on le charge en mémoire pour obtenir son état.
  - Une fois qu'on a son état, on peut exécuter la logique business comme avant, en jouant la bonne commande sur l'aggregate.
  - Notre aggregate génèrera alors des domain events, mais **ne modifiera jamais son state autrement que par ces events**.
  - La persistance se fera en persistant les domain event dans l'event store.
- Avec le domain model pattern classique on avait aussi des domain events qui informaient les composants extérieurs des changements importants, mais là la différence c'est que ces events doivent être émis pour chaque changement d'état de l'aggregate.
- Côté code, on a notre objet aggregate root avec ses propriétés de type value object ou entity, et cet objet a une méthode **`apply(event: EventType)`** qui est surchargée plusieurs fois pour chaque type d'event qui peut survenir.

  ```typescript
  public class Person {
      public id: PersonId;
      public name: Name;
      public email: Email;
      public version: number;

      function apply(event: PersonCreated) {
          this.id = event.id;
          this.name = event.name;
          this.email = event.email;
          this.version = 0;
      }
      function apply(event: EmailUpdated) {
          this.email = event.email;
          this.version += 1;
      }
  }
  ```

  - Le constructeur de la classe prend la liste d'events et la rejoue dans l'ordre à chaque instanciation.

- On peut rejouer la projection des événements avec des variantes :
  - En rejouant les X premiers événements on **remonte à n'importe quel état** **précis dans le temps**.
  - En nous intéressant seulement à certains événements et en implémentant des actions particulières dans les fonctions **`apply`** de ceux-ci, on peut rechercher des données particulières et **faire de l'analyse de données**.
    - Par exemple, si on cherche toutes les personnes qui ont pu avoir une modification de leur email plus de 3 fois en une journée, il suffit de créer une classe qui surcharge **`apply`** pour l'event `EmailUpdated`, et de rejouer les événements en comparant les dates de changement dans notre méthode **`apply`**.
- Avantages et inconvénients :
  - Avantages de la version event-sourced par rapport à la version classique :
    - On peut **remonter dans le temps**. Par exemple pour analyser, ou encore pour débugger en allant à l'état exact qui a causé le bug.
    - On a du **deep insight** sur ce qui se passe. Particulièrement utile pour les core subdomains qui sont complexes.
    - On a une forte **traçabilité** de tout ce qui se passe, très utile dans un contexte d'audit, par exemple si la loi l'oblige comme avec les transactions monétaires.
    - Quand deux transactions écrivent la donnée de manière concurrente, on va habituellement annuler une des deux transactions. Ici on a des infos plus fines sur les **deux changements concurrents**, et donc on peut plus facilement prendre des décisions business sur la manière de les **concilier au lieu de les annuler.**
  - Désavantages :
    - **Courbe d'apprentissage** pour l'équipe : il faut du temps et la volonté pour l'équipe d'apprendre ce paradigme.
    - **L'évolution du modèle est plus compliquée** : on ne change pas un schéma d'events comme on change la structure d'une BDD relationnelle.
      - A ce propos il y a le livre _Versioning in an Event Sourced System_ de Greg Young.
    - La **complexité globale de l'architecture** est plus grande.
  - RDV au chapitre 10 pour avoir des règles basiques sur le type de design à choisir.
- A propos de la **performance** :
  - Oui rejouer les events à chaque fois qu'on instancie un objet affecte la performance, mais dans les faits :
    - Il faut bien noter que les events sont propres à chaque aggregate, et que ceux-ci ont un cycle de vie.
      - Par ex un ticket de support passera par plusieurs étapes, et finira par être fermé, donc n'aura plus de changements d'état.
    - En deçà de 10 000 events par aggregate, l'impact de performance ne se ressent pas.
      - La moyenne du cycle de vie des objets dans la plupart des systèmes est de l'ordre de 100 events.
  - Dans les rares cas où la performance est un problème, il y a la technique du **snapshot**.
    - Il s'agit de générer de manière asynchrone une projection sous forme de BDD servant de cache, et ayant les données des états de chaque aggregates, mais aussi le numéro du dernier event pris en compte.
    - Quand l'application a besoin de charger un aggregate, elle le fait depuis ce cache, puis lit le numéro du dernier event pris en compte, et va chercher les events manquants dans l'event store pour les appliquer elle-même et obtenir l'état à jour de l'aggregate.
      - Ainsi donc, même si l'aggregate a 100 000 events, l'application n'a à chaque fois qu'à appliquer les quelques events à rattraper qui n'avaient pas encore été appliqués sur la BDD dénormalisée servant de cache.
    - Attention cependant, avant d'utiliser une technique avancée comme celle du snapshot il faut :
      - S'assurer qu'on a un vrai problème de performance (+ de 10 000 events par aggregate).
      - Vérifier les limites de notre aggregate et voir si il n'est pas trop gros et qu'on ne pourrait pas plutôt le scinder.
  - L'event sourcing **scale très bien** : vu qu'on a une séparation claire des données d'event par aggregate, on peut très bien sharder par identifiant d'aggregate.
  - Le CQRS (décrit au chapitre suivant) peut répondre à la problématique de performance de par sa nature.
- Comment **supprimer des données** de l'event store (par ex pour des considérations légales) ?
  - L'idée de l'event sourcing c'est qu'on ne supprime pas les données d'event, c'est la condition pour toujours pouvoir rejouer, mais aussi la condition pour qu'elles ne soient jamais corrompues.
  - On peut utiliser la technique du **forgettable payload** : on stocke dans notre event une version chiffrée des donnée sensibles, et on stocke une paire _clé de chiffrement / ID de l'aggregate_ dans un stockage key-value à part. Quand on a envie d'effacer une des données sensibles, il suffit de supprimer sa clé.
- Ne pourrait-on pas continuer à utiliser une BDD relationnelle comme BDD principale, et écrire en plus les events soit dans un log, soit en utilisant un database trigger pour écrire dans une table en même temps ?
  - C'est une mauvaise idée parce que le log qui ne serait pas la source de vérité finirait par se dégrader au fil du temps, parce qu'on y ferait moins attention.
  - Et en plus il est difficile de se donner du mal à bien représenter le sens des events dans ceux-ci si ils ne sont pas la source de vérité et que cette info n'est pas destinée à la BDD principale.

### 8 - Architectural Patterns

- Jusqu'ici on s'est intéressé aux patterns tactiques pour la logique business, ici on dézoome un peu et on s'intéresse à la relation entre les diverses parties du code du système (la logique business est une de ces parties).
- Si on ne maintient pas une limite claire entre le code business et le reste, on risque d'avoir une logique diffuse un peu partout.
  - Ça rend le code difficile à changer parce qu'il faut passer partout.
  - Ca fait qu'on peut oublier certains endroits et avoir des inconsistances.
- On voit dans ce chapitre 3 architectural patterns :

  - **Layered architecture**
    - C'est un des patterns les plus courants, il s'agit d'avoir 3 couches.
    - 1- la **couche de présentation** représente l'interface utilisateur, mais de nos jours c'est plus large : Interface graphique, interface en ligne de commande, API programmatique ou réseau.
      - On l'appelle parfois aussi la _user interface layer_.
    - 2- la **couche business** : c'est là où on a les patterns décrits avant : active record, domain model pattern etc.
      - On l'appelle parfois aussi le _domain layer_ ou le _model layer_.
    - 3- la **couche data access** où on stocke et manipule des données dans des bases de données, dans du cloud etc.
      - On l 'appelle parfois aussi l'_infrastructure layer_.
    - Côté dépendance, la couche de présentation n'a accès qu'à la couche business, et la couche business qu'à la couche data access : `presentation -> business -> data access`
    - Ce pattern est souvent étendu avec une **couche additionnelle appelée service** : elle va venir se placer entre la présentation et le business. `presentation -> service -> business -> data access`
      - Le service layer permet de faire la logique d'orchestration autour de la couche business, et permet de découpler davantage les couches présentation et business.
      - Par exemple, on peut avoir un controller (du pattern MVC) qui réceptionne un appel API, et qui va simplement faire appel au bon service, puis renvoyer la réponse de celui-ci. Le service quant à lui va commencer une transaction, faire appel par exemple à un objet Active Record, et soit valider soit annuler la transaction avant de renvoyer le résultat.
      - Le service layer peut être utile dans le cas où la partie business est un Active Record, mais si c'est un Transaction Script alors il n'y a rien à abstraire vis-à-vis de la couche présentation.
      - On l'appelle parfois aussi l'_application layer_.
    - Attention à ne pas confondre la layered architecture avec une architecture **N-Tier** :
      - N-Tier fait référence au fait d'avoir des composants déployables indépendamment et donc qu'on considère séparés “physiquement”, qui n'ont pas le même cycle de vie : par exemple ce qui tourne dans un navigateur, ce qui tourne sur un serveur applicatif, le serveur de base de données etc.
      - Quand on parle de layered architecture, on parle bien des couches logicielles au sein d'une même entité physique, avec le même cycle de vie.
    - Etant donné la dépendance entre couche business et couche de data access, **la layered architecture est mal adaptée à du code business sous forme de domain model pattern**. Elle est par contre adaptée pour du transaction script ou de l'active record.
  - **Ports and adapters**

    - Elle est bien plus **adaptée au domain model pattern** parce que le code business va être découplé du reste.
    - On va mettre le business layer d'abord, sans qu'il ne dépende de rien, et ensuite on va regrouper tout ce qui est communication avec la BDD, frameworks, providers externes etc. dans une couche qu'on appellera infrastructure. Et enfin on ajoute une couche application layer (c'est plus parlant que service layer) au milieu.
      - `Business layer <- Application layer <- infrastructure layer`
    - Grâce au **Dependency Inversion Principle**, on va à chaque fois injecter les dépendances des couches supérieures dans les couches du dessous : la couche business ne connaît pas la nature concrète de la couche application, et la couche application ne connaît pas la nature concrète de la couche infrastructure.

      - C'est ici que la notion de “ports and adapters” prend son sens : On a des **ports** qui sont des interfaces mises en place par les couches du dessus, et des **adapters** qui sont des implémentations concrètes faites par les couches du dessous, et passés en paramètre des fonctions des couches du dessus.
      - Exemple :

        ```typescript
        // le port (couche business)
        interface IMessaging {
            void publish(Message payload);
        }

        // l'adapter (couche infrastructure)
        public class SQBus implements Imessaging {
            void publish(Message payload) {
                // ...
            }
        }
        ```

        - Ici la classe adapter qui respecte le contrat (le port) établi par la couche business pourra être donnée en paramètre dans les fonctions de la couche business où elle accepte un objet `Imessaging`. Elle pourra utiliser l'objet sans savoir exactement ce qu'il fait (publier dans un bus, mettre l'info simplement dans une variable etc..).

    - On l'appelle aussi (avec quelques variations) : _hexagonal architecture_, _onion architecture_ ou encore _clean architecture_.
      - Dans ces variantes, l'application est parfois appelée _service layer_ ou encore _use case layer_.
      - Et le business layer est parfois appelé _domain layer_, ou encore _core layer_.

  - **CQRS** (Command-Query Responsibility Segregation)
    - Cette architecture est similaire au ports & adapters pour ce qui est de la place centrale du code business indépendant du reste. La différence se trouve dans la manière de gérer les données : on va vouloir adopter un “polyglot modeling”, c'est-à-dire plusieurs formes dénormalisées de la donnée pour la lecture.
    - A l'origine le CQRS a été pensé pour répondre au problème de l'event sourcing, en fournissant la possibilité de lire le state actuel des données directement, au lieu d'avoir à repartir de la forme primaire (qui est la liste d'événements) et de rejouer tous les events pour obtenir le state actuel.
      - Mais il est utile aussi sans event sourcing, c'est-à-dire dans le cas classique où la BDD “source de vérité” est par exemple relationnelle.
    - Comme l'indique le nom, on va faire une ségrégation (une séparation) entre les commands (les écritures) et les queries (les lectures).
      - D'un côté les **commands sont faites vers la base de données principale**, source de vérité, et fortement cohérente (les invariants sont respectés, les règles business validées etc.).
      - De l'autre les **queries sont faites vers des bases de données dérivées**, utiles pour chaque cas particulier.
        - Par exemple des data warehouses sous forme de colonnes pour l'analyse, des BDD adaptées à la recherche, des BDD relationnelles pour avoir le state des aggregates tout de suite etc.
        - Les BDD de lecture peuvent aussi être des fichiers, ou encore des BDD in-memory. Et on peut normalement les supprimer et les reconstituer à nouveau à partir de la BDD principale.
    - Il faut que les BDD secondaires soient mises à jour à chaque fois que la BDD principale est modifiée. On appelle ça les **projections**.
      - Il y a les **projections synchrones** :
        - Il s'agit d'un modèle de catch-up subscription.
          - Quand une requête est faite à la BDD de lecture, le système de projection va faire une requête vers la BDD principale pour obtenir tous les changements de la donnée voulue jusqu'au précédent checkpoint.
          - Les nouvelles données sont utilisées pour mettre à jour la BDD de lecture.
          - Le système de projection retient le nouveau checkpoint à partir duquel il faudra faire la prochaine mise à jour de la donnée en question.
        - Pour que ça puisse fonctionner, il faut que la BDD principale note toutes les modifications faites à l'ensemble des entrées qu'elle possède, avec des checkpoints (des versions) pour chacune.
          - NDLR : Ca ressemble à des domain events parce que chaque changement est noté en BDD. Mais pour que ça en soit il faut aussi que la signification métier de chaque changement soit noté, et pas juste “telle entrée est modifiée à telle valeur”. C'est dans cette mesure que le CQRS peut être utilisé même sans event sourcing.
      - Et les **projections asynchrones** :
        - Il s'agit d'un mécanisme de souscription :
          - La BDD principale publie chacun de ses changements dans un message bus, auquel souscrivent les projection engines de chaque BDD secondaire..
          - A chaque message chaque BDD secondaire est mise à jour.
        - Ce modèle est plus scalable et plus performant, mais il est soumis aussi à des problématiques de consistance des données des BDD secondaires, et une plus grande difficulté à les reconstruire.
          - Il est donc conseillé de rester sur le modèle synchrone et éventuellement construire une projection asynchrone par dessus.
    - Les queries ne peuvent pas écrire de données, par contre les commands qui écrivent les données peuvent aussi retourner des valeurs, notamment le statut de succès ou d'échec de la commande, avec l'erreur spécifique.
    - Le CQRS est adapté au domain model pattern, il est utile quand on veut avoir plusieurs modèles de données adaptés à plusieurs besoins. Il est aussi adapté à l'event sourced domain model pattern pour lequel il est quasi obligatoire.

- **Ces patterns doivent être appliqués au cas par cas pour les besoins de chaque brique logicielle**. Il ne faut pas choisir un seul pattern pour tout le système, ni même forcément un seul pattern pour un même bounded context, dans la mesure où plusieurs subdomains peuvent exister dans un même bounded context.

### 9 - Communication Patterns

- Dans ce chapitre il est question des patterns de communication entre briques logicielles distinctes et ayant chacune leur bounded context. C'est la mise en application technique du chapitre 4.
- Pour **implémenter un anticorruption layer ou un open-host service**, on peut procéder de plusieurs manières :
  - En mode **stateless** : chaque requête est autonome. Dans ce cas, le bounded context contient le layer de transformation.
    - On va utiliser le pattern Proxy pour que la requête passe d'abord par le layer (le proxy), puis soit traduite correctement vers la target.
    - La traduction peut être **synchrone**, auquel cas le proxy est dans le même composant.
      - On peut aussi vouloir utiliser un **API gateway pattern**, en ayant un composant externe tenant le rôle du proxy et permettant d'intégrer plusieurs flux vers une même target.
        - Dans ce cas, on pourra utiliser une solution open source comme **Kong** ou **KrakenD**, ou une solution payante comme **AWS API Gateway**, **Google Api-gee** ou **Azure API Management**.
    - La traduction peut aussi être **asynchrone**, auquel cas le proxy est un composant qui souscrit à des events, et envoie des messages.
      - Il pourrait en plus filtrer certains messages non pertinents.
      - Dans le cas d'un Open-Host Service, le modèle asynchrone est très pratique parce qu'il permet de **traduire correctement les événements du langage privé vers les événements du langage public**.
        - Et même de filtrer certains événements qui n'ont d'utilité qu'au sein du bounded context et pas pour l'intégration avec les autres.
  - En mode **statefull** où le layer possède sa BDD et implémente une logique complexe.
    - On a le cas où on veut **agréger les données entrantes**. Soit parce qu'on veut les traiter par batchs, soit parce qu'on veut créer un plus gros message à partir de plus petits messages arrivant.
      - Le BDD propre au proxy sert alors à gérer ce processus d'agrégation.
      - Parfois on peut utiliser un outil tout fait comme **Kafka**, **AWS Kinesis**, ou une solution de batch comme **Apache Nifi**, **AWS Glue**, **Spark**.
    - On a aussi le cas où on veut **unifier des sources de données**. Par exemple un backend for frontend qui va chercher des données chez plusieurs autres bounded contexts pour les afficher.
- A propos de l'**implémentation technique des domain events émis par les aggregates** et consommé par les autres composants.
  - Une façon de faire triviale mais mauvaise serait de laisser l'aggregate faire l'ajout de l'event dans le message bus.
    - L'event serait émis avant même que la transaction (qui a lieu dans le layer applicatif, après que le code de l'aggregate ait été exécuté) n'ait été commitée. On pourrait alors avoir des composants avertis d'une chose qui n'est pas encore vraie en DB.
      - Voir même qui ne sera jamais vraie en DB si la transaction échoue.
  - Une variante serait d'émettre l'event dans le layer applicatif juste après le commit dans la BDD.
    - Mais là encore que se passerait-il si la publication de l'event ne marchait pas ? Ou encore si le serveur crashait juste avant la publication de l'event ? On aurait la transaction validée mais pas d'event pour prévenir les autres.
  - Le pattern **Outbox** permet d'adresser ces cas.
    - 1- Le state et l'event sont ajoutés dans la BDD en une seule transaction.
      - L'event peut être ajouté dans une table différente si c'est possible. Si c'est une database NoSQL qui ne le permet pas, alors on peut le rajouter dans l'état de l'aggregate.
    - 2- Un message relay va s'occuper de traiter les events non publiés de la BDD en envoyant le message dans le message bus.
      - Le relay peut soit poller régulièrement la BDD (il faut faire attention à ne pas la surcharger), soit on peut utiliser les fonctionnalités proactives de la BDD pour déclencher le relay.
    - 3- Une fois que l'event est envoyé, le message relay va enlever l'event de la BDD, ou le marquer comme publié.
      - A noter que le relay garantit la publication du message au moins une fois : si il crash lui-même après avoir publié l'event mais avant de l'avoir enlevé de la BDD, il va le republier une 2ème fois.
- Le pattern **Saga** permet d'adresser les cas où on doit **écrire dans plusieurs aggregates**, vu que par principe on a décidé qu'une transaction ne devait écrire que dans un seul aggregate.
  - Le saga **va écouter des events émis par certains composants, et déclencher des commandes sur d'autres**. Si la commande échoue, le saga est responsable de déclencher des commandes pour assurer la logique business.
  - Exemple : imaginons deux entities qu'on ne veut pas mettre dans le même aggregate parce qu'elles sont trop peu liées entre elles : une campagne de publicité et un publisher.
    - On veut que l'activation d'une campagne déclenche sa publication auprès du publisher, et qu'en fonction de la décision du publisher, elle soit soit validée soit annulée.
    - Le saga va écouter l'event d'activation de l'aggregate de la campagne, et déclencher la commande de publication auprès de l'aggregate du publisher. Et il écoute aussi les events de validation ou d'annulation auprès de l'aggregate du publisher, pour déclencher une commande auprès de celui de la campagne en retour.
  - La logique de la saga peut aussi nécessiter de garder en mémoire son état, par exemple pour gérer correctement les actions de compensation. Dans ce cas les events de la saga peuvent être mis en BDD, et la saga peut être implémentée elle-même comme un event-sourced aggregate.
    - Dans ce cas il faut séparer la logique d'exécution des commandes de la mise à jour du state de la saga, et utiliser le même principe que pour l'outbox pattern. On aura un message relay qui garantira l'exécution de la commande même si on échoue à quelque étape que ce soit.
  - Attention tout de même à ne pas utiliser les saga pour compenser des limites d'aggregates mal pensées. L'action du saga étant asynchrone, les données seront eventually consistent entre-elles. **Les seules données fortement consistantes sont celles qui sont dans un même aggregate**.
- Le **Process Manager** se base sur le même principe que la saga, mais là où la saga associe juste un event à une commande, le process manager va implémenter une logique plus complexe liée à plusieurs events pour choisir les commandes à déclencher.
  - Il est implémenté sous forme de state-based ou event-sourced aggregate avec une persistance de son état en BDD.
  - Là où la saga est déclenchée implicitement quand un événement qu'elle doit écouter se produit, le process manager est instancié par l'application et s'occupe de mener à bien un flow en plusieurs étapes.
  - Exemple : la réservation d'un voyage business commence par la sélection du trajet le plus optimal, puis la validation par l'employé. Dans le cas où l'employé préfère un autre trajet, le manager doit valider. Puis l'hôtel pré-approuvé doit être réservé. Et en cas d'absence d'hôtels disponibles, l'ensemble de la réservation est annulée.
  - Là encore, vu qu'on a des commandes et un état, il vaut mieux implémenter l'outbox pattern pour jouer les commandes et mettre à jour l'état de manière sûre.

## Part III : Applying Domain-Driven Design in Practice

### 10 - Heuristics

- Ce chapitre donne des **heuristiques**, c'est-à-dire des règles à appliquer qui marchent dans la plupart des cas.
- **Bounded contexts**
  - Recomposer différemment des bounded contexts quand on se rend compte que le découpage n'est pas bon coûte cher. Il vaut donc mieux **commencer par des bounded contexts grands**, et les redécouper plus finement par la suite.
  - Cette règle s'applique en particulier pour les bounded contexts contenant un core subdomain, qui change par nature beaucoup. Il peut être sage de laisser d'autres subdomains avec lesquels il réagit beaucoup dans le même bounded context.
- **Business logic patterns**
  - Si le subdomain trace des transactions monétaires, ou doit permettre une traçabilité, ou encore des possibilités d'analyse approfondies des données, alors il faut choisir l'**event sourced domain model** pattern.
  - Sinon, si la logique business est quand même complexe (core subdomain), alors il faut choisir le **domain model** pattern classique.
  - Sinon, si la logique business inclut des transformations de données complexes, alors il faut choisir l'**active record** pattern.
  - Et sinon, il faut se rabattre sur le **transaction script** pattern.
  - Concernant le fait de savoir si une logique business est complexe :
    - Elle l'est si elle contient des règles business compliquées, des invariants et des algorithmes compliqués. On peut s'en rendre compte à partir de la validation des inputs (?).
    - Elle l'est aussi si l'ubiquitous language est compliqué. Et à l'inverse s'il décrit surtout des opérations CRUD, alors elle ne l'est pas.
    - Si on se retrouve à avoir une différence entre le pattern qu'on veut utiliser pour répondre à la nature de la logique business, et la catégorie de notre subdomain, alors c'est l'occasion de s'interroger sur la pertinence de notre catégorisation.
      - Mais il ne faut pas oublier non plus que l'avantage compétitif peut ne pas résider dans la technique.
- **Architectural patterns**
  - L'event sourced domain model nécessite le **CQRS**. Dans le cas contraire, on est vraiment limité : on ne pourra qu'obtenir les entities par leur ID.
  - Le domain model classique nécessite le **ports & adapters**, sinon on va avoir du mal à isoler la logique business de la persistance.
  - L'active record va bien avec une **layered architecture à 4 couches**, avec la couche service qui va contenir la logique qui manipule l'active record.
  - Le transaction script peut être implémenté avec une simple **layered architecture à 3 couches**.
  - Enfin, même dans le cas où on n'a pas choisi l'event sourcing pour notre logique business, on peut quand même adopter le CQRS dans le cas où on aurait besoin d'avoir la donnée sous plusieurs formes différentes.
- **Testing strategy**
  - La **testing pyramid** (beaucoup de unit tests, moins de tests d'intégration, et peu de tests end to end) est bien adaptée aux **domain model** patterns (event sourced ou classique). Les value objects et les aggregates font de parfaits units autonomes.
  - Le **testing diamond** (peu de test unitaires, beaucoup de tests d'intégration, et peu de tests end to end) est bien adapté à l'**active record** pattern. La logique business étant éparpillée à travers le layer service et business, il est pertinent de les tester ensemble. (avec la BDD du coup ?)
  - La **reversed testing pyramid** (peu de unit tests, plus de tests d'intégration, et beaucoup de tests end to end) est bien adaptée au **transaction script**. La logique business étant simple, et le nombre de couches faible, on peut directement tester de bout en bout.
- L'auteur a déjà rencontré des équipes qui utilisaient par exemple l'event sourced domain model pattern partout. Pour autant il ne le conseille pas, et a rencontré beaucoup plus d'équipes qui s'en sortaient bien avec ces règles d'heuristiques.

### 11 - Evolving Design Decisions

- **Identifier les subdomains** a des implications importantes. Mais il faut régulièrement questionner leur statut et éventuellement le réévaluer.
  - **Core -> supporting** : Si l'avantage compétitif apporté par le subdomain n'est plus justifié, l'entreprise peut décider de réduire la complexité du subdomain au minimum pour se concentrer sur ceux qui ont plus de valeur.
  - **Core -> generic** : On peut avoir un procédé qui constituait un avantage compétitif, qui se fait dépasser par un acteur qui se met à fournir le fournir sous forme de service sur le marché. Dans ce cas on se retrouve contraint d'utiliser sa solution pour rester à la page, et notre core subdomain devient un generic subdomain.
  - **Supporting -> core** : Il s'agit de cas où un supporting subdomain dont la logique se complexifie. Si elle n'apporte pas d'avantage compétitif, alors il n'y a pas de raison à cette complexification et il faut la corriger. Sinon c'est qu'on se retrouve avec un core subdomain.
  - **Supporting -> generic** : On peut imaginer qu'on ait créé un système simple pour gérer quelque chose. Puis il apparaît un système open source qui fait la même chose, mais a aussi des fonctionnalités que l'entreprise n'avait pas priorisé étant donné la faible valeur apportée par ce système. Mais si c'est à faible coût alors pourquoi pas : l'entreprise abandonne la solution maison pour la solution open source.
  - **Generic -> core** : Une entreprise utilisant une solution externe se retrouve limitée par celle-ci. Elle décide finalement d'implémenter une solution maison qui réponde vraiment à son besoin, et grâce à ça de mieux rendre son service. Un exemple est Amazon qui a créé une solution d'infrastructure web maison pour ses besoins, et a fini par la commercialiser sous le nom AWS.
  - **Generic -> supporting** : Pour la raison inverse de passer de supporting à generic, on peut décider que la solution open source ne vaut plus le coup parce qu'elle coûte trop cher à intégrer, et revenir à une solution maison simple.
- Quand **ajouter des fonctionnalités devient douloureux**, c'est un signe qu'on se retrouve avec **des patterns tactiques qui ne sont plus adaptés** à la complexité de notre logique business.
  - Par exemple, si un supporting subdomain se met à avoir une logique de plus en plus complexe, il est peut être temps de le transformer en core.
- Si on a fait nos choix en conscience, et qu'on a connaissance des différents patterns existants, migrer n'est pas si difficile.
  - **Transaction script -> active record** :
    - Il s'agit dans les deux cas de logique procédurale. Quand on a des structures de données compliquées, il faut les repérer et les encapsuler la logique de lecture/écriture dans des objets.
  - **Active record -> domain model** :
    - On pense à le faire quand on remarque que la logique business qui manipule les active records devient complexe, et qu'on se trouve face à des duplications qui mènent à des inconsistances à travers la codebase.
    - On **identifie d'abord les value objects** immuables dans notre logique. On identifie aussi la logique qui pourrait aller dans ces objets et on la migre dedans.
    - Ensuite on essaye de **délimiter les données qui doivent être mises à jour ensemble**.
      - On peut mettre les setters des active records privés, et constater où ils sont appelés dans le code.
      - On peut alors déplacer tout le code qui est cassé à l'intérieur de l'active record. On obtient un bon candidat pour un aggregate.
      - On va ensuite examiner la hiérarchie d'entities qu'il faut construire, et extraire éventuellement plusieurs aggregates du code : un aggregate sera la plus petite unité dont les données doivent être fortement consistantes.
      - Enfin on se débrouille pour que les méthodes de l'aggregate ne soient appelables que par l'aggregate root, et que seule l'interface publique soit appelable de l'extérieur.
  - **Domain model -> event sourced domain model** :
    - Il faut commencer par modéliser les domain events.
    - La partie la plus compliquée va être de **gérer les events passés qui n'existent pas**. On a deux manières de le faire :
      - **Générer de fausses transitions passées**.
        - On va analyser chaque aggregate et on va imaginer une manière réaliste d'obtenir l'état actuel par une succession de domain events successifs.
        - On va alors générer ces events en base et faire comme si notre aggregate avait toujours été event-sourced.
        - L'avantage c'est qu'on pourra toujours faire des projections, y compris en partant de zéro.
        - Le désavantage c'est que les events avant la migration seront inventés, et pourraient aussi induire en erreur sur une analyse.
      - **Modéliser des events de migration**.
        - L'autre solution c'est d'accepter explicitement qu'avant un certain point on n'a pas de données. On va alors créer un **event initial** pour chaque aggregate, qui va simplement avoir pour effet de mettre la valeur actuelle de l'état de toutes les valeurs de l'aggregate.
        - L'avantage c'est qu'on ne pourra pas avoir de fausses projections.
- Les **changements organisationnels peuvent affecter les patterns d'intégration des bounded contexts** :
  - Par exemple, si une seule équipe gérait un bounded context avec plusieurs subdomains, l'arrivée d'une 2ème équipe mènera à la séparation du bounded context en deux, parce qu'un bounded context ne doit pas être géré par deux équipes différentes.
  - Autre exemple : si un des bounded contexts est affecté à une autre équipe, et que la communication n'est pas bonne, on va passer du partnership à du customer-supplier.
  - Et quand on a encore plus de problèmes de communication, il vaut parfois mieux dupliquer la fonctionnalité et passer sur du separate ways.
- Il faut **prendre soin de la connaissance du domaine**.
  - En particulier pour les core subdomains, la modélisation du domaine est complexe et change souvent. Il faut donc régulièrement revoir les value objects, les aggregates etc.
  - Parfois la connaissance du domaine se perd, la documentation n'est plus à jour, les gens bougent etc. Il faut alors la retrouver, par exemple avec des sessions d'event storming.
- A mesure que le **code grossit**, les décisions de design deviennent obsolètes et doivent être adaptées. Si on ne le fait pas, on va obtenir un **big ball of mud**.
  - Il faut identifier et régulièrement éliminer l'**accidental complexity** résultant des décisions design obsolètes, et n'adresser que l'**essential complexity** inhérente au domaine avec les outils tactiques du DDD.
  - Les **subdomains** qui deviennent de plus en plus gros peuvent être **distillés pour en mettre en évidence d'autres**. Il faut en particulier le faire avec les core subdomains pour que la partie core soit circonscrite à ce qui est vraiment nécessaire.
  - Les **bounded contexts** peuvent aussi être revisités quand ils grossissent trop :
    - On peut simplifier le modèle du bounded context en extrayant un bounded context chargé d'un problème spécifique qui a grossi.
    - Parfois on se rend compte qu'un bounded context n'arrive pas à faire une action sans être dépendant d'un autre bounded context. On peut alors revoir ses limites pour augmenter son autonomie.
  - Les **aggregates** doivent rester des unités incluant **le plus petit set de données possible qui doivent être fortement consistantes entre elles**.
    - Au fil du temps on peut être amené à ajouter des choses dans des aggregates existants parce que c'est plus pratique.
    - Il faut donc régulièrement revérifier le contenu des aggregates. Et ne pas hésiter à extraire des fonctionnalités dans un nouvel aggregate pour simplifier le premier.
    - On se rend souvent compte que le nouvel aggregate révèle un nouveau modèle et amène à la création d'un nouveau bounded context.

### 12 - EventStorming

- L'event storming est un atelier qui permet de **modéliser ensemble un process business particulier.**
  - Il s'agit de construire la story du business process concerné, à travers une timeline d'events qu'on fait apparaître sur un tableau au cours de l'atelier.
- Les membres participant à l'atelier doivent être **le plus divers possible** (devs, domain experts, product owners, UI/UX, CSM etc.). Mais il vaut mieux ne pas dépasser 10 personnes.
- Côté matériel :
  - Ça se passe sur un mur, avec un gros rouleau de papier sur lequel on va pouvoir coller des post-its.
  - Il faut des post-its de couleur, chaque couleur représentant un concept particulier. Et des marqueurs pour écrire dessus.
  - L'atelier dure 2 à 4 heures, donc il faut prévoir de quoi grignoter.
  - Il faut une salle spacieuse, et rien qui ne gène pour que chaque participant accède librement au tableau.
    - Il ne faut pas de chaises, les participants sont debout.
- L'atelier se passe en 10 étapes, pendant lequel le modèle est enrichi collectivement :
  - **Étape 1 - Unstructured Exploration** : il s'agit d'une étape en mode brainstorming : tous les participants prennent des **post-its oranges**, et **écrivent dessus des events liés au process business** auquel on s'intéresse.
    - Les events sont collés sur le tableau sans se soucier de l'ordre ou de la redondance.
    - Les events sont formulés au passé.
    - Exemple d'events : “Notification sent”, “Destination chosen”, “Order shipped”.
  - **Étape 2 - Timelines** : Les participants vont **organiser les post-its dans le bon ordre**, en commençant par le “happy path scenario”, puis par les cas d'erreur.
    - On peut tracer des flèches à partir d'un post-it pour mener à plusieurs flows de posi-its possibles.
    - C'est aussi le moment d'enlever les doublons.
  - **Étape 3 - Pain Points** : On va marquer les points qui **requièrent une attention particulière** : des bottlenecks, des étapes manuelles à automatiser, de la doc ou du domain knowledge manquant etc.
    - On fait ça avec des **post-its roses, tournés de 45°** pour être “en diamant”.
    - Cette étape y est dédiée spécifiquement, mais le facilitateur doit aussi faire attention aux pain points levés tout au long de l'atelier et les marquer.
  - **Étape 4 - Pivotal Events** : On va s'intéresser à des events **marquant une étape importante** dans le processus, et tracer **une barre verticale** le long du tableau, pour délimiter l'avant et l'après cet event.
    - Exemple : dans un flow d'achat, “order initialized”, “order shipped” et “order delivered” peuvent être des events importants.
  - **Étape 5 - Commands** : On va ajouter des commandes sur des **post-its bleus**, juste avant l'event qui résulte de la commande.
    - Les commandes sont formulées à l'impératif.
    - Exemple : “Publish campaign”, “Submit order”.
    - Si l'actor (la persona du business domain, ex : client, administrateur etc.) qui émet la commande est évident, on l'ajoute sur le post-it de la commande avec un **post-it jaune**.
  - **Étape 6 - Policies** : Presque à chaque fois il y a des commandes qui n'ont pas d'actor associé. On peut alors leur ajouter une **automation policy** qui consiste à les **déclencher lors d'un domain event, sans intervention manuelle**.
    - On ajoute ces policies avec des **post-its violets**, et on va les placer entre la commande et l'event concernés.
    - On peut ajouter d'éventuels critères de déclenchement. Par exemple, si un event “complaint received” doit déclencher la commande “escalate” seulement si ça vient d'un client VIP, on peut mettre sur le post-it de policy que c'est seulement si client VIP.
    - Dans le cas où l'event et la commande sont loin, ne pas hésiter à tracer une flèche pour les joindre.
  - **Étape 7 - Read Models** : On ajoute les “read models”, c'est-à-dire les interfaces utilisateur (écran, rapport, notification etc.) utilisées par les actors pour prendre leurs décisions d'exécuter les commandes.
    - Ils seront ajoutés sur des **post-its verts**.
    - Exemple de read model : “Shopping cart”, pour un actor “customer”, qui va actionner la commande “submit order”.
  - **Étape 8 - External Systems** : On va représenter les intéractions avec des systèmes externes, c'est-à-dire qui ne font pas partie du domaine qu'on est en train d'explorer.
    - Ils seront ajoutés sur des **post-its roses**.
    - Ça peut être un composant qui déclenche une commande, par exemple un CRM qui déclenche l'ordre d'expédier une commande?
    - Ca peut aussi être un composant qu'on va notifier, par exemple notifier le CRM suite à un event qui dit que l'expédition est approuvée.
    - **A la fin de cette étape, toutes les commandes devront avoir leur origine associée** : soit un actor, soit une policy, soit un système externe.
  - **Étape 9 - Aggregates** : On va pouvoir organiser les commandes et events représentés en aggregates, sachant qu'un aggregate reçoit des commandes, et émet des events.
    - On va ajouter de **longs post-its jaunes verticaux** pour représenter l'aggregate. On va les placer entre les commandes et les events.
  - **Étape 10 - Bounded Contexts** : On va finalement essayer de **regrouper les aggregates entre eux**, soit parce qu'ils représentent des fonctionnalités liées entre elles, soit par leur couplage via des policies.
    - Ces groupes d'aggregates seront de bons candidats pour des bounded contexts.
    - On peut les matérialiser avec des pointillés entourant les groupes d'aggregates liés.
- Le créateur de l'event storming lui-même (Alberto Brandolini) dit qu'il s'agit simplement de conseil sur la manière de mener l'atelier. On peut très **bien l'adapter comme ça nous convient**.
  - L'auteur du livre applique d'abord les étapes 1 à 4 sur le domaine tout entier pour obtenir une vision large du domaine et identifier les process business.
  - Ensuite il organise un atelier pour chaque process business pertinent, en suivant cette fois toutes les étapes.
- La **vraie valeur de l'event storming c'est surtout le processus en lui-même**, et le fait qu'il permette la communication entre les différentes parties prenantes, en leur permettant d'aligner leur modèle mental, de découvrir d'éventuels modèles en conflit, et de formuler un ubiquitous language.
  - Les objets obtenus sur le tableau sont un bonus. Tous les ingrédients sont là pour implémenter un event-sourced domain model pattern si on est bien face à un core subdomain et qu'on veut faire ce choix.
- L'utilisation de l'event storming permet non seulement de construire l'ubiquitous language, et construire le domain model, mais il est aussi utile pour **explorer de nouvelles fonctionnalités, retrouver de la connaissance du domaine perdue, onboarder de nouvelles recrues** en leur donnant de la connaissance du domaine.
  - En revanche, l'event storming sera peu utile si le process business examiné est simple et évident.
- Tips pour les facilitateurs :
  - Au début de l'atelier, donner une vision d'ensemble du processus, et présenter les éléments de modélisation qui vont être utilisés tout au long (en construisant une légende avec les post-its de chaque couleur par exemple).
  - Si on sent un ralentissement du dynamisme du groupe, on peut relancer avec une question par exemple, ou peut être que c'est le moment de passer à l'étape suivante.
  - Tout le monde doit participer, si certain(e)s sont en retrait, essayer de les inclure en leur posant une question sur l'état actuel du modèle.
  - L'activité étant intense, il y aura au moins un break.
    - Il ne faut pas reprendre l'activité tant que tout le monde n'est pas de retour.
    - On peut reprendre en récapitulant le modèle tel qu'il a été défini pour le moment.
  - Les ateliers en remote sont plus difficiles à mener, l'auteur conseille de se limiter à 5 personnes.
    - Parmi les outils existants, miro.com est celui qui est le plus connu au moment de l'écriture du livre.
  - Pas besoin d'être ceinture noire pour faciliter une session : on apprend en faisant.

### 13 - Domain-Driven Design in the Real World

- Les techniques du domain driven design **apporteront le plus de bénéfices aux brownfield projects** : les projets ayant déjà un business et s'étant éventuellement embourbés sous forme de big ball of mud.
- Pas besoin que tous les devs soient des ceintures noires du DDD, ni d'appliquer toutes les techniques que le DDD propose.
  - Par exemple, si on préfère d'autres patterns tactiques que ceux du DDD, c'est tout à fait OK.
- Il faut d'abord **commencer par l'analyse stratégique** :
  - D'abord comprendre le domaine avec une vue haut niveau (qui sont les clients, que fait l'entreprise, qui sont les compétiteurs etc.).
  - Puis **identifier les subdomains**. Pour ça on peut partir de l'organigramme de l'entreprise.
    - Pour les core subdomains, on peut se demander ce qui différencie l'entreprise de ses compétiteurs :
      - Peut-être un algorithme maison que les autres n'ont pas ?
      - Peut être un avantage non-technique comme la capacité à embaucher du personnel top niveau, ou de produire un design artistique unique ?
      - Une heuristique qui fonctionne bien est de **trouver les composants qui sont dans le pire état** : ceux qui sont devenus des big balls of mud, que les devs détestent mais que les dirigeants refusent de faire réécrire à cause du risque business.
    - Pour les generic subdomains il s'agit simplement de trouver les solutions prêtes à l'emploi, soit open source, soit payantes.
    - Les supporting subdomains sont les composants restants.
      - Ils peuvent être dans un mauvais état mais suscitent moins de plainte de la part des devs parce qu'ils sont moins souvent modifiés.
    - On n'est pas obligés d'identifier tous les subdomains. On peut commencer par les plus importants.
  - On peut ensuite **identifier et analyser les différents composants logiciels**.
    - Le critère ici c'est le cycle de vie : les différents composants sont ceux qu'on peut faire évoluer et déployer indépendamment des autres.
    - On peut alors regarder les patterns d'architecture utilisés pour chaque composant, et vérifier si un pattern complexe serait plus adapté, ou à l'inverse si on pourrait utiliser un plus simple ou même une solution existante.
    - On peut ensuite faire comme si ces composants étaient des bounded contexts, et tracer le context map avec la relation entre chacun d'entre eux.
      - On peut là aussi vérifier si les patterns d'intégration de bounded context sont améliorables : plusieurs équipes travaillant sur le même composant, implémentations de core subdomain dupliquées, implémentation de core subdomain sous-traitée, frictions à cause d'une mauvaise communication entre équipes etc.
  - On pourra par la suite utiliser l'event storming pour construire un ubiquitous language, et éventuellement retrouver de la connaissance du domaine perdue.
- Ensuite on peut **mettre en place une stratégie de modernisation** :
  - Il ne s'agira pas de tout réécrire parce que ça marche rarement, et que c'est supporté par le management encore plus rarement.
  - Il faut accepter que dans un grand système, tout ne sera pas bien designé, et **se concentrer sur les composants qu'on estime stratégiques**.
    - Mais pour ça il faut déjà s'assurer qu'on a bien une délimitation, au moins logique, entre les subdomains (code séparé sous forme de modules, namespaces, packages etc.).
    - Ne pas oublier aussi les bouts de code du subdomain qui seraient ailleurs, sous forme de stored procedures d'une BDD, ou sous forme de serverless functions.
  - On peut alors commencer à extraire des composants logiques en bounded contexts physiques, en commençant par ceux qui apportent le plus de valeur.
  - Il faut ensuite bien examiner les composants extraits et réfléchir à comment les moderniser :
    - Il faut examiner leur intégration vis-à-vis de la relation entre les équipes qui en ont la charge et de leur niveau de communication (partnership vers customer/supplier, shared kernel vers separate ways etc.).
    - Côté tactique, il faut se concentrer sur les composants qui apportent beaucoup de valeur (core) mais dont l'implémentation ne serait pas adaptée et rendrait difficile la maintenance, en utilisant plutôt un domain model pattern.
    - Il faut aussi ne pas oublier de construire un ubiquitous language avec les domain experts, notamment au travers d'ateliers d'event storming.
  - Concernant la modernisation de chaque bounded context extrait, on peut procéder de deux manières :
    - Le **strangler pattern** : on va créer un nouveau bounded context à côté de l'ancien, et toutes les nouvelles fonctionnalités iront dedans. On migrera aussi les anciennes progressivement jusqu'à ce qu'il ne reste plus rien du premier.
      - On peut mettre en place une façade devant les deux bounded contexts pour rediriger les appels vers l'un ou l'autre. Elle disparaît quand le composant legacy meurt.
      - Les deux bounded contexts peuvent temporairement partager une même base de données (chose qu'on ne fait pas d'habitude pour deux bounded contexts).
    - Le **refactoring progressif** : on va changer le modèle et les patterns progressivement au sein du bounded context comme expliqué au chapitre 11.
      - Il faut procéder par étapes, et ne pas sauter d'un transaction script à un event sourced domain model par exemple.
      - Il faut passer du temps à trouver les bonnes limites pour les aggregates, en particulier si on va jusqu'à l'event sourced domain model, les changer ensuite est plus difficile.
      - L'introduction du domain model pattern elle-même peut se faire en plusieurs étapes : par exemple commencer par trouver les objets immutables pour les extraire en value objects.
- Comment **introduire le DDD au sein de mon organisation** ?
  - Étant donné les changements importants, y compris organisationnels et d'implication des effectifs hors ingénierie, **avoir un appui du top management** peut beaucoup aider. Mais c'est plutôt rare.
  - Ceci dit, l'essentiel du DDD reste des pratiques d'ingénierie logicielle, donc on peut commencer à l'utiliser dans ses activités quotidiennes **même sans mise en place à l'échelle de l'organisation**.
    - On peut déjà commencer à **construire un ubiquitous language et l'utiliser**.
      - Écouter les domain experts parler, leur demander des clarifications, repérer les doublons et demander à ce qu'on n'utilise qu'un des termes.
      - Parler avec les domain experts plus souvent, pas forcément au cours de meetings formels. En général ils sont ravis de parler aux devs qui sont sincèrement intéressés par comprendre le domaine.
      - Utiliser la terminologie qu'on a constitué dans le code, et dans tous les autres supports ou échanges.
    - Pour les **bounded contexts**, l'important est de **comprendre les principes sous-jacents** pour pouvoir les utiliser :
      - Pourquoi créer plusieurs modèles utiles à chaque problème ? Parce qu'utiliser un très gros modèle est rarement efficace.
      - Pourquoi un bounded context ne doit pas avoir des modèles en conflit en son sein : à cause de la complexité que ça engendre.
      - Pourquoi plusieurs équipes ne devraient pas travailler sur un même bounded context ? A cause de la friction et de la mauvaise collaboration que ça engendre.
    - C'est la même chose pour les **patterns tactiques** : il faut comprendre la logique de chaque pattern et utiliser cette logique pour améliorer son design, plutôt que faire appel à l'argument d'autorité “DDD” qui ne mènera nulle part.
      - Pourquoi faire des limites transactionnelles explicites ? Pour protéger la consistance de la donnée.
      - Pourquoi une transaction DB ne peut pas modifier plus d'une instance d'un aggregate à la fois ? Pour être sûr que les limites de consistance sont correctes.
      - Pourquoi l'état d'un aggregate ne peut pas être modifié directement par un autre composant ? Pour s'assurer que toute la logique business est située au même endroit et pas dupliquée.
      - Pourquoi ne pourrait-on pas déplacer une partie de la logique d'un aggregate dans une _stored procedure_ ? Pour être sûr de ne pas dupliquer la logique, parce que la logique dupliquée dans un autre composant a tendance à se désynchroniser et mener à de la corruption de données.
      - Pourquoi essayer d'avoir des limites d'aggregates petites ? Parce que des limites transactionnelles larges augmentent la complexité de l'aggregate et impactent négativement la performance.
      - Pourquoi, à la place de l'event sourcing, ne pourrait-on pas écrire la donnée dans un fichier de log ? Parce qu'on n'aura pas de garantie de consistance de longue durée pour la donnée.
    - A propos de l'**event sourcing** en particulier, **expliquer le principe aux domain experts**, et en particulier le niveau d'insight qu'on acquiert sur la donnée, va en général les convaincre que c'est une bonne idée.

## Part IV : Relationships to Other Methodologies and Patterns

### 14 - Microservices

- Un **service** est une entité qui reçoit des données en entrée et envoie des données en sortie. Ca peut être de manière synchrone comme avec le modèle request/response, ou asynchrone comme avec le modèles basé sur les events.
  - Il a une **interface publique** qui décrit comment on peut communiquer avec. Elle est en général suffisante pour comprendre ce que le service fait.
- Un **microservice** est un service avec une **petite interface publique**.
  - En limitant son interface on le rend plus facilement compréhensible, et on réduit les raisons qu'il a de changer.
  - C'est aussi pour ça qu'un microservice possède sa propre base de données et ne l'expose pas.
  - NDLR : cette courte définition n'est pas partagée par tout le monde.
    - Dave Farley définit le microservice comme étant d'abord une unité déployable indépendamment, et donc conçue de telle manière qu'elle n'ait pas besoin d'être testée avec d'autres unités. cf. [vidéo 1](https://www.youtube.com/watch?v=zzMLg3Ys5vI), [vidéo 2](https://www.youtube.com/watch?v=bWZVx6TgVvc)
- Quand on se pose la question d'à quel point notre service devrait avoir une petite interface, il faut prendre en compte la **complexité locale** (la complexité interne de chaque microservice) et la **complexité globale**. (la complexité de l'ensemble résultant de l'interaction entre microservices).
  - Pour réduire au maximum la complexité globale, il suffit d'implémenter l'ensemble sous forme d'un unique service monolithique. La complexité locale est alors maximale, et le risque est de finir avec un **big ball of mud**.
  - Pour réduire au maximum la complexité locale, on pourrait mettre chaque fonction dans un microservice, avec sa base de données. La complexité globale est alors maximale, et on risque alors de se retrouver avec un **distributed big ball of mud**.
  - Il s'agit donc de trouver un **juste milieu entre complexité locale et globale**.
  - L'auteur évoque le concept de _depth_ proposé par John Ousterhout dans son livre _The philosophy of Software Design_ : un deep module a une petite interface mais une plus grande implémentation, alors qu'un shallow module a une grande interface comparé à son implémentation (donc beaucoup de choses sont exposées).
    - C'est la même chose pour les microservices, il faut les concevoir avec l'interface la plus petite possible, tout en ayant une implémentation importante en comparaison. **Il faut que le microservice en tant qu'entité d'encapsulation encapsule des choses**, sinon il ne fera qu'ajouter de la complexité accidentelle.
- Les microservices sont souvent confondus avec les **bounded contexts**.
  - Il est vrai qu'un microservice est forcément un bounded context : il ne peut pas être géré par deux équipes, et il ne peut pas y avoir plusieurs modèles en conflit en son sein.
  - En revanche, un bounded context peut être plus large que ce qui serait raisonnable pour un microservice, il peut contenir plusieurs subdomains tant qu'un même modèle (et un même ubiquitous language) permet d'en rendre compte.
  - Du coup :
    - Des ensembles trop vastes contenant des modèles en conflit mènent à du big ball of mud.
    - Des ensembles moins vastes avec un modèle unique sont des bounded contexts.
    - Des ensemble suffisamment petits pour avoir une petite interface publique sont des microservices et aussi des bounded contexts.
    - Si on découpe au-delà, on tombe sur du distributed big ball of mud.
- Parfois on a envie d'utiliser des **aggregates** comme microservices.
  - Il s'agit de regarder le couplage entre l'aggregate et les autres composants du subdomain : s'il communique souvent avec d'autres composants, si le changer mènerait souvent à changer d'autres composants etc. Plus ce couplage est fort, plus le microservice résultant sera “shallow”.
  - Parfois l'aggregate est suffisamment indépendant et ça marche bien, mais **la plupart du temps c'est une mauvaise idée** : l'aggregate se trouve être un découpage trop petit et mène au distributed big ball of mud.
- Enfin, une autre possibilité est d'utiliser les **subdomains** pour les microservices.
  - Les subdomains sont de bons candidats pour des deep modules.
    - Ils sont concentrés sur les use-cases plutôt que sur la manière dont ceux-ci seront implémentés.
    - Les use-cases sont dépendants les uns des autres.
    - Le subdomain agit sur un même jeu de données.
  - **Choisir les subdomains comme heuristique pour ses microservices est donc une bonne idée**.
    - Parfois il sera préférable de choisir un ensemble plus vaste qui sera un bounded context, parfois un ensemble plus restreint qui sera un aggregate. Mais la plupart du temps les subdomains feront l'affaire.
- Les **Open-Host Services**, et **Anticorruption Layers** peuvent contribuer à réduire davantage l'interface publique des microservices, en n'exposant qu'une version réduite du modèle interne.
  - Dans le cas de l'ACL, il peut être mis dans un service à part, réduisant donc l'interface publique du service consommateur qui se protège.

### 15 - Event-Driven Architecture

- L'**event driven architecture** est une architecture dans laquelle les composants souscrivent et réagissent à des événements de manière **asynchrone**, plutôt que sous forme de requête/réponse de manière synchrone.
  - Le pattern saga en est un exemple.
- On parle bien ici de communication entre composants (ie. bounded contexts). **Alors que l'event sourcing porte sur l'utilisation d'events à l'intérieur du BC, l'event driven architecture s'intéresse aux events utilisés pour communiquer entre BCs**.
- L'event et la commande sont tous deux des **messages**.
  - L'**event** décrit quelque chose qui s'est déjà passé et ne peut pas être annulé ou refusé.
  - La **commande** décrit quelque chose qui doit être fait, et qui pourrait être refusé, auquel cas on peut déclencher des commandes de compensation.
- On peut classer les events en 3 catégories :
  - 1- L'**event notification** est un event qui sert à notifier un composant de quelque chose, pour le **pousser à faire une query** dès qu'il est disponible.
    - On ne va pas mettre toute l'info dans l'event, étant donné qu'on s'attend à ce que le composant refasse une query quand il est prêt.
    - Avantages d'obliger à faire une query :
      - Ca peut être bien niveau sécurité : notifier avec des infos non sensibles dans l'event, et vérifier avec une autorisation plus forte quand la query explicite est faite.
      - Ça peut permettre au composant d'être sûr d'avoir des infos à jour au moment du processing vu que la query sera synchrone, contrairement à l'event.
      - Ça peut permettre de mettre en place une opération bloquante du point de vue concurrence pour que ce soit fait par une seule instance.
    - Exemple : ici on n'a quasi aucune info à part l'ID et un lien pour en savoir plus.
      ```json
      {
        "type": "mariage-recorded",
        "person-id": "01b9a761",
        "payload": {
          "person-id": "01b9a761",
          "details": "/01b9a761/mariage-data"
        }
      }
      ```
  - 2- L'**event-carried state transfer** (ECST) est un event qui va donner l'information complète permettant à un composant externe de **maintenir un cache de l'état interne de nos objets**.
    - On peut soit envoyer à chaque fois un snapshot complet de l'état d'un objet, ou alors ne renvoyer que les modifications de cet état et le receveur se débrouille pour maintenir la cohérence de l'état au fur et à mesure.
    - Avantages de maintenir un cache à distance :
      - On est plus tolérant aux fautes : le composant qui nous consomme peut continuer à fonctionner même si on est down.
      - On économise des requêtes : le composant consommateur n'a pas à faire de requête à chaque fois pour obtenir la donnée, et si elle ne change pas il n'y aura pas non plus d'event vu que le cache sera déjà à jour.
    - Exemple : ici on a la donnée qui a changé dans le state de la personne concernée, pour qu'on mette à jour notre cache.
      ```json
      {
        "type": "personal-details-changed",
        "person-id": "01b9a761",
        "payload": {
          "new-last-name": "Williams"
        }
      }
      ```
  - 3- Le **domain event** décrit un événement lié au business domain, et qui s'est produit dans le passé. Il est là dans un but de **modélisation du business domain** et pas spécialement pour des considérations techniques vis-à-vis des autres composants.
    - Par rapport à l'event notification :
      - Le domain event inclut toute l'info nécessaire pour décrire l'event, alors que l'event notification non.
      - Le domain event est utile en interne même si aucun composant externe s'y intéresse, alors que le but de l'event notification est uniquement l'intégration avec les composants externes.
    - Par rapport à l'ECST :
      - Le but est différent : le domain event décrit le fonctionnement du domaine, alors que l'ECST est là pour exposer l'état interne des objets pour des raisons techniques.
      - En s'abonnant à un type précis de domain event, on n'a pas du tout la garantie d'obtenir tous les changements d'un aggregate par exemple, contrairement à ce que permet l'ECST.
    - Exemple : on modélise l'événement qui s'est produit au plus près possible du domaine.
      ```json
      {
        "type": "married",
        "person-id": "01b9a761",
        "payload": {
          "person-id": "01b9a761",
          "assumed-partner-last-name": "true"
        }
      }
      ```
- Pour montrer qu'il ne suffit pas de saupoudrer un système d'events pour le transformer en event driven architecture, voici un exemple réel d'architecture produisant un distributed big ball of mud :
  - Imaginons un composant (bounded context) CRM utilisant l'event sourced domain model. Trois autres composants choisissent de consommer l'ensemble de ses domain events :
    - Le BC Marketing les transforme en état, puis les utilise pour se mettre à jour.
    - Le BC AdsOptimization fait quelque chose de similaire à Marketing pour d'autres besoins.
    - Le BC Reporting les consomme, puis attend 5 mn avant de faire une query vers AdsOptimization, pour espérer qu'AdsOptimization aura déjà traité les données liées à cet event.
  - On se retrouve avec 3 problèmes :
    - Un **couplage temporel** : Reporting attend 5 mn avant de faire sa requête, mais rien de garantit que ce sera suffisant. Si AdsOptimization est surchargé, qu'on a des problèmes réseau ou autres, Reporting risque de faire sa requête avant qu'AdsOptimization n'ait fini de processer les données liées à cet event.
      - Solution : plutôt que de faire consommer les events de CRM à Reporting, on peut faire en sorte que ce soit AdsOptimization qui envoie un **notification event** à Reporting quand il a processé quelque chose de nouveau, pour que Reporting fasse sa requête.
    - Un **couplage fonctionnel** : Marketing et AdsOptimization consomment tous deux les mêmes events, qu'ils transforment tous deux en objets avec état exactement de la même manière. Ça crée une duplication de logique dans ces deux bounded contexts.
      - Solution : CRM peut implémenter l'**open-host service** pour y implémenter la logique qui était dupliquée dans Marketing et AdsOptimization. Comme ça les consommateurs ne s'encombrent pas d'implémenter ça.
    - Un **couplage à l'implémentation** : vu que CRM est event-sourced, les autres BCs sont couplés à l'implémentation de CRM. A chaque fois qu'il change quelque chose, il faut qu'ils mettent à jour leur code aussi.
      - Solution : CRM, bien qu'event-sourced, n'a pas besoin d'exposer tous ses domain events. Il peut choisir publiquement d'exposer seulement certains d'entre eux, ou **choisir d'exposer un type d'event différent**, par exemple ici des ECST vu que Marketing et AdsOptimization sont intéressés par le fait d'avoir un cache de l'état de certains des objets de CRM.
- Quelques bonnes pratiques pour l'event driven architecture :
  - Il faut toujours **s'attendre au pire** : éviter le mindset “things will be ok” et partir du principe que tout peut échouer.
    - On parle d'architecture distribuée : le réseau peut avoir des problèmes, les serveurs peuvent crasher, les events peuvent arriver deux fois ou pas du tout etc.
    - Il faut donc s'assurer à tout prix que les events arrivent bien correctement :
      - Utiliser l'**outbox pattern** pour publier les messages.
      - S'assurer que les messages pourront être **dédupliqués** ou **réordonnés** grâce à leur numéro s' ils arrivent dans le mauvais ordre.
      - Utiliser les patterns **saga** et **process manager** pour orchestrer des process cross-bounded contexts qui nécessitent des actions de compensation.
  - Il faut bien **distinguer les events publics des privés** : ne pas tout exposer tel quel mais traiter les events qu'on expose comme une interface publique classique.
    - Ne jamais exposer l'ensemble des domain events d'un event sourced bounded context.
    - Quand on implémente un open-host service, bien s'assurer que le modèle qu'on veut exposer est bien différent du modèle interne à notre bounded context, ce qui peut impliquer de transformer certains events, et pas seulement les filtrer.
    - Exposer plutôt des ECST et des notification events, et des domain events avec parcimonie, en distinguant bien ceux qu'on expose.
  - On peut utiliser le **besoin de consistance comme critère supplémentaire pour le choix du type d'event** :
    - Si l'eventual consistency est OK, on peut utiliser l'ECST.
    - Si le BC consommateur a besoin de lire la dernière écriture dans le state du producteur, alors le notification event appelant à faire une query est sans doute plus adapté.

### 16 - Data Mesh

- Les transactions **OLTP** (T pour transaction) et **OLAP** (A pour analytics) ont des buts, des consommateurs et des temporalités différents.
  - Les OLTP permet de servir les clients sur des opérations plutôt en temps réel, et dont les fonctionnalités sont connues et optimisées.
  - Les OLAP sont là pour obtenir des insights à partir des données, et permettre à l'entreprise d'optimiser le business.
    - Ils prennent en général plus de temps parce qu'ils portent sur de grandes quantités de données, et utilisent des données moins à jour.
    - Ils portent sur des données normalisées qui permettent une grande flexibilité de requêtes de la part des data analysts.
    - NDLR : les OLAP utilisent souvent des BDD orientées colonne plutôt que lignes, c'est-à-dire que les données de toutes les entrées d'une même colonne sont stockées physiquement les unes à la suite des autres, pour faciliter les requêtes qui portent sur peu de colonnes et beaucoup d'entrées. (cf. Designing Data-Intensive Applications).
- Les OLTP vont typiquement utiliser des données organisées **sous forme relationnelle** avec des **entités individuelles** reliées entre elles, pour faciliter le fonctionnement des systèmes opérationnels.
- Les OLAP en revanche s'intéressent plus aux **activités business** et vont utiliser un modèle de données basé sur les fact tables et dimension tables.
  - Les **fact tables** sont des tables qu'on va remplir avec des événements liés au business qui se sont produits dans le passé.
    - Exemple : **`Fact_Sales`** peut être un fact table contenant une entrée pour chaque vente qui a eu lieu.
    - Ils sont choisis pour répondre aux besoins des data analysts qui vont utiliser la BDD.
    - Selon les besoins, on peut choisir d'y mettre seulement certaines données, par exemple des changements de statut dont on garde seulement une entrée toutes les 30 minutes, parce que plus serait inutile ou inefficace pour ce qu'on veut.
    - Les données y sont ajoutées pour être lues, on n'y fait pas de modification.
  - Les **dimension tables** sont référencées par des foreign keys à partir de la fact table, et décrivent des propriétés du fact en question.
    - Exemple : **`Dim_Agents`**, **`Dim_Customers`** etc. qui vont chacun avoir leurs propres champs qui les décrivent.
    - On remarque bien la forte **normalisation** avec la fact table au centre, et les dimension tables autour, qui permettent de maximiser la flexibilité des requêtes possibles sur les données autour de ce fact.
  - On appelle ce modèle fact tables / dimension tables le **star schema**.
    - Il existe une variante appelée le **snowflake schema**, où les dimensions sont sur plusieurs niveaux : les dimension tables ont elles-mêmes des foreign keys vers d'autres dimensions qui les décrivent.
    - L'avantage du snowflake c'est qu'il prend moins de place pour les mêmes données, par contre il faudra faire plus de jointures et donc les requêtes seront plus lentes.
- Le **data warehouse** consiste à extraire les données des systèmes opérationnels, et les mettre dans une grande BDD avec un modèle orienté analytics (star, snowflakes etc.). Les data analysts et ingénieurs BI vont alors consommer cette BDD avec du SQL.
  - On a des flows de type ETL qui vont consommer les BDD mais aussi éventuellement des events, des logs etc. pour construire le data warehouse.
  - Il peut y avoir de la déduplication, de l'élimination d'informations sensibles, de l'agrégation etc.
  - Le data warehouse pose plusieurs **problèmes** :
    - On retombe sur la problématique d'un **modèle unique** pour régler tous les problèmes (dont on s'était sorti par la création de bounded contexts), qui est inefficace.
      - Une des solutions qui a été trouvée c'est de créer des **data marts** qui vont constituer des BDD spécifiques, soit extraites à partir de la data warehouse, soit extraite directement à partir d'un système opérationnel.
      - Le problème c'est que si le data mart est extrait de la data warehouse on ne règle pas le problème du modèle unique dont on dépend. Et si on extrait à partir d'un système opérationnel, on a du mal ensuite à faire des requêtes cross-database entre plusieurs data marts, à cause de problèmes de performance.
    - On a aussi un **couplage à l'implémentation des systèmes opérationnels**. Or être couplé à l'implémentation de gens à qui on parle peu c'est catastrophique. Dès qu'ils modifient leur schéma de données, ça casse les scripts d'ETL qui nourrit le data warehouse.
- Le **data lake** est censé résoudre certains de ces problèmes, en se plaçant entre les systèmes opérationnels et la data warehouse.
  - Elle centralise au même endroit les données opérationnelles **sans changer leur schéma**. Le data warehouse va alors pouvoir être rempli avec un ou plusieurs flows ETL à partir du data lake.
  - Quand le modèle du warehouse n'est plus satisfaisant, les data analysts vont pouvoir piocher dans le data lake avec d'autres scripts ETL.
  - Il y a cependant des **problèmes** :
    - Le processus est plus complexe, et les data engineers se retrouvent souvent à maintenir plusieurs versions d'un même script ETL pour gérer plusieurs versions d'un système opérationnel.
      - => on n'a pas vraiment réglé le souci de **couplage à une implémentation gérée par une autre équipe**.
    - Le data lake étant schema-less, il n'apporte pas de garantie vis-à-vis de la consistance des données venant des systèmes opérationnels.
      - Quand les données deviennent grandes, le data lake se transforme en data swamp (marécage de données), rendant le travail des data scientists très difficile.
- Le **data mesh** tente de répondre à son tour à ces problématiques, en adoptant d'une certaine manière une approche DDD appliquée à la data.
  - Le data mesh a 4 grands principes :
    - **Decompose data around domains** :
      - On va prendre les bounded contexts qu'on avait créés, et y intégrer une partie data correspondant aux données issues des BDD opérationnelles de ce bounded context.
      - L'équipe en charge du bounded context aura alors **sous sa responsabilité à la fois la partie OLTP et la partie OLAP de son bounded context**.
      - On élimine donc la friction qu'il y avait entre équipe feature et équipe data **en intégrant une personne avec les compétences data dans l'équipe feature**.
    - **Data as a product** :
      - Fini les scripts ETL douteux pour construire la data, chaque bounded context expose sa data proprement avec une **API publique**, à laquelle il accorde le même soin qu'une API publique destinée à être consommée par un composant OLTP du système.
        - On va mettre en place des SLA/SLO, on versionne le modèle de données exposé, les endpoints doivent être faciles à trouver et le schéma clairement défini etc.
      - La data étant un produit plutôt qu'un élément de seconde classe, chaque équipe gérant le bounded context a la **responsabilité d'assurer la qualité et l'intégrité de la data exposée**. Mais aussi servir la donnée sous les formats qui pourront intéresser les consommateurs (SQL, fichier etc.).
        - L'idée c'est que les data analysts/BI puissent aller chercher facilement des données de plusieurs bounded context, appliquer éventuellement des transformations en local, et faire leur analyse.
    - **Enable autonomy** :
      - Créer une infrastructure pour gérer la data étant difficile, il faut une **équipe centrale** dédiée à l'infrastructure de la plateforme data.
      - Elle sera en charge de maintenir la plateforme qui permet aux feature teams de créer facilement leur produit data sous les différents formats possibles.
      - Par contre elle reste la plus agnostique possible par rapport au modèle de données, qui est de la responsabilité des équipes qui produisent la donnée.
    - **Build ecosystem** :
      - Il faut un corps de “**gouvernance fédérale**” pour penser l'écosystème data au sein de l'entreprise, et notamment la question de **l'interopérabilité**.
      - Ce corps est composé de représentants data et produit des équipes feature, et de représentants de l'équipe plateforme centrale.
  - Côté relation avec le DDD :
    - Le DDD aide à structurer la donnée analytique en amenant l'ubiquitous language et la connaissance du domaine.
    - Exposer une donnée différente de la donnée utilisée est l'**open-host service** pattern du DDD.
    - Grâce au **CQRS**, on peut facilement mettre en place une ou plusieurs formes dénormalisées de plus qui auront un modèle analytique.
    - Les patterns de relation entre bounded context s'appliquent aussi aux données analytiques (partnership, ACL, separate ways etc.).

## Appendix A - Applying DDD: A Case Study

- Vlad a passé quelques années dans une **startup** qui appliquait le DDD. Il s'agit ici d'une description de ce qu'ils ont fait, et des **erreurs commises**.
  - L'entreprise portait sur le marketing en ligne, de la stratégie marketing aux éléments graphiques, campagnes marketing et appels des prospects récoltés.
  - Il y avait aussi une importante partie data analytics sur l'optimisation des campagnes de marketing, le fait de faire travailler les agents sur les prospects les plus intéressants etc.
- Les 5 bounded contexts qu'il nous présente pour en tirer des leçons :
  - 1- Marketing : il s'agit d'une solution de gestion des campagnes.
    - Quasiment chaque nom présent dans les requirements était un aggregate. Pour autant la plupart n'avaient que peu de logique, celle-ci étant essentiellement dans un énorme service layer.
      - Quand on essaye d'implémenter un domain model mais qu'on finit avec un active record pattern, on appelle ça un **anemic domain model** antipattern.
    - Malgré l'architecture mal adaptée, le projet a été un succès, grâce à l'**ubiquitous language** mis en place dès le début, et les **conversations très fréquentes avec les domain experts**.
  - 2- CRM : les sales l'utilisaient pour se répartir les prospects de manière optimisée.
    - Ils ont d'abord commencé à développer le CRM à l'intérieur du même monolithe que Marketing. Puis voyant que le modèle avait des incohérences, ils ont extrait CRM dans son propre bounded context (au niveau du code seulement).
    - Ils ont cette fois tenté d'utiliser le domain model pattern en mettant **beaucoup de logique dans les aggregates**, et chaque transaction n'affectant qu'un aggregate.
    - Le tout prenant beaucoup de temps, le management a décidé de donner certaines fonctionnalités à l'équipe database, qui l'a fait sous forme de stored procedures.
    - **Deux équipes qui ne se parlent que peu, et qui travaillent sur le même bounded context** : de la duplication de logique, des corruptions de données etc. la cata.
  - 3- Event Crunchers : ils ont remarqué que les événements venant des clients amenaient à modifier les deux bounded contexts, alors ils ont extrait la logique dans un 3ème.
    - Initialement pensé comme un supporting subdomain, et développé avec du **transaction script**, la logique s'est vite complexifiée, et la qualité dégradée.
    - Finalement au fil du temps la logique était devenue tellement complexe et bordélique, qu'ils ont dû le refaire sous forme d'**event-sourced domain model**, avec les autres bounded contexts souscrivant à ses events.
  - 4- Bonuses : il s'agit de calculer les bonus des sales.
    - Là encore ça partait d'une logique simple, donc un **supporting subdomain**. Ils ont choisi d'utiliser l'**active record pattern**.
    - Là encore la logique s'est complexifiée assez vite, grâce à l'**ubiquitous language** qui était en place avec les domain experts, ils ont pu se rendre compte que le modèle ne convenait plus à la complexité plus tôt que pour Event Crunchers : ils l'ont recodé sous forme d'**event-sourced domain** model.
  - 5- Marketing Hub : une nouvelle idée du management : se servir des nombreux prospects acquis pour les vendre à des clients.
    - Ils ont dès le début catalogué le subdomain comme **core**, et utilisé l'**event-sourcing et CQRS**.
    - Ils ont aussi utilisé les **microservices**, un concept qui devenait populaire à ce moment. Mais ils ont fait **un microservice par aggregate**, avec un aggregate event-sourced, et les autres state-based.
    - Au fil du temps chaque micro service avait besoin de quasiment tous les autres, et ils se sont retrouvés avec un distributed monolith.
    - Finalement, même si le subdomain était une source de profit et donc core, la partie logicielle en elle-même était très simple, et le pattern utilisé s'est révélé être largement overkill, amenant de la **complexité accidentelle**.
- L'**ubiquitous language** est selon Vlad le “core subdomain” du DDD : à chaque fois qu'ils l'ont bien mis en place, le projet a plutôt marché, et à chaque fois qu'ils ne l'ont pas fait, le projet a plutôt échoué.
  - Plus on le met en place tôt, plus on évite des problèmes.
- Les **subdomains** sont aussi très importants. Mal les identifier amène à utiliser les mauvais patterns.
  - Vlad propose d'**inverser la relation entre le subdomain et les patterns tactiques** : d'abord choisir le pattern tactique qui convient le mieux au requirement, puis qualifier le type de subdomain, et enfin vérifier ce type avec les gens du business.
    - Si le business pense que le subdomain est core, mais qu'on peut le réaliser facilement, on l'a peut être mal découpé et analysé, ou il faut se poser des questions sur la viabilité de l'idée business.
    - Si le business pense que c'est supporting, mais qu'on ne peut le réaliser qu'avec des patterns complexes, alors :
      - Soit le business s'enflamme sur les requirements et ajoute de l'**accidental business complexity** (mettre beaucoup de ressources sur une activité non rentable).
      - Soit les gens du business ne se rendent pas compte qu'ils obtiennent un gain compétitif qu'ils n'avaient pas envisagé avec le subdomain en question.
  - **Il ne faut pas ignorer la douleur** : si elle se manifeste, c'est sans doute que les patterns utilisés ne sont pas en phase avec la problématique business. Il ne faut pas hésiter à **requalifier le subdomain**.
- A propos des **bounded contexts**, comme évoqué précédemment, le mieux est de **les choisir larges**, puis de les découper quand le besoin se fait sentir, et que la connaissance du domaine augmente.
- Finalement la startup a été profitable assez vite, et a fini par être rachetée par un de leur gros client. Pendant ces années, ils ont été en mode startup : changer les priorités et requirements rapidement, des timeframes agressives, et une petite équipe R&D. Pour Vlad le DDD a rempli ses promesses.
