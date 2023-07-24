# Refactoring: Improving the Design of Existing Code

## 1 - Refactoring : exemple introductif

- Ce chapitre montre un exemple de refactoring selon les techniques qui seront décrites dans le livre.
- Description :
  - Le logiciel sert une entreprise de vente de billets de pièces de théâtre, et permet de créer des factures.
  - Les clients pourront avoir des rabais en fonction de divers paramètres (par exemple le type de la pièce).
- Le programme :

  - On a deux objets/json auxquels notre fonction a accès :
    - Les _plays_ qui sont les pièces de théâtre.
      Exemple :
      ```json
      {
        "hamlet": { "name": "Hamlet", "type": "tragedy" },
        "othello": { "name": "Othello", "type": "tragedy" }
      }
      ```
    - Les _invoices_ qui sont
      ```json
      {
        "customer": "BigCo",
        "performances": [
          {
            "playID": "hamlet",
            "audience": 55
          },
          {
            "playID": "othello",
            "audience": 40
          }
        ]
      }
      ```
  - Le code initial est le suivant :

    ```typescript
    function statement(invoice, plays) {
      let totalAmount = 0;
      let volumeCredits = 0;
      let result = `Statement for ${invoice.customer}\n`;
      const format = new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
        minimumFractionDigits: 2,
      }).format;

      for (let perf of invoice.performances) {
        const play = plays[perf.playID];
        let thisAmount = 0;
        switch (play.type) {
          case "tragedy":
            thisAmount = 40000;
            if (perf.audience > 30) {
              thisAmount += 1000 * (perf.audience - 30);
            }
            break;
          case "comedy":
            thisAmount = 30000;
            if (perf.audience > 20) {
              thisAmount += 10000 + 500 * (perf.audience - 20);
            }
            thisAmount += 300 * perf.audience;
            break;
          default:
            throw new Error(`unknown type: ${play.type}`);
        }
        // add volume credits
        volumeCredits += Math.max(perf.audience - 30, 0);
        // add extra credit for every ten comedy attendees
        if ("comedy" === play.type)
          volumeCredits += Math.floor(perf.audience / 5);
        // print line for this order
        result += ` ${play.name}: ${format(thisAmount / 100)}`;
        result += ` (${perf.audience} seats)\n`;
        totalAmount += thisAmount;
      }
      result += `Amount owed is ${format(totalAmount / 100)}\n`;
      result += `You earned ${volumeCredits} credits\n`;
      return result;
    }
    ```

    - Le programme est court pour les besoins du livre. Pour un si petit programme la question ne se poserait peut-être pas, mais pour un programme de plusieurs centaines de lignes, il faut refactorer.

  - Les utilisateurs veulent deux modifications :
    - 1- Afficher un relevé de compte en HTML et pas seulement sous format texte comme actuellement.
    - 2- Ajouter de nouveaux types de pièces de théâtre, avec chacune ses règles particulières pour le système de rabais. On ne sait pas encore quels types on voudra.
  - Une première (mauvaise) solution simple serait de dupliquer la fonction pour faire la version HTML sur l'autre fonction.
    - Mais il faudrait ensuite mettre à jour les règles des nouveaux types de pièces sur les deux fonctions.

