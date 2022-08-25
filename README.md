# @manques/keycloak
> keycloak setup web, cordova and cordova-native

---

- [About](#about)
- [Installation](#installation)
- [Setup](#setup)
- [Example project](#example-project)

---

## About

This library helps you to use [keycloak-js offical](https://www.keycloak.org/docs/latest/securing_apps/index.html#_javascript_adapter) in Angular applications providing the following features:

- **Keycloak Module** 
- **Keycloak Service** 
- **Keycloak Auth Gaurd** 

## Installation

Run the following command to install both Keycloak Angular and the official Keycloak client library:

```sh
npm install @manques/keycloak-js
```

## Setup

### app.module.ts

```ts
...
import { AuthModule } from './auth/auth.module';
import { AuthGuard } from './auth/auth.guard';
...

@NgModule({
  declarations: [
    .......
  ],
  imports: [
   ....
    AuthModule
  ],
  providers: [AuthService, DataService,
    ...
    AuthGuard
  ],
  bootstrap: [AppComponent]
})

export class AppModule{...}
```
### environment.ts

```
import { Environment } from './environment.d';
import { getPlatform } from './environment.platform';

export const environment: Environment  = {
	production: false,
	// auth server(keycloak)
	keycloakConfigOptions:  {
		url: 'http://localhost:8080', // your auth server url
		realm: 'your realm name',
		clientId: 'your client id',
		enableLogging: true,
	},
	keycloakInitOptions : {
		onLoad: 'login-required',
		adapter: 'cordova', // inAppbrowser 
		// adapter: 'cordova-native', // system browser
		responseMode: 'query',
		redirectUri: 'http://localhost' // inAppbrowser,
		// redirectUri: https://www.example.com' // safari system browser ios 
		// redirectUri:'android-app://com.example.www/https/www.example.com' // chrome system browser
	},
};
```


### module(auth.module.ts)

```ts
import { NgModule } from "@angular/core";
import { AuthOIDCService } from "./authOIDC.service";

@NgModule({
    providers: [
        AuthOIDCService,
    ]
})

export class AuthModule{}
```

### service (authOIDC.service.ts)

```ts
import { Injectable, NgZone } from '@angular/core';
import * as Keycloak from '@manques/keycloak-js';
import { Subject } from 'rxjs';
import { environment } from "../../environments/environment";

@Injectable()

export class AuthOIDCService {
    keycloak = Keycloak(environment.keycloakConfigOptions);
    authState = new Subject<boolean>();


    constructor(private ngZone: NgZone){}

    AuthInit(path: string = '/'){
        return new Promise((resolve, reject) => {
            this.keycloak
            .init(/** @type {?} */ ({
                ...environment.keycloakInitOptions,
                redirectUri: environment.keycloakInitOptions.redirectUri + path
            }))
            .success(isAuth => {
                this.authState.next(true);
                resolve(isAuth);
            })
            .error(error => {
                this.authState.next(false);
                console.log('auth service init error', error);
                reject(false);
            });
        });
    }

    login(path: string = '/'){
        return new Promise((resolve, reject) => {
            this.keycloak
            .init(({
                redirectUri: environment.keycloakInitOptions.redirectUri + path
            }))
            .success(isAuth => {
                this.authState.next(true);
                resolve(isAuth);
            })
            .error(error => {
                this.authState.next(false);
                console.log('auth service login error', error);
                console.log(error);
                reject('Auth server login unsuccessful!');
            });
        });
    }

    refreshToken(){
        this.keycloak.isTokenExpired()
    }

    updateToken(number = 0){
        this.keycloak.updateToken(number)
        .success((isAuth) => {
            this.authState.next(true);
        }).error((err) => {
            this.authState.next(false);
            console.log(err);
        });
	}

    logout(path = '/'){
        return new Promise((resolve, reject) => {
            this.keycloak
            .logout(({
                redirectUri: environment.keycloakInitOptions.redirectUri + path
            }))
            .success(isAuth => {
               this.ngZone.run(() => {
                    this.authState.next(false);
                    resolve(true);
               });
            })
            .error(err => {
                this.authState.next(true);
                console.log(err);
                reject(err)
            });
        });
    }

    get isAuthenticated(){
        return this.keycloak.authenticated;
    }

    get token(){
        return this.keycloak.token;
    }

    get idToken(){
        return this.keycloak.idToken;
    }

    get user(){
        return this.keycloak.tokenParsed;
    }

}

```


### Guard (auth.guard.ts)

```ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { AuthOIDCService } from './authOIDC.service';
@Injectable()
export class AuthGuard implements CanActivate {

    constructor(
        private authOIDCService: AuthOIDCService,
    ) {}

    canActivate( next: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
        if(!this.authOIDCService.token){
            this.onAuthInit();
            return false;
        } else return true;
    }

    private onAuthInit() {
		this.authOIDCService.AuthInit()
        .then(isAuth => {
			// after auth processs
        }).catch(err => {
            console.log(err);
        });
    }

}

```


## Example project

[cordova-keycloak-angular](https://github.com/manques/cordova-keycloak-angular)
[cordova official](https://github.com/keycloak/keycloak/tree/main/examples/cordova)
[cordova-native official)](https://github.com/keycloak/keycloak/tree/main/examples/cordova-native)




