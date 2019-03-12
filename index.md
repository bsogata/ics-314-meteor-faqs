# ICS 314 Meteor FAQs

This page is intended to answer questions about Meteor for ICS 314 Spring 2019<sup><a href="#footnote-1">1</a></sup>.  This also serves the secondary purpose of providing an example of how to use GitHub Pages<sup><a href="#footnote-2">2</a></sup>.

## Table of Contents
1. <a href="#project-structure">Walkthrough of System User Interface</a>
2. <a href="#data">Data and Accounts: Structure and Initialization</a>
3. <a href="#navigation">Navigation, Routing, Pages, and Components</a>
4. <a href="#forms">Forms</a>
5. <a href="#roles">Authorization, Authentication, and Roles</a>
6. <a href="#misc">Miscellaneous Questions</a>
7. <a href="#unanswered">Unanswered and Unclassified Questions</a>

## <span id="project-structure">Walkthrough of System User Interface</span>
#### Since the initial passwords for the template are stored in plain text in _/config/settings.development.json_, should projects based on this template be stored in a private repository?
Yes; you should make repositories private unless there is some compelling reason for them to be public.  (Those of you who took my ICS 111 will recall taking a similar approach to `public` and `private` instance variables.)

#### When should we use _.jsx_ files instead of _.js_ files and vice versa?
In general, use _.jsx_ whenever you use (React) components in the file and _.js_ otherwise.

#### Do the functions (ex. `addData` in _/app/imports/startup/server/stuff.js_) have to be in specific files?
As far as your teaching assistant knows, functions can be in any file that is `import`ed where the function is used.  However, if a function is only used on one file it makes sense to place the function in that one file.  `addData` is only used in initializing the seed data for the `Stuffs` collection, so it is located in the file that handles the `Stuffs` collection on the server side.

