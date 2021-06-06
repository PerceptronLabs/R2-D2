# How To Add Authentication to Angular Web App using Routing Guards

![image](images/angular_web_app.jpeg)

## Introduction 
In web applications, authentication plays a very important role. Whenever there is a prompt to login or check if this user should be authenticated to enter the website or not, it is handled using username and password authentication. While building a web application in angular js, one can build  a user authentication system using Angular Route Guards. In this blog, we will talk about what are route guards, how easy it is to configure them in your angular js based web applications and then you can build your own authentication using these! 


## Angular Authentication With Angular Route Guards

In your Angular application, you probably have pages you only want authenticated users to access, but how can you control this? Router in Angular is a service that provides navigation among views and URL manipulation capabilities. Route guards are then implemented to prevent users from navigating.

![image](images/user_routing.png)


With Router Guards we can prevent users from accessing areas that they’re not allowed to access, or, we can ask them for confirmation when leaving a certain area. And these are particularly my favourite for authentication since route guards make this integration quick and easy! 

### Lets dive deeper :swimmer:
Now as we have got some idea on routing and use case of Route Guard lets jump into the way it’s created and implemented in Angular Js. 

Let us first build your code as follows: 
  - Create an Auth Directory so that it’s easy to understand the folder you are working in for your requirements.  
  - The route guard will work in conjunction with an auth service that contains an HTTP request to your server (or a serverless SDK request) that determines the user’s authenticated state.

Next, We can generate guards with [Angular Cli](https://angular.io/cli) from command line inside the directory you want to create the app.

First make sure you are in the right directory:
`
cd src/app/
`

At the end of this step, we will have the following directory structure: 

`
src -> app -> auth -> (auth.guard.ts & auth.service.ts)
`

#### Let us create our first file - `auth.guard.ts`
In order to create the guard file, angular has this amazing one-liner command:

`
ng generate guard auth
`

Here we call the `ng generate` to generate the `guard` authentication file. 

If the CLI (command line interface or your terminal) asks you which interface you would like to implement, choose `CanActivate`. You will see that the CLI will create the `/auth` directory for you if you haven’t created it yourself so far. 

The `auth.guard.ts` file should look like this:

````
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, UrlTree } from '@angular/router';
import { Observable } from 'rxjs’;
@Injectable({
    providedIn: 'root'
})
export class AuthGuard implements CanActivate {
    canActivate(next: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
    return true;
}}
````

Before we jump to he service file, you can read more about the different kinds of interfaces availble for implementation here: 

Note that in the above code, class `AuthGuard` implements `CanActivate` as we had previously selected `CanActivate` from the CLI. We will add more steps in this method later, but for now we can keep it as is. If you dont understand all of this now, it is completely fine, we will get to the further steps soon! 

> **CanActivate** is the interface that determines if the user can proceed to a specific route. I’ve edited the return type to be just `Observable < boolean >`. Your application might be different.


#### Let us create our service file - `auth.service.ts`

Now that we have created our guard file we also need a file which checks the authentication state of the user which is written in `auth.service.ts`. 

This function might call an API endpoint or a serverless SDK function. In this example we will assume you are calling an API endpoint. Here is an example of what your `auth.service.ts` file might look like:

````
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core’;
@Injectable({
providedIn: 'root'
})
export class AuthService { constructor(private http: HttpClient) { } isAuthenticated() {
return this.http.get('/auth/isAuthenticated');
}}:
````

Let us understand the above code. Here we are create a class `AuthService` which uses the method `isAuthenticated()`. This function  is going to call an API endpoint `/auth/isAuthenticated` that will return an object indicating the authenticated status. 

`{ authenticated: (true | false) }`

Authentication status would be true or false (a boolean field) which can then be used for routing purposes.

#### Implementing Guards with Authservice

Now that we have our guard and service file ready, we need to import it and define it in the constructor. We also need to `import Router` to redirect the user to the login page if they are not authenticated. 

So let us make the following changes in our first file - `auth.guard.ts`:

````
...
import { AuthService } from './auth.guard';
import { ActivatedRouteSnapshot, CanActivate, Router, RouterStateSnapshot } from '@angular/router’;
...

	 constructor(private authService: AuthService, private router: Router) { } 
````


Now we will edit the `CanActivate` method in the guard file to use the `isAuthenticated()` function from service file.

````
import { Observable, of } from 'rxjs';
import { catchError, map } from 'rxjs/operators’;
... 
    canActivate(next: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> 
    {
        return this.authService.isAuthenticated().pipe(map((response: { authenticated: boolean}) => {
            if (response.authenticated) {
                return true;
            }
        this.router.navigate(['/login']);
        return false;
        }), catchError((error) => {
        this.router.navigate(['/login']);
        return of(false);
        }));
    }}
````

In the above code, we are calling the `isAuthenticated()` function. We make use of the `pipe` and `map` operator to access the response. We then use the response to determine the user’s authenticated state. If the user is authenticated, we return true. Otherwise we use the Router to redirect the user to the login page and return false.

We are also using the `catchError` operator to catch any errors from the api. If there is an error, we will redirect the user to the homepage and return false. Did you notice that false is wrapped in an of `rxjs` operator? This is because catchError requires you to return an observable. Of takes a value and wraps that value in an observable.

##### Adding Guards to your Routes:
In the routing modules, `app-routing.module.ts`, you need to add the guard to all the routes you want to protect. 

In this example, lets protect the user’s profile page. Add the `canActivate property` to the `profile` route that takes an array of guards with only your `AuthGuard`. Also add the `AuthGuard` to the providers array.

````
...
import { AuthGuard } './auth/auth.guard’;
const routes: Routes = [
...
{
    path: 'profile',
    component: ProfileComponent,
    canActivate: [ AuthGuard ]
}
];
@NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule],
    providers: [ AuthGuard ]
})
export class AppRoutingModule { }

````

Adding the guard to the route’s canActivate property ensures that this guard will be run anytime someone tries to access the profile page, only allowing them to proceed if they are authenticated. Additionally, without adding it to the providers array, your guard will not be registered and the application won’t run. Don’t forget to add any guards you may have in your application to this array.

##### We can also use `CanActivateChild`

`CanActivateChild` is almost similar to `CanActivate ` interface, the only difference is `CanActivate` is used to control the accessibility of the current route but `CanActivateChild` is used to prevent access to child routes of a given route, so by using this you don’t need to add canActive on each child route, in other words, you just need to add canActiveChild to parent route and it will work for child routes as well.

````
{
    path: 'dashboard’,
    canActivate: [AuthGuard],
    canActivateChild: [AuthGuard],
    component: DashboardComponent,
    children: [
        { path: ':id', component: InfoComponent},
        { path: ':id/edit', component: EditInfoComponent}
    ]
}
````
