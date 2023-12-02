# GlideAjax - Cheet Sheet

> This guide will evolve, both in terms of content and format. Every feedback is nice to have, please don't hesitate to contact me, all of my contact information are on [my portfolio.](https://www.valentinvirot.fr) Thank you, have a nice day!

In order to communicate between client and server (for exemple: get data from related table from a client script), you need to use a mecanism called GlideAjax.

> Some of you might recognize the Ajax term, this one being used in other web languages. You won't be lost, it almost works the same way!
> {.is-info}

For this guide, we will create a Client Script on the incident table, to get selected Assignment Group's email.

## Creating a Script Include

To get data from the server, first step is to create a Script Include.

> Of course, this step is only mandatory in case you're developping something from scratch. If you need to do this multiple times for the same perimeter, you can use the same script include!
> {.is-info}

When creating a Script Include, you need to check the "Client Callable" checkbox before typing script in the dedicated field, in order to make it heritate from the AbstractAjaxProcessor script. For you english speakers, it's natural, but for everyone else, good practice requires you to use english as language while implementing, to make life easier for everyone else that can work on your code!

You should have something like that:

```js
var NameOfTheScriptInclude = Class.create();
NameOfTheScriptInclude.prototype = Object.extendsObject(
  global.AbstractAjaxProcessor,
  {
    type: "NameOfTheScriptInclude",
  }
);
```

> It is important to understand why this script needs to be called client side.
> When ServiceNow displays a form in your web browser, it pre-loads data to display before giving you the page, and temporarely stores it in your web browser. By doing that, it removes latency when displaying the page (information are prefilled), and no opened link are opened between your PC and the instance (to make it really simple). If the script include is designated as "Client Callable", your webbrowser will be able to call back the instance, to get additionnal data for exemple.
> {.is-info}

## Define execution ACL

Since some ServiceNow releases, for every client callable script include, an ACL is automatically generated, to limit what population can execute it. This is very important to limit Cyberattacks, the script being executed from the end user's machine.

If the script that you're developping have a destinated population, please update the associated ACL!

> This can make your script unusable! Please think about it before updating the ACL.
> {.is-warning}

## Adding desired method

Now that we have a Script Include that can host our code, let's implement our method!

It's very important to think about 3 major points before trying to implement:

- What do I want to do with my code (what do I want to get?)
- How can I get it (where is it stored?)
- What do I need to get it (what parameters?)

Once you've identified theses points, let's create our solution!

In the Script Include, like every other script, let's add a method. For its name, please find something pertinent, for a future consultant / the future you to understand quickly what it does!

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

> See the comments above the method? It is very important to add details to it everytime, so that later on, you or one of your coworkers that needs to work on / use your code can understand what parameters it needs, what it does, and what it returns!
> For non english speakers, please use english as primary language for this!
> {.is-info}

Now that the method is created, let's implement what you need. For this guide, I want to get the email of the assignment group. Because I need to know what assignment group is selected, let's add our first parameter: assignment group's sys_id!

To do that, let's add the following code:

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

> In this case, the convention says that variable name will start by "sysparm\_", and the rest of it needs to be written using the Snake Case convention. Please note this variable name somewhere, you'll need it later.
> {.is-info}

Now that this parameter is added, let's write code, like other scripts in the platform, to get what we need.

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

Et voilà! Server side code is completed, now we only need to call it from client side!

## Calling Script Include from Client Script

Let's call our Server Script now!
In our case here, I want to get selected Assignment group's email. So naturally, I will create a Client Script, onChange, on the assignment group, that gives me this code by default.

```js
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue == "") {
    return;
  }
}
```

> Even if in this case, we're using an onChange type client script, of course you can use other types, for your use case.
> {.is-info}

Like our Server Script, first of all, we will get our parameter value (here, the new value of the assignment group field), and prepare it to be sent.

```js
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue == "") {
    return;
  }

  // Since this script is onChange, on the Assignment group field, we can use newValue. You can also use g_form.getValue().
  var assignmentGroup = newValue;
}
```

Now, we need 3 things to call our Script Include:

- Name of the script include created on step 1
- Name of our method
- Name of the parameters that you've created

Let's do it!

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

> Once theses parameters have been defined, we only need to execute our code. Here, for junior developers/Consultant, it's normal to not understand what we are doing. We're going to use a mecanism called "Callback". We'll use a function to receive data from the server, and do something with it. The most important thing is to understand where to implement what you want to do. In this case, we'll get the assignment group's email from the server, and store it in the description field.
> {.is-info}

This is what it looks like at the end:

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

Done! You've successfully created your first Client Script with GlideAjax in it!

> If you want more informations / go further, official documentation is your best friend!
> You'll find everything that you need [on ServiceNow's official documentation.](https://developer.servicenow.com/dev.do#!/reference/api/utah/client/c_GlideAjaxAPI)
