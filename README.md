
# Securely Manage access to your Applications Sensitive Data on IBM Cloud

Ensuring that the correct people have the approved access when they need it can be difficult when you are coding your application. 

In this workshop we will walk you through how you can grant access to specific resources by defining runtime actions and assigning role-based permissions to your users using App ID. You will also learn how the sample application validates users' scopes to provide a different experience according to their role.


![image](https://user-images.githubusercontent.com/1867657/135224421-963787fe-111c-4535-846e-5c2835c62116.png)


# Pre-Requisite:

Before you can start creating roles and scopes, you need to be sure that you have the following prerequisites:

- An instance of App ID: We will see it through IBM Cloud
- An application: I will provide sample app to you.
- App users. For this Tutorial, we will creat the users Bob, Peter, and Ted in Cloud Directory.

## Step 1: Sign-up/Login to IBM Cloud

1.1- If you are an existing user please this link to Login: https://ibm.biz/appsecurity

And if you are not, don't worry! We have got you covered! There are 3 steps to create your account on IBM Cloud: 

1.2- Put your email and password. 

1.3- You get a verification link with the registered email to verify your account. 

1.4- Fill the personal information fields. 

** Please make sure you select the country you are in when asked at any step of the registration process.

## Step 2: Create an instance of AppID service

2.1- In the search bar type "App ID", click the instance from search result, it will take you to a new window. Click create botton to start your App ID instance
![image](https://user-images.githubusercontent.com/16270682/134592823-3f5f8eda-253c-422a-b27c-04b72ebc5704.PNG)

2.2- Click create botton to start your App ID instance

2.3 - Download the sample app From workshop's [Git Repository](https://github.com/IBMDeveloperMEA/Securely-Manage-access-to-your-Applications-Sensitive-Data-on-IBM-Cloud) click on code and download zip file. Unzip the file to a path of your choice. 

You can also use this link to [Download the sample app.](https://github.com/ibm-cloud-security/appid-sample-code-snippets/tree/master/access-control-demo-apps)

Add localhost:3000/* to your list of allowed redirect URIs. You can edit your allowed redirect URIs on the **Manage authentication > Authentication settings** page of the App ID dashboard.

## Step 3: Creating Scopes and Roles

3.1. Now that App ID is configured and I have the users that we need, let's walk through creating scopes and roles.
Register your application with App ID by going to **Applications > Add Application.** I created an application called Elevators and I defined the scopes elevator.call, elevator.stop, and elevator.service to represent the specific operations that the different roles can perform:

![image](https://user-images.githubusercontent.com/1867657/135224994-c6d38d8b-877a-4414-a69f-6d913603e0d1.png)

3.2. Create your roles by going to **Roles and profiles > Roles > Create** role. I created the roles Caller and Technician. I added only the elevator.call scope to the Caller role. 

![image](https://user-images.githubusercontent.com/1867657/135225011-c9dd4643-981a-41a4-a10d-2ba4afd341b5.png)

While for the Technician role, I gave the ability to complete more operations by adding the scopes elevator.call, elevator.stop, and elevator.service: 

3.3. Assign the roles to specific application users by going to **Roles and profiles > User profiles.** Then, choose the user that you want to assign the role to and click the **More options menu > Assign role.** I assigned the Technician role to Ted and the Caller role to Peter. In the following image, you can see what it looks like when I assign the Technician role to Ted: 

![image](https://user-images.githubusercontent.com/1867657/135225122-a1661f7d-896d-45e6-a296-7f817980c799.png)

Note: I used the credentials of my elevator application in the configuration of my sample applications in localdev-config.json. To grab your credentials, open your application in the Applications page of the App ID dashboard.

## Step 4: Configuring Your App's Endpoints

After creating the roles and assigning them to users, I want my application to verify the users' scopes when performing certain operations. To do this, I will define some endpoints on my application that will either use App ID's WebApp Strategy or send a request to backend endpoints that I will also define, which will perform the validation by using App ID's API Strategy. 

## Step 5 WebAppStrategy (Web app endpoints)

Using the elevator app as an example, you can see how I secure the following endpoint by using the hasScope method in WebAppStrategy to check whether the access token contains the elevator.service and elevator.stop scopes:

```
// This endpoint will return true if we should show the technician menu.
// we show this menu if request's access token contains the scopes "elevator.stop" and "elevator.service".
app.get("/protected/api/shouldShowTechnicianMenu", (req, res) => {
 res.send(WebAppStrategy.hasScope(req, 'elevator.service elevator.stop')); });
```
 
These three endpoints call backend server endpoints to validate that the access token contains the elevator.call, elevator.service, and elevator.stop scopes respectively:

```

// Call protected endpoint on backend, where we will check if the access token has the "elevator.call" scope.
         app.get("/protected/callElevator", async function(req, res) {
           const options = {
           url: 'http://localhost:1234/protected/callElevator',
           headers: {
           'Authorization': 'Bearer ' + req.session[WebAppStrategy.AUTH_CONTEXT].accessToken
          }
         };
 try {
  await request(options);
  res.send('The elevator will arrive shortly.');
 }catch(e) {
  res.send("You don't have permissions to perform this action.");
 }
});
 
 
// Call protected endpoint on backend, where we will check if the access token has the "elevator.service" scope.
app.get("/protected/serviceMode", async function(req, res) {
 const options = {
  url: 'http://localhost:1234/protected/serviceMode',
  headers: {
   'Authorization': 'Bearer ' + req.session[WebAppStrategy.AUTH_CONTEXT].accessToken
  }
 };
 try {
  await request(options);
  res.send("Elevator is on service mode.");
 }catch(e) {
  res.send("You don't have permissions to perform this action.");
 }
});
 
 
// Call protected endpoint on backend, where we will check if the access token has the "elevator.stop" scope.
app.get("/protected/stopElevator", async function(req, res) {
 const options = {
  url: 'http://localhost:1234/protected/stopElevator',
  headers: {
   'Authorization': 'Bearer ' + req.session[WebAppStrategy.AUTH_CONTEXT].accessToken
  }
 };
 try{
  await request(options);
  res.send("Elevator is now stopped.");
 } catch (e) {
  res.send("You don't have permissions to perform this action.");
 }
});

```

## Step 6: APIStrategy (Backend endpoints)

These endpoints on the backend server use APIStrategy to validate that the access token contains the elevator.call, elevator.service, and elevator.stop scopes respectively:

```
app.get("/protected/callElevator",
passport.authenticate(APIStrategy.STRATEGY_NAME, {
audience: config.clientId,
scope: "elevator.call",
session: false
}),

function(req, res) {
  res.send('success');
 });
app.get("/protected/serviceMode",
 passport.authenticate(APIStrategy.STRATEGY_NAME, {
  audience: config.clientId,
  scope: "elevator.service",
  session: false
 }),
 function(req, res) {
  res.send('success');
 });
app.get("/protected/stopElevator",
 passport.authenticate(APIStrategy.STRATEGY_NAME, {
  audience: config.clientId,
  scope: "elevator.stop",
  session: false
 }),
 function(req, res) {
  res.send('success');
 });

```


## Conclusion:

In this workshop, we showed you how to use App ID to manage users access permissions to your application's resources. Hopefully, now you feel confident in your ability to set up your App ID instance with scopes and roles and set up your application to manage access according to the roles that you assigned. 

## Workshop Resources

Login/Sign Up for IBM Cloud: https://ibm.biz/appsecurity

Workshop Replay: https://www.crowdcast.io/e/journey-to-low-code-no-code-app-security-3

## Done with the workshop? Here are some things you can try further

configure social identity providers to set up a single sign-on experience for your app : https://cloud.ibm.com/docs/appid?topic=appid-social&interface=ui

## Links to the previous sessions in "Journey to Low Code/ No Code Application Security"

## 24th September- 7PM-9PM (GMT+5)

Securely Manage access to your Applications Sensitive Data on IBM Cloud
https://www.crowdcast.io/e/journey-to-low-code-no-code-app-security-1

## 28th September- 7PM-9PM (GMT+5)

Easily Secure your Spring boot app with Low code/No Code
https://www.crowdcast.io/e/journey-to-low-code-no-code-app-security-2

