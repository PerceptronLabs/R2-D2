# How To Add Authentication to Angular Web App using Routing Guards

![image](images/angular_web_app.jpeg)

## Introduction 
In web applications, authentication plays a very important role. Whenever there is a prompt to login or check if this user should be authenticated to enter the website or not, it is handled using username and password authentication. While building a web application in angular js, one can build  a user authentication system using Angular Route Guards. In this blog, we will talk about what are route guards, how easy it is to configure them in your angular js based web applications and then you can build your own authentication using these! 


## Angular Authentication With Angular Route Guards

In your Angular application, you probably have pages you only want authenticated users to access, but how can you control this? Router in Angular is a service that provides navigation among views and URL manipulation capabilities. Route guards are then implemented to prevent users from navigating.

![image](images/user_routing.png)

