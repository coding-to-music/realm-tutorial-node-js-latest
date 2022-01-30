# Realm Node.js CLI Tutorial

Follow along at https://docs.mongodb.com/realm/tutorial/nodejs-cli/

## Troubleshooting

The most common issue is schema mismatch due to frequent little tweaks to the
schema as you develop your app.

- Be sure to **check the logs in Realm UI** for more information as well as the console in your app.
- Purge local data by deleting any realm files (e.g. in `localRealmDb/`).
- **Restart Sync** in the Realm UI by clicking "Delete Synced Data" on the Sync page.
- Be sure to **deploy your changes** in the Realm UI.
- If your schema does not match the server, **compare the class definitions from the SDKs tab in the Realm UI** with those in the client code.
- When creating objects, make sure the **partition value of your new object matches** the partition value you opened the Realm with.

## Issues & Pull Requests

If you find an issue or have a suggestion, please let us know using the feedback
widget on the [docs site](http://docs.mongodb.com/realm/tutorial).

This repo is automatically derived from our main docs repo. If you'd like to
submit a pull request -- thanks! -- please feel free to do so at
https://github.com/mongodb/docs-realm/ (see the tutorial/ subdirectory).

# MongoDB Realm Node.js CLI Tutorial

Follow along at https://docs.mongodb.com/realm/tutorial/nodejs-cli/

## Troubleshooting

The most common issue is schema mismatch due to frequent little tweaks to the
schema as you develop your app.

- Be sure to **check the logs in Realm UI** for more information as well as the console in your app.
- Purge local data by deleting any realm files (e.g. in `localRealmDb/`).
- **Restart Sync** in the Realm UI by clicking "Delete Synced Data" on the Sync page.
- Be sure to **deploy your changes** in the Realm UI.
- If your schema does not match the server, **compare the class definitions from the SDKs tab in the Realm UI** with those in the client code.
- When creating objects, make sure the **partition value of your new object matches** the partition value you opened the Realm with.

## Issues & Pull Requests

If you find an issue or have a suggestion, please let us know using the feedback
widget on the [docs site](http://docs.mongodb.com/realm/tutorial).

This repo is automatically derived from our main docs repo. If you'd like to
submit a pull request -- thanks! -- please feel free to do so at
https://github.com/mongodb/docs-realm/ (see the tutorial/ subdirectory).

Node.js CLI Tutorialicons/link.png

# Overview

In this tutorial, you will use Node.js to create a task tracker command line interface (CLI) that allows users to:

- Register themselves with email and password.
- Sign in to their account with their email and password.
- View a list of projects they are a member of.
- View, create, modify, and delete tasks in projects.
- View a list of team members in their project.
- Add and remove team members to their project.
- This tutorial should take around 30 minutes to complete.

NOTE
Check Out the Quick Start
If you prefer to explore on your own rather than follow a guided tutorial, check out the Quick Start. It includes copyable code examples and the essential information that you need to set up a MongoDB Realm application.

## Prerequisites

Before you begin, ensure you have:

- Node.js installed.
- Set up the backend. https://docs.mongodb.com/realm/tutorial/realm-app/#std-label-tutorial-task-tracker-create-realm-app

Once you're set up with these prerequisites, you're ready to start the tutorial.

## A. Clone the Client App Repository

We've already put together a task tracker CLI application that has most of the code you'll need. You can clone the client application repository directly from GitHub:

We will get the latest branch, not the start branch

```java
git clone https://github.com/mongodb-university/realm-tutorial-node-js.git realm-tutorial-node-js-latest
cd realm-tutorial-node-js-latest
```

Now you can go to the end of this tutorial and just run

```java
npm install
node index.js.
```

The rest of this document is from the Start tutorial https://github.com/coding-to-music/realm-tutorial-node-js

TIP
The start branch is an incomplete version of the app that we will complete in this tutorial. To view the finished app, check out the final branch, install dependencies, and update realmAppId in the config.js to point to your Realm app ID.

In your terminal, run the following command to install its dependencies:

```java
npm install
```

## B. Explore the App Structure & Components

This application has a flat project structure: all of the files are in the root directory. Open a text editor to explore the directory and files. In this tutorial, we'll be focusing on 5 files: config.js, users.js, tasks.js, team.js, projects.js. The other files provide the underlying structure for the CLI. The following table describes the role of important files in this project:

```java
- File        Purpose
- config.js   Provides a single location for configuration data. You will put your Realm app ID here.
- index.js    The entry point for the app. Creates the Realm App that you app will use throughout its lifecycle and displays the initial logon screen.
- main.js     Displays the main menu of choices. Users can view a list of projects they are a member of or select a project.
- output.js   Responsible for displaying text in the terminal window.
- tasks.js    Handles all task-related communication between the CLI and Realm. The methods for listing, creating, editing, and deleting tasks live here.
- schemas.js  Contains the schema definitions for the collections used in this project.
- users.js    Handles Realm user authentication, including logging in, registering a new user, and logging out.
- team.js     Handles the team member related communication between the CLI and Realm. The methods for listing, adding, and removing team members are contained in this file.
- projects.js Handles project retrieval and listing.
```

## C. Connect to Your MongoDB Realm App

To get the app working with your backend, you first need to add your Realm App ID to the config.js file. The config.js module exports a single property, realmAppId, which is currently set to "TODO":

```java
exports.realmAppId = "<your Realm app ID here>";
```

Change this value to your Realm App ID.

NOTE
To learn how to find your MongoDB Realm appId, see the Find Your App Id doc.

Once you have made that change, you now need to complete the code needed to open a realm. In index.js, find the openRealm function. Replace the TODO line with a line of code that opens a realm and assigns it to the realm property. It will look like this:

```java
async function openRealm(partitionKey) {
  const config = {
    schema: [schemas.TaskSchema, schemas.UserSchema, schemas.ProjectSchema],
    sync: {
      user: users.getAuthedUser(),
      partitionValue: partitionKey,
    },
  };
  return Realm.open(config);
}
```

Now that you have implemented the openRealm function, you will now need to complete the code that retrieves the realm. In index.js, find the getRealm function. It will look like this:

```java
async function getRealm(partitionKey) {
  if (realms[partitionKey] == undefined) {
    realms[partitionKey] = openRealm(partitionKey);
  }
  return realms[partitionKey];
}
```

At this point, your app is pointing to your backend and opens a connection to it when you start the app. However, users cannot log in yet, so let's update that code next.

## D. Enable authentication

In the users.js file, we have a logIn function that prompts the user for an email address and password, and then, within a try-catch block, creates an emailPassword credential and passes it to the Realm logIn() method.

Find the the logIn function and add the following code to create a emailPassword credential and call the logIn() method.

```java
async function logIn() {
  const input = await inquirer.prompt([
    {
      type: "input",
      name: "email",
      message: "Email:",
    },
    {
      type: "password",
      name: "password",
      message: "Password:",
      mask: "*",
    },
  ]);
  try {
    const credentials = Realm.Credentials.emailPassword(
      input.email,
      input.password
    );
    const user = await app.logIn(credentials);
    if (user) {
      output.result("You have successfully logged in as " + app.currentUser.id);
      return main.mainMenu();
    } else {
      output.error("There was an error logging you in");
      return logIn();
    }
  } catch (err) {
    output.error(err.message);
    return logIn();
  }
}
```

## E. Define the Object Schemas

In order to model data in the database, we need to define some schemas for the data we store. In schemas.js:

### TaskSchema

The task model contains information about a user's tasks. Find the TaskSchema code and replace it with the following:

```java
const TaskSchema = {
  name: "Task",
  properties: {
    _id: "objectId",
    name: "string",
    owner: "string?",
    status: "string",
  },
  primaryKey: "_id",
};
```

### UserSchema

The user model represents a user and their permissions. Find the UserSchema code and replace it with the following:

```java
const UserSchema = {
  name: "User",
  properties: {
    _id: "string",
    memberOf: "Project[]",
    name: "string",
  },
  primaryKey: "_id",
};
```

### ProjectSchema

We also need a project model to represent the projects a user is a member of. Find the ProjectSchema code and replace it with the following:

```java
const ProjectSchema = {
  name: "Project",
  embedded: true,
  properties: {
    name: "string?",
    partition: "string?",
  },
};
```

## F. Implement the CRUD methods

In the tasks.js and projects.js files, there are a number of functions to handle typical CRUD functionality: getTasks, getTask, createTask, deleteTask, editTask, changeStatus, and getProjects. Each of these functions (except getTasks and getProjects) prompts the user for input and then makes the appropriate call to Realm. Your job is to implement the calls to Realm. The following list provides guidance on how to complete this task for each function.

In tasks.js:

### getTasks

To get all objects, call the objects() method and pass in the name of the collection:

```java
exports.getTasks = async (partition) => {
  const realm = await index.getRealm(partition);
  const tasks = realm.objects("Task");
  output.header("MY TASKS:");
  output.result(JSON.stringify(tasks, null, 2));
};
```

### getTask

In the Tasks collection, a task's id field is the primary key, so we call the objectForPrimaryKey() function to get a task by its Id.

```java
exports.getTask = async (partition) => {
  const realm = await index.getRealm(partition);
  try {
    const task = await inquirer.prompt([
      {
        type: "input",
        name: "id",
        message: "What is the task ID (_id)?",
      },
    ]);
    let result = realm.objectForPrimaryKey("Task", new bson.ObjectID(task.id));
    if (result !== undefined) {
      output.header("Here is the task you requested:");
      output.result(JSON.stringify(result, null, 2));
    }
  } catch (err) {
    output.error(err.message);
  }
};
```

### createTask

Whenever we modify an object in realm, we must do so within a transaction. The write() method takes care of transaction handling for us. So, within the write function, we call the create() function, passing in all of the required properties:

```java
exports.createTask = async (partition) => {
  const realm = await index.getRealm(partition);
  try {
    output.header("*** CREATE NEW TASK ***");
    const task = await inquirer.prompt([
      {
        type: "input",
        name: "name",
        message: "What is the task text?",
      },
      {
        type: "rawlist",
        name: "status",
        message: "What is the task status?",
        choices: ["Open", "In Progress", "Closed"],
        default: function () {
          return "Open";
        },
      },
    ]);
    let result;
    realm.write(() => {
      result = realm.create("Task", {
        _id: new bson.ObjectID(),
        _partition: partition,
        name: task.name,
        status: task.status.replace(/\s/g, ""), // Removes space from "In Progress",
      });
    });
    output.header("New task created");
    output.result(JSON.stringify(result, null, 2));
  } catch (err) {
    output.error(err.message);
  }
};
```

NOTE
The write function replaces the need to call the beginTransaction(), commitTransaction(), and cancelTransaction() methods.

### deleteTask

Deleting objects must also take place within a transaction. As with modifying an object, we'll use the write() function to handle the transaction for us. We'll first call the objectForPrimaryKey method to get the specific we want to delete and then the delete() function on that task:

```java
exports.deleteTask = async (partition) => {
  const realm = await index.getRealm(partition);
  output.header("DELETE A TASK");
  const answers = await inquirer.prompt([
    {
      type: "input",
      name: "id",
      message: "What is the task ID (_id)?",
    },
    {
      type: "confirm",
      name: "confirm",
      message: "Are you sure you want to delete this task?",
    },
  ]);
  if (answers.confirm) {
    let task = realm.objectForPrimaryKey("Task", new bson.ObjectID(answers.id));
    realm.write(() => {
      realm.delete(task);
      output.result("Task deleted.");
    });
    return;
  }
};
```

### modifyTask

This function is called by both the editTask and changeStatus functions. Like the createTask and deleteTask methods, when you change an object, you do so within a transaction. Other than that, though, there is no specific call to a Realm API to change an object. Rather, you change the local object and Sync ensures the object is updated on the server.

```java
async function modifyTask(answers, partition) {
  const realm = await index.getRealm(partition);
  let task;
  try {
    realm.write(() => {
      task = realm.objectForPrimaryKey("Task", new bson.ObjectID(answers.id));
      task[answers.key] = answers.value;
    });
    return JSON.stringify(task, null, 2);
  } catch (err) {
    return output.error(err.message);
  }
}
```

NOTE
To learn more about Realm Sync, see Sync Overview.

### In projects.js:

### getProjects

As defined by our data model, projects are an embedded object of the users object. To get all projects the user is a part of, we need to get the current user. Then we'll use the refreshCustomData method to get the current user's memberOf property.

```java
async function getProjects() {
  const user = users.getAuthedUser();
  try {
    const { memberOf: projects } = await user.refreshCustomData();
    // Make sure that the user object has been created
    if (!projects) {
      output.error("The user object hasn't been created yet. Try again soon.");
      throw new Error("No projects for user");
    }
    return projects;
  } catch (err) {
    output.error("There was a problem accessing custom user data");
  }
}
```

NOTE
How Do We Know Which Projects a User Can Access?
The backend you imported makes exactly one custom user data object for each user upon signup. This custom user data object contains a list of partitions a user can read and a list of partitions a user can write to.

The backend is set up so that every user has read-only access to their own custom user data object. The backend also has functions to add and remove access to projects, which we will use later when we add the Manage Team view.

By managing the custom user data object entirely on the backend and only providing read-only access on the client side, we prevent a malicious client from granting themselves arbitrary permissions.

## G. Use Realm Functions

In the team.js file, there are functions that rely on Realm functions. Realm functions allow you to execute server-side logic for your client applications. Each of the following functions require you to implement the calls to Realm.

### getTeamMembers

To get all team members, call the getMyTeamMembers Realm function using the User.functions method.

```java
exports.getTeamMembers = async () => {
  const currentUser = users.getAuthedUser();
  try {
    const teamMembers = await currentUser.functions.getMyTeamMembers();
    output.result(JSON.stringify(teamMembers, null, 2));
  } catch (err) {
    output.error(err.message);
  }
};
```

### addTeamMember

This function prompts the user for the email of the new team member. You will need to call the addTeamMember Realm function and pass it the email parameter.

```java
exports.addTeamMember = async () => {
  try {
    output.header("*** ADD A TEAM MEMBER ***");
    const currentUser = users.getAuthedUser();
    const { email } = await inquirer.prompt([
      {
        type: "input",
        name: "email",
        message: "What is the new team member's email address?",
      },
    ]);
    await currentUser.functions.addTeamMember(email);
    output.result("The user was added to your team.");
  } catch (err) {
    output.error(err.message);
  }
};
```

### removeTeamMember

This functions prompts the user for the email of the team member they would like to remove from their project. You will need to call the removeTeamMember Realm function and pass it the email parameter.

```java
exports.removeTeamMember = async () => {
  const currentUser = users.getAuthedUser();
  const teamMembers = await currentUser.functions.getMyTeamMembers();
  const teamMemberNames = teamMembers.map((t) => t.name);
  try {
    output.header("*** REMOVE A TEAM MEMBER ***");
    const { selectedTeamMember } = await inquirer.prompt([
      {
        type: "rawlist",
        name: "selectedTeamMember",
        message: "Which team member do you want to remove?",
        choices: [...teamMemberNames, new inquirer.Separator()],
      },
    ]);
    let result = await currentUser.functions.removeTeamMember(
      selectedTeamMember
    );
    output.result("The user was removed from your team.");
  } catch (err) {
    output.error(err.message);
  }
};
```

## H. Run and Test

Once you have completed the code, you should run the app and check functionality.

Open a terminal window and change to your app's directory.
Run the following commands to install all of the dependencies and start the app:

```java
npm install
node index.js
```

Your terminal window will clear and you will see the initial menu prompting you to log in or register as a new user:

### This is what I am seeing

```java
8888888b.                   888                    88888888888                888
888   Y88b                  888                        888                    888
888    888                  888                        888                    888
888   d88P .d88b.   8888b.  888 88888b.d88b.           888   8888b.  .d8888b  888  888 .d8888b
8888888P" d8P  Y8b     "88b 888 888 "888 "88b          888      "88b 88K      888 .88P 88K
888 T88b  88888888 .d888888 888 888  888  888          888  .d888888 "Y8888b. 888888K  "Y8888b.
888  T88b Y8b.     888  888 888 888  888  888          888  888  888      X88 888 "88b      X88
888   T88b "Y8888  "Y888888 888 888  888  888          888  "Y888888  88888P' 888  888  88888P'




*** WELCOME ***


Please log in to your Realm account or register as a new user.

? What do you want to do? Register as a new user

WELCOME, NEW USER

? Email: first@example.com
? Password: *****

 ❗
password must be between 6 and 128 characters
 ❗


WELCOME, NEW USER

? Email: first@example.com
? Password: **********
You have successfully created a new Realm user and are now logged in.

? What would you like to do? Show all of my projects

MY PROJECTS:

[
  {
    "name": "My Project",
    "partition": "project=61f5efa9eadae6363c9a6222"
  }
]

? What would you like to do? Select a project
? Which project do you want to access? My Project
? What would you like to do? Show all of my tasks

MY TASKS:

[]

? What would you like to do? Create a task

*** CREATE NEW TASK ***

? What is the task text? olajsdfapsdfjasdfadsf
? What is the task status? In Progress

New task created

{
  "_id": "61f5f0374ce2da4cb431eeed",
  "name": "olajsdfapsdfjasdfadsf",
  "owner": null,
  "status": "InProgress"
}

? What would you like to do? Show all of my tasks

MY TASKS:

[
  {
    "_id": "61f5f0374ce2da4cb431eeed",
    "name": "olajsdfapsdfjasdfadsf",
    "owner": null,
    "status": "InProgress"
  }
]

? What would you like to do? Create a task

*** CREATE NEW TASK ***

? What is the task text? sdfjaspdfj
? What is the task status? Open

New task created

{
  "_id": "61f5f0714ce2da4cb431eeee",
  "name": "sdfjaspdfj",
  "owner": null,
  "status": "Open"
}

? What would you like to do? Create a task

*** CREATE NEW TASK ***

? What is the task text? llsdfgdsflgj
? What is the task status? Open

New task created

{
  "_id": "61f5f1504ce2da4cb431eeef",
  "name": "llsdfgdsflgj",
  "owner": null,
  "status": "Open"
}

? What would you like to do? Manage my team
? What would you like to do?
  1) Get my team members
  2) Add a team member
  3) Remove a team member
  4) Return to project
  5) Return to main menu
  6) Log out / Quit
(Move up and down to reveal more choices)
  Answer:
>> Please enter a valid index
```

### Initial menu

If the app builds successfully, here are some things you can try in the app:

- Create a user with email first@example.com
- Explore the app, then log out.
- Start up the app again and register as another user with email second@example.com
- Select second@example.com's project
- Add, update, and remove some tasks
- Select the "Manage Team" menu option
- Add first@example.com to your team
- Log out and log in as first@example.com
- See two projects in the projects list
- Navigate to second@example.com's project
- Collaborate by adding, updating, and removing some new tasks

TIP
If something isn't working for you, you can check out the final branch of this repo to compare your code with our finished solution.

## What's Next?

You just built a functional task tracker web application built with MongoDB Realm. Great job!

Now that you have some hands-on experience with MongoDB Realm, consider these options to keep practicing and learn more:

- Extend the task tracker app with additional features. For example, you could:

- allow users to log in using another authentication provider
- Follow another tutorial to build a mobile app for the task tracker. We have task tracker tutorials for the following platforms:

  - iOS (Swift)
  - Android (Kotlin)
  - React Native (JavaScript)
  - Web with React and GraphQL (Javascript)

- Dive deeper into the docs to learn more about MongoDB Realm. You'll find information and guides on features like:

  - Serverless functions that handle backend logic and connect your app to external services. You can call functions from a client app, either directly or as a custom GraphQL resolver.
  - Triggers and HTTPS Endpoints, which automatically call functions in response to events as they occur. You can define database triggers which respond to changes in your data, authentication triggers which respond to user management and authentication events, and scheduled triggers which run on a fixed schedule.
  - Built-in authentication providers and and user management tools. You can allow users to log in through multiple methods, like API keys and Google OAuth, and associate custom data with every user.
