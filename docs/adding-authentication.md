# Adding Authentication

Now that we have a simple React app, let’s let users sign up and sign in to our app. They won’t be able to do anything yet, but it will be helpful to have this in place so that when we add in the ability to query our backend API, we’ll know which users are accessing our system.

## 1. Initializing Amplify

1.1\. Install Amplify CLI tool https://github.com/aws-amplify/amplify-cli

``` bash
npm install -g @aws-amplify/cli
```

1.2\. Adde the **aws-amplify** and **aws-amplify-react** modules to your React application.

``` bash
cd ~/environment/react-my-todos/
npm install aws-amplify @aws-amplify/ui-react
```

1.3\. Initialize the Amplify project and use the following values.

``` bash
amplify init
```

? Enter a name for the project **react-my-todos**

? Enter a name for the environment **prod**

? Choose your default editor: **Visual Studio Code**

? Choose the type of app that you're building **javascript**

Please tell us about your project

? What javascript framework are you using **react**

? Source Directory Path:  **src**

? Distribution Directory Path: **build**

? Build Command:  **npm run-script build**

? Start Command: **npm run-script start**

Using default provider  awscloudformation


For more information on AWS Profiles, see:

https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html


? Do you want to use an AWS profile? **Yes**

? Please choose the profile you want to use **default**

![Amplify init](images/amplify-init.png)

## 2. Adding Amplify authentication

2.1\. Using the Amplify CLI add authentication to the application and use the following values.

``` bash
amplify add auth
```

 Do you want to use the default authentication and security configuration? **Default configuration**

 Warning: you will not be able to edit these selections. 

 How do you want users to be able to sign in? **Username**

 Do you want to configure advanced settings? **No, I am done.**

![Amplify add auth](images/amplify-add-auth.png)

2.2\. Push to create these changes in the cloud.

``` bash
amplify push
```

? Are you sure you want to continue? **Yes**

⠏ Updating resources in the cloud. This may take a few minutes...

![Amplify add auth push](images/amplify-add-auth-push.png)

!!! info
    The Amplify CLI will take care of provisioning the appropriate cloud resources and it will update src/aws-exports.js with all of the configuration data we need to be able to use the cloud resources in our app.

**Congratulations! You’ve just created a serverless backend for user registration and authorization capable of scaling to millions of users with Amazon Cognito.**

## 3. Configure React application with authentication

3.1\. Update the contents of **src/App.js** with the following.

``` javascript hl_lines="1 11 12 13 14 15 26 27 28 29 30 31 35 36 37 38 61"
import React, { useEffect } from "react";
import { makeStyles } from "@material-ui/core/styles";
import CssBaseline from "@material-ui/core/CssBaseline";
import Container from "@material-ui/core/Container";
import IndexTodos from "./components/indexTodos.js";
import AddTodo from "./components/addTodo.js";
import EditTodo from "./components/editTodo.js";
import Paper from "@material-ui/core/Paper";
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";

import { Amplify, Auth } from 'aws-amplify';
import { withAuthenticator, AmplifyGreetings  } from '@aws-amplify/ui-react'
import { AuthState, onAuthUIStateChange } from "@aws-amplify/ui-components";
import awsExports from "./aws-exports";
Amplify.configure(awsExports);

const useStyles = makeStyles(theme => ({
  root: {
    padding: theme.spacing(5, 2)
  }
}));

function App() {
  const classes = useStyles();

  useEffect(() => {
    return onAuthUIStateChange((nextAuthState, authData) => {
      setAuthState(nextAuthState);
      setUser(authData);
    });
  }, []);
  
  return (
    <div className="App">
      { authState === AuthState.SignedIn && user ?
        (<AmplifyGreetings username={authState === AuthState.SignedIn && user ? user.username : ""}></AmplifyGreetings>)
       : ""
      }
      <Router>
        <React.Fragment>
          <CssBaseline />
          <Container
            fixed
            maxWidth="sm"
            style={{ height: "100vh", paddingTop: 20 }}
          >
            <Paper className={classes.root}>
              <Switch>
                <Route path="/" exact component={IndexTodos} />
                <Route path="/addTodo" component={AddTodo} />
                <Route path="/editTodo/:idTodo" component={EditTodo} />
              </Switch>
            </Paper>
          </Container>
        </React.Fragment>
      </Router>
    </div>
  );
}

export default withAuthenticator(App);
```

3.2\. Go back to your application running and create an account in the app by providing a username, password, and a valid email address (to receive a confirmation code at).

![React Auth](images/react-auth.png)

**What we changed in App.js**

* Imported and configured the AWS Amplify JS library
* Imported the withAuthenticator higher order component from aws-amplify-react
* Wrapped the App component using withAuthenticator

3.3\. Check your email. You should have received a confirmation link and after the validation you should then be able to log in with the username and password you entered during sign up.

3.3\. Once you sign in, the form disappears and you can see our App component rendered below a header bar that contains your username and a **Sign Out** button.

![React sign in](images/react-sign-in.png)