- On va **refactorer** le programme :

  - **1 - Les tests**
    - Avant tout refactoring, la 1ère chose à faire c'est de s'assurer que cette partie du code a des tests solides. Si c'est pas le cas on les écrit.
  - **2 - On décompose la fonction en sortant le switch**
    - On identifie d'abord à l'œil les **parties qui vont ensemble**.
    - Ici c'est d'abord le switch qu'on choisit de traiter comme un bloc.
    - On va utiliser la technique **[Extract Function](#extract-function)** pour sortir le bloc dans une autre fonction (qui se trouvera dans la portée de _statement_).
    - On regarde d'abord les **variables qui sont utilisées** par la nouvelle fonction extraite :
      - Deux variables ne sont que lues (_invoices_ et _plays_), on va pouvoir les passer en paramètre.
      - Une autre variable est assignée (_thisAmount_), on va pouvoir retourner la valeur pour l'assigner par retour de fonction.
    - Une fois qu'on a déplacé la fonction, on compile (si nécessaire), on joue les tests, et on commit.
      - Pour éviter de passer du temps à débugger son propre refactoring, il faut faire de **petites étapes avec un commit à chaque fois**.
    - On va ensuite renommer la fonction, et ses paramètres :
      ```typescript
      function amountFor(aPerformance, play) {
        // …
      ```
      - Le fait de mettre _For_ dans le nom de la fonction et _a_ dans le nom du paramètre fait partie du style de Fowler, qu'il a pris à Kent Beck.
  - **3 - On supprime la variable play**
    - Pour chaque variable prise en paramètre de notre nouvelle fonction _amountFor_, on vérifie sa provenance pour voir si on ne peut pas s'en débarrasser :
      - _aPerformance_ est différente à chaque tour de boucle, on doit la garder.
      - _play_ est par contre toujours le même : on pourrait le supprimer comme paramètre, et recalculer sa valeur dans notre nouvelle fonction.
    - On va utiliser la technique **[Replace Temp with Query](#replace-temp-with-query)** pour remplacer dans la boucle :
      - `const play = plays[perf.playID];`
      - par
      - `const play = playFor(perf);`
      - Avec la nouvelle fonction :
        ```typescript
        function playFor(aPerformance) {
          return plays[aPerformance.playID];
        }
        ```
    - On compile, teste, commit, puis on utilise **[Inline Variable](#inline-variable)** :
      - On supprime la variable `const play = playFor(perf);`
      - Et on appelle la fonction au moment d'appeler _amountFor_ :
        ```typescript
        let thisAmount = amountFor(perf, palyFor(perf));
        ```
      - Et on fait le remplacement par l'appel partout où _play_ était utilisé.
    - On compile, teste, commit, puis on peut utiliser **[Change Function Declaration](#change-function-declaration)** pour éliminer le paramètre _play_ dans _amountFor_ :
      - On utilise la nouvelle fonction _playFor_ à l'intérieur d'_amountFor_, en remplacement du paramètre _play_.
      - On compile, teste, commit.
      - On supprime le paramètre _play_ qui n'était plus utilisé.
    - Remarques :
      - Faire 3 fois l'accès à play est moins performant que de mettre la valeur dans une variable et y accéder les deux autres fois.
        - Mais **cette différence est négligeable**.
        - Et **même si elle était significative**, rendre le code plus clair permettra de mieux l'optimiser ensuite.
      - **Supprimer les variables locales** permet en général de faciliter les extractions, c'est pour ça qu'on le fait.
    - On utilise encore **[Inline Variable](#inline-variable)** pour appeler plusieurs fois _amountFor_ dans _statement_, comme ça on supprime cette variable locale aussi.
  - **4 - on extrait les crédits de volume**
    - On va extraire le bloc de calcul de crédits de volume. Pour ça on vérifie les variables qui devront être passées :
      - _perf_ doit être passé.
      - _play_ a été supprimé par le refactoring d'avant.
      - _volumeCredits_ est un accumulateur. On peut créer une variable locale dans la nouvelle fonction et la retourner.
    - Ca donne :
      ```typescript
      function volumeCreditsFor(perf) {
        let volumeCredits = 0;
        volumeCredits += Math.max(perf.audience - 30, 0);
        if ("comedy" === playFor(perf).type)
          volumeCredits += Math.floor(perf.audience / 5);
        return volumeCredits;
      }
      ```
    - Et côté appel dans _statement_ :
      `volumeCredits += volumeCreditsFor(perf);`
    - On compile, teste, commit.
    - On va renommer les variables dans _volumeCreditsFor_ :
      ```typescript
      function volumeCreditsFor(aPerformance) {
        let result = 0;
        result += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" === playFor(aPerformance).type)
          result += Math.floor(aPerformance.audience / 5);
        return result;
      }
      ```
      - On a ici fait deux changements de variable (_aPerformance_, et _result_), il faut compiler, tester et commiter entre chaque changement.
      - La convention _result_ fait partie du style de convention de Martin Fowler, au même titre que _aVariable_ pour les variables en paramètre.
  - **5 - On va supprimer la variable format**
    - On continue à supprimer des variables temporaires dans _statement_ pour faciliter l'extraction de blocs.
    - Ici on s'attaque à _format_, c'est une variable contenant une fonction. On va la supprimer pour déclarer la fonction en question :
      ```typescript
      function format(aNumber) {
        return new Intl.NumberFormat("en-US", {
          style: "currency",
          currency: "USD",
          minimumFractionDigits: 2,
        }).format(aNumber);
      }
      ```
    - Cette technique de transformer une fonction dans une variable en fonction déclarée n'est pas assez importante pour figurer dans les techniques du livre.
    - On va ensuite utiliser **[Change Function Declaration](#change-function-declaration)** pour renommer la fonction en quelque chose qui indique mieux ce qu'elle fait.
      - Vu qu'elle est utilisée dans une petite portée et est peu importante, Fowler privilégie un nom plus court que _formatAsUSD_. Il choisit juste _usd_ pour mettre en avant l'aspect monétaire.
  - **6 - On va déplacer le calcul du volume de crédits**
    - On avait précédemment extrait le calcul du volume de crédits pour une pièce. On va maintenant extraire le calcul du volume de crédits pour l'ensemble des pièces hors de _statement_.
    - On va utiliser **[Split Loop](#split-loop)** pour couper la boucle en deux, et avoir le calcul du volume de crédits dans une boucle à part.
      ```typescript
      for (let perf of invoice.performances) {
        // print line for this order
        result += ` ${play.name}: ${format(thisAmount / 100)}`;
        result += ` (${perf.audience} seats)\n`;
        totalAmount += thisAmount;
      }
      for (let perf of invoice.performances) {
        volumeCredits += volumeCreditsFor(perf);
      }
      ```
      - On compile, teste, commit.
    - On peut alors utiliser **[Slide Statements](#slide-statements)** pour déplacer la déclaration de la variable _volumeCredits_ juste au-dessus de la 2ème boucle.
      - On compile, teste, commit.
    - On va pouvoir utiliser **[Replace Temp with Query](#replace-temp-with-query)**, la première étape pour pouvoir le faire c'est d'utiliser **[Extract Function](#extract-function)** :
      ```typescript
      function totalVolumeCredits() {
        let volumeCredits = 0;
        for (let perf of invoice.performances) {
          volumeCredits += volumeCreditsFor(perf);
        }
        return volumeCredits;
      }
      ```
    - On peut donc appeler la nouvelle fonction dans statement :
      ```typescript
      volumeCredits = totalVolumeCredits();
      ```
      - On compile, teste, commit.
    - On peut alors utiliser **[Inline Variable](#inline-variable)** pour supprimer la variable locale _volumeCredits_, et appeler _totalVolumeCredits_ là où elle était utilisée.
      - On compile, teste, commit.
    - Remarques :
      - Pour le côté **performance**, la plupart du temps créer des boucles supplémentaires ne créera pas de ralentissement significatif.
        - Même dans les rares cas où c'est le cas, on pourra toujours mieux optimiser après avoir rendu le code plus clair par du refactoring.
      - Les étapes présentées avant chaque compilation/test/commit sont très courtes. Il arrive à Martin Fowler de ne pas faire à chaque fois des étapes aussi courtes, mais il fait quand même des étapes relativement courtes.
        - Et surtout, dès que les tests échouent, il **annule ce qu'il a fait depuis le dernier commit** et reprend avec des étapes plus courtes (comme celles-là).
        - Le but c'est de ne pas **perdre de temps à débugger** pendant un refactoring.
    - On va répéter la même séquence pour extraire complètement _totalAmount_ :
      - **[Split Loop](#split-loop)** pour extraire l'instruction qui nous intéresse dans une boucle à part.
      - **[Slide Statements](#slide-statements)** pour déplacer la variable locale près de la nouvelle boucle.
      - **[Extract Function](#extract-function)** pour extraire la boucle dans une nouvelle fonction.
        - Le meilleur nom pour cette fonction est déjà pris par la variable _totalAmount_, donc on lui met un nom au hasard pour garder un code qui marche et commiter.
      - **[Inline Variable](#inline-variable)** nous permet d'éliminer la variable locale, et de renommer la nouvelle fonction _totalAmount_.
      - On en profite aussi pour renommer la variable locale dans la nouvelle fonction en _result_ pour respecter notre convention de nommage.
    - Ca donne :
      ```typescript
      function totalAmount() {
        let result = 0;
        for (let perf of invoice.performances) {
          result += amountFor(perf);
        }
        return result;
      }
      ```
  - **7 - On va fractionner les phases de calcul et de formatage**

    - Jusqu'ici on avait refactoré pour rendre le code plus clair et mieux le comprendre. On va maintenant le mener vers l'objectif qui est de pouvoir créer des factures HTML en plus des factures texte.
    - On va mettre en œuvre **[Split Phase](#split-phase)** pour faire la division logique / formatage.
    - Pour ça on commence par appliquer **[Extract Function](#extract-function)** au code qui constituera la 2ème phase : on va déplacer l'ensemble du code de _statement_ et les fonctions imbriquées dans une fonction _renderPlainText_.

      ```typescript
      function statement(invoice, plays) {
        return renderplainText(invoice, plays);
      }

      function renderPlainText(invoice, plays) {
        let result = `Statement for ${invoice.customer}\n`;
        // ...
        return result;

        function totalAmount() {
          // ...
        }
        // ...
      }
      ```

      - On compile, teste et commit.

    - On va créer un objet qui servira de **structure de données intermédiaire entre les deux phases** :

      ```typescript
      function statement(invoice, plays) {
        const statementData = {};
        return renderplainText(statementData, invoice, plays);
      }

      function renderPlainText(data, invoice, plays) {
      //...
      ```

    - Le but c'est de déplacer tout le calcul de logique hors de _renderPlainText_, et de tout lui passer au travers de l'objet _data_.

      - (On va compiler / tester / commiter entre chaque étape)
      - On va mettre `invoice.customer` dans _data_, pour obtenir `data.customer` de l'autre côté.
        ```typescript
        const statementData = {};
        statementData.customer = invoice.customer;
        ```
      - On fait pareil avec `invoice.performances`.
      - On veut maintenant que les valeurs des performances soient précalculées dans le paramètre _data_ qu'on donne à _renderPlainText_. On commence par créer une fonction pour enrichir les performances :

        ```typescript
        statementData.performances =
          invoice.performances.map(enrichPerformance);

        function enrichPerformance(aPerformance) {
          const result = { ...aPerformance };
          return result;
        }
        ```

      - On va déplacer les informations des pièces (plays) dans les _performances_ qu'on a mis dans _data_.
        - Pour ça on ajoute la valeur dans notre nouvelle fonction.
          ```typescript
          function enrichPerformance(aPerformance) {
            const result = { ...aPerformance };
            result.play = playFor(result);
            return result;
          }
          ```
        - Ensuite on utilise **[Move Function](#move-function)** pour déplacer _playFor_ en dessous d'_enrichPerformance_.
        - Et enfin on remplace toutes les utilisations de _playFor_ dans les fonctions de _renderPlainText_ par les valeurs précalculées issues de _data_. On va y accéder typiquement par `perf.play`.
      - On va ensuite appliquer la même chose que pour _playFor_, mais cette fois pour _amountFor_ dont on élimine les appels de _renderPlainText_.
      - Puis on fait la même chose pour _volumeCreditsFor_.
      - Et encore la même chose pour _totalAmount_, et _totalVolumeCredits_.

    - On en profite pour utiliser **[Replace Loop with Pipeline](#replace-loop-with-pipeline)** sur les boucles de _totalAmount_ et _totalVolumeCredits_.
      ```typescript
      function totalAmount(data) {
        return data.performances.reduce((total, p) => total + p.amount, 0);
      }
      ```
    - On va maintenant extraire le code de première phase qu'on vient de créer (la création des data pré-calculées dans statement) dans une fonction à part :

      ```typescript
      function statement(invoice, plays) {
        return renderplainText(
          createStatementData(invoice, plays)
        );
      }

      function renderPlainText(data) {
        //...
      }

      function createStatementData(invoice, plays) {
        const result = {};
        result.customer = invoice.customer;
        result.performances =
            invoice.performances.map(enrichPerformance);
        result.totalAmount = totalAmount(result);
        result.totalVolumeCredits =
            totalVolumeCredits(result);
        return result;

        function enrichPerformance(aPerformance) {
          // ...
        }
      // ...
      ```

    - On peut alors extraire _createStatementData_ et ses fonctions imbriquées (le code de la phase 1 donc) dans un fichier à part qu'on appelle _createStatementData.js_.
    - On peut maintenant facilement écrire la version HTML de statement dans le fichier statement.js :

      ```typescript
      function statement(invoice, plays) {
        return renderplainText(createStatementData(invoice, plays));
      }

      function renderPlainText(data) {
        let result = `Statement for ${data.customer}\n`;
        // ...
        return result;
      }

      function htmlStatement(invoice, plays) {
        return renderHtml(createStatementData(invoice, plays));
      }

      function renderHtml(data) {
        let result = `<h1>Statement for ${data.customer}</h1>\n`;
        // ...
        return result;
      }
      ```

    - Remarque : Fowler propose de suivre la **règle du camping** : il faut toujours laisser le code un peu plus propre que l'état dans lequel on l'a trouvé. Le code ne sera jamais parfait, mais il sera meilleur qu'avant.

  - **8 - On va créer un calculateur pour types de performances**

    - On s'intéresse ici au fait de faciliter l'ajout de nouveaux types de pièces de théâtre, avec chacun ses conditions et valeurs de calcul pour la facturation et les crédits de volume.
    - La solution qu'on retient c'est de créer un calculateur sous forme de classes, avec des classes filles pour contenir la logique de chaque type de pièce. On va donc mettre en œuvre la technique **[Replace Conditional with Polymorphism](#replace-conditional-with-polymorphism)**.
      - NDLR : dans Clean Code, Uncle Bob disait qu'un code procédural (qui utilise les structures de données pour représenter les objets) est adapté pour ajouter des fonctionnalités supplémentaires (par exemple dans notre cas une fonctionnalité en plus du calcul de facturation et du calcul de volume de crédits). Un code orienté objet par contre est plutôt adapté pour l'ajout de nouveaux types d'objets sans ajout de nouvelles fonctionnalités (par exemple dans notre cas ajouter un nouveau type de pièce de théâtre avec ses propres règles de facturation et volume de crédits).
        - L'idée est de minimiser le nombre d'éléments qui seront changés quand on fera notre changement.
    - On commence par créer le calculateur sans qu'il ne fasse rien :

      ```typescript
      function enrichPerformance(aPerformance) {
        const calculator = new PerformanceCalculator(aPerformance);
        // ...
      }

      class PerformanceCalculator {
        constructor(aPerformance) {
          this.performance = aPerformance;
        }
      }
      ```

    - Ensuite, pour plus de clarté, on déplace les informations des pièces :

      ```typescript
      function enrichPerformance(aPerformance) {
        const calculator = new PerformanceCalculator(
          aPerformance,
          playFor(aPerformance)
        );
        // ...
      }

      class PerformanceCalculator {
        constructor(aPerformance, aPlay) {
          this.performance = aPerformance;
          this.play = aPlay;
        }
      }
      ```

  - **9 - On va déplacer les fonctions de calcul dans le calculateur**
    - On va d'abord déplacer _amountFor_ dans le calculateur.
      - Pour ça on commence par utiliser **[Move Function](#move-function)** et copier le code dans un getter, et utiliser les variables d'instance `this.play` et `this.performance` :
        ```typescript
        get amount() {
          switch (this.play.type) {
          case "tragedy":
            thisAmount = 40000;
            if (this.performance.audience > 30) {
        //...
        }
        ```
      - On va ensuite instancier le calculateur et utiliser le getter _amount_ dans _amountFor_, pour tester que jusque là les tests passent.
        ```typescript
        function amountFor(aPerformance) {
          return new PerformanceCalculator(aPerformance, playFor(aPerformance))
            .amount;
        }
        ```
      - Enfin on va utiliser **[Inline Function](#inline-function)** pour éliminer _amountFor_ et utiliser `calculator.amount` à la place.
    - On fait la même chose pour _volumeCreditsFor_, pour se retrouver à utiliser `calculator.volumeCredits`.
  - **10 - On va rendre le calculateur polymorphe**

    - On va commencer par utiliser **[Replace Type Code with Subclasses](#replace-type-code-with-subclasses)** pour ça.

      - Pour obtenir la bonne classe, on va utiliser **[Replace Constructor with Factory Function](#replace-constructor-with-factory-function)** :

        ```typescript
        function enrichPerformance(aPerformance) {
          const calculator = createPerformanceCalculator(
            aPerformance,
            playFor(aPerformance)
          );
          //...
        }

        function createPerformanceCalculator(aPerformance, aPlay) {
          switch (aPlay.type) {
            case "tragedy":
              return TragedyCalculator(aPerformance, aPlay);
            case "comedy":
              return ComedyCalculator(aPerformance, aPlay);
            default:
              throw new Error(`Unknown type: ${aPlay.type}`);
          }
        }

        class TragedyCalculator extends PerformanceCalculator {}

        class ComedyCalculator extends PerformanceCalculator {}
        ```

      - On peut maintenant utiliser **[Replace Conditional with Polymorphism](#replace-conditional-with-polymorphism)** pour déplacer les fonctions de calcul dans les classes filles.
        - On peut créer un getter du même nom (_amount_) dans _TragedyCalculator_ par exemple, pour y déplacer le code lié à la facturation des tragédies.
        - Puis on le fait pour la facturation des comédies.
        - On peut alors supprimer `get amount()` de _PerformanceCalculator_. Ou alors le laisser et y throw une erreur indiquant que la fonctionnalité est déléguée aux classes filles.
        - On peut faire pareil avec les volumes de crédits. Pour celui-là on pourra laisser le cas le plus courant (attribuer des crédits si l'auditoire est supérieur à 30 personnes) dans _PerformanceCalculator_, et ne surcharger que pour _ComedyCalculator_.

- Comme souvent, **les premières phases** du refactoring permettent de **comprendre** ce que fait le code en le clarifiant. On peut ensuite réinjecter cette compréhension dans la suite des refactorings pour le faire aller dans le sens qu'on veut.
- Les **petites étapes** sont étonnantes au premier abord, mais cette méthode est vraiment efficace, et permet d'avancer sereinement et rapidement pour faire au final des refactorings importants.

## 2 - Principes du refactoring

- L'auteur propose une définition plus restreinte du refactoring que ce qu'on entend habituellement : il s'agit pour lui d'une **succession de petits changements qui permettent de rendre le code plus facile à comprendre et à changer, sans changer son comportement**.
  - Comme ce sont de petits changements indépendants, on peut arrêter à tout moment en gardant le code fonctionnel.
  - Il propose _restructuration_ (restructuring) comme mot plus général pour désigner le fait de réorganiser le code, le refactoring étant une forme particulière de restructuration.
- Les développeurs ont **deux casquettes** distinctes qu'ils peuvent porter, une à la fois : celle d'**ajout de changements, et celle de refactoring**.
  - Il est important d'essayer de garder ces deux casquettes distinctes pour être efficace dans ce qu'on fait et avancer sereinement.
  - La casquette de changement mène normalement à l'ajout ou à la modification de tests, alors que la casquette de refactoring ne devrait pas mener à toucher aux tests.
  - NDLR : à petite échelle il s'agit aussi des casquettes qu'on adopte en TDD : red-green avec celle du changement, et refactor avec celle du refactoring.
- Pourquoi faire du refactoring ?
  - Pour conserver et améliorer l'architecture du logiciel qui se délite peu à peu.
  - Pour rendre le code plus lisible et compréhensible.
    - On met la connaissance qu'on a au moment où on a passé du temps sur le code dans le code lui-même, comme ça on peut nous-mêmes l'oublier sans souci.
  - Pour révéler les bugs qui se cachaient dans du code fouilli.
  - Pour programmer **plus rapidement**.
    - Si on met en place du code bien structuré et compréhensible, on pourra s'appuyer sur le code existant pour coder plus vite de nouvelles features.
    - Cette hypothèse est basée sur l'expérience de Fowler et celle de centaines de programmeurs qu'il connaît.
- Quand faire du refactoring ?
  - La **règle de trois** : on fait quelque chose une fois, la 2ème fois qu'on le fait on laisse passer, la troisième fois on fait un refactoring.
  - Le meilleur moment est le **refactoring préparatoire** : juste avant de faire une modification, on remanie le code pour rendre cette modification plus facile.
    - Ça peut être un refactoring de compréhension, ou un refactoring de ramassage d'ordures quand on se rend compte que le code est mal structuré pour ce qu'on veut en faire.
  - La plupart des refactorings doivent être **opportunistes**, c'est-à-dire s'intégrer au flux habituel d'ajout de fonctionnalités ou de correction de bugs.
    - On peut parfois aussi faire des refactorings **planifiés** si on a vraiment négligé le code ou qu'on tombe sur quelque chose de spécifique qui le nécessite.
  - Le refactoring est aussi **nécessaire pour le code qui est déjà de bonne qualité**, parce qu'il ne s'agit d'adapter en permanence le code à notre compréhension actuelle du système, et celle-ci varie tout le temps.
  - L'idée de séparer les features et les refactorings dans des features séparées n'a pas que des avantages, Fowler est plutôt réticent..
    - Elle permet de faire des reviews plus ciblées, mais d'un autre côté le refactoring est souvent lié au contexte du changement qu'il accompagne, et on risque d'avoir plus de mal à le comprendre et à le justifier isolément.
    - NDLR : [Kent Beck a récemment](https://www.youtube.com/watch?v=BFFY9Zor6zw) conseillé de faire des PRs séparées avec de petites granularités pour pouvoir choisir le moment où on fait des refactorings et fournir des fonctionnalités régulières aux personnes qui attendent.
  - On a parfois besoin de **gros refactorings**, par exemple pour remplacer une librairie. Dans ce cas, l'auteur conseille de **procéder petit à petit** quand même.
    - Pour la librairie, on peut mettre en place une abstraction devant la librairie actuelle, et remplacer les fonctionnalités derrière l'abstraction petit à petit. On appelle ça _Branch By Abstraction_.
  - Le refactoring est aussi utile pendant les **code reviews** : on retravaille le code pour comprendre plus en profondeur ce que la personne a fait, et pour avoir des idées d'amélioration qu'on pourra mettre en place immédiatement.
    - Ca implique de faire la code review **en présence de la personne qui a fait la PR**. Faire des Code reviews sans la personne ne fonctionne de toute façon pas très bien selon Fowler.
    - La conclusion logique de la pratique est le pair programming.
  - Quand ne pas faire de refactoring :
    - Quand on tombe sur du code qui part dans tous les sens mais qu'on n'a pas besoin de le modifier.
    - Quand il est préférable de réécrire le code plutôt que de le remanier. Ce genre de décision se fait avec l'expérience.
- Que dire aux managers pour le refactoring ?
  - Les managers qui ont une bonne compréhension de la technique vont de toute façon encourager le refactoring parce qu'ils sauront que ça permet de rester productif.
  - Pour ceux qui n'ont pas de bonne compétences techniques, le conseil controversé de Fowler est de **ne pas leur dire qu'on en fait**.
    - Nous sommes les professionnels du développement, nous sommes payés pour notre expertise à coder vite, et le refactoring nous permet justement de coder vite.
- De manière générale, et y compris auprès des développeurs, il ne faut pas justifier le refactoring par “la beauté du code” ou “les bonnes pratiques”, mais par le critère économique qui met tout le monde d'accord : **ça permet d'aller plus vite**.
- Il vaut mieux éviter des granularités trop fines pour ce qui est de la propriété du code, notamment au sein d'une équipe : chaque membre de l'équipe devrait pouvoir modifier toute la codebase pour pouvoir faire des refactorings qui toucheraient éventuellement jusque ces endroits-là.
  - On peut étendre ça entre les équipes où n'importe qui de l'entreprise pourrait faire un PR chez la codebase d'une autre équipe.
- L'utilisation de _feature branches_ est problématique par rapport aux conflits de mege, et en particulier par rapport aux refactorings qui vont provoquer beaucoup de conflits.
  - La pratique du refactoring va de pair avec la Continuous Integration (CI), aussi appelée trunk-based development, qui consiste à **intégrer au moins une fois par** jour son travail sur le branche principale.
    - Les deux pratiques font partie de l'_Extreme Programming_.
    - A noter que la CI est prouvée comme plus efficace que les autres pratiques d'intégration (cf. le livre **_Accelerate_**).
- Pour faire des refactorings, il faut soit des **tests** qui assurent que le comportement n'est pas changé, soit utiliser des outils qui font des **refactorings automatiquement** (par exemple renommer une variable ou extraire une fonction).
  - Dans **_Working Effectively with Legacy Code_**, Michael Feathers décrit comment ajouter des tests à du code legacy : il faut trouver des points d'entrée où insérer des tests, et pour ça il faut prendre des risques en faisant du refactoring.
- Pour ce qui est du **refactoring de bases de données**, il faut aussi y aller par petits pas, en créant de petites migrations successives.
  - Il y a un livre à ce sujet : **_Refactoring Databases_**.
  - Une bonne pratique est le **changement parallèle** (aussi appelé expand-contract) où on va d'abord créer la nouvelle structure, puis l'alimenter avec l'ancienne, puis migrer tout petit à petit pour utiliser la nouvelle. Et finalement supprimer l'ancienne.
- Le refactoring implique d'adopter une **approche incrémentale de l'architecture**.
  - On appelle ça aussi **YAGNI** (you aren't going to need it) : il s'agit de ne pas rendre le code inutilement flexible pour plus tard “au cas où”.
    - Par exemple, ne pas ajouter des paramètres non utilisés à une fonction, au cas où on en aurait besoin plus tard. On les ajoutera avec du refactoring quand on en aura effectivement besoin.
    - Souvent le besoin imaginé ne se réalise pas, ou pas comme on l'avait imaginé.
    - Ça a bien sûr des limites : parfois un refactoring sera beaucoup plus coûteux plus tard. Dans ce cas, on peut l'envisager tout de suite. Mais ce n'est pas si courant.
  - Cette approche de l'architecture est aussi étudiée sous le nom d'**evolutionary architecture**.
- Le refactoring fait partie d'un ensemble de techniques interdépendantes et cohérentes qui permet l'agilité. Elles sont regroupées au sein de l'**Extreme Programming**.
  - Parmi ces techniques il y a notamment : le refactoring, l'intégration continue (CI), la livraison continue (CD), YAGNI, et les tests automatisés.
- Il y a 3 manières d'aborder la question de la **performance** :
  - La 1ère est la **budgétisation du temps** : au moment de la conception on attribue un budget temps qui ne doit pas être dépassé à chaque composant.
    - C'est utile pour les **systèmes temps réel** comme les simulateurs cardiaques, mais inadapté à des systèmes web classiques.
  - La 2ème est l'**attention constante** : on essaye de faire attention à la performance sur tout le code qu'on écrit.
    - Le problème c'est qu'en général l'essentiel du temps d'exécution des programmes se concentre sur très peu de code. Et on passe 90% du temps à optimiser des choses qui n'ont aucun impact.
    - Un autre problème c'est qu'on a en général une mauvaise idée de la manière dont se comporte le compilateur, le runtime, le matériel etc. et on optimise des choses à tort.
    - Cette solution mène à perdre beaucoup de temps à faire des **optimisations inutiles**, et à obtenir du **code peu maintenable**.
  - La 3ème méthode consiste à **séparer l'optimisation de performance dans une phase à part** : on code sans prendre en compte la performance, puis on passe du temps dédié à l'améliorer.
    - On utilise un profiler pour repérer les endroits du code qui consomment le plus (de temps, de mémoire etc.), et on se concentre sur ça seulement.
    - On procède là aussi itérativement par petites touches, en annulant ce qu'on a fait si ça n'améliore pas.
    - Le fait d'avoir du code bien refactoré permet de plus facilement comprendre ce qui se passe, et aide donc à optimiser.
    - Cette approche est **la plus efficace**.
- Quelques livres intéressants sur le refactoring :
  - **_Refactoring Workbook_** de Bill Wake : un livre avec des exercices pour mettre en application le refactoring.
  - **_Refactoring to Patterns_** de Josh Kerievsky : comment appliquer le refactoring en utilisant les design patterns du gang of four.
  - **_Refactoring Databases_** de Scott Ambler et Pramod Sadalage et **_Refactoring HTML_** de Elliotte Rusty : des livres appliquant le refactoring à des domaines spécifiques.
  - **_Working Effectively with Legacy Code_** de Michael Feathers : comment faire du refactoring sur du code avec peu ou pas de tests.

## 3 - Quand le code sent mauvais

- On ne peut pas savoir quand un refactoring est nécessaire mieux que l'intuition d'un programmeur expérimenté, mais ce chapitre contient une liste de 24 **code smells** qui devraient au moins nous alerter quand on les croise.
- **1 - Mysterious Name** : quand on ne comprend pas un nom de variable ou fonction au premier coup d'œil, il faut la renommer.
  - Si on n'arrive pas à trouver un bon nom, c'est sans doute qu'on a d'autres problèmes plus profonds avec le code qu'on essaye de nommer.
  - Parmi les techniques, il y a **[Change Function Declaration](#change-function-declaration)**, **[Rename Function](#change-function-declaration)** et **[Rename Field](#rename-field)**.
- **2 - Duplicated Code** : quand on a du code dupliqué, il faut essayer de le factoriser pour avoir moins d'endroits à maintenir à jour à chaque modification.
  - En général on va utiliser **[Extract Function](#extract-function)**.
  - Si le code dupliqué n'est pas tout à fait identique, on peut d'abord utiliser **[Slide Statements](#slide-statements)** pour obtenir un morceau de code identique à factoriser.
  - Si le code dupliqué se trouve dans des classes filles d'une même hiérarchie, on peut la remonter dans la mère avec **[Pull Up Method](#pull-up-method)**.
- **3 - Long Function** : les fonctions courtes sont plus efficaces pour la compréhension du code.
  - L'idée c'est de nommer les fonctions avec **l'intention de leur code** plutôt que par ce qu'il fait. A chaque fois qu'on veut commenter, on peut remplacer ça par une fonction qui encapsule le bout de code.
  - C'est tout à fait OK de faire des fonctions qui ne contiennent **qu'une ligne**, pour peu que le nommage apporte une meilleure information sur l'intention.
  - En général, on va utiliser **[Extract Function](#extract-function)**.
  - Les **conditions** peuvent être divisées avec **[Decompose Conditional](#decompose-conditional)**.
    - Un grand switch devrait avoir ses clauses transformées en un seul appel de fonction avec **[Extract Function](#extract-function)**.
    - S'il y a plus d'un switch sur la même condition, alors il faut appliquer **[Replace Conditional with Polymorphism](#replace-conditional-with-polymorphism)**.
  - Les **boucles** peuvent être extraites dans leur propre fonction.
    - Si on a du mal à nommer la fonction, alors on peut appliquer d'abord **[Split Loop](#split-loop)**.
- **4 - Long Parameter List** : trop de paramètres porte à confusion, il faut essayer de les éliminer.
  - Si on peut obtenir un paramètre à partir d'un autre, alors on peut appliquer **[Replace Temp with Query](#replace-temp-with-query)** pour l'éliminer.
  - Si plusieurs paramètres sont toujours ensemble, on peut les combiner avec **[Introduce Parameter Object](#introduce-parameter-object)**.
  - Si un argument est utilisé pour choisir une logique dans la fonction, on peut diviser la logique en plusieurs fonctions avec **[Remove Flag Argument](#remove-flag-argument)**.
  - On peut aussi regrouper les fonctions qui ont des paramètres communs en classes avec **[Combine Functions into Class](#combine-functions-into-class)**, pour remplacer les paramètres par des champs.
- **5 - Global Data** : le problème des données globales c'est qu'on peut les modifier de n'importe où, et donc c'est très difficile de suivre ce qui se passe.
  - Pour traiter le problème, il faut les encapsuler avec **[Encapsulate Variable](#encapsulate-variable)**.
- **6 - Mutable Data** : le fait que les structures soient mutables fait qu'on peut changer une structure quelque part, et provoquer un bug ailleurs sans s'en rendre compte.
  - La recherche de l'immutabilité vient de la programmation fonctionnelle.
  - On peut utiliser **[Encapsulate Variable](#encapsulate-variable)** pour s'assurer qu'on modifie la structure à partir de petites fonctions.
  - Si une variable est mise à jour pour stocker plusieurs choses, on peut utiliser **[Split Variable](#split-variable)** pour rendre ces updates moins risquées.
  - Il faut essayer de garder la logique qui n'a pas de side effects et le code qui modifie la structure séparés, avec **[Slide Statements](#slide-statements)** et **[Extract Function](#extract-function)**. Et dans les APIs, on peut utiliser **[Separate Query from Modifier](#separate-query-from-modifier)** pour que l'appelant fasse des queries sans danger.
  - Dès que c'est possible, il faut utiliser **[Remove Setting Method](#remove-setting-method)** pour enlever les setters.
  - Les données mutables qui sont calculées ailleurs sont sources de bugs, il faut les remplacer par **[Replace Derived Variable with Query](#replace-derived-variable-with-query)**.
  - Il faut essayer de limiter le scope du code qui a accès aux variables mutables. Par exemple avec **[Combine Functions into Class](#combine-functions-into-class)**, ou **[Combine Functions into Transform](#combine-functions-into-transform)**.
  - Si une variable contient déjà une structure avec d'autres données, il vaut mieux remplacer la structure entière d'un coup, plutôt que de modifier la variable, avec **[Change Reference to Value](#change-reference-to-value)**.
- **7 - Divergent Change** : quand on a un module qui doit être modifié pour plusieurs raisons, on est face à des changements divergents.
  - Par exemple si on se dit “Je devrai modifier ces trois fonctions si j'ajoute une nouvelle base de données, et ces quatre fonctions si j'ajoute un nouvel instrument financier” : les bases de données et les instruments financiers sont deux contextes différents qu'il vaut mieux traiter séparément.
  - Si les deux contextes forment deux phases (par exemple il faut obtenir les infos de la base de données, puis appliquer un instrument financier), alors on peut utiliser **[Split Phase](#split-phase)** pour séparer les deux avec une structure de données.
  - Sinon on peut utiliser **[Extract Function](#extract-function)** pour les séparer dans plusieurs fonctions.
  - Et si c'est des classes : **[Extract Class](#extract-class)**.
- **8 - Shotgun Surgery** : c'est l'inverse du Divergent Change, on a une fonctionnalité qui est dispersée à plusieurs endroits qu'il faut à chaque fois aller modifier.
  - On peut utiliser **[Move Function](#move-function)** et **[Move Field](#move-field)** pour replacer le code au même endroit.
  - Si on a des fonctions qui opèrent sur les mêmes données, on peut les associer avec **[Combine Functions into Class](#combine-functions-into-class)**.
  - On peut aussi combiner le code éparpillé dans une grande fonction ou classe (avec **[Inline Function](#inline-function)** et **[Inline Class](#inline-class)**) avant de séparer ça en plus petites fonctions.
- **9 - Feature Envy** : on essaye en général d'avoir des modules à l'intérieur desquels il y a beaucoup de communication, et entre lesquels il y en a peu. On parle de feature envy quand un module communique plus avec du code ‘un module voisin qu'avec le module où il est.
  - En général on va utiliser **[Move Function](#move-function)**, parfois précédé d'**[Extract Function](#extract-function)** si seule une partie de la fonction a besoin de changer d'endroit.
- **10 - Data Clumps** : quand on a un groupe de données qui se retrouvent toujours ensemble, c'est qu'elles doivent peut-être rejoindre une même structure.
  - On va d'abord chercher où ces données apparaissent sous forme de champs pour les extraire dans une nouvelle classe avec **[Extract Class](#extract-class)**.
    - On parle bien d'extraire dans une classe et pas dans une simple structure, parce que ça va permettre ensuite d'y ajouter du comportement propre à ces données, typiquement quand on a des Feature Envies.
  - Au niveau des paramètres des fonctions on va alors pouvoir utiliser **[Introduce Parameter Object](#introduce-parameter-object)** et **[Preserve Whole Object](#preserve-whole-object)**.
- **11 - Primitive Obsession** : il s'agit d'utiliser des Value Objects à la place des types primitifs comme number ou string.
  - Exemple : un numéro de téléphone doit être validé, et correctement affiché.
  - La règle typique c'est **[Replace Primitive with Object](#replace-primitive-with-object)**.
  - Si le type primitif est impliqué dans une structure conditionnelle, on peut encapsuler les conditions dans une hiérarchie de classes avec **[Replace Type Code with Subclasses](#replace-type-code-with-subclasses)** puis **[Replace Conditional with Polymorphism](#replace-conditional-with-polymorphism)**.
- **12 - Repeated Switches** : on repère les switchs portant sur la même condition, et on les remplace par des classes.
  - Il s'agit d'utiliser **[Replace Conditional with Polymorphism](#replace-conditional-with-polymorphism)**.
- **13 - Loops** : les fonctions issues de la programmation fonctionnelle (map, filter, reduce) permettent de voir plus rapidement les éléments qui sont inclus et ce qui est fait avec eux, par rapport à des boucles.
  - On peut remplacer les boucles par des pipelines avec **[Replace Loop with Pipeline](#replace-loop-with-pipeline)**.
- **14 - Lazy Element** : parfois certaines classes ou fonctions sont inutiles.
  - Par exemple une fonction dont le corps se lit de la même manière que son nom, ou une classe qui n'a qu'une méthode et qui pourrait être une fonction.
  - On peut les éliminer avec **[Inline Function](#inline-function)** ou **[Inline Class](#inline-class)**.
  - Dans le cas où on veut réduire une hiérarchie de classe, on peut utiliser **[Collapse Hierarchy](#collapse-hierarchy)**.
- **15 - Speculative Generality** : quand on ajoute des mécanismes de flexibilité pour plus tard, au cas où il y en aurait besoin. Il faut s'en débarrasser parce que YAGNI.
  - On peut se débarrasser de classes qui ne font pas grand chose avec **[Collapse Hierarchy](#collapse-hierarchy)**.
  - Les délégations inutiles peuvent être éliminées avec **[Inline Function](#inline-function)** et **[Inline Class](#inline-class)**.
  - Les paramètres inutilisés par les fonctions peuvent être enlevés avec **[Change Function Declaration](#change-function-declaration)**.
  - Si les seuls utilisateurs d'une fonction sont des tests, il faut les supprimer, puis appliquer **[Remove Dead Code](#remove-dead-code)**.
- **16 - Temporary Field** : quand une classe contient un champ utilisé seulement dans certains cas, ça rend le code plus difficile à comprendre.
  - On peut utiliser **[Extract Class](#extract-class)** puis **[Move Function](#move-function)** pour déplacer le code qui utilise le champ qui est à part.
  - Il se peut aussi qu'on puisse réduire le problème au fait de traiter le cas où les variables ne sont pas valides en utilisant **[Introduce Special Case](#introduce-special-case)**.
- **17 - Message Chains** : il s'agit de longues chaînes d'appels d'objet en objet pour obtenir quelque chose au bout du compte. Si une des méthodes d'un des objets de la chaîne change, notre appelant doit changer aussi.
  - On peut utiliser **[Hide Delegate](#hide-delegate)** sur les objets intermédiaires.
  - Une autre solution est de voir si on peut utiliser **[Extract Function](#extract-function)** suivi de **[Move Function](#move-function)** pour déplacer l'utilisation de la chaîne d'appels plus bas dans la chaîne.
- **18 - Middle Man** : il est normal d'encapsuler et de déléguer des choses, mais si une classe délègue la moitié de ses méthodes à une autre classe, c'est qu'il est peut être temps de s'interfacer directement avec la classe qui sait ce qui se passe.
  - La technique à utiliser est **[Remove Middle Man](#remove-middle-man)**.
  - On peut aussi utiliser **[Replace Superclass with Delegate](#replace-superclass-with-delegate)** ou **[Replace Subclass with Delegate](#replace-subclass-with-delegate)** pour fondre le middle man dans la classe cible.
- **19 - Insider Trading** : il s'agit de code de modules différents qui communique trop entre eux, et donc un couplage trop important entre modules.
  - On peut utiliser **[Move Function](#move-function)** et **[Move Field](#move-field)** pour séparer le code qui ne devrait pas être trop couplé.
  - Dans le cas où les modules ont des choses en commun, on peut aussi en extraire une classe commune, ou utiliser **[Hide Delegate](#hide-delegate)** pour utiliser un module comme intermédiaire.
- **20 - Large Class** : une classe avec trop de champs doit être divisée en plusieurs classes.
  - On peut typiquement repérer les noms de variable qui partagent un préfixe ou suffixe commun, et utiliser **[Extract Class](#extract-class)**, ou encore **[Extract Superclass](#extract-superclass)** ou **[Replace Type Code with Subclasses](#replace-type-code-with-subclasses)**.
  - Si la classe a trop de code, on va probablement avoir des duplications. Le mieux est alors de la refactorer en petites fonctions.
    - Par exemple pour une classe de 500 lignes, on peut refactorer en méthodes de 5 à 10 lignes, avec une dizaine de méthodes de 2 lignes extraites dans une classe à part.
  - Un bon indicateur pour découper une classe c'est en regardant le code qui l'utilise, souvent on peut repérer des parties dans la classe.
- **21 - Alternative Classes with Different Interfaces** : ça peut être intéressant de pouvoir substituer une classe par une autre. Pour ça il faut faire correspondre leur prototype en les faisant adhérer à une même interface.
  - Pour faire correspondre les deux classes, on peut utiliser **[Change Function Declaration](#change-function-declaration)** et **[Move Function](#move-function)**.
- **22 - Data Class** : les classes qui ont des getters/setters mais peu ou pas de logique sont un signe que la logique n'est pas au bon endroit.
  - Leurs champs publics doivent être encapsulés avec **[Encapsulate Record](#encapsulate-record)**.
  - Les setters pour les méthodes qui ne doivent pas être changés doivent être enlevés avec **[Remove Setting Method](#remove-setting-method)**.
  - Ensuite on peut chercher où ces getters et setters sont utilisés, et appliquer **[Move Function](#move-function)** (et au besoin **[Extract Function](#extract-function)**) pour déplacer la logique dans la classe qu'on cherche à enrichir.
  - Il y a des exceptions : certaines classes peuvent être légitimes en tant que structure de données, et dans ce cas il n'y a pas besoin de getters et setters. Leurs champs seront immutables et donc n'auront pas besoin d'être encapsulés.
    - Exemple : la structure de données qui permet de communiquer entre les deux phases quand on utilise **[Split Phase](#split-phase)**.
- **23 - Refused Bequest** : parfois certaines classes filles refusent certaines implémentations venant du parent.
  - Il ne s'agit pas d'une forte odeur, donc on peut parfois le tolérer.
  - Si on veut régler le problème, on peut utiliser une classe soeur et pousser le code qui ne devrait pas être partagé vers elle avec **[Push Down Method](#push-down-method)** et **[Push Down Field](#push-down-field)**.
  - Parfois, ce n'est pas l'implémentation d'une méthode, mais l'interface que la classe fille ne veut pas. Dans ce cas l'odeur est beaucoup plus forte et il faut éliminer l'héritage pour le remplacer par de la délégation avec **[Replace Subclass with Delegate](#replace-subclass-with-delegate)** ou **[Replace Superclass with Delegate](#replace-superclass-with-delegate)**.
- **24 - Comments** : la plupart des commentaires cachent des code smells, et sont inutiles si on les refactore.
  - Quand on en rencontre, il faut essayer de voir si on ne peut mieux expliquer ce que fait un bloc de code avec **[Extract Function](#extract-function)** et **[Change Function Declaration](#change-function-declaration)**. Ou encore déclarer des règles sur l'état du système avec **[Introduce Assertion](#introduce-assertion)**.
  - Si malgré ça on a toujours besoin du commentaire, alors c'est qu'il est légitime. Il peut servir à décrire ce qui se passe, indiquer les endroits où on n'est pas sûr, ou encore expliquer pourquoi on a fait quelque chose.

## 4 - Création de tests

- Les tests sont utiles pour le refactoring, mais aussi plus généralement ils permettent d'économiser un **temps conséquent à débugger**.
- Ecrire le **test avant le code** permet :
  - De mieux se concentrer sur le fonctionnalité qu'on veut faire, en s'intéressant d'abord à l'interface plutôt qu'à l'implémentation.
  - De savoir quand la fonctionnalité est terminée : quand le test passe.
- Exemple de programme à tester :
  - On a un plan de production qui doit montrer la production des producteurs classés par province.
  - Le code est organisé avec deux classes : Producer qui contient les données du producteur, et Province qui prend des données JSON, et calcule les éléments liés à la production.
  - Exemple de test pour le déficit de production :
    ```typescript
    describe("province", () => {
      it("shortfall", () => {
        const asia = new Province(sampleProvinceData());
        expect(asia.shortfall).toBe(5);
      });
    });
    ```
- Concernant la manière de **nommer les tests**, Fowler ne prend pas position mais se contente de dire que certains développeurs préfèrent faire des phrases, et d'autres écrire peu de mots, et que lui écrit suffisamment de mots pour reconnaître les tests quand ils sont en erreur.
- Quand on ajoute des tests à du code non testé pour le refactorer, on ne peut pas faire la séquence “red” puis “green”. On peut donc à la place jouer le test qui marche, puis introduire une erreur dans le code et vérifier que le test échoue bien comme on l'aurait pensé.
- L'auteur conseille d'**exécuter les tests souvent** : si on ne fait pas de TDD, les tests du module sur lequel on travaille au moins toutes les 10 minutes, et l'ensemble des tests au moins une fois par jour.
- Il ne faut tester que le code qui en vaut la peine. Typiquement, les getters/setters n'ont pas à être testés parce que le risque d'erreur est faible.
- Il faut **factoriser les répétitions** dans les tests comme on le fait pour le code de prod.
- Comme pour le code de prod, il faut à tout prix **éviter les variables globales** (et y compris si elles sont assignées dans un `beforeAll`), et utiliser soit des variables locales aux tests, soit des `beforeEach`.
- Fowler aime bien avoir un **dispositif de test standard** pour chaque groupe de tests, avec lequel il faut se familiariser avant de les regarder.
  - `beforeEach` peut être une convention pour ce dispositif.
  - Le `describe` permet de créer des groupes de tests où il y aura un `beforeEach`. Si notre test ne rentre pas dans le contexte du dispositif, on pourra créer un autre bloc avec éventuellement un autre dispositif.
- Il conseille d'essayer de se **limiter à une assertion par test**, sauf si les éléments testés sont étroitement liés.
  - L'argument est que si le test échoue pour un des premiers asserts, on ne saura pas si les autres échouaient ou non.
  - NDLR : C'est en contradiction avec le conseil de Vladimir Khorikov sur le fait qu'il faille un seul “act”, et qu'au niveau du “assert” on teste toujours toutes les conséquences.
- Il faut aussi **tester les cas aux limites** :
  - Si on a une collection, on peut voir ce qui se passe si elle est vide.
  - Si on a un nombre, on peut voir ce qui se passe avec 0 ou un nombre négatif.
  - Si on a une chaîne, on peut voir ce qui se passe avec la chaîne vide.
  - Ça nous oblige à chaque fois à nous poser la question de savoir si ça a un sens. Par exemple, si le nombre négatif n'aurait pas de sens, on peut s'attendre à une erreur.
  - Il est possible que l'entrée vienne d'un module déjà testé, dans ce cas il n'est pas nécessaire de tester notre module avec des cas qui ne devraient pas se produire.
  - Si un cas d'erreur conduit à une potentielle corruption de données, on peut utiliser **[Introduce Assertion](#introduce-assertion)** pour éviter que ça arrive. Il n'y a pas besoin de tester ce genre de cas.
    - NDLR : on est sur du _fail fast_.
- Quand on a un bug, l'auteur conseille d'abord d'écrire un test qui reproduit le bug, et ensuite de le faire passer au vert.
- La bonne quantité de tests c'est quand on est **suffisamment confiant pour faire du refactoring** et savoir que si on introduit un bug, il sera révélé par les tests.

## 5 - Présentation du catalogue

- Les chapitres suivants présentent un catalogue de techniques de refactoring suffisamment importants pour être nommés et décrits.
- Martin Fowler utilise lui-même le catalogue pour se refamiliariser avec des techniques de refactorings qu'il n'a pas faites depuis longtemps.

## 6 - Premier ensemble de refactorings

### Extract Function

- **Exemple :**

  - Avant :
    ```javascript
    function printOwig(invoices) {
      printBanner();
      let outstanding = calculateOutstanding();
      // display details;
      console.log(`name: ${invoice.customer}`);
      console.log(`amount: ${outstanding}`);
    }
    ```
  - Après :

    ```javascript
    function printOwing(invoice) {
      printBanner();
      let outstanding = calculateOutstanding();
      printDetails(outstanding);

      function printDetails(outstanding) {
        console.log(`name: ${invoice.customer}`);
        console.log(`amount: ${outstanding}`);
      }
    }
    ```

- **Étapes :**
  - 1. On crée une nouvelle fonction pour accueillir le code à extraire, on la nomme correctement, et on **copie** le code à extraire dedans.
    - Si le langage le supporte (comme JavaScript), on peut extraire la fonction **dans la portée de la première fonction**, ne serait-ce que pour visualiser si l'extraction a du sens sans que ce soit coûteux. On pourra la déplacer éventuellement plus tard.
  - 2. Dans le cas où la fonction est exportée hors de la portée de la fonction source, on lui passe toutes les variables dont elle a besoin en paramètre.
    - Si une variable n'est utilisée que dans le code extrait, on la déplace dedans.
    - Si une variable utilisée en dehors est assignée à l'intérieur du code extrait, alors il faut faire en sorte que la fonction extraite la retourne.
    - Si on a trop de variables assignées dans le code extrait, on abandonne l'extraction au profit d'abord de **[Split Variable](#split-variable)** ou de **[Replace Temp with Query](#replace-temp-with-query)**.
  - 3. On Compile.
  - 4. On remplace le code initialement extrait par un appel à la nouvelle fonction.
  - 5. On teste.
  - 6. On recherche d'autres bouts de code similaires au code extrait pour les remplacer par un appel à la nouvelle fonction avec **[Replace Inline Code with Function Call](#replace-inline-code-with-function-call)**.
- **Théorie :**
  - Le bon argument sur pourquoi extraire une fonction est de **séparer l'intention de l'implémentation**, pour que l'intention saute aux yeux à la première lecture.
  - L'auteur a tendance à écrire des **fonctions courtes**, une fonction dépassant une demi-douzaine de lignes commence à sentir mauvais, et les fonctions d'une ligne ne sont pas rares pour peu qu'elles expriment mieux l'intention.
  - La qualité du **nommage** est fondamentale avec les petites fonctions.
    - Si on n'arrive pas à trouver un nom pour la fonction extraite, c'est peut être un signe que l'extraction est inutile.
    - Il est normal d'extraire, de manipuler le code, puis de se rendre compte que l'extraction était inutile. Tant qu'on a appris quelque chose, on n'a pas perdu son temps.
  - Typiquement si on a un commentaire qui marque la séparation d'un bloc de code, c'est une bonne heuristique pour extraire.

### Inline Function

- **Exemple :**

  - **Avant :**

    ```typescript
    function getRating(driver) {
      return moreThanFiveLateDeliveries(driver) ? 2 : 1;
    }

    function moreThanFiveLateDeliveries(driver) {
      return driver.numberOfLateDeliveries > 5;
    }
    ```

  - **Après :**
    ```typescript
    function getRating(driver) {
      return driver.numberOfLateDeliveries > 5 ? 2 : 1;
    }
    ```

- **Étapes :**
  - 1. On vérifie que la fonction n'est pas une méthode polymorphe.
  - 2. On trouve les endroits où la fonction est appelée et on remplace par le corps de la fonction.
  - 3. On teste après chaque remplacement.
  - 4. On peut supprimer la fonction qui n'est plus appelée.
- **Théorie :**
  - Les fonctions courtes sont plus lisibles, mais parfois on peut avoir des fonctions qui n'apportent rien, parce que leur contenu est déjà clair. Dans ce cas on les enlève.
  - Une autre raison d'enlever les fonctions c'est pour d'abord **regrouper le code dans un grand bloc avant de mieux le découper**.
  - Dans le cas où on rencontre des difficultés importantes pour faire l'incorporation, l'auteur conseille de ne pas faire ce refactoring.

### Extract Variable

- **Exemple :**
  - **Avant :**
    ```javascript
    return (
      order.quantity * order.itemPrice -
      Math.max(0, order.quantity * 500) * order.itemPrice -
      0.05 +
      Math.min(order.quantity * order.itemPrice * 0.1, 100)
    );
    ```
  - **Après :**
    ````javascript
    const basePrice = order.quantity * order.itemPrice;
    const quantityDiscount = Math.max(0, order.quantity - 500) *
      order.itemPrice * 0.05;
    const shipping = Math.min(basePrice * 0.1, 100);
    return basePrice - quantityDiscount + shipping;
    ```
    ````
- **Étapes :**
  - 1. On vérifie que l'expression qu'on veut extraire n'a pas de side effects.
  - 2. On crée une nouvelle variable immutable avec la valeur de l'expression.
  - 3. On remplace l'expression d'origine par la nouvelle variable.
  - 4. On teste.
- **Théorie :**
  - Ces variables permettent de décomposer le code pour le rendre plus lisible.
  - Une fois qu'on a choisi le nom, on réfléchit au contexte : si c'est un contexte local une variable est très bien, si c'est plus global il vaut mieux une fonction qu'on réutilisera dans plusieurs endroits avec le même nom.
    - Typiquement si on est dans une classe, il y a des chances pour que le concept qu'on extrait puisse devenir un membre de la classe (par exemple avec `get monConcept()`).

### Inline Variable

- **Exemple :**
  - **Avant :**
    ```javascript
    let basePrice = anOrder.basePrice;
    return basePrice > 1000;
    ```
  - **Après :**
    ```javascript
    return anOrder.basePrice > 1000;
    ```
- **Étapes :**
  - 1. On vérifie que l'expression qu'on veut réintégrer n'a pas de side effects.
  - 2. Si elle ne l'est pas, on rend la variable immutable pour éviter qu'elle ne soit réassignée plus bas.
  - 3. On remplace l'utilisation de la variable par son expression, occurrence par occurrence, en testant à chaque fois.
  - 4. On supprime la variable et on teste.
- **Théorie :**
  - Quand un nom de variable devient inutile, on peut la supprimer.

### Change Function Declaration

- **Exemple :**
  - **Avant :**
    ```javascript
    function circum(radius) {...}
    ```
  - **Après :**
    ```javascript
    function circumference(radius) {...}
    ```
- **Étapes (version simple) :**
  - 1. Si on supprime des paramètres, on s'assure qu'ils ne sont plus référencés.
  - 2. On remplace la déclaration de la fonction par la nouvelle déclaration.
  - 3. On trouve tous les endroits où l'ancienne déclaration était appelée, et on les met à jour.
  - 4. On teste.
- **Étapes (version progressive) :**
  - 1. Si besoin, on refactore le corps de la fonction pour faciliter l'extraction.
  - 2. On extrait l'ensemble du corps de la fonction dans une nouvelle fonction avec **[Extract Function](#extract-function)**.
    - Si elle va avoir le même nom que l'ancienne, on la nomme avec un nom temporaire.
  - 3. Si on a des changements de paramètres, on peut utiliser la version simple pour les changer sur la nouvelle fonction.
    - L'ancienne fonction devra donner des valeurs par défaut pour les nouveaux paramètres pour continuer à marcher.
  - 4. On teste.
  - 5. On applique **[Inline Function](#inline-function)** à l'ancienne fonction pour la faire disparaître.
  - 6. Si besoin de modifier le nom parce qu'on avait un nom temporaire, on le fait.
  - 7. On teste.
- **Théorie :**
  - Les noms des fonctions sont difficiles à choisir, et on doit souvent s'y prendre à plusieurs fois. Dès qu'on a un nom meilleur pour une fonction, il faut l'utiliser.
  - Une bonne manière de trouver un bon nom est d'écrire en commentaire ce que fait la fonction, puis de transformer ça en nom.
  - La même difficulté se retrouve avec les paramètres, on doit donc régulièrement refactorer pour améliorer petit à petit..
    - Par exemple, il n'y a pas de bonne réponse au fait de savoir s'il faut passer un objet entier à une fonction ou seulement l'une de ses propriétés qui est actuellement utilisée.
  - La version simple sert pour les cas simples, alors que la version progressive est à utiliser dans le cas où on a de nombreuses occurrences à remplacer, ou d'autres difficultés comme une méthode polymorphe.
    - Il est préférable de faire la modification de nom et de paramètres en deux étapes pour la version simple.
    - De manière générale, si la méthode simple ne marche pas du premier coup, il faut annuler les changements et recommencer avec la méthode progressive.
    - Si on refactore une API, il faut appliquer la méthode progressive et faire une pause dès qu'on a la nouvelle fonction, sans supprimer l'ancienne. On la supprimera quand on sera sûr que les clients ont migré.
- **Exemple détaillé :**

  - On a une fonction qui détermine si un client est basé en Nouvelle-Angleterre ou pas. On veut qu'elle prenne plutôt la valeur indiquant le lieu et non plus l'objet client entier.

    ```javascript
    function inNewEngland(aCustomer) {
      return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(
        aCustomer.address.state
      );
    }

    const newEnglanders = someCustomers.filter((c) => inNewEngland(c));
    ```

  - On va utiliser la version progressive :

    - D'abord on utilise **[Extract Variable](#extract-variable)** pour faciliter l'extraction de fonction.
      ```javascript
      function inNewEngland(aCustomer) {
        const stateCode = aCustomer.address.state;
        return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(stateCode);
      }
      ```
    - Ensuite on applique **[Extract Function](#extract-function)**.

      ```javascript
      function inNewEngland(aCustomer) {
        const stateCode = aCustomer.address.state;
        return xxNEWinNewEngland(stateCode);
      }

      function xxNEWinNewEngland(stateCode) {
        return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(stateCode);
      }
      ```

    - Puis on applique **[Inline Variable](#inline-variable)** à la fonction initiale.
      ```javascript
      function inNewEngland(aCustomer) {
        return xxNEWinNewEngland(aCustomer.address.state);
      }
      ```
    - Puis on utilise **[Inline Function](#inline-function)** pour remplacer les appels à l'ancienne fonction par des appels à la nouvelle fonction, petit à petit. A la fin on supprime l'ancienne fonction.
      ```javascript
      const newEnglanders = someCustomers.filter((c) =>
        xxNEWinNewEngland(c.address.state)
      );
      ```
    - On réutilise **[Change Function Declaration](#change-function-declaration)**, cette fois avec la version simple, pour changer le nom de la nouvelle fonction. On obtient alors la version finale.

      ```javascript
      function inNewEngland(stateCode) {
        return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(stateCode);
      }

      const newEnglanders = someCustomers.filter((c) =>
        inNewEngland(c.address.state)
      );
      ```

### Encapsulate Variable

- **Exemple :**

  - **Avant :**
    ```javascript
    let defaultOwner = { firstName: "Martin", lastName: "Fowler" };
    ```
  - **Après :**

    ```javascript
    let defaultOwnerData = { firstName: "Martin", lastName: "Fowler" };

    export function defaultOwner() {
      return defaultOwnerData;
    }
    export function setDefaultOwner(arg) {
      defaultOwnerData = arg;
    }
    ```

- **Étapes :**
  - 1. On crée des fonctions pour lire et écrire dans la variable qu'on veut encapsuler.
  - 2. On remplace chaque référence à la variable par un appel à ces fonctions, en testant à chaque fois.
  - 3. On limite la visibilité de la variable.
    - Si le langage ne le permet pas, on la renomme pour voir si ça casse quelque chose.
  - 4. On teste.

5. Si la variable est un record, on peut envisager **[Encapsulate Record](#encapsulate-record)**.

- **Théorie :**

  - Alors qu'il est facile de déplacer une fonction, ça l'est beaucoup moins pour des données, par exemple des variables globales. L'encapsulation peut donc être une première étape pour ce déplacement.
  - L'autre avantage c'est de pouvoir ajouter un traitement systématique à chaque lecture ou écriture de la donnée.
  - Dans le cas où la donnée est immutable, ces problèmes se posent moins : on peut les copier facilement au lieu de les déplacer, et on n'a pas besoin d'ajouter un traitement à la lecture ou à l'écriture.
  - L'auteur **encapsule toutes les données mutables qui dépassent la portée d'une seule fonction**.
    - Pour y arriver, il saisit chaque occasion d'accéder à nouveau à une donnée mutable qui est hors de la fonction pour l'encapsuler.
  - En JavaScript on peut implémenter cette technique avec l'utilisation de ES modules : on exporte la fonction getter et setter mais pas la variable.
  - Pour ce qui est de la convention de nommage, l'auteur déconseille le préfixe _get_, mais préfère laisser le préfixe _set_ parce qu'il n'aime pas la pratique de l'_overloaded getter setter_ qui consiste à leur donner le même nom.
  - Dans le cas où on veut contrôler ce qui arrive au contenu de notre variable (si elle est une référence) :

    - On peut renvoyer une copie au moment de la lecture.
    - Ou on peut utiliser **[Encapsulate Record](#encapsulate-record)** :

      ```javascript
      let defaultOwnerData = { firstName: "Martin", lastName: "Fowler" };
      export function defaultOwner() {
        return new Person(defaultOwnerData);
      }
      export function setDefaultOwner(arg) {
        defaultOwnerData = arg;
      }

      class Person {
        constructor(data) {
          this._lastName = data.lastName;
          this._firstName = data.firstName;
        }
        get lastName() {
          return this._lastName;
        }
        get firstName() {
          return this._firstName;
        }
      }
      ```

### Rename Variable

- **Exemple :**
  - **Avant :**
    ```javascript
    let a = height * width;
    ```
  - **Après :**
    ```javascript
    let area = height * width;
    ```
- **Étapes :**
  - 1. Dans le cas où la variable est utilisée au-delà de la fonction, on peut envisager **[Encapsulate Variable](#encapsulate-variable)**.
  - 2. On recherche toutes les références à la variable, et on les modifie.
  - 3. On teste.
- **Théorie :**
  - Les noms de variable sont moins importants quand leur portée est petite, et requièrent une plus grande attention quand elle est plus grande.
  - Un nom peut évoluer pour diverses raisons, parmi lesquelles une meilleure compréhension du problème, ou l'évolution de la solution.
  - Dans le cas où le changement de nom est difficile, il vaut mieux passer par encapsuler la variable, et en général on va vouloir garder cette encapsulation.
  - Si la variable est immutable, on peut passer par des remplacements progressifs en testant au fur et à mesure, sans avoir besoin d'encapsuler.

### Introduce Parameter Object

- **Exemple :**
  - **Avant :**
    ```javascript
    function amountInvoiced(startDate, endDate) {...}
    function amountReceived(startDate, endDate) {...}
    function amountOverdue(startDate, endDate) {...}
    ```
  - **Après :**
    ```javascript
    function amountInvoiced(aDateRange) {...}
    function amountReceived(aDateRange) {...}
    function amountOverdue(aDateRange) {...}
    ```
- **Étapes :**
  - 1. Si on n'a pas encore de structure pour regrouper les paramètres visés, on la crée.
    - L'auteur préfère créer une **classe** pour y mettre de la logique, dans l'idée d'avoir un **Value Object**.
  - 2. On teste.
  - 3. On utilise **[Change Function Declaration](#change-function-declaration)** pour ajouter une instance de notre nouvelle structure en paramètre aux fonctions qui prennent les paramètres qu'on veut grouper.
  - 4. On teste.
  - 5. On modifie chaque appelant pour qu'il ajoute l'instance de la nouvelle structure, et on teste à chaque fois.
  - 6. On remplace l'utilisation de chacun des anciens paramètres par les éléments de la nouvelle structure déjà passée en paramètre. Et on supprime à chaque fois l'ancien paramètre concerné.
  - 7. On teste.
- **Théorie :**
  - Grouper les paramètres qui voyagent souvent ensemble dans le même objet permet :
    - d'avoir **moins de paramètres** à passer aux fonctions.
    - de créer des groupements cohérents, permettant de **mieux comprendre le domaine**.
    - de **rapatrier de la logique** spécifique à l'objet en question dans une classe qu'on crée pour contenir les paramètres qui vont ensemble.
- **Exemple détaillé :**

  - On a plusieurs fonctions qui prennent des paramètres _min_ et _max_.

    ```javascript
    function readingsOutsideRange(station, min, max) {
      return station.readings.filter((r) => r.temp < min || r.temp > max);
    }

    alerts = readingsOutsideRange(
      station,
      operatingPlan.temperatureFloor,
      operatingPlan.temperatureCeiling
    );
    ```

  - On va créer une classe pour contenir _min_ et _max_, et ajouter un paramètre supplémentaire à notre fonction.

    ```javascript
    class NumberRange {
      constructor(min, max) {
        this._data = {min: min, max: max};
      }
      get min() {
        return this._data.min;
      }
      get max() {
        return this._data.max;
      }
    }

    function readingsOutsideRange(station, min, max, range) {
      // ...
    ```

  - On modifie les appelants pour qu'ils fournissent le nouveau paramètre _range_.
    ```javascript
    const range = new NumberRange(
      operatingPlan.temperatureFloor,
      operatingPlan.temperatureCeiling
    );
    alerts = readingsOutsideRange(
      station,
      operatingPlan.temperatureFloor,
      operatingPlan.temperatureCeiling,
      range
    );
    ```
  - On supprime les anciens paramètres un par un, _min_ puis _max_.

    ```javascript
    function readingsOutsideRange(station, range) {
      return station.readings.filter(
        (r) => r.temp < range.min || r.temp > range.max
      );
    }

    alerts = readingsOutsideRange(station, range);
    ```

  - Et enfin pour aller un peu plus loin que ce refactoring, on va tirer parti du fait qu'on a un Value Object pour y placer de la logique.

    ```javascript
    class NumberRange {
      // ...
      contains(arg) {
        return arg >= this.min && arg <= this.max;
      }
      // ...
    }

    function readingsOutsideRange(station, range) {
      return station.readings.filter((r) => !range.contains(r.temp));
    }
    ```

### Combine Functions into Class

- **Exemple :**
  - **Avant :**
    ```javascript
    function base(aReading) {...}
    function taxableCharge(aReading) {...}
    function calculateBaseCharge(aReading) {...}
    ```
  - **Après :**
    ```javascript
    class Reading {
      base() {...}
      taxableCharge() {...}
      calculateBaseCharge() {...}
    }
    ```
- **Étapes :**
  - 1. On applique **[Encapsulate Record](#encapsulate-record)** aux paramètres communs entre les fonctions.
    - Si ces paramètres ne sont pas déjà au sein d'une même structure, on applique d'abord **[Introduce Parameter Object](#introduce-parameter-object)** pour les regrouper.
  - 2. On utiliser **[Move Function](#move-function)** pour déplacer chaque fonction visée dans la nouvelle classe qu'on vient de créer.
    - On peut supprimer les arguments de ces fonctions qui sont déjà membres de la classe.
  - 3. On va ensuite à la recherche de la logique restante qui manipule la donnée qu'on a encapsulée, pour l'extraire sous forme de fonctions avec **[Extract Function](#extract-function)** et la rapatrier dans la nouvelle classe.
- **Théorie :**

  - Un des usages des classes c'est de grouper des fonctions qui prennent des paramètres communs, pour donner ces paramètres au constructeur et éviter d'avoir à les donner à chaque appel de fonction.
  - En fonction du contexte, si on n'a pas besoin de modifier les instances indépendamment mais plutôt grouper deux fonctions à appeler ensemble, on voudra peut-être plutôt utiliser **[Combine Functions into Transform](#combine-functions-into-transform)**.
  - On peut aussi utiliser les fonctions imbriquées à la place des classes pour mutualiser les paramètres, mais dans ce cas on ne pourra accéder qu'à la fonction de plus haut niveau. La classe est plus flexible.
  - Il ne faut pas hésiter à créer des méthodes sous forme de getter pour aller dans le sens du **Uniform Access Principle**.

    - Exemple : l'appelant accède à _baseCharge_ comme si c'était une valeur, et ne sait pas si on a une fonction qui est appelée derrière ou pas.

      ```javascript
      class Reading {
        // ...
        get baseCharge() {
          return this.baseRate(this.month, this.year) * this.quantity;
        }
      }

      const aReading = new Reading(rawReading);
      const basicChargeAmount = aReading.baseCharge;
      ```

### Combine Functions into Transform

- **Exemple :**
  - **Avant :**
    ```javascript
    function base(aReading) {...}
    function taxableCharge(aReading) {...}
    ```
  - **Après :**
    ```javascript
    function enrichReading(argReading) {
      const aReading = _.cloneDeep(argReading);
      aReading.baseCharge = base(aReading);
      aReading.taxableCharge = taxableCharge(aReading);
      return aReading;
    }
    ```
- **Étapes :**
  - 1. On crée une fonction qui prend le record à transformer et en retourne une deep copy.
    - On aura sans doute besoin d'un test pour vérifier la nature de la copie.
  - 2. On prend une partie de la logique qu'on veut utiliser pour la transformation et on la déplace dans la fonction de transformation.
  - 3. On teste.
  - 4. On déplace les autres parties de logique, et on teste à chaque fois.
- **Théorie :**
  - On aurait pu simplement utiliser Extract Function pour extraire la logique commune, mais l'intérêt de combiner des fonctions avec les données sur lesquelles elles opèrent (avec **[Combine Functions into Class](#combine-functions-into-class)** et **[Combine Functions into Transform](#combine-functions-into-transform)**) c'est qu'on les retrouve plus facilement qu'une fonction qui se balade.
  - D'un point de vue convention de nommage, l'auteur aime bien le préfixe _enrich_ quand la fonction renvoie la même donnée modifiée, et _transform_ quand c'est une donnée qu'il estime être autre chose.
  - Pour la deep copy, on peut utiliser lodash.

### Split Phase

- **Exemple :**

  - **Avant :**
    ```javascript
    const orderData = orderString.split(/\s+/);
    const productPrice = priceList[orderData[0].split("-")[1]];
    const orderPrice = parseInt(orderData[1]) * productPrice;
    ```
  - **Après :**

    ```javascript
    const orderRecord = parseOrder(order);
    const orderPrice = price(orderRecord, priceList);

    function parseOrder(aString) {
      const values = aString.split(/\s+/);
      return {
        productID: values[0].split("-")[1],
        quantity: parseInt(values[1]),
      };
    }

    function price(order, priceList) {
      return order.quantity * priceList[order.productID];
    }
    ```

- **Étapes :**
  - 1. On extrait le code de la 2ème phase dans sa fonction avec **[Extract Function](#extract-function)**.
  - 2. On teste.
  - 3. On introduit une structure de données en paramètre de la nouvelle fonction.
  - 4. On teste.
  - 5. On examine chaque paramètre pris par la fonction de la 2ème phase :
    - Si elle est aussi utilisée dans la 1ère phase, on la déplace dans la structure de données.
    - Si elle n'est pas utilisée par la 1ère phase (la fonction principale prend le paramètre et le transfert à la fonction de la 2ème phase), on la laisse en paramètre.
    - On teste après chaque déplacement.
  - 6. On extrait le code de la 1ère phase dans une fonction à part retournant la structure de données avec **[Extract Function](#extract-function)**.
- **Théorie :**
  - Quand on a un code qui fait deux choses différentes, on peut le séparer en deux phases distinctes, pour que la prochaine fois qu'on le modifie, on puisse éventuellement intervenir sur une seule des deux phases indépendamment de l'autre.
  - Souvent on va séparer une phase de mise en forme d'une phase de calcul, mais ça peut aussi être par exemple deux phases de calcul indépendantes.

## 7 - Encapsulation

### Encapsulate Record

- **Exemple :**
  - **Avant :**
    ```javascript
    organization = { name: "Acme Gooseberries", country: "GB" };
    ```
  - **Après :**
    ```javascript
    class Organization {
      constructor(data) {
        this._name = data.name;
        this._country = data.country;
      }
      get name() {
        return this._name;
      }
      set name(arg) {
        this._name = arg;
      }
      get country() {
        return this._country;
      }
      set country(arg) {
        this._country = arg;
      }
    }
    ```
- **Étapes :**
  - 1. On utilise **[Encapsulate Variable](#encapsulate-variable)** sur la variable qui a la valeur du record.
  - 2. On wrap la variable par une classe qui permet d'y accéder avec un accesseur, et on fait en sorte que les fonctions d'accès/modification qu'on vient de créer à l'étape 1 utilisent cet accesseur.
  - 3. On teste.
  - 4. On crée des fonctions d'accès/modification qui renvoie l'objet wrappant le record.
  - 5. Pour chaque utilisation extérieure du record, on va utiliser les nouvelles fonctions qui renvoient l'objet wrapper.
    - Si le record est complexe, on va s'intéresser d'abord aux occurrences qui le modifient.
    - On peut envisager de retourner une copie ou une version read-only pour les occurrences qui ne font que de la lecture.
  - 6. On supprime les fonctions qui permettent d'accéder au record non wrappé sur l'objet, et en dehors.
  - 7. On teste.
  - 8. Si le record a des champs qui sont eux-mêmes des records, on peut envisager d'utiliser **[Encapsulate Record](#encapsulate-record)** et **[Encapsulate Collection](#encapsulate-collection)** de manière récursive sur ces champs aussi.
- **Théorie :**
  - Les _records_ sont des structures permettant de regrouper des données, souvent sous forme de HashMaps, en javascript des objets `{}`.
  - Quand ces structures sont utilisées dans de nombreux endroits, on peut avoir du mal à trouver comment les utiliser. C'est pour cela qu'il est pratique d'y ajouter de la logique en les transformant en classe.
  - Il peut y avoir un trade off entre encapsuler tous les champs (en particulier pour les structures imbriquées) dans la classe qui les englobe, et retourner la structure pour que le client puisse l'explorer en mode read-only ou en donnant une copie.
    - Wrapper tous les champs permet d'avoir plus de flexibilité mais demande plus de travail et n'en vaut pas toujours la peine.
    - Comme on ne veut pas casser l'encapsulation dans le cas des modifications, on peut soit créer une copie (mais ça peut poser des problèmes de performance), soit renvoyer un objet read-only (mais c'est difficile à faire en JavaScript.
      - NDLR : Immer permet par exemple de retourner l'équivalent d'une valeur read-only. Typescript permet aussi de le faire à la compilation.
- **Exemple détaillé :**

  - On a un record :
    ```javascript
    const organization = { name: "Acme Gooseberries", country: "GB" };
    ```
  - On va encapsuler la variable en la déplaçant dans un module, et en ajoutant un accesseur avec un nom moche qui sera bientôt supprimé.
    ```javascript
    function getRawDataOfOrganization() {
      return organization;
    }
    ```
  - On encapsulate la variable dans une classe pour pouvoir ensuite la protéger plus finement, et on ajoute un accesseur pour l'instance de cette classe.

    ```javascript
    class Organization {
      constructor(data) {
        this._data = data;
      }
    }

    const organization = new Organization({
      name: "Acme Gooseberries",
      country: "GB",
    });

    function getRawDataOfOrganization() {
      return organization._data;
    }

    function getOrganization() {
      return organization;
    }
    ```

  - On va maintenant chercher tous les endroits où on lit ou écrit dans notre record, et ajouter des getters ou setters dans notre classe.

    ```javascript
    class Organization {
      // ...
      set name(aString) {
        this._data.name = aString;
      }
      get name() {
        return this._data.name;
      }
    }

    // là où on met à jour le record
    getOrganization().name = newName;
    // là où on lit le record
    result += `<h1>${getOrganization().name}</h1>`;
    ```

  - On peut maintenant supprimer l'accesseur du record brut qui avait un nom bâclé (`getRawDataOfOrganization()`).
  - Et enfin on peut déplacer les champs du record dans la classe, et créer des getters/setters pour ces champs.
    ```javascript
    class Organization {
      constructor(data) {
        this._name = data.name;
        this._country = data.country;
      }
      get name() {
        return this._name;
      }
      set name(aString) {
        this._name = aString;
      }
      get country() {
        return this._country;
      }
      set country(aCountryCode) {
        this._country = aCountryCode;
      }
    }
    ```

### Encapsulate Collection

- **Exemple :**
  - **Avant :**
    ```javascript
    class Person {
      // ...
      get courses() {
        return this._courses;
      }
      set courses(aList) {
        this._courses = aList;
      }
    }
    ```
  - **Après :**
    ```javascript
    class Person {
      // ...
      get courses() {
        return this._courses.slice();
      }
      addCourse(aCourse) {
        // ...
      }
      removeCourse(aCourse) {
        // ...
      }
    }
    ```
- **Étapes :**
  - 1. On applique **[Encapsulate Variable](#encapsulate-variable)** sur la collection si elle n'est pas déjà encapsulée avec la valeur originale privée, et l'accès par getter et setter.
  - 2. On ajoute des méthodes pour ajouter et supprimer un objet de la collection.
  - 3. Si il existe un setter pour la collection, on le supprime avec **[Remove Setting Method](#remove-setting-method)**.
  - 4. On lance la vérification statique du code.
  - 5. On remplace toutes les modifications de la collection par des appels aux nouvelles méthodes d'ajout et de suppression. On teste à chaque fois.
  - 6. On modifie le getter pour renvoyer une copie ou un objet en lecture seule.
  - 7. On teste.
- **Théorie :**
  - L'encapsulation est utile pour les données mutables, pour pouvoir contrôler leur modification, mais si on donne une référence à une collection, on ne contrôle plus la manière dont elle sera modifiée.
  - L'auteur trouve dommage cependant d'empêcher entièrement l'accès à la collection, étant donné que le langage offre de nombreuses options pour la parcourir et la manipuler, options qu'on aurait du mal à recoder dans notre classe.
    - Ce qu'on peut faire c'est en fournir une copie, et utiliser les méthodes _add_ et _remove_ de la classe quand on veut vraiment la modifier.
    - On peut aussi, à la place de la copie, fournir la collection en lecture seule. Mais il faut choisir l'une des deux comme convention pour la codebase et s'y tenir.

### Replace Primitive with Object

- **Exemple :**
  - **Avant :**
    ```javascript
    orders.filter((o) => "high" === o.priority || "rush" === o.priority);
    ```
  - **Après :**
    ```javascript
    orders.filter((o) => o.priority.higherThan(new Priority("normal")));
    ```
- **Étapes :**
  - 1. On applique **[Encapsulate Variable](#encapsulate-variable)** si la valeur n'est pas déjà encapsulée, en créant un getter et un setter.
  - 2. On crée une classe qui prend la valeur dans son constructeur, et fournit un getter pour cette donnée qu'elle contient.
  - 3. On lance la vérification statique du code.
  - 4. On modifie le setter de la variable encapsulée pour qu'il crée une instance de la classe qu'on a créée et qu'il la stocke.
  - 5. On modifie le getter de la variable encapsulée pour qu'il renvoie le résultat du getter de l'instance de la classe qu'on a créée.
  - 6. On teste.
  - 7. On utilise éventuellement **[Rename Function](#change-function-declaration)** sur le getter et le setter pour leur donner un meilleur nom (étant donné qu'on manipule maintenant une classe qui contient la valeur et non pas la valeur elle-même).
  - 8. On peut clarifier le rôle de notre nouvel objet en tant que Value Object ou Reference Object, en appliquant **[Change Reference to Value](#change-reference-to-value)** ou **[Change Value to Reference](#change-value-to-reference)**.
- **Théorie :**
  - De nombreux programmeurs expérimentés considèrent que c'est un des refactorings les plus précieux.
  - A chaque fois qu'une valeur sert à autre chose qu'à un simple affichage, l'auteur crée une classe pour contenir la valeur et de la logique liée à la valeur.
- **Exemple détaillé :**

  - On a une classe _Order_ avec un champ _priority_.

    ```javascript
    class Order {
      constructor(public priority) {}
    }

    const highPriorityCount = orders.filter(
      o => "high" === o.priority || "rush" === o.priority
    ).length;
    ```

  - On encapsule la variable avec un getter et un setter.
    ```javascript
    class Order {
      constructor(private _priority) {}
      get priority() {
        return this._priority;
      }
      set priority(aString) {
        this._priority = aString;
      }
    }
    ```
  - On crée une classe pour encapsuler la valeur.
    ```javascript
    class Priority {
      constructor(value) {
        this._value = value;
      }
      toString() {
        return this._value;
      }
    }
    ```
  - On modifie les getter/setter d'_Order_ pour manipuler une instance de la nouvelle classe _Priority_.
    ```javascript
    class Order {
      constructor(private _priority) {}
      get priority() {
        return this._priority.toString();
      }
      set priority(aString) {
        this._priority = new Priority(aString);
      }
    }
    ```
  - On renomme le getter en _priorityString_ pour mieux refléter ce qu'il fait.
  - On décide que Priority va devenir un Value Object, pour qu'il puisse être utile aussi en dehors d'_Order_.
    ```javascript
    class Priority {
      constructor(value) {
        if (value instanceof Priority) return value;
        if (Priority.legalValues().includes(value)) this._value = value;
        else throw new Error(`<${value}> is invalid for Priority`);
      }
      toString() {
        return this._value;
      }
      get _index() {
        return Priority.legalValues().findIndex((s) => s === this._value);
      }
      static legalValues() {
        return ["low", "normal", "high", "rush"];
      }
      higherThan(other) {
        return this._index > other._index;
      }
    }
    ```

### Replace Temp with Query

- **Exemple :**

  - **Avant :**
    ```javascript
    const basePrice = this._quantity * this._itemPrice;
    if (basePrice > 1000) return basePrice * 0.95;
    else return basePrice * 0.98;
    ```
  - **Après :**

    ```javascript
    get basePrice() {
      this._quantity * this._itemPrice;
    }

    //...
    if (this.basePrice > 1000)
      return this.basePrice * 0.95;
    else
      return this.basePrice * 0.98;
    ```

- **Étapes :**
  - 1. On s'assure que la valeur de la variable est calculée avant son utilisation, et que le code qui la calcule ne renvoie pas des valeurs différentes à chaque appel.
  - 2. On transforme si possible la variable en lecture seule (const).
  - 3. On teste.
  - 4. On extrait le calcul de la variable dans une fonction.
    - Si la variable et la fonction doivent avoir le même nom, on choisit un nom temporaire pour la fonction.
    - Si la fonction extraite a des effets secondaires, on va les séparer en utilisant **[Separate Query from Modifier](#separate-query-from-modifier)**.
  - 5. On teste.
  - 6. On utilise **[Inline Variable](#inline-variable)** pour enlever la variable temporaire initiale.
- **Théorie :**
  - Utiliser des appels de fonction au lieu de variables temporaires permet de :
    - faciliter le refactoring parce qu'on a moins de paramètres à passer aux fonctions qu'on extrait.
    - éviter la duplication de logique en appelant la même fonction à chaque fois qu'on croise cette logique-là.
  - Les classes sont particulièrement propices à l'utilisation de ce genre de fonctions.
  - Ce refactoring ne marche qu'avec les variables qui sont calculées une fois puis lues mais pas assignées à nouveau. Il faut aussi que leur calcul soit déterministe.

### Extract Class

- **Exemple :**

  - **Avant :**
    ```javascript
    class Person {
      get officeAreaCode() {
        return this._officeAreaCode;
      }
      get officeNumber() {
        return this._officeNumber;
      }
    }
    ```
  - **Après :**

    ```javascript
    class Person {
      get officeAreaCode() {
        return this._telephoneNumber.areaCode;
      }
      get officeNumber() {
        return this._telephoneNumber.number;
      }
    }

    class TelephoneNumber {
      get areaCode() {
        return this._areaCode;
      }
      get number() {
        return this._number;
      }
    }
    ```

- **Étapes :**
  - 1. On réfléchit d'abord aux responsabilités à séparer dans une nouvelle classe.
  - 2. On crée une nouvelle classe pour accueillir ces responsabilités.
    - Au besoin, on renomme la classe initiale pour mieux refléter ce qu'elle fera après le refactoring.
  - 3. On crée une instance de la classe enfant dans le constructeur de la classe parente.
  - 4. On utilise **[Move Field](#move-field)** sur chaque champ à déplacer, en testant à chaque fois.
  - 5. On utilise **[Move Function](#move-function)** sur chaque méthode à déplacer, en testant à chaque fois.
  - 6. On revoit l'interface de chaque classe, en changeant le nom des méthodes si besoin.
  - 7. On décide si on veut exposer l'instance de la nouvelle classe ou pas.
    - Si oui, on peut envisager d'utiliser **[Change Reference to Value](#change-reference-to-value)** sur la nouvelle classe pour la transformer en value object.
- **Théorie :**
  - Les classes ont tendance à grossir au cours du temps. Cette technique permet d'extraire une partie de la logique dans une classe distincte.
  - On se rend compte qu'il y a une classe à extraire quand on a des méthodes et des champs qui sont souvent modifiés et utilisés ensemble.

### Inline Class

- **Exemple :**

  - **Avant :**

    ```javascript
    class Person {
      get officeAreaCode() {
        return this._telephoneNumber.areaCode;
      }
      get officeNumber() {
        return this._telephoneNumber.number;
      }
    }

    class TelephoneNumber {
      get areaCode() {
        return this._areaCode;
      }
      get number() {
        return this._number;
      }
    }
    ```

  - **Après :**
    ```javascript
    class Person {
      get officeAreaCode() {
        return this._officeAreaCode;
      }
      get officeNumber() {
        return this._officeNumber;
      }
    }
    ```

- **Étapes :**
  - 1. On crée les fonctions de la classe à faire disparaître dans la classe qui accueille. Ces fonctions doivent déléguer à celles de la classe qui va disparaître.
  - 2. On change toutes les occurrences qui utilisent la classe qui va disparaître pour qu'elles utilisent la classe qui accueille, avec les fonctions qui délèguent.
    - On teste après chaque changement.
  - 3. On déplace un par un le contenu des fonctions de la classe qui va disparaître vers les fonctions qui délèguent.
    - On teste à chaque fois.
  - 4. On supprime la classe.
- **Théorie :**
  - On utilise en général cette technique pour réorganiser une classe : on la vide de ses responsabilités en extrayant des classes, puis on réintègre intelligemment dans de nouvelles classes.
  - Une autre approche peut être de fusionner une classe dans une autre, et réextraire mieux ensuite.

### Hide Delegate

- **Exemple :**

  - **Avant :**

    ```javascript
    manager = aPerson.department.manager;

    class Person {
      // …
      get department() {
        return this._department;
      }
    }
    ```

  - **Après :**

    ```javascript
    manager = aPerson.manager;

    class Person {
      // …
      get manager() {
        return this._department.manager;
      }
    }
    ```

- **Étapes :**
  - 1. Pour chaque méthode à laquelle on accède sur le délégué, on crée une méthode de délégation sur notre classe initiale.
  - 2. On ajuste les occurrences de code qui utilisaient le délégué à partir de notre classe pour qu'elles appellent les nouvelles méthodes de délégation.
    - On teste à chaque fois.
  - 3. Si possible, on supprime le getter qui renvoyait le délégué directement.
  - 4. On teste.
- **Théorie :**
  - L'idée de cette technique est d'encapsuler les objets (les délégués) qui sont accessibles à partir des champs d'une classe, comme ça quand leur interface change, seule la classe qui les expose sera impactée et pas les appelants.

### Remove Middle Man

- **Exemple :**

  - **Avant :**

    ```javascript
    manager = aPerson.manager;

    class Person {
      // …
      get manager() {
        return this._department.manager;
      }
    }
    ```

  - **Après :**

    ```javascript
    manager = aPerson.department.manager;

    class Person {
      // …
      get department() {
        return this._department;
      }
    }
    ```

- **Étapes :**
  - 1. On crée un getter pour renvoyer le délégué.
  - 2. On ajuste les occurrences de code qui utilisaient les méthodes de délégation pour qu'elles utilisent le délégué par chaînage.
    - On teste à chaque fois.
    - On supprime les méthodes de délégation une par une quand elles ne sont plus utilisées.
- **Théorie :**
  - L'ajout de délégation va dans le sens de la _Loi de Demeter_, mais Fowler aimerait qu'on l'appelle plutôt la _Suggestion Utile de Demeter_, dans la mesure où elle n'est pas à suivre à la lettre, et les délégations peuvent parfois devenir trop coûteuses.

### Substitute Algorithm

- **Exemple :**
  - **Avant :**
    ```javascript
    function foundPerson(people) {
      for (let i = 0; i < people.length; i++) {
        if (people[i] === "Don") {
          return "Don";
        }
        if (people[i] === "John") {
          return "John";
        }
        if (people[i] === "Kent") {
          return "Kent";
        }
      }
      return "";
    }
    ```
  - **Après :**
    ```javascript
    function foundPerson(people) {
      const candidates = ["Don", "John", "Kent"];
      return people.find((p) => candidates.includes(p)) || "";
    }
    ```
- **Étapes :**
  - 1. On organise le code pour avoir l'algorithme dans une même fonction.
  - 2. On crée des tests pour cette fonction, pour vérifier qu'elle ne change pas de comportement.
  - 3. On code notre algorithme alternatif.
  - 4. On joue nos vérifications statiques et on teste.
    - Si les testent ne passent pas, on débogue.
- **Théorie :**
  - Parfois il est plus simple de réécrire complètement un algorithme plutôt que de le faire par petites étapes. Cette technique dit comment le faire.

## 8 - Déplacement des fonctionnalités

### Move Function

- **Exemple :**

  - **Avant :**

    ```javascript
    class Account {
      // ...
      get overdraftCharge() {...}
    }

    class AccountType {
      // ...
    }
    ```

  - **Après :**

    ```javascript
    class Account {
      // ...
    }

    class AccountType {
      // ...
      get overdraftCharge() {...}
    }
    ```

- **Étapes :**
  - 1. On examine le contexte de notre fonction : est-ce qu'on veut aussi déplacer d'autres fonctions que notre fonction appelle ?
    - Si oui, l'auteur conseille de déplacer ces fonctions en premier parce qu'elles ont moins de dépendances.
    - Si la sous-fonction est appelée par une seule fonction, on peut l'imbriquer, les bouger ensemble, puis la ressortir.
  - 2. On vérifie si la fonction est une méthode polymorphe, auquel cas il faudra aussi déplacer les fonctions des parents.
  - 3. On copie la fonction dans son nouveau contexte, en ajustant son nom si besoin.
  - 4. On lance l'analyse statique.
  - 5. On change la fonction initiale en une fonction de délégation, pour qu'elle ne fasse qu'appeler la nouvelle fonction.
  - 6. On teste.
  - 7. On envisage de supprimer l'ancienne fonction devenue fonction de délégation, en utilisant **[Inline Function](#inline-function)**.
- **Théorie :**
  - Les fonctions sont en général dans un contexte d'encapsulation (module, classe etc.). A mesure qu'on gagne en connaissance, on réorganise les fonctions.
  - Les critères pour placer la fonction peuvent être par exemple de la mettre proche des fonctions qu'elle appelle, ou des fonctions qui l'appellent.
  - Dans le cadre d'un refactoring, on peut souvent travailler avec les fonctions dans un même contexte, et les regrouper dans d'autres contextes ensuite.
  - L'auteur est plutôt méfiant à l'égard des fonctions imbriquées parce qu'elles gardent des relations de données cachées. Il préfère les modules ES6 pour encapsuler les fonctions (l'imbrication est souvent utilisée comme étape intermédiaire dans un refactoring).

### Move Field

- **Exemple :**
  - **Avant :**
    ```javascript
    class Customer {
      get plan() {
        return this._plan;
      }
      get discountRate() {
        return this._discountRate;
      }
    }
    ```
  - **Après :**
    ```javascript
    class Customer {
      get plan() {
        return this._plan;
      }
      get discountRate() {
        return this.plan.discountRate;
      }
    }
    ```
- **Étapes :**
  - 1. On s'assure que notre champ initial est encapsulé, sinon on fait.
    - Par exemple, on auto-encapsule le champ dans sa propre classe en y accédant à travers un getter et setter privés.
  - 2. On teste.
  - 3. On crée le champ et des getter/setter dans la classe que le champ veut rejoindre.
  - 4. On lance les vérifications statiques.
  - 5. On s'assure qu'il y a une référence de la classe initiale à la classe qui accueille le nouveau champ, pour pouvoir le manipuler depuis l'ancienne classe.
    - Ça peut être la nouvelle classe dont l'instance fait partie d'un champ de l'ancienne, ou une fonction dans l'ancienne permet d'y accéder. Au pire il faudra créer un champ dans l'ancienne classe, pour y mettre une instance de la nouvelle.
  - 6. On ajuste les getter/setter de la classe initiale pour qu'ils lisent et écrivent dans le champ de la nouvelle classe.
    - Si on est dans une situation complexe où le nouveau champ va être mis à jour par plusieurs classes, on peut choisir de mettre à jour l'ancien champ et le nouveau champ en vérifiant la cohérence des deux valeurs avec **[Introduce Assertion](#introduce-assertion)**, puis en arrêtant d'écrire dans l'ancien champ quand on pense que c'est bon.
  - 7. On teste.
  - 8. On supprime le champ initial.
  - 9. On teste.
- **Théorie :**
  - Le design des structures de données est très important pour avoir un code clair.
  - Plus notre connaissance du domaine est importante, mieux on va les concevoir. Donc on va être amenés à les modifier au fur et à mesure.
  - Deux champs dans deux structures qui sont toujours lues ensemble, ou mises à jour ensemble, doivent rejoindre la même structure.
  - Ce refactoring est plus facile à faire si notre structure est en fait une classe, avec ses champs encapsulés, et qu'on peut donc ajouter de la logique aux getters/setters.
    - Si la structure n'est pas une classe, l'auteur conseille de la transformer en une classe avec Encapsulate Record avant de faire le refactoring.

### Move Statement into Function

- **Exemple :**

  - **Avant :**

    ```javascript
    result.push(`<p>title: ${person.photo.title}</p>`);
    result.concat(photoData(person.photo));

    function photoData(aPhoto) {
      return [
        `<p>location: ${aPhoto.location}</p>`,
        `<p>date: ${aPhoto.date.toDateString()}</p>`,
      ];
    }
    ```

  - **Après :**

    ```javascript
    result.concat(photoData(person.photo));

    function photoData(aPhoto) {
      return [
        `<p>title: ${aPhoto.title}</p>`,
        `<p>location: ${aPhoto.location}</p>`,
        `<p>date: ${aPhoto.date.toDateString()}</p>`,
      ];
    }
    ```

- **Étapes :**
  - 1. Si l'instruction exécutée en même temps que la fonction n'est pas à côté d'elle, on déplace l'instruction à côté avec **[Slide Statements](#slide-statements)**.
  - 2. Si on n'avait en fait qu'une seule occurrence des instructions utilisées avec l'appel à la fonction, alors on peut simplement couper l'instruction et la coller dans la fonction, en ignorant le reste des étapes.
  - 3. Si on a plusieurs occurrences, on utilise **[Extract Function](#extract-function)** sur une des occurrences, pour extraire les instructions et l'appel à la fonction.
    - On nomme la nouvelle fonction avec un nom temporaire facile à rechercher.
  - 4. On utilise la nouvelle fonction dans toutes les occurrences où on peut, en testant à chaque fois.
  - 5. On utilise **[Inline Function](#inline-function)** pour mettre le contenu de la fonction initiale dans la nouvelle fonction.
  - 6. On utilise **[Rename Function](#change-function-declaration)** pour trouver un meilleur nom à la nouvelle fonction.
- **Théorie :**
  - Quand on remarque que des instructions sont tout le temps exécutées en même temps que l'appel à une fonction, il faut se demander si elles poursuivent le même but que la fonction :
    - Si oui il faut les déplacer dans la fonction avec cette technique.
    - Si non il faut extraire les instructions et la fonction dans une nouvelle fonction qui fera quelque chose de spécifique et d'utile puisque souvent utilisée.
  - Vu les étapes, on peut parfois extraire les instructions et la fonction dans une nouvelle fonction, puis se dire qu'on veut aller plus loin et intégrer les instructions dans la fonction.

### Move Statement to Callers

- **Exemple :**

  - **Avant :**

    ```javascript
    emitPhotoData(outStream, person.photo);

    function emitPhotoData(outStream, photo) {
      outStream.write(`<p>title: ${photo.title}</p>\n`);
      outStream.write(`<p>location: ${photo.location}</p>\n`);
    }
    ```

  - **Après :**

    ```javascript
    emitPhotoData(outStream, person.photo);
    outStream.write(`<p>location: ${person.photo.location}</p>\n`);

    function emitPhotoData(outStream, photo) {
      outStream.write(`<p>title: ${photo.title}</p>\n`);
    }
    ```

- **Étapes :**
  - 1. Si on est dans un cas simple avec un ou deux endroits où notre fonction est appelée, on peut simplement couper le code à sortir de la fonction, et le coller là où on l'appelle.
  - 2. Si on est dans un cas plus complexe, on va d'abord utiliser **[Extract Function](#extract-function)** pour toutes les instructions qu'on ne va pas déplacer de notre fonction, pour les mettre dans une fonction temporaire.
  - 3. On applique ensuite **[Inline Function](#inline-function)** sur la fonction initiale pour la faire disparaître.
  - 4. On utilise enfin **[Change Function Declaration](#change-function-declaration)** pour renommer la fonction qu'on avait extraite avec le nom de la fonction qui vient de disparaître (ou un meilleur nom).
- **Théorie :**
  - On peut utiliser ce refactoring quand une fonction fait trop de choses, notamment quand on commence à vouloir la personnaliser pour répondre à des besoins particuliers où il faut éviter d'exécuter une partie de la fonction.
  - Ce refactoring marche bien si on a peu de changements à déplacer, sinon il faut d'abord tout remettre à plat avec **[Inline Function](#inline-function)**, et extraire de meilleures fonctions à partir de là.

### Replace Inline Code with Function Call

- **Exemple :**
  - **Avant :**
    ```javascript
    let appliesToMass = false;
    for (const s of states) {
      if (s === "MA") appliesToMass = true;
    }
    ```
  - **Après :**
    ```javascript
    appliesToMass = states.includes("MA");
    ```
- **Étapes :**
  - 1. On remplace du code existant par un appel à une fonction qui fait déjà la même chose.
  - 2. On teste.
- **Théorie :**
  - Ce refactoring consiste tout simplement à voir si on n'a pas déjà une fonction (maison ou dans la bibliothèque) qui fait déjà ce que fait un bout de code qu'on a, et si oui à l'appeler.
  - Dans le cas où le code est similaire au code d'une fonction, mais que cette similitude est une coïncidence, il ne faut pas faire le remplacement puisque ces deux codes ne doivent alors pas évoluer ensemble.
    - Le nom de la fonction peut nous permettre de comprendre ce qu'elle est censée faire, pour savoir si c'est bien la même chose qu'on veut faire avec notre code.

### Slide Statements

- **Exemple :**
  - **Avant :**
    ```javascript
    const pricingPlan = retrievePricingPlan();
    const order = retreiveOrder();
    let charge;
    const chargePerUnit = pricingPlan.unit;
    ```
  - **Après :**
    ```javascript
    const pricingPlan = retrievePricingPlan();
    const chargePerUnit = pricingPlan.unit;
    const order = retreiveOrder();
    let charge;
    ```
- **Étapes :**
  - 1. On examine le code pour voir si les instructions qu'on veut déplacer vont créer des interférences avec du code avant ou après. S'il y a des interférences on abandonne le refactoring.
    - On ne peut pas déplacer une variable avant un élément qu'elle référence.
    - On ne peut pas déplacer une variable après un élément qui la référence.
    - Une variable ne peut pas aller après une instruction qui modifie un élément qu'elle référence.
    - Une variable qui modifie un élément ne peut pas aller après une instruction qui référence l'élément modifié.
  - 2. On coupe les instructions et on les colle là où on veut qu'elles soient.
  - 3. On teste.
- **Théorie :**
  - On va souvent vouloir regrouper les instructions qui agissent sur la même structure.
    - L'auteur déclare en général les variables juste au-dessus de leur première utilisation (et non pas en haut de la fonction).
  - En général on regroupe les instructions ensemble pour ensuite faire un autre refactoring, par exemple **[Extract Function](#extract-function)**.
  - Si après le refactoring nos tests ne passent plus, on peut recommencer avec moins d'instructions déplacées. On peut aussi laisser de côté pour le moment pour faire d'abord d'autres refactorings.
  - Le fait qu'une valeur soit modifiée par l'instruction qu'on déplace et lue par l'instruction par dessus laquelle on déplace (ou l'inverse) n'est en fait pas forcément éliminatoire.
    - Il peut y avoir des cas où les modifications sont interchangeables, mais il faut être très prudent. Appliquer **[Split Variable](#split-variable)** peut parfois clarifier la situation.
  - L'auteur adhère au principe de **séparation de commandes et queries**, et donc ses fonctions vont soit retourner une valeur, soit avoir un side effect, mais pas les deux.

### Split Loop

- **Exemple :**
  - **Avant :**
    ```javascript
    let averageAge = 0;
    let totalSalary = 0;
    for (const p of people) {
      averageAge += p.age;
      totalSalary += p.salary;
    }
    averageAge = averageAge / people.length;
    ```
  - **Après :**
    ```javascript
    let totalSalary = 0;
    for (const p of people) {
      totalSalary += p.salary;
    }
    let averageAge = 0;
    for (const p of people) {
      averageAge += p.age;
    }
    averageAge = averageAge / people.length;
    ```
- **Étapes :**
  - 1. On copie la boucle en dessous de l'autre.
  - 2. On supprime ce qu'il faut dans chacune deux boucles, et notamment les side-effects dupliqués auxquels il faut faire attention.
  - 3. On teste.
  - 4. On envisage **[Extract Function](#extract-function)** pour mettre la nouvelle boucle dans une fonction.
- **Théorie :**
  - On utilise souvent une seule boucle pour faire plusieurs choses parce qu'on a peur des performances.
    - Pourtant séparer la boucle en plusieurs boucles permet d'avoir du code plus compréhensible, et c'est avec du code compréhensible qu'on pourra au mieux optimiser si vraiment on a besoin.
  - En ayant des boucles qui font une seule chose, on arrive aussi à obtenir des fonctions qui exécutent la boucle et renvoient une seule valeur.
    - En général l'auteur fractionne une boucle avant d'extraire la fraction en boucle.

### Replace Loop with Pipeline

- **Exemple :**
  - **Avant :**
    ```javascript
    const names = [];
    for (const i of input) {
      if (i.job === "programmer") names.push(i.name);
    }
    ```
  - **Après :**
    ```javascript
    const names = input
      .filter((i) => i.job === "programmer")
      .map((i) => i.name);
    ```
- **Étapes :**
  - 1. On crée une nouvelle variable à laquelle on assigne la collection sur laquelle on boucle.
  - 2. On prend chaque groupe d'instructions cohérentes de la boucle en commençant par le début, et on en crée une opération de pipeline.
    - On teste à chaque fois.
  - 3. Une fois que toutes les instructions de la boucle sont supprimées, on supprime la boucle.
- **Théorie :**
  - Les pipelines permettent de suivre les opérations de manière linéaire, avec les entrées et sorties claires. L'auteur les trouve plus claires que les boucles.
  - Parmi les opérations de pipeline on a par exemple les classiques _map_, _filter_, _reduce_.
  - Pour plus d'exemples, Fowler propose son essai **_Refactoring with Loops and Collection Pipelines_**.

### Remove Dead Code

- **Exemple :**

  - **Avant :**
    ```javascript
    if (false) {
      doSomethingThatUsedToMatter();
    }
    ```
  - **Après :**

    ```javascript

    ```

- **Étapes :**
  - 1. Si c'est du code appelable, on recherche les références pour bien s'assurer que le code est mort.
  - 2. On supprime le code.
  - 3. On teste.
- **Théorie :**
  - Le code mort alourdit le code pour rien. Il faut le supprimer pour des raisons de maintenabilité.
  - Le gestionnaire de version retient le code de toute façon. Et au pire, si on veut _vraiment_ garder une trace visible, on peut mettre une trace sous forme de commentaire disant dans quel commit se trouve le bout de code qu'on a supprimé.

## 9 - Organisation des données

### Split Variable

- **Exemple :**
  - **Avant :**
    ```javascript
    let temp = 2 * (height + width);
    console.log(temp);
    temp = height * width;
    console.log(temp);
    ```
  - **Après :**
    ```javascript
    const perimeter = 2 * (height + width);
    console.log(perimeter);
    const area = height * width;
    console.log(area);
    ```
- **Étapes :**
  - 1. On renomme la variable par ce qui exprime le mieux sa première affectation.
    - Si les affectations suivantes sont sous la forme `variable += variable + valeur`, on est sans doute face à une variable de collecte, il faut donc abandonner le refactoring.
  - 3. Si possible on déclare la variable comme immutable (const).
  - 4. On change toutes les références à la variable jusqu'à sa prochaine affectation.
  - 5. On teste.
  - 6. On répète l'ensemble des étapes à chaque affectation, jusqu'à la dernière, en testant à chaque fois.
    - A la fin on a bien des variables différentes pour chaque affectation.
- **Théorie :**
  - Ce refactoring permet d'éliminer les **variables qui sont assignées plusieurs fois** et qui ont **plusieurs responsabilités**, pour en faire une variable par responsabilité.
    - Les variables qui comptant les tours de boucle, font des sommes ou des concaténations de chaîne sont réassignées de nombreuses fois, mais elles ont bien une seule responsabilité. Ce sont des **variables de collecte**.

### Rename Field

- **Exemple :**
  - **Avant :**
    ```javascript
    class Organization {
      get name() {...}
    }
    ```
  - **Après :**
    ```javascript
    class Organization {
      get title() {...}
    }
    ```
- **Étapes :**
  - 1. Dans le cas où la structure a une petite portée, il suffit de renommer les références au champ qu'on renomme directement. Pas besoin d'aller plus loin dans les étapes.
  - 2. Dans le cas contraire, si on est face à une structure non encapsulée dans une classe, on appliqué **[Encapsulate Record](#encapsulate-record)** pour l'encapsuler.
  - 3. On renomme le champ privé dans la classe, et on met à jour le getter et le setter pour que ça fonctionne.
  - 4. On teste.
  - 5. On applique **[Change Function Declaration](#change-function-declaration)** pour modifier le nom du getter et du setter.
- **Théorie :**
  - L'intérêt de ce refactoring est de faire en sorte que les structures de données restent en cohérence avec la connaissance nouvelle qu'on gagne à mesure qu'on travaille sur le projet.
  - On parle ici des structures de données simples mais aussi des classes.
  - Quand on encapsule une structure de données dans une classe d'abord, on doit ensuite renommer le getter, le setter, le constructeur et la variable privée. Mais on se facilite en fait la vie parce qu'on traite chaque aspect indépendamment au lieu de tout faire d'un coup.
- **Exemple détaillé :**
  - On a initialement une structure. On veut renommer _name_ en _title_.
    ```javascript
    const organization = {
      name: "Acme Gooseberries",
    };
    ```
  - On va l'encapsuler dans une classe.
    ```javascript
    class Organization {
      constructor(data) {
        this._title = data.name;
      }
      get name() {
        return this._title;
      }
      set name(aString) {
        this._title = aString;
      }
    }
    ```
  - On permet ensuite au constructeur d'accepter le nouveau nom du paramètre, pour pouvoir changer les références une par une en testant.
    ```javascript
    class Organization {
      constructor(data) {
        this._title = data.title !== undefined ? data.title : data.name;
      }
      // ...
    }
    ```
  - Une fois qu'on a changé toutes les références, on peut définitivement renommer name en title.
    ```javascript
    class Organization {
      constructor(data) {
        this._title = data.title;
      }
      get title() {
        return this._title;
      }
      set title(aString) {
        this._title = aString;
      }
    }
    ```

### Replace Derived Variable with Query

- **Exemple :**
  - **Avant :**
    ```javascript
    get discountedTotal() {
      return this._discountedTotal;
    }
    set discount(aNumber) {
      const old = this._discount;
      this._discount = aNumber;
      this._discountedTotal += old - aNumber;
    }
    ```
  - **Après :**
    ```javascript
    get discountedTotal() {
      return this._baseTotal - this._discount;
    }
    set discount(aNumber) {
      this._discount = aNumber;
    }
    ```
- **Étapes :**
  - 1. On identifie la variable qu'on veut remplacer par un calcul, et on liste les endroits où elle est mise à jour.
    - SI besoin on peut utiliser **[Split Variable](#split-variable)** pour la séparer en plusieurs variables avec une responsabilité chacune.
  - 2. On crée une fonction qui calcule la valeur de la variable.
  - 3. On utilise **[Introduce Assertion](#introduce-assertion)** pour vérifier que la variable et la fonction fournissent la même valeur.
    - Si besoin, on peut utiliser **[Encapsulate Variable](#encapsulate-variable)** pour avoir un endroit où mettre l'assertion.
  - 4. On teste.
  - 5. On remplace les accès à la variable par un appel à la fonction.
  - 6. On teste.
  - 7. On utilise **[Remove Dead Code](#remove-dead-code)** pour éliminer la variable et le code qui la met à jour.
- **Théorie :**
  - Les variables mutables sont une source de problèmes. Ce refactoring permet de les limiter, en remplaçant certaines variables par des calculs qui permettent de les réobtenir.
  - Dans le cas où les données à partir desquels la variable est calculée sont immutables, c'est OK de la laisser et de la rendre immutable aussi.
    - Et si la donnée dérivée est temporaire, ça peut être OK aussi de la laisser dans une variable.
- **Exemple détaillé :**
  - On a une classe avec une duplication de structure de données : _production_ est calculable depuis _adjustment_, mais on stocke les deux valeurs en tant que variable membre.
    ```javascript
    class ProductionPlan {
      // ...
      get production() {
        return this._production;
      }
      applyAdjustment(anAdjustment) {
        this._adjustments.push(anAdjustment);
        this._production += anAdjustment.amount;
      }
    }
    ```
  - On crée une fonction qui calcule la valeur de production, et on vérifie avec un assert qu'elle renvoie la même valeur que production.
    ```javascript
    class ProductionPlan {
      // ...
      get production() {
        assert(this._production === this.calculatedProduction);
        return this._production;
      }
      get calculatedProduction() {
        return this._adjustments.reduce((sum, a) => sum + a.amount, 0);
      }
    }
    ```
  - Si ça marche bien, on peut enlever l'assert, puis placer le contenu de la fonction calculatedProduction dans le getter de production.
    ```javascript
    class ProductionPlan {
      // ...
      get production() {
        return this._adjustments.reduce((sum, a) => sum + a.amount, 0);
      }
    }
    ```
  - Et enfin on peut éliminer le code mort représenté par la variable privée `this._production`.

### Change Reference to Value

- **Exemple :**
  - **Avant :**
    ```javascript
    class Product {
      applyDiscount(arg) {
        this._price.amount -= arg;
      }
    }
    ```
  - **Après :**
    ```javascript
    class Product {
      applyDiscount(arg) {
        this._price = new Money(this._price.amount - arg, this._price.currency);
      }
    }
    ```
- **Étapes :**
  - 1. On vérifie que l'objet qu'on veut transformer en valeur est déjà, ou peut devenir immutable.
  - 2. On va effectivement le rendre immutable : pour chaque setter qu'on appelle pour modifier l'objet, on applique **[Remove Setting Method](#remove-setting-method)**. De cette manière l'objet ne pourra plus être changé autrement qu'à sa construction.
  - 3. On lui ajoute une méthode de comparaison d'égalité, basée sur la valeur des propriétés de notre objet.
- **Théorie :**
  - Une instance d'objet qui est une propriété d'un autre objet peut être traitée soit comme une référence, soit comme une valeur.
    - Si elle est une référence, on garde la même et on modifie des choses dessus si besoin.
    - Si elle est une valeur, on la recrée avec les bonnes propriétés à chaque fois qu'elle est modifiée. Elle est alors un **value object**.
  - L'avantage à manipuler des value objects est que ce sont des valeurs immutables, et donc on n'a pas à s'inquiéter qu'elles soient modifiées sans qu'on le sache (sans passer par notre setter).
    - Les value objects sont particulièrement utiles pour les systèmes distribués et concurrents.
  - La plupart des langages permettent de surcharger l'opérateur `==`, et en général on doit aussi surcharger l'opérateur de hachage pour que la valeur puisse servir de clé dans une hashmap.
    - Dans le cas où on ne peut pas surcharger l'opérateur d'égalité, on peut toujours créer une méthode `equals()`.

### Change Value to Reference

- **Exemple :**
  - **Avant :**
    ```javascript
    let customer = new Customer(customerData);
    ```
  - **Après :**
    ```javascript
    let customer = customerRepository.get(customerData.id);
    ```
- **Étapes :**
  - 1. On crée un repository pour les instances de l'objet concerné (si ça n'existe pas déjà).
  - 2. On s'assure que le constructeur des objets qui instancient notre objet concerné a la possibilité de rechercher les bonnes instances de l'objet concerné.
  - 3. On change le constructeur des objets qui instancient notre objet concerné, pour qu'ils utilisent le repository pour obtenir la bonne instance au lieu de construire l'objet directement.
- **Théorie :**
  - Quand il faut mettre à jour des données partagées, si elles sont immutables, il faut toutes les trouver et les changer. Alors que quand on a une référence, on peut la changer une fois pour la voir changée partout.
- **Exemple détaillé :**

  - On a une classe _Order_ qui a une instance d'un objet _Customer_ qu'on veut transformer en référence pour que les customers qui ont le même ID soient partagés :

    ```javascript
    class Order {
      constructor(data) {
        this._number = data.number;
        this._customer = new Customer(data.customer);
      }
    }

    class Customer {
      constructor(id) {
        this._id = id;
      }
    }
    ```

  - On va créer un repository qui nous permet de créer un customer d'il n'existe pas déjà, ou d'obtenir le customer existant qui a le même ID.

    ```javascript
    let _repositoryData;

    export function initialize() {
      _repositoryData = {};
      _repositoryData.customers = new Map();
    }

    export function registerCustomer(id) {
      if (!_repositoryData.customers.has(id))
        _repositoryData.customers.set(id, new Customer(id));
      return findCustomer(id);
    }

    export function findCustomer(id) {
      return _repositoryData.customers.get(id);
    }
    ```

  - On peut maintenant utiliser le repository dans _Order_ pour obtenir la bonne instance de _Customer_.
    ```javascript
    class Order {
      constructor(data) {
        this._number = data.number;
        this._customer = registerCustomer(data.customer);
      }
    }
    ```

## 10 - Simplification de la logique conditionnelle

### Decompose Conditional

- **Exemple :**
  - **Avant :**
    ```javascript
    if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
      charge = quantity * plan.summerRate;
    else charge = quantity * plan.regularRate + plan.regularServiceCharge;
    ```
  - **Après :**
    ```javascript
    if (summer()) charge = summerCharge();
    else charge = regularCharge();
    ```
- **Étapes :**
  - 1. On applique **[Extract Function](#extract-function)** sur la condition, et on l'applique sur le contenu de chaque branche de manière à obtenir un code tout petit.
- **Théorie :**
  - Le but est de rendre lisible les conditions.

### Consolidate Conditional Expression

- **Exemple :**

  - **Avant :**
    ```javascript
    if (anEmployee.seniority < 2) return 0;
    if (anEmployee.monthsDisabled > 12) return 0;
    if (anEmployee.isPartTime) return 0;
    ```
  - **Après :**

    ```javascript
    if (isNotEligableForDisability()) return 0;

    function isNotEligableForDisability() {
      return (
        anEmployee.seniority < 2 ||
        anEmployee.monthsDisabled > 12 ||
        anEmployee.isPartTime
      );
    }
    ```

- **Étapes :**
  - 1. On s'assure qu'aucune des conditions (contenu des parenthèses du if) n'a de side effects.
    - Si il y en a, on les sépare avec **[Separate Query from Modifier](#separate-query-from-modifier)**.
  - 2. On prend deux conditions et on les combine dans une nouvelle condition avec des opérateurs logiques.
    - Des suites de conditions se combinent avec un OR, des conditions imbriquées se combinent avec un AND.
  - 3. On répète jusqu'à ce qu'il ne reste qu'une condition, en testant à chaque fois.
  - 4. On envisage **[Extract Function](#extract-function)** sur la condition résultante.
- **Théorie :**
  - Il s'agit ici du cas où les **conditions sont différentes mais le contenu des branches est le même**.
  - Regrouper les conditions permet d'indiquer l'intention : le fait que ces conditions permettent en fait de protéger la même branche de code.
    - Si le contenu des branches est le même par hasard, alors on ne fait pas ce refactoring parce qu'on pourrait induire en erreur en indiquant une mauvaise intention.

### Replace Nested Conditional with Guard Clauses

- **Exemple :**
  - **Avant :**
    ```javascript
    function getPayAmount() {
      let result;
      if (isDead) result = deadAmount();
      else {
        if (isSeparated) result = separatedAmount();
        else {
          if (isRetired) result = retiredAmount();
          else result = normalPayAmount();
        }
      }
      return result;
    }
    ```
  - **Après :**
    ```javascript
    function getPayAmount() {
      if (isDead) return deadAmount();
      if (isSeparated) return separatedAmount();
      if (isRetired) return retiredAmount();
      return normalPayAmount();
    }
    ```
- **Étapes :**
  - 1. On prend la branche la plus externe de la condition et on la transforme en clause de garde.
  - 2. On teste.
  - 3. On répète si nécessaire.
- **Théorie :**
  - L'idée est de laisser le if/else dans le cas où **les deux conditions font partie du flow normal de la fonction**, et de mettre la branche if comme clause de garde si **le else serait tout le reste de la fonction**.
    - Le but principal est de mettre en avant la nature de la condition.
  - Ce refactoring est souvent utilisé avec une inversion de la condition :
    ```javascript
    function adjustedCapital(anInstrument) {
      let result = 0;
      if (anInstrument.capital > 0) {
        if (anInstrument.interestRate > 0 && anInstrument.duration > 0) {
          result =
            (anInstrument.income / anInstrument.duration) *
            anInstrument.adjustmentFactor;
        }
      }
      return result;
    }
    ```
  - Donne :
    ```javascript
    function adjustedCapital(anInstrument) {
      if (
        anInstrument.capital <= 0 ||
        anInstrument.interestRate <= 0 ||
        anInstrument.duration <= 0
      )
        return 0;
      return (
        (anInstrument.income / anInstrument.duration) *
        anInstrument.adjustmentFactor
      );
    }
    ```

### Replace Conditional with Polymorphism

- **Exemple :**

  - **Avant :**
    ```javascript
    switch (bird.type) {
      case 'EuropeanSwallow':
        return "average";
      case 'AfricanSwallow':
        return (bird.numberOfCoconuts > 2) ? "tired" : "average";
      case 'NorwegianBlueParrot':
        return (bird.voltage > 100) ? "scorched" : "beautiful";
      default:
        return "unknown";
    ```
  - **Après :**

    ```javascript
    class EuropeanSwallow {
      get plumage() {
        return "average";
      }
    }

    class AfricanSwallow {
      get plumage() {
        return this.numberOfCoconuts > 2 ? "tired" : "average";
      }
    }

    class NorwegianBlueParrot {
      get plumage() {
        return this.voltage > 100 ? "scorched" : "beautiful";
      }
    }
    ```

- **Étapes :**
  - 1. On crée des classes pour le comportement conditionnel, et on crée une fonction _factory_ qui permet de renvoyer une instance de la bonne classe.
  - 2. Si la logique conditionnelle n'est pas déjà dans une fonction, on utilise **[Extract Function](#extract-function)** pour qu'elle le soit.
  - 3. On déplace la fonction avec la logique conditionnelle dans la classe mère de la hiérarchie de classes qu'on a créée à l'étape 1.
    - Ca peut être plusieurs fonctions, si on a plusieurs logiques conditionnelles similaires (comme par exemple plusieurs switchs similaires).
  - 4. On crée une méthode qui surcharge la méthode avec la logique conditionnelle dans une des classes filles, et on y déplace le corps de la bonne branche de l'instruction conditionnelle.
  - 5. On répète pour chaque branche conditionnelle et classe fille.
  - 6. On laisse un cas par défaut dans la classe mère, ou alors si elle est abstraite on rend la méthode abstraite (ou on fait en sorte qu'elle lève une erreur).
- **Théorie :**
  - A partir du moment où on a **plusieurs fois le même switch** pour faire des choses, ça devient plus avantageux d'utiliser une hiérarchie de classes pour remplacer ces switch.
  - Une situation aussi où il est pertinent d'utiliser le polymorphisme c'est quand on a **un cas de base, et des variantes secondaires** qui vont venir apporter des changements au cas de base.
    - La classe mère pourra avoir une grande partie de la logique non redéfinie dans les classes filles, et seules certaines méthodes surchargées vont venir apporter des changements secondaires dans les classes filles.

### Introduce Special Case

- **Exemple :**

  - **Avant :**
    ```javascript
    if (aCustomer === "unknown") {
      customerName = "occupant";
    }
    ```
  - **Après :**

    ```javascript
    customerName = aCustomer.name;

    class UnknownCustomer {
      get name() {
        return "occupant";
      }
    }
    ```

- **Étapes :**
  - **Cette description est cryptique sans regarder l'exemple détaillé plus bas**.
  - On démarre avec un conteneur (classe ou structure) qui a une propriété (qu'on va appeler _sujet_) que le code appelant compare avec une valeur, et dont on aimerait que la valeur soit dans une classe special case.
  - 1. On ajoute une propriété vérifiant le fait d'être le spatial case ou pas (et retournant _false_) au _sujet_.
  - 2. On crée la classe special case avec la même propriété vérifiant le fait d'être spatial case ou pas, mais celle-là retourne _true_.
  - 3. On applique **[Extract Function](#extract-function)** pour extraire la condition (contenu des parenthèses du _if_) des occurrences de code appelant, dans une fonction qui vérifiera si notre objet est un special cas ou non.
  - 4. On introduit les instances de special case depuis le conteneur initial, quand le sujet est un special case.
  - 5. On met à jour la fonction qui vérifie si l'objet est special case ou pas, pour qu'elle prenne enfin en compte la classe special case qu'on a créée.
  - 6. On teste.
  - 7. On utilise **[Combine Function into Class](#combine-functions-into-class)** ou **[Combien Function into Transform](#combine-functions-into-transform)** pour déplacer toutes les valeurs et comportements liés aux conditions de special case dans l'objet special case.
  - 8. On utilise **[Inline Function](#inline-function)** sur la fonction qui vérifie si l'objet est special case, pour les occurrences de code appelant où on en a besoin parce qu'on ne peut pas simplement utiliser l'objet special case (parce qu'on est sur du cas particulier de cas particulier).
    - S'il n'y a pas de tels endroits, on peut supprimer la fonction qui vérifie si l'objet est special case.
- **Théorie :**
  - Quand on a des conditions dupliquées qui donnent lieu aux mêmes instructions, ça peut être utile de créer un objet special case pour les regrouper au même endroit.
  - L'objet special case peut retourner des valeurs spécifiques, ou contenir des méthodes avec de la logique à mettre en commun entre les conditions où il est utilisé.
  - `null` est un exemple d'objet de special case.
  - L'objet de special case est un value object.
- **Exemple détaillé :**

  - On a un _Site_ avec un une propriété _Customer_.

    ```javascript
    class Site {
      get customer() {
        return this._customer;
      }
    }

    class Customer {
      get name() {
        // ...
      }
    }
    ```

  - La plupart du temps un _Site_ a un _Customer_, mais parfois il n'en a pas, et dans ce cas la propriété customer vaut _“unknown”_.
    ```javascript
    // exemple code appelant
    const aCustomer = site.customer;
    let customerName;
    if (aCustomer === "unknown") {
      customerName = "occupant";
    }
    ```
  - Vu qu'on a de nombreux cas de code appelant qui traite le cas particulier de customer non existant, et que la plupart du temps le nom du customer va être _“occupant”_, on va mettre ça dans un objet special case.
  - On commence par ajouter une méthode à _Customer_ pour indiquer qu'il n'est pas _unknown_.
    ```javascript
    class Customer {
      get isUnknown() {
        return false;
      }
    }
    ```
  - Puis on crée notre objet special case qui lui va indiquer avec la même méthode qu'il est _unknown_.
    ```javascript
    class UnknownCustomer {
      get isUnknown() {
        return true;
      }
    }
    ```
  - Vu qu'on a beaucoup d'occurrences de code appelant qui traite le cas _unknown_, on a envie de pouvoir les changer petit à petit en obtenant à chaque fois un code qui marche et fait passer les tests. Donc on ajoute une fonction isUnknown qui sera utilisée par le code appelant.

    ```javascript
    function isUnknown(arg) {
      if (!(arg instanceof Customer || arg === "unknown")) {
        throw new Error(`Investigate bad value &lt;${arg}>`);
      }
      return arg === "unknown";
    }

    // exemple code appelant
    const aCustomer = site.customer;
    let customerName;
    if (isUnknown(aCustomer)) {
      customerName = "occupant";
    }
    ```

  - Une fois qu'on a utilisé _isUnknown_ dans tout le code appelant, on peut modifier _Site_ pour renvoyer l'objet special case au lieu de la chaîne _unknown_ dans le getter de _customer_.
    ```javascript
    class Site {
      get customer() {
        return this._customer === "unknown"
          ? new UnknownCustomer()
          : this._customer;
      }
    }
    ```
  - On change aussi isUnknown pour prendre ça en compte et on teste.
    ```javascript
    function isUnknown(arg) {
      if (!(arg instanceof Customer || arg instanceof UnknownCustomer)) {
        throw new Error(`Investigate bad value &lt;${arg}>`);
      }
      return arg === "unknown";
    }
    ```
  - On ajoute une méthode au special case pour pouvoir éliminer la condition du code appelant.

    ```javascript
    class UnknownCustomer {
      get name() {
        return "occupant";
      }
    }

    // exemple code appelant
    const aCustomer = site.customer;
    const customerName = aCustomer.name;
    ```

  - Finalement, une fois qu'on a remplacé partout, on supprime _isUnknown_ que plus personne n'utilise. Elle nous a servi seulement pour le refactoring.

### Introduce Assertion

- **Exemple :**
  - **Avant :**
    ```javascript
    if (this.discountRate) base = base - this.discountRate * base;
    ```
  - **Après :**
    ```javascript
    assert(this.discountRate >= 0);
    if (this.discountRate) base = base - this.discountRate * base;
    ```
- **Étapes :**
  - 1. Quand on voit dans le code qu'une condition doit toujours être vraie, on ajoute une assertion pour l'indiquer.
    - Les assertions ne doivent pas modifier le comportement du système, elles l'arrêtent juste dans des cas qui ne doivent de toute façon pas se produire.
- **Théorie :**
  - Quand on a des conditions qui ne sont jamais censées se produire, on peut placer une assertion qui arrête le programme au cas où elle n'est pas vérifiée.
  - C'est utile pour trouver des bugs, mais aussi pour communiquer l'information sur l'état que le code n'est pas censé avoir au lecteur du code.
  - Attention à ne pas en abuser, il faut vérifier ce qui doit être vrai, pas tout ce qu'on pense être vrai.
    - Par exemple, si on lit des données externes qu'on doit parser, il ne faut pas utiliser des assertions sur elles parce qu'il y a des chances pour qu'elles ne soient pas dans le bon format sans que ce soit une erreur dans notre programme.

## 11 - Refactoring des APIs

### Separate Query from Modifier

- **Exemple :**

  - **Avant :**
    ```javascript
    function getTotalOutstandingAndSendBill() {
      const result = customer.invoices.reduce(
        (total, each) => each.amount + total,
        0
      );
      sendBill();
      return result;
    }
    ```
  - **Après :**

    ```javascript
    function totalOutstanding() {
      return customer.invoices.reduce((total, each) => each.amount + total, 0);
    }

    function sendBill() {
      emailGateway.send(formatBill(customer));
    }
    ```

- **Étapes :**
  - 1. On copie la fonction et on la renomme en tant que query.
    - Pour le choix du nom, on peut voir où la fonction est utilisée, et ce qui est fait de sa valeur de retour (par exemple le nom de la variable dans laquelle on la met).
  - 2. On supprime les side effects de la nouvelle fonction.
  - 3. On lance la vérification statique du code.
  - 4. Pour chaque appel à la fonction initiale, si l'appel utilise la valeur de retour, on le remplace par un appel à la nouvelle fonction.
    - On teste à chaque fois.
  - 5. On supprime la valeur de retour de la fonction initiale.
  - 6. On teste.
- **Théorie :**
  - Fowler essaye de séparer les fonctions qui ont des side effects mais ne renvoient pas de résultat (modifier) des fonctions qui n'en ont pas et qui renvoient un résultat (query).
    - C'est pas une règle absolue, mais ça permet d'être plus serein sur le code des queries.
  - S'il y a beaucoup de duplication entre la fonction query et la fonction modifier, on peut voir s'il y a moyen d'utiliser **[Substitute Algorithm](#substitute-algorithm)** pour utiliser la fonction query dans la fonction modifier, et la raccourcir.

### Parameterize Function

- **Exemple :**

  - **Avant :**

    ```javascript
    function tenPercentRaise(aPerson) {
      aPerson.salary = aPerson.salary.multiply(1.1);
    }

    function fivePercentRaise(aPerson) {
      aPerson.salary = aPerson.salary.multiply(1.05);
    }
    ```

  - **Après :**
    ```javascript
    function raise(aPerson, factor) {
      aPerson.salary = aPerson.salary.multiply(1 + factor);
    }
    ```

- **Étapes :**
  - 1. On choisit une des fonctions qui doit être paramétrisée.
  - 2. On utilise **[Change Function Declaration](#change-function-declaration)** pour ajouter des paramètres pour les valeurs en dur à paramétriser.
  - 3. On ajoute les nouveaux paramètres pour chaque code qui appelle la fonction qu'on vient de modifier.
  - 4. On teste.
  - 5. On change le corps de la fonction pour utiliser les nouveaux paramètres.
    - On teste à chaque changement dans la fonction.
  - 6. On remplace les fonctions similaires avec la nouvelle fonction, en ajustant si besoin.
    - On teste à chaque fois.
- **Théorie :**
  - Il s'agit de regrouper plusieurs fonctions qui font presque la même chose avec une différence de valeur en dur, dans une fonction similaire avec la valeur donnée en paramètre.

### Remove Flag Argument

- **Exemple :**

  - **Avant :**
    ```javascript
    function setDimension(name, value) {
      if (name === "height") {
        this._height = value;
        return;
      }
      if (name === "width") {
        this._width = value;
        return;
      }
    }
    ```
  - **Après :**

    ```javascript
    function setHeight(value) {
      this._height = value;
    }

    function setWidth(value) {
      this._width = value;
    }
    ```

- **Étapes :**
  - 1. On crée une fonction explicite pour chaque valeur possible du flag argument.
    - On peut utiliser **[Decompose Conditional](#decompose-conditional)** pour faciliter la séparation de la logique conditionnelle.
  - 2. On remplace chaque fonction qui utilisait l'ancienne fonction avec un flag argument par un appel à une des nouvelles fonctions.
- **Théorie :**
  - On a un flag argument dans une fonction quand un des paramètres permet de choisir la logique qu'on déclenche dans la fonction, par exemple avec un booléen, ou avec une enum.
  - L'auteur n'aime pas les flag arguments (et encore plus les booléens) parce qu'ils rendent moins clair ce que font les fonctions.
  - Les flag arguments peuvent être acceptables s'il y en a plusieurs dans la fonction, et que créer des fonctions avec chaque combinaison en ferait trop.
    - Mais dans ce cas, il faut essayer d'appliquer un autre refactoring parce que notre fonction est probablement trop complexe.

### Preserve Whole Object

- **Exemple :**
  - **Avant :**
    ```javascript
    const low = aRoom.daysTempRange.low;
    const high = aRoom.daysTempRange.high;
    if (aPlan.withinRange(low, high))
    ```
  - **Après :**
    ```javascript
    if (aPlan.withinRange(aRoom.daysTempRange))
    ```
- **Étapes :**
  - 1. On crée une fonction avec un nom bidon, et qui prend les paramètres tels qu'on les veut, avec l'objet au lieu de ses valeurs.
  - 2. On remplit le corps de la nouvelle fonction avec un appel à l'ancienne, en faisant un mapping des paramètres.
  - 3. On lance les vérifications statiques.
  - 4. On change le code appelant un par un pour qu'il utilise la nouvelle fonction au lieu de l'ancienne.
    - On peut au passage utiliser **[Remove Dead Code](#remove-dead-code)** pour supprimer le code en trop qui devrait apparaître pendant le remplacement.
    - On teste à chaque fois.
  - 5. Quand tous les remplacements sont faits, on utilise **[Inline Function](#inline-function)** sur la fonction initiale pour que son contenu se retrouve dans la nouvelle et qu'elle disparaisse.
  - 6. On change le nom de la nouvelle fonction.
- **Théorie :**
  - Quand on a plusieurs valeurs issues d'un même objet ou structure, qui sont données en argument à une fonction, il faut penser à donner plutôt l'objet entier.
  - Si on doit extraire des valeurs d'un objet pour en faire quelque chose, on peut aussi être face à un cas de feature envy où il vaut mieux utiliser **[Extract Class](#extract-class)** pour extraire les valeurs qui veulent sortir de la classe avec la logique associée utilisée à chaque fois dans la fonction qui prend ces valeurs.
  - Si une classe passe plusieurs de ses variables membre à une autre classe ou fonction, on peut passer _this_ à la place.
  - Quand la fonction et l'objet se trouvent dans deux modules différents par contre, et qu'on veut garder de l'encapsulation entre ceux-ci, on peut ne pas vouloir appliquer ce refactoring.

### Replace Parameter with Query

- **Exemple :**

  - **Avant :**

    ```javascript
    availableVacation(anEmployee, anEmployee.grade);

    function availableVacation(anEmployee, grade) {
      // ...
    }
    ```

  - **Après :**

    ```javascript
    availableVacation(anEmployee);

    function availableVacation(anEmployee) {
      const grade = anEmployee.grade;
      // ...
    }
    ```

- **Étapes :**
  - 1. Si besoin, on utilise **[Extract Function](#extract-function)** pour extraire la logique qui permet de calculer la valeur passée en paramètre qu'on veut enlever.
  - 2. On remplace l'utilisation de chaque paramètre à enlever de la fonction par un appel qui permet d'obtenir la même valeur depuis le corps de la fonction.
    - On teste à chaque fois.
  - 3. On utilise **[Change Function Declaration](#change-function-declaration)** pour supprimer le paramètre qui n'est plus utilisé.
- **Théorie :**
  - La liste des paramètres d'une fonction devrait montrer les diverses manières dont on peut la faire varier.
    - Il vaut mieux éviter les duplications dans les paramètres, et un paramètre que la fonction peut **facilement obtenir dans son corps** est une forme de duplication.
    - Dans le cas où elle ne peut pas facilement l'obtenir, il vaut peut être mieux ne pas faire le refactoring. Par exemple, s' il vaut mieux qu'elle ne sache pas la manière dont on l'obtient pour des raisons d'encapsulation.
  - Typiquement, si on peut **obtenir un paramètre à partir d'un autre**, on est quasi sûr qu'il vaut mieux n'en passer qu'un des deux.

### Replace Query with Parameter

- **Exemple :**

  - **Avant :**

    ```javascript
    targetTemperature(aPlan);

    function targetTemperature(aPlan) {
      currentTemperature = thermostat.currentTemperature;
      // ...
    }
    ```

  - **Après :**

    ```javascript
    targetTemperature(aPlan, thermostat.currentTemperature);

    function targetTemperature(aPlan, currentTemperature) {
      // ...
    }
    ```

- **Étapes :**
  - 1. On utilise **[Extract Variable](#extract-variable)** sur le code qui fait l'appel à la référence extérieure, de manière à ce que cet appel soit isolé du reste du corps de la fonction.
  - 2. On applique **[Extract Function](#extract-function)** avec une fonction ayant un nom temporaire, pour extraire la partie du corps de notre fonction qui _ne fait pas_ l'appel à la référence extérieure.
  - 3. On utilise **[Inline Variable](#inline-variable)** pour se débarrasser de la variable qu'on avait créée à l'étape 1.
  - 4. On utilise **[Inline Function](#inline-function)** pour fondre la fonction initiale dans la nouvelle fonction extraite à l'étape 2.
  - 5. On change le nom de la fonction qu'on a créée à l'étape 2 pour lui donner le nom de la fonction initiale qui vient de disparaître à l'étape 4.
- **Théorie :**
  - On a en fait une opposition entre interface de fonction simple (avec peu de paramètres), et faible couplage entre le corps de la fonction et d'autres fonctions.
    - **La décision d'avoir une query ou un parameter n'est parfois pas évidente**, et il faut tester pour voir ce que ça donne.
  - On pourra appliquer ce refactoring par exemple pour éviter la dépendance à une variable globale, ou un élément qu'on veut déplacer.
  - Une autre possibilité c'est dans le cas où la fonction n'a pas de _referential transparency_, c'est à dire qu'elle ne donne pas le même résultat à chaque appel : on pourra vouloir créer des fonctions pures d'un côté, et des fonctions avec side effect passant des paramètres de l'autre.

### Remove Setting Method

- **Exemple :**
  - **Avant :**
    ```javascript
    class Person {
      get name() {...}
      set name(aString) {...}
    }
    ```
  - **Après :**
    ```javascript
    class Person {
      get name() {...}
    }
    ```
- **Étapes :**
  - 1. Si la valeur qu'on set n'est pas donnée au constructeur, on l'ajoute au constructeur avec **[Change Function Declaration](#change-function-declaration)**, et on appelle le setter depuis le constructeur avec la valeur qu'on reçoit.
  - 2. On remplace l'utilisation extérieure du setter par le passage de la valeur au constructeur un par un.
    - On teste à chaque fois.
  - 3. On utilise **[Inline Function](#inline-function)** sur le setter pour le faire disparaître complètement.
    - Si possible on rend la valeur membre immutable.
  - 4. On teste.
- **Théorie :**
  - On veut supprimer un setter à chaque fois qu'on veut que le champ soit immutable de l'extérieur.

### Replace Constructor with Factory Function

- **Exemple :**
  - **Avant :**
    ```javascript
    leadEngineer = new Employee(document.leadEngineer, "E");
    ```
  - **Après :**
    ```javascript
    leadEngineer = createEngineer(document.leadEngineer);
    ```
- **Étapes :**
  - 1. On crée une fonction _factory_, avec un appel au constructeur dans son corps.
  - 2. On remplace chaque appel au constructeur par un appel à la fonction _factory_.
    - On teste à chaque fois.
  - 3. Si possible, on limite la visibilité du constructeur.
- **Théorie :**
  - Le but de ce refactoring est de répondre au cas où on n'a pas envie d'utiliser la fonction de constructeur directement :
    - parce qu'il peut avoir des limitations comme renvoyer une instance spécifique.
    - parce qu'on ne peut en général pas personnaliser son nom.
    - parce qu'on ne peut pas le passer comme une simple fonction.

### Replace Function with Command

- **Exemple :**
  - **Avant :**
    ```javascript
    function score(candidate, medicalExam, scoringGuide) {
      let result = 0;
      let healthLevel = 0;
      // long body code
    }
    ```
  - **Après :**
    ```javascript
    class Scorer {
      constructor(candidate, medicalExam, scoringGuide) {
        this._candidate = candidate;
        this._medicalExam = medicalExam;
        this._scoringGuide = scoringGuide;
      }
      execute() {
        this._result = 0;
        this._healthLevel = 0;
        // long body code
      }
    }
    ```
- **Étapes :**
  - 1. On crée une classe vide avec un nom basé sur celui de la fonction.
  - 2. On utilise Move Function pour déplacer la fonction dans la classe.
    - On peut appeler la fonction _execute_ ou _call_ par exemple.
  - 3. On envisage de créer un champ pour chaque argument de la fonction, en déplaçant ces arguments sur le constructeur.
- **Théorie :**
  - Il s'agit d'encapsuler une fonction dans une classe, pour lui donner la possibilité d'avoir des méthodes associées, potentiellement de l'héritage etc.
  - 95 fois sur 100 l'auteur utilise une fonction normale non encapsulée.
  - Une des raisons pour utiliser le refactoring est aussi de diviser une fonction complexe en plus petits morceaux : on peut notamment transformer ses variables locales en variables membres qui seraient utilisées dans toutes les fonctions de la classe.

### Replace Command with Function

- **Exemple :**
  - **Avant :**
    ```javascript
    class ChargeCalculator {
      constructor(customer, usage) {
        this._customer = customer;
        this._usage = usage;
      }
      execute() {
        return this._customer.rate * this._usage;
      }
    }
    ```
  - **Après :**
    ```javascript
    function charge(customer, usage) {
      return customer.rate * usage;
    }
    ```
- **Étapes :**
  - 1. On utilise **[Extract Function](#extract-function)** pour extraire la création de l'instance de commande et de l'appel à la fonction.
  - 2. Pour chaque méthode appelée par la méthode principale, on utilise **[Inline Function](#inline-function)** pour l'éliminer.
  - 3. On utilise **[Change Function Declaration](#change-function-declaration)** pour ajouter les paramètres du constructeur à la méthode principale.
  - 4. Pour chaque champ de l'objet, on remplace son utilisation par l'utilisation des paramètres dans la méthode.
    - On teste à chaque fois.
  - 5. On inline la construction de l'objet et l'appel à la méthode dans le code appelant.
  - 6. On teste.
  - 7. On utilise **[Remove Dead Code](#remove-dead-code)** pour éliminer la classe de commande.
- **Théorie :**
  - L'objet de commande est puissant, mais il arrive aussi avec une certaine complexité : si la fonction est simple en général on n'en a pas besoin.

## 12 - Gestion de l'héritage

### Pull Up Method

- **Exemple :**

  - **Avant :**

    ```javascript
    class Employee {...}

    class Salesman extends Employee {
      get name() {...}
    }

    class Engineer extends Employee {
      get name() {...}
    }
    ```

  - **Après :**

    ```javascript
    class Employee {
      get name() {...}
    }

    class Salesman extends Employee {...}

    class Engineer extends Employee {...}
    ```

- **Étapes :**
  - 1. On examine les méthodes des classes filles à remonter pour s'assurer qu'elles sont **identiques**.
    - Si c'est pas le cas, on refactore jusqu'à obtenir une méthode identique à remonter dans chaque classe.
  - 2. On s'assure que tous les appels de méthode ou de champ seront accessibles depuis la classe mère.
  - 3. SI la signature des méthodes est différente, on les change avec **[Change Function Declaration](#change-function-declaration)** pour qu'elles soient similaires.
  - 4. On crée une méthode dans la classe mère, et on copie le code d'une des méthodes à remonter dedans.
  - 5. On lance les vérifications statiques.
  - 6. On supprime une à une les méthodes des classes filles, en testant à chaque fois.
- **Théorie :**
  - Le but de ce refactoring est d'éviter la duplication de code dans les classes filles, en le remontant dans la classe mère.
  - On l'utilise souvent après avoir généralisé une méthode avec **[Parameterize Function](#parameterize-function)** pour faire en sorte que les deux classes filles aient la même méthode, qu'on peut alors remonter.
  - Si la méthode à remonter fait référence à des champs dans les classes filles, on peut utiliser **[Pull Up Field](#pull-up-field)** pour les remonter d'abord.

### Pull Up Field

- **Exemple :**

  - **Avant :**

    ````javascript
    class Employee {...}

    class Salesman extends Employee {
      private name: string;
    }

    class Engineer extends Employee {
      private name: string;
    }```
    ````

  - **Après :**

    ```typescript
    class Employee {
      protected name: string;
    }

    class Salesman extends Employee {...}

    class Engineer extends Employee {...}
    ```

- **Étapes :**
  - 1. On inspecte bien l'utilisation du champ pour vérifier qu'il s'agit vraiment de la même chose.
  - 2. Si les deux champs ont des noms différents, on utilise Rename Field pour leur redonner le même nom.
  - 3. On crée un champ dans la classe mère, avec une protection suffisamment lâche pour que les classes filles y aient accès (_protected_).
  - 4. On supprime les champs des classes filles.
  - On teste.
- **Théorie :**
  - On se retrouve parfois avec le même champ présent dans deux classes filles, **pas forcément sous le même nom**.
    - Pour savoir si c'est le même champ, il faut voir comment il est utilisé dans la classe.
  - On va souvent vouloir déplacer le champ et ensuite déplacer les méthodes qui manipulent ce champ.

### Pull Up Constructor Body

- **Exemple :**

  - **Avant :**

    ```javascript
    class Party {...}

    class Employee extends Party {
      constructor(name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
      }
    }
    ```

  - **Après :**

    ````javascript
    class Party {
      constructor(name){
        this._name = name;
      }
    }

    class Employee extends Party {
      constructor(name, id, monthlyCost) {
        super(name);
        this._id = id;
        this._monthlyCost = monthlyCost;
      }
    }```
    ````

- **Étapes :**
  - 1. On définit un constructeur dans la classe mère (s'il n'existe pas déjà), et on l'appelle dans les constructeurs des classes filles.
  - 2. On utilise **[Slide Statements](#slide-statements)** pour déplacer les instructions communes du constructeur juste après l'appel à `super()`.
  - 3. On déplace le code commun dans le constructeur parent, en donnant tous les paramètres nécessaires dans l'appel à `super()`.
  - 4. On teste.
  - 5. Si on ne peut pas déplacer une partie du code vers le haut du constructeur juste après `super()`, on peut utiliser **[Extract Function](#extract-function)** pour l'extraire, puis **[Pull Up Method](#pull-up-method)** pour le remonter dans le parent et l'utiliser dans le constructeur parent.
- **Théorie :**
  - Ce refactoring fait la même chose que **[Pull Up Method](#pull-up-method)**, mais en répondant aux règles spécifiques des constructeurs.
  - Si on a du mal à appliquer ce refactoring, on peut utiliser **[Replace Constructor with Factory Function](#replace-constructor-with-factory-function)** à la place.

### Push Down Method

- **Exemple :**

  - **Avant :**

    ```javascript
    class Employee {
      get quota {...}
    }

    class Engineer extends Employee {...}

    class Salesman extends Employee {...}
    ```

  - **Après :**

    ```javascript
    class Employee {...}

    class Engineer extends Employee {...}

    class Salesman extends Employee {
      get quota {...}
    }
    ```

- **Étapes :**
  - 1. On copie la méthode dans les classes filles qui en ont besoin.
  - 2. On supprime la méthode depuis la classe mère.
  - 3. On teste.
  - 4. On supprime la méthode dans chaque classe fille qui n'en a pas besoin.
  - 5. On teste.
- **Théorie :**
  - Si une méthode n'est utilisée que par une classe fille (ou un petit nombre de classes filles par rapport au total) et qu'elle se trouve dans la mère, on peut la descendre pour rendre plus clair que les autres filles n'en ont pas l'usage.

### Push Down Field

- **Exemple :**

  - **Avant :**

    ```typescript
    class Employee {
      private quota: string;
    }

    class Engineer extends Employee {...}

    class Salesman extends Employee {...}
    ```

  - **Après :**

    ```typescript
    class Employee {...}

    class Engineer extends Employee {...}

    class Salesman extends Employee {
      protected quota: string;
    }
    ```

- **Étapes :**
  - 1. On crée le champ dans les classes filles qui en ont besoin.
  - 2. On supprime le champ de la classe mère.
  - 3. On teste.
  - 4. On supprime le champ des classes filles qui n'en ont pas besoin.
  - 5. On teste.
- **Théorie :**
  - Si un champ n'est utilisé que par une classe fille (ou un petit nombre de classes filles par rapport au total), on le descend dans les classes qui en ont l'usage.

### Replace Type Code with Subclasses

- **Exemple :**
  - **Avant :**
    ```javascript
    function createEmployee(name, type) {
      return new Employee(name, type);
    }
    ```
  - **Après :**
    ```javascript
    function createEmployee(name, type) {
      switch (type) {
        case "engineer":
          return new Engineer(name);
        case "salesman":
          return new Salesman(name);
        case "manager":
          return new Manager(name);
      }
    }
    ```
- **Étapes :**
  - 1. On va auto-encapsuler le champ qui contient la valeur de type (on crée des getter/setter et on les utilise en interne dans la classe, pour ne plus accéder au champ directement autrement que par ces getter/setter).
  - 2. On choisit une valeur de type, et on crée une classe fille pour cette valeur. En fonction de la méthode directe ou indirecte, on aura la classe mère à créer ou alors ce sera notre classe initiale.
  - 3. On surcharge le getter du champ de type de cette nouvelle classe pour qu'il renvoie la valeur de type littérale (celle qu'on avait initialement).
  - 4. On ajoute de la logique pour utiliser le getter issu de la nouvelle classe.
    - Avec la méthode directe, il faudra qu'on instancie la bonne classe dès le début, donc on peut utiliser **[Replace Constructor with Factory Function](#replace-constructor-with-factory-function)**.
    - Avec la méthode indirecte, on peut instancier la bonne classe de type dans le constructeur de la classe initiale.
  - 5. On teste.
  - 6. On répète la création de classe fille pour chaque valeur de type.
    - On teste à chaque fois.
  - 7. On supprime le champ de type initial.
  - 8. On teste.
  - 9. On utilise **[Push Down Method](#push-down-method)** et **[Replace Conditional with Polymorphism](#replace-conditional-with-polymorphism)** sur les méthodes qui utilisent les getter/setter créés à l'étape 1.
  - 10. On peut supprimer les getter/setter pour la valeur de type (plus besoin d'auto-encapsulation du type).
- **Théorie :**
  - Quand on veut faire des choses différentes en fonction du type de chose, par exemple des employés différenciés par leur fonction, on peut se contenter d'une variable qui nous indique cette information. Mais on peut parfois vouloir une hiérarchie de classes.
  - La hiérarchie de classe est utile quand :
    - On veut transformer des conditions similaires en polymorphisme comme avec Replace Conditional with Polymorphism.
    - On a des fonctionnalités qui ne concernent que certains types, qu'on peut du coup mettre dans une méthode de la classe fille concernée.
  - Il y a deux manières de le faire :
    - 1- Directement : transformer la classe qui prend le type en classe mère, et créer des classes filles pour chaque type.
      - Il faut bien réfléchir au critère sur lequel on crée notre hiérarchie : si on le fait pour ce type, on ne pourra pas en même temps le faire pour un autre critère.
      - Si le type est mutable, il vaut mieux partir sur la 2ème méthode.
    - 2- Indirectement : créer une hiérarchie de classes juste pour le type, et y placer la logique liée au type. On utiliserait ici la composition en gardant une instance de ce type dans la classe qui prend le type.

### Remove Subclass

- **Exemple :**

  - **Avant :**

    ```javascript
    class Person {
      get genderCode() {
        return "X";
      }
    }

    class Male extends Person {
      get genderCode() {
        return "M";
      }
    }

    class Female extends Person {
      get genderCode() {
        return "F";
      }
    }
    ```

  - **Après :**
    ```javascript
    class Person {
      get genderCode() {
        return this._genderCode;
      }
    }
    ```

- **Étapes :**
  - 1. On utilise **[Replace Constructor with Factory Function](#replace-constructor-with-factory-function)** pour que la classe fille qu'on veut supprimer soit construite depuis une fonction.
  - 2. Si on a du code qui applique une condition sur le type de classe fille, on extrait ce code avec **[Extract Function](#extract-function)**, et on le remonte vers la classe mère avec **[Move Function](#move-function)**.
  - 3. On crée un champ pour représenter le type de sous-classes dans la classe mère.
  - 4. On modifie les méthodes qui utilisent la classe fille pour qu'elles fassent référence au champ de type.
  - 5. On supprime la classe fille.
  - 6. On teste.
- **Théorie :**
  - Quand on a une hiérarchie de classes, on peut en profiter pour mettre des comportements dans chaque classe fille et obtenir quelque chose de flexible.
    - Mais parfois ces comportements ne sont pas (ou plus) suffisants pour justifier la complexité induite par la hiérarchie. Dans ce cas, il faut supprimer les classes filles.

### Extract Superclass

- **Exemple :**

  - **Avant :**

    ```javascript
    class Department {
      get totalAnnualCost() {...}
      get name() {...}
      get headCount() {...}
    }

    class Employee {
      get annualCost() {...}
      get name() {...}
      get id() {...}
    }
    ```

  - **Après :**

    ```javascript
    class Party {
      get name() {...}
      get annualCost() {...}
    }

    class Department extends Party {
      get annualCost() {...}
      get headCount() {...}
    }

    class Employee extends Party {
      get annualCost() {...}
      get id() {...}
    }
    ```

- **Étapes :**
  - 1. On crée une classe vide, et on fait hériter nos classes avec du code dupliqué de cette classe.
  - 2. On teste.
  - 3. Pour chaque classe fille, on utilise **[Pull Up Constructor Body](#pull-up-constructor-body)**, **[Pull Up Field](#pull-up-field)** et **[Pull Up Method](#pull-up-method)** pour déplacer le code commun aux classes filles dans la nouvelle classe mère.
  - 4. On refait une passe sur les classes filles, et si on constate encore du code commun mais pas dans des méthodes isolées, on utilise **[Extract Function](#extract-function)** pour l'isoler, puis **[Pull Up Method](#pull-up-method)** pour le remonter.
  - 5. On vérifie le code appelant, pour voir s' il ne faudrait pas utiliser l'interface de la classe mère quelque part.
- **Théorie :**
  - Le but de ce refactoring est de rassembler une logique dupliquée dans plusieurs classes vers une classe mère commune.
  - L'auteur conseille par défaut d'utiliser ce refactoring à la place de **[Extract Class](#extract-class)**, quitte à le transformer en délégation ensuite avec **[Replace Superclass with Delegate](#replace-superclass-with-delegate)**.

### Collapse Hierarchy

- **Exemple :**

  - **Avant :**

    ```javascript
    class Employee {...}

    class Salesman extends Employee {...}
    ```

  - **Après :**
    ```javascript
    class Employee {...}
    ```

- **Étapes :**
  - 1. On choisit la classe qu'on veut supprimer (mère ou fille).
  - 2. On utilise **[Pull Up Field](#pull-up-field)**, **[Pull Up Method](#pull-up-method)**, **[Push Down Field](#push-down-field)** et **[Push Down Method](#push-down-method)** pour déplacer tous les éléments de la classe à supprimer vers l'autre qui reste.
  - 3. On ajuste les références des méthodes déplacées dans la classe qui reste pour qu'elles s'intègrent avec le reste du code de la classe.
  - 4. On supprime la classe vide.
  - 5. On teste.
- **Théorie :**
  - Quand le fait d'avoir une hiérarchie de classes n'apporte plus suffisamment de choses pour contrebalancer la complexité induite par la hiérarchie elle-même, on supprime un les classes d'un des niveaux pour éliminer la hiérarchie.

### Replace Subclass with Delegate

- **Exemple :**

  - **Avant :**

    ```javascript
    class Order {
      get daysToShip() {
        return this._warehouse.daysToShip;
      }
    }

    class PriorityOrder extends Order {
      get daysToShip() {
        return this._priorityPlan.daysToShip;
      }
    }
    ```

  - **Après :**

    ```javascript
    class Order {
      get daysToShip() {
        return this._priorityDelegate
          ? this._priorityDelegate.daysToShip
          : this._warehouse.daysToShip;
      }
    }

    class PriorityOrderDelegate {
      get daysToShip() {
        return this._priorityPlan.daysToShip;
      }
    }
    ```

- **Étapes :**
  - 1. S'il y a de nombreux endroits où le constructeur de la classe fille qu'on va supprimer est utilisé, on va d'abord encapsuler la création des objets avec **[Replace Constructor with Factory Function](#replace-constructor-with-factory-function)**.
  - 2. On crée une classe vide pour le délégué. On lui fait prendre au constructeur tous les paramètres spécifiques à la classe fille à supprimer.
  - 3. On ajoute un champ à la classe mère pour stocker l'instance du délégué.
  - 4. On modifie le code de création de la classe fille à supprimer, pour qu'il crée l'instance du délégué et le place sur la classe mère : soit dans le constructeur de la classe fille, soit dans la factory function (si on l'a créée à l'étape 1).
  - 5. On choisit une méthode de la classe fille à déplacer dans le délégué.
  - 6. On utilise **[Move Function](#move-function)** pour la déplacer dans le délégué.
    - On ne fait pas la dernière étape de supprimer la fonction de délégation restée dans la classe fille.
    - Si la méthode a besoin d'autres champs ou méthodes pour fonctionner, on les déplace aussi.
    - Si elle a besoin de méthodes ou de champs qui doivent rester sur la classe mère, on passe une référence vers la classe mère au délégué.
  - 7. Si la méthode dans la classe fille qu'on est en train de traiter est appelée depuis l'extérieur de sa hiérarchie de classe, on déplace la méthode (qui est maintenant une méthode de délégation vers l'objet délégué) vers la classe mère. On y fait une vérification de l'existence du délégué avant l'appel.
    - Si il n'y avait pas d'appelants externes, on applique simplement **[Remove Dead Code](#remove-dead-code)** sur la méthode dans la classe fille.
  - 8. On teste.
  - 9. On répète les étapes 7 et 8 jusqu'à ce que toutes les méthodes de la classe fille soient transférées vers le délégué.
  - 10. On trouve toutes les occurrences de construction d'objet de la classe fille, et on le modifie pour construire la classe mère à la place.
  - 11. On teste.
  - 12. On applique **[Remove Dead Code](#remove-dead-code)** sur la classe fille vide.
- **Théorie :**
  - L'héritage permet de représenter naturellement la catégorisation des objets.
  - Elle a par contre deux problèmes :
    - On ne peut utiliser l'héritage pour classer que selon **un seul axe** de catégorie. Et donc la thématique qui n'est pas choisie pour l'axe utilisé pour l'héritage doit bien être traitée autrement.
    - Elle induit un grand couplage entre classe mère et fille, avec les changements dans la mère qui impactent toutes les filles.
  - La délégation (ou composition) règle ces deux problèmes.
    - L'auteur a connaissance du principe populaire “_Favor object composition over class inheritance_”, mais l'aurait bien remplacé par “_Favor a judicious mixture of composition and inheritance over either alone_”.
    - Pour autant, il préfère utiliser par défaut l'héritage qui a ses propres avantages, quitte à utiliser ce refactoring pour passer sur de la délégation quand il sent qu'il y a des frictions dans la hiérarchie.
    - Cette question de choix entre héritage et délégation est aussi discutée dans le livre de GoF.
  - Une des possibilités peut aussi être d'avoir une délégation qui elle-même a une hiérarchie pour laisser l'héritage de la classe principale à un autre axe, et profiter quand même de la puissance de l'héritage pour l'axe sur lequel on délègue.

### Replace Superclass with Delegate

- **Exemple :**

  - **Avant :**

    ```javascript
    class List {...}

    class Stack extends List {...}
    ```

  - **Après :**

    ```javascript
    class Stack {
      constructor() {
        this._storage = new List();
      }
    }

    class List {...}
    ```

- **Étapes :**
  - 1. On crée un champ dans la classe fille avec le type de la classe mère, et on l'instancie dans le constructeur de la classe fille.
  - 2. Pour chaque méthode de la classe mère, on crée une forwarding function dans la classe fille, qui appelle simplement la bonne méthode sur le champ créé à l'étape 1.
    - On teste à chaque fois.
    - Parfois il faut déplacer plusieurs méthodes interdépendantes, par exemple getter/setter.
  - 3. Quand on a créé des forwarding functions pour toutes les méthodes, on supprime le lien d'héritage.
  - 4. On teste.
- **Théorie :**
  - Un signe typique qui indique qu'il ne faut pas utiliser l'héritage, c'est quand on a des fonctions de la classe mère qui n'ont pas d'utilité dans les classes filles.
    - Un exemple connu c'est la liste qu'on prend comme classe mère de la pile : la plupart des opérations de liste ne sont pas utiles pour la pile.
  - Un autre problème aussi c'est que l'héritage induit d'un point de vue modélisation l'idée que les classes filles sont des instances de la mère, à tout point de vue y compris dans le monde physique, ce qui est assez restrictif pour bien des situations.
    - La délégation pose une vraie limite entre les deux classes.
  - Un des inconvénients de la délégation c'est que pour les fonctionnalités similaires entre une classe et son délégué, il faut faire une forwarding function.
  - L'auteur conseille quand même, comme dans la technique précédente, de partir sur de l'héritage par défaut, et de remplacer si besoin par la délégation avec ce refactoring.