#### How does `if (this.userId) {…}` check if the user is logged in or not?  What does the `this` keyword refer to here?
The documentation for `this.userId` is [here](https://docs.meteor.com/api/pubsub.html#Subscription-userId).  If the user is logged in, `this.userId` exists and is a truthy value.  If the user is not logged in, `this.userId` is `undefined`, which is a falsy value.

#### How does `if (Meteor.settings.defaultData)` work?
As with the previous question, `Meteor.settings.defaultData` will be a truthy value if it exists and a falsy value otherwise.

#### How would we make a Meteor project from scratch (that is, without using meteor-application-template-react)?  What files would we have to create?
[You have already done something like this](http://courses.ics.hawaii.edu/ics314s19/morea/meteor-1/experience-meteor-react-tutorial.html); note though that the template basically includes everything that you would need on a site.  Most websites that are complex enough to merit using Meteor will have some sort of user accounts and data associated with those accounts.

#### Are the names of the directories significant?
Sometimes; certainly the files in _/app/imports/startup_ should have something to do with what happens when the server starts up, _/client_ directories are only visible on the client side, _/server_ directories contain files only visible on the server side, etc.  Something like the _stuff_ directory for _/app/imports/api/stuff/stuff.js_ could be named differently though; however, we name it _stuff_ in accordance with convention and general common sense.

Essentially, anything covered in [Example directory layout](https://guide.meteor.com/structure.html#example-app-structure) should be exactly as is; you have some flexibility in naming anything else.

#### Is it safe to log in while already logged in?
Yes, as far as I know.

#### Can we just write HTML files in _/app/client_?
Yes, but if you are only making static web pages with HTML and CSS there is not much point in using Meteor in the first place.

#### Is the landing page basically the home page?
Yes.

---

## <span id="data">Data and Accounts: Structure and Initialization</span>
#### How does the application remember the user accounts and other data even after terminating the Meteor process?
When you use a computer, you have to save your work before shutting down the computer.  That data is stored in a file and you can open that file later to resume your work.  Databases serve the same purpose: your application stores data in the database so the data is preserved even after the application has closed.

#### Where is all of the data stored?  
The technically correct answer is _/app/.meteor/local/db_, though that is probably not of direct use to you since the data is not stored in a human-readable format.  Aside from a tired attempt at sarcasm, this does show that the data is in the project directory, so you do not have to worry about data from one project intermingling with data from another project<sup><a href="#footnote-3">3</a></sup>.

The more useful answer is that you do not have to worry about where the data is stored: MongoDB takes care of preserving the data for you.

#### How are passwords stored?  What prevents an attacker<sup><a href="#footnote-4">4</a></sup> from accessing those passwords?
If we look at the `users` collection in the MongoDB console, we see:

```
meteor:PRIMARY> db.users.find()
{ "_id" : "RvkbtmXXDoRDYeohc", "createdAt" : ISODate("2019-03-11T01:48:08.812Z"), "services" : { "password" : { "bcrypt" : "$2a$10$mS4VO9apDZC.Ul6U1YtfHOtgsY8JwVlCNq7muej4GgCTGTRyiW9K6" } }, "username" : "admin@foo.com", "emails" : [ { "address" : "admin@foo.com", "verified" : false } ], "roles" : [ "admin" ] }
{ "_id" : "Bf2XizdLC5fRq4BCk", "createdAt" : ISODate("2019-03-11T01:48:09.014Z"), "services" : { "password" : { "bcrypt" : "$2a$10$OZh60hVYwNNBVJ.SZ4q6oegTEHdFRS4SSpDKbNMTxSZIa7y/pnE7." }, "resume" : { "loginTokens" : [ { "when" : ISODate("2019-03-11T02:24:40.264Z"), "hashedToken" : "Oz02OPH48WBQeyIJ3M6nIztlYHfok6V/uWBG9VDPuSM=" } ] } }, "username" : "john@foo.com", "emails" : [ { "address" : "john@foo.com", "verified" : false } ] }
```

In particular, note that the `password` field for the first user has a value of `{ "bcrypt" : "$2a$10$mS4VO9apDZC.Ul6U1YtfHOtgsY8JwVlCNq7muej4GgCTGTRyiW9K6" }`.  This is very clearly not the default password provided in _/config/settings.development.json_.  Storing passwords as plain text is very much a bad idea; instead, we apply a hash function to the password and store that hashed value.  When someone attempts to log in as the user, we then use the same hash function.  If the hashed value of the input matches the value stored in the database, then we are reasonably certain that the password is correct<sup><a href="#footnote-5">5</a></sup>; otherwise we know we can reject the login.

#### The **Meteor** &gt; **MiniMongo** tab shows all of the collections in the database that are published on the current page; what stops the `users` collection from displaying passwords?
Per [the documentation for `meteor/accounts-base`](https://docs.meteor.com/api/accounts.html), "by default, the current user's `username`, `emails` and `profile` are published to the client"; our `users` have no `profile` field and we add in `roles`, but the `password` field is not included in the list of published fields and therefore will not appear in the developer tools.

#### What is bcrypt?
[bcrypt](https://www.npmjs.com/package/bcrypt) allows us to hash passwords.

#### Is it possible to get rid of the bcrypt warning message in the Meteor console?
Yes, but it is probably not worth the time.  As noted on [the course website](http://courses.ics.hawaii.edu/ics314s19/morea/meteor-1/reading-meteor-tips.html), the performance issues with bcrypt will only occur on logins and those are relatively uncommon.  One might even argue that slower logins are a security feature as it forces an attacker to wait longer between attempts to break in, and after a certain point said attacker may well decide that your site is not a time-efficient target.

#### What is the purpose of having sample data in _/config/settings.development.json_?
We use the seed data in _/config/settings.development.json_ to populate the database the first time that we start the application.  This gives us enough data to test the application: the seed data for _meteor-application-template-react_ allows us to immediately log in and interact with the website without having to touch the MongoDB console.

#### Can you define multiple schemas for the same collection?
Yes; see [the `attachSchema` documentation](https://github.com/aldeed/meteor-collection2#attaching-multiple-schemas-to-the-same-collection).

#### Can we expand the `users` collection to contain more data?
Yes, just as you can change the schema for any collection.  Since `users` is built into Meteor though rather than a collection that you defined yourself, you would have to follow [a slightly different process](https://guide.meteor.com/accounts.html#custom-user-data).

#### How can we enforce password constraints (ex. a minimum length, must contain numbers, etc.)?
See the [Simple Schema documentation](https://github.com/aldeed/simple-schema-js#schema-rules); `min` is easy enough for the minimum length, though you may have to write a separate function to perform [custom validation](https://github.com/aldeed/simple-schema-js#custom-field-validation) or use regular expressions.

#### How can we allow users to change their passwords?
See the documentation for [`Accounts.changePassword`](https://docs.meteor.com/api/passwords.html#Accounts-changePassword). 

#### How are passwords case-sensitive while user names are not?
See the Meteor notes on [case sensitivity](https://guide.meteor.com/accounts.html#case-sensitivity).

#### What is that `"verified" : false` in the "emails" field?  How do we change that value?
This indicates that the email address has not been verified yet: it may be a valid email address but we do not know that it actually exists and belongs to the user.  [`Accounts.sendVerificationEmail`](https://docs.meteor.com/api/passwords.html#Accounts-sendVerificationEmail) allows us to start that verification process.

#### What is the `{ tracker: Tracker }` in _/app/imports/api/stuff/stuff.js_ used for?
`Tracker` listens for changes on your data sources; see the [Simple Schema documentation](https://github.com/aldeed/simple-schema-js#enable-meteor-tracker-reactivity) for notes on how this is integrated with Simple Schema.

#### If we remove `"defaultData"` from _/config/settings.development.json_ after running the application for the first time, will those values be removed from the database?
No, by that point the values from `"defaultData"` are already in the database and removing them from the seed data will have no impact on the database.

#### Is `StuffAdmin` another database that runs alongside `Stuff`?
No, both `Stuff` and `StuffAdmin` are publications: they contain data from the database but are not databases themselves.

#### What does `withTracker` do in some of the pages (ex. _/app/imports/ui/pages/EditStuff.jsx_)?
The `withTracker` function provides access to the necessary publications from the database through setting up subscriptions; see more details in [the official documentation](https://guide.meteor.com/react.html#using-withTracker).

---

## <span id="navigation">Navigation, Routing, Pages, and Components</span>
#### The template shows how to add, edit, and show documents in a collection; how do you delete a document?
Use [`Collection#remove`](https://docs.meteor.com/api/collections.html#Mongo-Collection-remove).

#### Why do we need `export` statements for the components?  Does `import` not load in everything from the file?
`import` is not analagous to the `#include` in C and C++: it does not insert the entire imported file.  Instead, you must export whatever you want to import in a different file.  This may seem cumbersome at first, but this system allows for more flexibility and efficiency since you only import what you actually need.  For example, you only import the Semantic UI React components that you actually use in the file, not the entire `semantic-ui-react` library.

#### Why do we use `export` in some files and `export default` in others?
As the keyword suggests, `export default` is the default export: if you do not provide a specific item to import, Meteor will import the default export.  Any other exported elements must be referred to by name.  We see an example of this in _/app/imports/ui/pages/ListStuff.jsx_:

```
import { Stuffs } from '/imports/api/stuff/stuff';
import StuffItem from '/imports/ui/components/StuffItem';
```

`Stuffs` used `export` because _app/imports/api/stuff/stuff.js_ exports both `Stuffs` and `StuffSchema`.  `StuffItem` on the other hand was the only item exported from _/app/imports/ui/components/StuffItem_ and therefore used `export default`.  

Note that the name referenced when importing a default export is irrelevant.  The `export` statement in _/app/imports/ui/components/NavBar.jsx_ is:

```
export default withRouter(NavBarContainer);
```

However, the `import` statement in _/app/imports/ui/layouts/App.jsx_ is:

```
import NavBar from '../components/NavBar';
```

Even though the actual value exported from _NavBar.jsx_ is `NavBarContainer`, we can still refer to it as `NavBar` in _App.jsx_.

#### How does `publish` work?  Does it just send data from the server to the client?  When does this data transfer happen?
'publish' defines a set of data on the server.  The subscriptions you create on the client side with `Meteor.subscribe` retrieve that data from the server to use in your components and pages.  Unlike SQL views, MongoDB will automatically update these subscriptions if the original collection is modified.

#### Can we use object-oriented programming in Meteor?
You already are; everything in _/app/imports/ui_ extends `React.Component`.  You could thus make a component that extends `React.Component` and have other components that inherit from the first component.

#### How is a protected route different from a regular route?
Both `ProtectedRoute` and `AdminProtectedRoute` are defined in _/app/imports/ui/layouts/App.jsx_; `ProtectedRoute` is a `Route` with additional logic to determine whether the user is logged in (the `const isLogged = Meteor.userId() !== null`).  

#### How is an exact path different from a regular path?
An exact path must be an exact match.  The `Landing` page requires that we be at _"/"_, not some other path such as _"/signin"_ that merely contains a _/_.

#### The pages export `withTracker`; where is `withTracker` called?
`withTracker` is actually called in the export itself; note the presence of parentheses for the `withTracker` function.  

#### Is it possible to conditionally render parts of pages depending on whether the user is an admin or not?  
Yes; see _/app/imports/ui/components/NavBar.jsx_ for an example.

#### Can users add their own fields to the table in _/app/imports/ui/pages/ListStuff.jsx_?
Not in the template application, no.  Actually implementing this is theoretically possible but decidedly non-trivial: adding a new document with the new field would not be a problem, but adding said document to a collection with an existing schema would cause problems.

#### What happens if you remove the subscription from _/app/imports/ui/pages/ListStuff.jsx_?
Without the subscription to `Stuff`, the _ListStuff_ page would not have access to the `Stuffs` to display.  You can test what happens in this case on your side.

#### Are the `Route`, `ProtectedRoute`, and `AdminProtectedRoute` components built into Meteor?
Look through the rest of the file: `Route` is imported from `react-router-dom` and both `ProtectedRoute` and `AdminProtectedRoute` are defined in _App.jsx_.

#### What is the relationship between publications and subscriptions?  
The publication on the server side provides the data; the subscription on the server side receives the data.  By analogy, Dr. Johnson publishes a screencast and you subscribe to it (in the form of watching the video).

#### What would happen if you removed the `return this.ready()` from the publications?
The `this.ready()` call signals that the data transfer is complete.  Without this, any subscriptions on the publications would continue to wait for data indefinitely.

#### Is it possible to place the header and footer into individual pages rather than in _/app/imports/ui/layouts/App.jsx_?
Yes, though the question itself hints at why this is a bad idea: you would have to include the header and footer in each page in your application.  Placing the header and footer in _/layouts/App.jsx_ makes the header and footer appear on every page.

#### Is it possible to modify `ProtectedRoute` to only allow users with some combination of roles?
Yes, though you would have to `&&` the clauses together yourself.

#### What is the `:_id` in the `"/edit/:_id"` route?
The `:_id` is replaced with the `_id` of the `Stuff` being edited.

---

## <span id="forms">Forms</span>
#### How does error checking work?
This course uses [Uniforms](https://github.com/vazco/uniforms) in conjunction with [Simple Schema](https://github.com/aldeed/simple-schema-js) to accomplish this; see the documentation for those tools for more details.  Essentially, Uniforms uses the schema created with Simple Schema to generate and format the form displayed on the page.

#### Why "Bert Alert"?
[Ask the developer](https://github.com/themeteorchef/bert/wiki/Contribution-Guide#asking-questions).  

#### What options are available so the Bert alert does not block the navbar?
See the [Bert documentation](https://github.com/themeteorchef/bert/), especially the [API & Defaults](https://github.com/themeteorchef/bert/#api--defaults) and [Customization](https://github.com/themeteorchef/bert/#customization) sections.

#### Dr. Johnson mentions "spinners" in [the screencast](https://youtu.be/s77Vg6hgevI), but the actual component is [`Loader`](https://react.semantic-ui.com/elements/loader/).  Why the difference?
The Semantic UI developers did not want you to confuse the `Loader` component with [Spinners](https://youtu.be/4YSbGXNXfVg).

#### How should we handle forgotten passwords: provide a button or link to reset the password or automatically reset the password after *n* invalid login attempts?
These are not mutually exclusive options: you can provide the *Reset Password* link while still tracking the number of unsuccessful login attempts.  You could add an integer field to the `users` collection to store the number of consecutive failed logins and update that in `handleSubmit` in the `Signin` page.

#### How can we validate that an email address is an actual email address?
To verify that the string is a _valid_ email address, use [`SimpleSchema.RegEx.Email` or `SimpleSchema.RegEx.EmailWithTLD`](https://github.com/aldeed/simple-schema-js#regex).  Checking if the email address actually exists is more difficult and your teaching assistant does not have an answer to that beyond sending an email to that address.

#### Is it possible to sort the data displayed on the page?
Yes, and [you have already done something like this](http://courses.ics.hawaii.edu/ics314s19/morea/mongo/experience-mongodb-shell.html).

#### Can we combine pages together?  For example, could we merge the _Add Stuff_, _Edit Stuff_, and _List Stuff_ pages to form one _Stuff_ page where we can see the existing Stuffs as we add and modify items?
This is absolutely possible; you would just have to modify the relevant _.jsx_ files.  Whether you _should_ do this is a philosophical question.  On one hand, combining all of the possible actions on a collection into a single page reduces the amount of navigation necessary, and seeing what is already in the collection may be useful when adding new items (ex. ensuring that you are not creating a duplicate of something already in the database).  On the other hand, doing this will put a lot of data on one page, resulting in a potentially confusing and cluttered user interface.

#### Is it possible to combine the two `Meteor.publish` calls in _/app/imports/startup/server/stuff.js_ so the application only uses one subscription for `Stuffs`?
It is certainly possible to combine the conditional logic so the `Stuff` subscription returns all `Stuffs` for admins and only the items that the user owns for non-admins.  The template application separates the publications because it displays the `Stuffs` differently on different pages: an admin may want to see all `Stuffs` or just the `Stuffs` that he or she is directly associated with.  

#### How can we further customize the forms?  Can we use CSS to add new styles to the form elements?
See the [documentation](https://github.com/vazco/uniforms) and [assigned reading](https://blog.meteor.com/managing-forms-in-a-meteor-react-project-with-uniforms-33d60602b43a) for Uniforms; in particular, note that you are not restricted to only text fields and dropdowns.  You may also use _/app/client/style.css_ (or any other _.css_ file that you import) to style the elements in the form. 

#### Is it possible to validate the `AutoForm` without the fake username?
Try moving the `const owner` line from `submit` to `render` and setting the `value` of the `HiddenField` to `{ owner }`.

#### _/app/imports/ui/pages/AddStuff.jsx_ defines `insertCallback` and `submit` methods with parameters, but those parameters are not provided when the method is used.  How does the method receive those parameters?
The methods are used as follows:

```
<AutoForm ref={(ref) => { this.formRef = ref; }} schema={StuffSchema} onSubmit={this.submit}>
```

```
Stuffs.insert({ name, quantity, condition, owner }, this.insertCallback);
```  

In both cases, the method is not actually called at that point: `this.submit` is set as the handler for the `onSubmit` event and `this.insertCallback` is passed as an argument to `Stuffs.insert`.  As you will recall from the functional programming module earlier in the semester, functions are data in JavaScript and so you can store them in variables, pass them as arguments to other functions, etc.  The methods are actually called at some other time within the code for `AutoForm` and `Collection#insert` respectively.

#### Are there any restrictions on the components we can create?
None that I am aware of.

#### Where does the `error` in _/app/imports/ui/pages/EditStuff.jsx_ come from?
The code referenced is:

```
(error) => (error ?
        Bert.alert({ type: 'danger', message: `Update failed: ${error.message}` }) :
        Bert.alert({ type: 'success', message: 'Update succeeded' }))
```

This is an arrow function, so `error` is the parameter for the function defined here.  The function is not being called, only defined.  For a complete look at how this function is used:

```
submit(data) {
  const { name, quantity, condition, _id } = data;
  Stuffs.update(_id, { $set: { name, quantity, condition } }, (error) => (error ?
      Bert.alert({ type: 'danger', message: `Update failed: ${error.message}` }) :
      Bert.alert({ type: 'success', message: 'Update succeeded' })));
}
```

So the arrow function (with `error` as its parameter) is passed as an argument to `Stuffs.update`.

#### Is it possible to filter the range of possible inputs based on a specific set of valid values?
If you want to restrict the range of inputs, it is best to design the form to only allow those specific inputs rather than waiting until the user submits the form to perform validation.  If the input is a string, use [a dropdown](https://react.semantic-ui.com/modules/dropdown/#types-search-selection-two); if the input is a number, use [a number field](https://www.w3schools.com/html/tryit.asp?filename=tryhtml_input_number), etc.

#### How does the Uniforms package save the form data into the database?
The `onSubmit` attribute for the `AutoForm` component indicates the code to execute when the form is submitted.  In both _/app/imports/ui/pages/AddStuff.jsx_ and _/app/imports/ui/pages/EditStuff.jsx_, `onSubmit` is set to `this.submit`, referring to a `submit` method defined within the class.  This `submit` function takes care of the database operations.

#### Can we perform more specific validations, ex. limiting the name to only alphanumeric characters?
Yes, though this would likely require you to use [regular expressions](https://blog.codinghorror.com/regular-expressions-now-you-have-two-problems/) (and not just [the constants referenced in the Simple Schema documentation](https://github.com/aldeed/simple-schema-js#regex)).

#### Does `input` prevent users from overflowing the input to gain admin access?  
No, though the type of vulnerability you are referring to would be attacked through a different route.

---

## <span id="roles">Authorization, Authentication, and Roles</span>
#### How does the application differentiate between regular and admin users?
One example of this is in _/app/imports/ui/components/Navbar.jsx_:

```
{Roles.userIsInRole(Meteor.userId(), 'admin') ? (
    <Menu.Item as={NavLink} activeClassName="active" exact to="/admin" key='admin'>Admin</Menu.Item>
) : ''}
```

This uses the ternary operator to display the **Admin** menu item if the user is an admin and `''` otherwise.  The condition here is `Roles.userIsInRole(Meteor.userId(), 'admin')`; `Roles.userIsInRole` takes the ID of the user to examine and the role to search for as its arguments.  For more details, see [the documentation for `Roles.userIsInRole`](http://alanning.github.io/meteor-roles/classes/Roles.html#method_userIsInRole).

#### Why is there no explicit check for the user being a regular user?
In the template you are using, the `roles` property is set to `['admin']` if the user is an admin user; if that is not the case, the only other option is that the user is a standard user.

#### How do you add new admin users without using _/config/settings.development.json_?
You can add the "admin" role to an existing user through a MongoDB command, ex. `db.users.update({username: "john@foo.com"}, { $set: { roles: ["admin"] }});`.  

#### Are there any roles besides "user" and "admin" in the [template](http://ics-software-engineering.github.io/meteor-application-template-react/)?
The template only provides these two roles; these roles are ultimately just strings stored in an array field in the `users` collection though, so all you need to do in order to create new roles is decide on a name for the role and add conditional logic to perform certain actions for that role.  For more information, see [the documentation for `meteor-roles`](https://github.com/alanning/meteor-roles).  

#### Are there any limits on the number of regular or admin users?
After a while your machine would probably run out of memory, but other than that there are no limits on the number of documents in a collection.

#### How can a user modify a document that he or she did not create?
Creating data does not inherently protect it from modification by others; if you want to ensure that users can only modify their own data, you would have to add in the logic for that yourself.  

#### Are there ethical concerns about having an admin user who can see all of the data on the site?
Absolutely; we have an **Ethics in Software Engineering** module later in the semester and you might also consider ICS 390 for further discussion of ethical issues in Computer Science.  

#### What prevents users from changing their own roles, ex. a regular user changing his or her role to be an admin?
Mostly the fact that not even admins can make new admins in the template application.  If you add that functionality in, you can use `AdminProtectedRoute` from _/app/imports/ui/layouts/App.jsx_ to prevent standard users from accessing pages that only admins should have access to.

#### Does Meteor support group roles for users?
Yes, though it is more accurate to write that `meteor/alanning:roles` provides this; documentation is available [here](https://github.com/alanning/meteor-roles).

#### What prevents users from accessing admin pages?  
_/app/imports/ui/layouts/App.jsx_ uses `AdminProtectedRoute` for the admin page.  `AdminProtectdRoute` redirects the user to the signin page if the current user is not an admin (the `isLogged && isAdmin`).  Consequently, the current user must be an admin to access the page even if the user has typed in the URL of the admin page manually.

---

## <span id="misc">Miscellaneous Questions</span>
#### Is Meteor used for real-life applications<sup><a href="#footnote-6">6</a></sup>?
See the [Meteor showcase](https://www.meteor.com/showcase), though the most important example for you is clearly [RadGrad](https://www.radgrad.org/).

#### What is a hook function?
See [the answers to this Stack Overflow question](https://stackoverflow.com/q/467557); this should not be confused with [hooking](https://youtu.be/HxCIYUBT5-A).

#### How can we easily know what Semantic UI tools are available to us?
See the [Semantic UI React documentation](https://react.semantic-ui.com/).

#### How difficult would it be to use another UI framework in Meteor?  
In theory, not very difficult; in the worst-case scenario where the framework cannot be installed through _npm_ it will still be possible to add the `link` tags into _/app/client/main.html_ manually.

---

## <span id="unanswered">Unanswered and Unclassified Questions</span>
#### Can Meteor be used for real-time apps involving large quantities of data (ex. multiplayer games)?

#### Is this publication and subscription model similar to how Google, Amazon, Facebook, etc. retrieve and display data?

#### Why the split between _/client_ and _/server_?  Why not make all data visible on both sides?

#### What is the relationship between _/app/client/main.html_, _/app/imports/startup/client/startup.jsx_, and _/app/imports/ui/layouts/App.jsx_?

#### How does the `<Switch>` statement work in _/app/imports/ui/layouts/App.jsx_?

#### How exactly does `ProtectedRoute` create a new route?

#### What does the `render` function do?

#### How do you generate the URL of a `Stuff`?

#### Why do we split the components on a page across multiple files in Meteor when we did everything in a single file in React?

#### If you have multiple subscriptions on a page, is it possible to wait for a set of those subscriptions to become ready before updating the page or will the page automatically update upon any change to any of the subscriptions?

#### What makes Meteor unique among all the web application frameworks out there?
~~It is the only one named Meteor.~~

#### The _Add Stuff_ and _Edit Stuff_ pages are essentially the same; why do we need two separate pages to do the same thing?
Although the pages _look_ the same, they serve different roles.  

#### What is the `model={this.props.doc}` attribute in the `AutoForm` in _/app/imports/ui/pages/EditStuff.jsx_?

---

## Footnotes
<ol>
  <li id="footnote-1">Given the author, these answers are likely to make matters even more confusing.</li>
  <li id="footnote-2">
    For example, note the judicious use of HTML to work around some of the simplifications inherent to Markdown.
  </li>
  <li id="footnote-3">
    Your teaching assistant remembers one instance where two projects both used PostgreSQL and the `Users` table in one project overwrote all the users for the other project.
  </li>
  <li id="footnote-4">
    The correct term for this is <a href="http://www.catb.org/jargon/html/C/cracker.html">cracker</a>, not <a href="http://www.catb.org/jargon/html/H/hacker.html">hacker</a>.
  </li>
  <li id="footnote-5">
    It is theoretically possible for two different passwords to have the same hash value; each character in this hashed value can be one of sixty-five characters and the bcrypt value is sixty characters long, so there are 5,953,898,114,759,757,583,498,133,232,325,552,989,993,673,976,517,266,254,986,001,716,363,076,818,883,115,493,008,517,660,200,595,855,712,890,625 possible hash values.  
  </li>
  <li id="footnote-6">As opposed to fake-life applications.</li>
</ol>
