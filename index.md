# ICS 314 Meteor FAQs

This page is intended to answer questions about Meteor for ICS 314 Spring 2019<sup><a href="#footnote-1">1</a></sup>.  This also serves the secondary purpose of providing an example of how to use GitHub Pages<sup><a href="#footnote-2">2</a></sup>.

## Table of Contents
1. <a href="#forms">Forms</a>
2. <a href="#mongodb">MongoDB</a>
3. <a href="#rendering-data">Rendering Data</a>
4. <a href="#roles">Roles</a>

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

## <span id="mongodb">MongoDB</span>
#### How are passwords stored?  What prevents an attacker<sup><a href="#footnote-3">3</a></sup> from accessing those passwords?
If we look at the `users` collection in the MongoDB console, we see:

```
meteor:PRIMARY> db.users.find()
{ "_id" : "RvkbtmXXDoRDYeohc", "createdAt" : ISODate("2019-03-11T01:48:08.812Z"), "services" : { "password" : { "bcrypt" : "$2a$10$mS4VO9apDZC.Ul6U1YtfHOtgsY8JwVlCNq7muej4GgCTGTRyiW9K6" } }, "username" : "admin@foo.com", "emails" : [ { "address" : "admin@foo.com", "verified" : false } ], "roles" : [ "admin" ] }
{ "_id" : "Bf2XizdLC5fRq4BCk", "createdAt" : ISODate("2019-03-11T01:48:09.014Z"), "services" : { "password" : { "bcrypt" : "$2a$10$OZh60hVYwNNBVJ.SZ4q6oegTEHdFRS4SSpDKbNMTxSZIa7y/pnE7." }, "resume" : { "loginTokens" : [ { "when" : ISODate("2019-03-11T02:24:40.264Z"), "hashedToken" : "Oz02OPH48WBQeyIJ3M6nIztlYHfok6V/uWBG9VDPuSM=" } ] } }, "username" : "john@foo.com", "emails" : [ { "address" : "john@foo.com", "verified" : false } ] }
```

In particular, note that the `password` field for the first user has a value of `{ "bcrypt" : "$2a$10$mS4VO9apDZC.Ul6U1YtfHOtgsY8JwVlCNq7muej4GgCTGTRyiW9K6" }`.  This is very clearly not the default password provided in _/config/settings.development.json_.  Storing passwords as plain text is very much a bad idea; instead, we apply a hash function to the password and store that hashed value.  When someone attempts to log in as the user, we then use the same hash function.  If the hashed value of the input matches the value stored in the database, then we are reasonably certain that the password is correct<sup><a href="#footnote-4">4</a></sup>; otherwise we know we can reject the login.

## <span id="rendering-data">Rendering Data</span>
#### Is it possible to sort the data displayed on the page?
Yes, and [you have already done something like this](http://courses.ics.hawaii.edu/ics314s19/morea/mongo/experience-mongodb-shell.html).

## <span id="roles">Roles</span>
#### How does the application differentiate between regular and admin users?
One example of this is in _/app/imports/ui/components/Navbar.jsx_:

```
{Roles.userIsInRole(Meteor.userId(), 'admin') ? (
    <Menu.Item as={NavLink} activeClassName="active" exact to="/admin" key='admin'>Admin</Menu.Item>
) : ''}
```

This uses the ternary operator to display the **Admin** menu item if the user is an admin and `''` otherwise.  The condition here is `Roles.userIsInRole(Meteor.userId(), 'admin')`; `Roles.userIsInRole` takes the ID of the user to examine and the role to search for as its arguments.  For more details, see [the documentation for `Roles.userIsInRole`](http://alanning.github.io/meteor-roles/classes/Roles.html#method_userIsInRole).

#### How do you add new admin users without using _/config/settings.development.json_?
You can add the "admin" role to an existing user through a MongoDB command, ex. `db.users.update({username: "john@foo.com"}, { $set: { roles: ["admin"] }});`.  

#### Are there any roles besides "user" and "admin" in the [template](http://ics-software-engineering.github.io/meteor-application-template-react/)?
The template only provides these two roles; these roles are ultimately just strings stored in an array field in the `users` collection though, so all you need to do in order to create new roles is decide on a name for the role and add conditional logic to perform certain actions for that role.  For more information, see [the documentation for `meteor-roles`](https://github.com/alanning/meteor-roles).  

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
    It is theoretically possible for two different passwords to have the same hash value; each character in this hashed value can be one of sixty-five characters and the bcrypt value is sixty characters long, so there are 5,953,898,114,759,757,583,498,133,232,325,552,989,993,673,976,517,266,254,986,001,716,363,076,818,883,115,493,008,517,660,200,595,855,712,890,625 possible hash values.  
  </li>
</ol>
