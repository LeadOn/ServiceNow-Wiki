# GlideAjax - Cheet Sheet

> Ce guide est destiné à évoluer, à la fois en termes de format et de contenu. Tout feedback est bon à prendre, n'hésitez pas à me faire mes retours (mes contacts sont disponibles sur [mon portfolio.](https://www.valentinvirot.fr) Merci, et bonne lecture !

Pour communiquer entre client et serveur (exemple concret : récupérer des données d'une autre table depuis un client script), il est nécessaire d'utiliser un mécanisme utilisé GlideAjax.

> Certains curieux vont reconnaître le terme Ajax, celui-ci étant utilisé de base dans les langages web. Vous ne serez pas dépaysés, car le fonctionnement est très similaire !
> {.is-info}

Pour cet exemple, nous allons créer un Client Script sur la table des incidents, permettant de récupérer l'adresse email du groupe de support sélectionné en "Assignment Group".

## Création d'un script include

Pour récupérer des données côté serveur, la première étape est la création d'un Script Include.

> Bien évidemment, cette étape est nécessaire uniquement dans le cas d'un nouveau développement de A à Z. Vous pouvez tout à fait ajouter une méthode à un script existant, si celui-ci est pertinant dans votre cas d'usage !
> {.is-info}

Lors de la création du Script Include, il est essentiel de cocher la case "Client Callable" avant de commencer à taper du script, afin que celui-ci soit configuré pour hériter du script AbstractAjaxProcessor. La bonne pratique veut également que vous utilisiez l'anglais, et que vous lui donniez un nom pertinent, pour faciliter la compréhension par d'autres, ainsi que la maintenance long terme !

Vous devriez avoir quelque chose de similaire à cela :

```js
var NameOfTheScriptInclude = Class.create();
NameOfTheScriptInclude.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    type: "NameOfTheScriptInclude",
  }
);
```

> Il est important de comprendre pourquoi le script doit être appelé côté client.
> Lorsque ServiceNow affiche un formulaire sur votre navigateur web, celui-ci prépare les données à afficher en amont du chargement, et les stocke dans votre navigateur. De ce fait, aucun lien n'est actif entre le navigateur et ServiceNow, et il est donc impossible de récupérer de nouvelles données. Si le script est "Client Callable", votre navigateur peux contacter ServiceNow, lui demandant de récupérer des informations, et donc rafraîchîr les données de votre navigateur, ou bien les compléter.
> {.is-info}

## Définir l'ACL d'exécution

Depuis plusieurs releases de ServiceNow, pour chaque Script Include client callable créé, une ACL est automatiquement générée, afin de limiter la population pouvant y faire appel. Ceci est très important pour réduire les risques de Cyberattaque, le script étant exécuté sur le navigateur de l'utilisateur, celui-ci pourrait être utilisé dans un cadre malicieux.

Si le script que vous développez est destiné à une population ciblée, pensez à modifier cette ACL en conséquence !

> Ceci peut également empêcher votre script de fonctionner ! Renseignez-vous bien sur les rôles de vos utilisateurs cibles avant de tester !
> {.is-warning}

## Ajout de la méthode désirée

Maintenant que nous avons un Script Include pouvant accueillir notre code, implémentons la méthode !

Il est important de savoir 3 éléments clés :

- Quel est la finalité de mon code (que dois-je récupérer ?)
- Comment la récupérer (où est stocké les données concernées ?)
- De quoi ai-je besoin pour y arriver (quels paramètres ?)

Une fois ces éléments identifiés, créons notre solution !

Dans le Script Include, comme tout autre script, ajoutons une méthode. Pour le nom, essayez de trouver quelque chose de pertinent, pour qu'un futur consultant puisse comprendre directement ce qu'il voit !

```js
var NameOfTheScriptInclude = Class.create();
NameOfTheScriptInclude.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    /**
     * Gets the support group email, from an assignment group record.
     *
     * @param {string} - Assignment group Sys ID
     * @return {string} - email as a string, or null if nothing found
     */
    getSupportEmailFromAssignmentGroup: function () {
      return "emailofthegroup@valentinvirot.fr";
    },

    type: "NameOfTheScriptInclude",
  }
);
```

> Vous voyez les commentaires au-dessus de la méthode ? Il est très important de compléter ceux-ci de manière précise à chaque fois, pour que plus tard, que ce soit vous ou bien un futur collègue qui doit intervenir ou réutiliser ce code, puisse comprendre ce que fait la méthode, ces paramètres et son format de retour simplement !
> Également, il est de bonne pratique d'utiliser l'anglais comme langue principale. Essayez de respecter cela !
> {.is-info}

Maintenant que la méthode est créée, complétez-la par le fonctionnement souhaité. Ici, je souhaite récupérer l'email de support du groupe d'assignation. J'aurai donc besoin d'abord de savoir de quel groupe il s'agit. Ajoutons donc un paramètre en entrée : le sys_id du groupe d'affectation.

Pour ce faire, dans un script voué à être utilisé côté client, il suffit d'ajouter la ligne suivante :

```js
var NameOfTheScriptInclude = Class.create();
NameOfTheScriptInclude.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    /**
     * Gets the support group email, from an assignment group record.
     *
     * @param {string} - Assignment group Sys ID
     * @return {string} - email as a string, or null if nothing found
     */
    getSupportEmailFromAssignmentGroup: function () {
      var groupSysId = this.getParameter("sysparm_group_sys_id");

      return "emailofthegroup@valentinvirot.fr";
    },

    type: "NameOfTheScriptInclude",
  }
);
```

> La convention veut ici que le nom de la variable commence par sysparm\_, et soit rédigée en convention Snake case. Notez bien le nom de votre paramètre, nous en aurons besoin plus tard.
> {.is-info}

Une fois ce paramètre ajouté, je peux traiter le script comme n'importe quel script plateforme, et retourner la valeur souhaitée :

```js
var NameOfTheScriptInclude = Class.create();
NameOfTheScriptInclude.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    /**
     * Gets the support group email, from an assignment group record.
     *
     * @param sysparm_group_sys_id {string} - Assignment group Sys ID
     * @return {string} - email as a string, or null if nothing found
     */
    getSupportEmailFromAssignmentGroup: function () {
      var groupSysId = this.getParameter("sysparm_group_sys_id");

      // Getting group from table
      var groupGr = new GlideRecord("sys_user_group");
      if (groupGr.get(groupSysId)) {
        return groupGr.getValue("email");
      } else {
        return null;
      }
    },

    type: "NameOfTheScriptInclude",
  }
);
```

Et voilà ! Le code côté serveur est terminé ! Maintenant, il ne nous reste plus qu'à l'appeler dans notre client script !

## Appel depuis le client script

Maintenant que le code est prêt côté serveur, appelons le depuis notre client script !
Ici, pour cet exemple, nous sommes dans le cas d'un client script, onChange, sur le champ Assignment group. Le code de base ressemble donc à cela :

```js
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue == "") {
    return;
  }
}
```

> Même si dans cet exemple, nous utilisons un client script onChange, vous pouvez bien entendu utiliser l'appel du script dans les autres types
> {.is-info}

Tout comme le script serveur, nous allons dans un premier temps prendre le paramètre souhaité, et le préparer pour l'envoyer à notre serveur.

```js
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue == "") {
    return;
  }

  // Since this script is onChange, on the Assignment group field, we can use newValue. You can also use g_form.getValue().
  var assignmentGroup = newValue;
}
```

Maintenant, faisons appel à notre script include ! Il vous faut 3 éléments ici :

- Le nom du script include créé à l'étape 1
- Le nom de votre méthode
- Les noms des paramètres que vous avez défini

Prêt ? C'est parti !

```js
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue == "") {
    return;
  }

  // Since this script is onChange, on the Assignment group field, we can use newValue. You can also use g_form.getValue().
  var assignmentGroup = newValue;

  // Calling script include
  // Here, call it by a recognizable name!
  var getEmailGa = new GlideAjax("NameOfTheScriptInclude");

  // Here, type the name of the method that you've created in the script include
  getEmailGa.addParam("sysparm_name", "getSupportEmailFromAssignmentGroup");

  // Setting parameters
  // Put one line per parameter that you need
  // Syntax is : 'variable name', value
  getEmailGa.addParam("sysparm_group_sys_id", newValue);
}
```

> Une fois les paramètres définis, il ne reste plus que l'exécution. Ici, pour des développeurs juniors, il est possible que ce soit effrayant/incompréhensible, car nous utilisons un mécanisme appelé "Callback". Nous allons faire appel à une fonction pour traiter le résultat de l'appel Serveur. L'essentiel est de comprendre l'emplacement où vous devez implémenter ce que vous désirez faire avec les données récupérées dans le serveur. Pour cet exemple, nous allons définir la description du ticket comme étant l'email du groupe.
> {.is-info}

Voilà ce que donne le résultat avec le traitement final :

```js
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue == "") {
    return;
  }

  // Since this script is onChange, on the Assignment group field, we can use newValue. You can also use g_form.getValue().
  var assignmentGroup = newValue;

  // Calling script include
  // Here, call it by a recognizable name!
  var getEmailGa = new GlideAjax("NameOfTheScriptInclude");

  // Here, type the name of the method that you've created in the script include
  getEmailGa.addParam("sysparm_name", "getSupportEmailFromAssignmentGroup");

  // Setting parameters
  // Put one line per parameter that you need
  // Syntax is : 'variable name', value
  getEmailGa.addParam("sysparm_group_sys_id", newValue);

  // Executing the Ajax call
  getEmailGa.getXMLAnswer(useAjaxResult);
}

// Callback function called when result is received from server
// Do what ever you want with it, answer contains what is returned from the Script Include.
function useAjaxResult(answer) {
  if (answer == null) {
    g_form.setValue("description", "No email found.");
  } else {
    g_form.setValue("description", answer);
  }
}
```

> Si vous désirez plus d'informations / aller plus loin, comme je le dis si souvent, la documentation est votre meilleure amie !
> Vous pouvez trouver votre bonheur sur [la documentation officielle.](https://developer.servicenow.com/dev.do#!/reference/api/utah/client/c_GlideAjaxAPI)
