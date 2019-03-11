# ICS 314 Meteor FAQs

This page is intended to answer questions about Meteor for ICS 314 Spring 2019<sup><a href="#footnote-1">1</a></sup>.  This also serves the secondary purpose of providing an example of how to use GitHub Pages<sup><a href="#footnote-2">2</a></sup>.

## Table of Contents
1. <a href="#forms">Forms</a>
2. <a href="#mongodb">MongoDB</a>
3. <a href="#project-hierarchy">Project Hierarchy</a>
4. <a href="#rendering-data">Rendering Data</a>
5. <a href="#roles">Roles</a>

## <span id="forms">Forms</span>
#### How does error checking work?
This course uses [Uniforms](https://github.com/vazco/uniforms) in conjunction with [Simple Schema](https://github.com/aldeed/simple-schema-js) to accomplish this; see the documentation for those tools for more details.  

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

#### How can we enforce password constraints (ex. a minimum length, must contain numbers, etc.)?
See the [Simple Schema documentation](https://github.com/aldeed/simple-schema-js#schema-rules); `min` is easy enough for the minimum length, though you may have to write a separate function to perform [custom validation](https://github.com/aldeed/simple-schema-js#custom-field-validation) or use regular expressions.

#### How can we allow users to change their passwords?
See the documentation for [`Accounts.changePassword`](https://docs.meteor.com/api/passwords.html#Accounts-changePassword). 

## <span id="mongodb">MongoDB</span>
#### How does the application remember the user accounts and other data even after terminating the Meteor process?
When you use a computer, you have to save your work before shutting down the computer.  That data is stored in a file and you can open that file later to resume your work.  Databases serve the same purpose: your application stores data in the database so the data is preserved even after the application has closed.

#### Where is all of the data stored?  
The technically correct answer is _/app/.meteor/local/db_, though that is probably not of direct use to you since the data is not stored in a human-readable format.  Aside from a tired attempt at sarcasm, this does show that the data is in the project directory, so you do not have to worry about data from one project intermingling with data from another project<sup><a href="#footnote-4">4</a></sup>.

The more useful answer is that you do not have to worry about where the data is stored: MongoDB takes care of preserving the data for you.

#### How are passwords stored?  What prevents an attacker<sup><a href="#footnote-3">3</a></sup> from accessing those passwords?
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

## <span id="rendering-data">Rendering Data</span>
#### Is it possible to sort the data displayed on the page?
Yes, and [you have already done something like this](http://courses.ics.hawaii.edu/ics314s19/morea/mongo/experience-mongodb-shell.html).

#### Can we combine pages together?  For example, could we merge the _Add Stuff_, _Edit Stuff_, and _List Stuff_ pages to form one _Stuff_ page where we can see the existing Stuffs as we add and modify items?
This is absolutely possible; you would just have to modify the relevant _.jsx_ files.  Whether you _should_ do this is a philosophical question.  On one hand, combining all of the possible actions on a collection into a single page reduces the amount of navigation necessary, and seeing what is already in the collection may be useful when adding new items (ex. ensuring that you are not creating a duplicate of something already in the database).  On the other hand, doing this will put a lot of data on one page, resulting in a potentially confusing and cluttered user interface.

## Project Hierarchy
#### When should we use _.jsx_ files instead of _.js_ files and vice versa?
In general, use _.jsx_ whenever you use (React) components in the file and _.js_ otherwise.

## <span id="roles">Roles</span>
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

## Footnotes
<ol>
  <li id="footnote-1">Given the author, these answers are likely to make matters even more confusing.</li>
  <li id="footnote-2">
    For example, note the judicious use of HTML to work around some of the simplifications inherent to Markdown.
  </li>
  <li id="footnote-3">
    The correct term for this is <a href="http://www.catb.org/jargon/html/C/cracker.html">cracker</a>, not <a href="http://www.catb.org/jargon/html/H/hacker.html">hacker</a>.
  </li>
  <li id="footnote-4">
    Your teaching assistant remembers one instance where two projects both used PostgreSQL and the `Users` table in one project overwrote all the users for the other project.
  </li>
  <li id="footnote-5">
    It is theoretically possible for two different passwords to have the same hash value; each character in this hashed value can be one of sixty-five characters and the bcrypt value is sixty characters long, so there are 5,953,898,114,759,757,583,498,133,232,325,552,989,993,673,976,517,266,254,986,001,716,363,076,818,883,115,493,008,517,660,200,595,855,712,890,625 possible hash values.  
  </li>
</ol>
