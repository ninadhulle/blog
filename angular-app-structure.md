Below key points are for basic folder structure for Angular app
1. Create folders for your bounded context(for each <domain> or functional area). Each folder should have a <domain>module.ts and <domain>-routing.module.ts for 
eg. 
```
  app
    |__core  (core module, this will include common code like security, error handling, logging, etc) 
    |__home (your home page - your default navigation page)
    |__functionality1 (this is root menu item)
    |__functionality2 (this is root menut items)
```
Don’t make module huge. If you have lot of sub menus and menus – please split it up with sub-modules.
2. Your app-routing.module.ts should only have root level menus. This also enables default lazy loading strategy of Angular.
  
```typescript
  const routes: Routes = [{
    path: '', 
    component: HomeComponent,
    children: [
    { 
      path: 'domain-menu-1', 
      loadChildren: 'app/domain1/domain1.module#Domain1Module' 
    },
    { 
      path: 'domain-menu-2', 
      loadChildren: 'app/domain2/domain2.module#Domain2Module' 
    }]
  }];
  
```  
  
3. Each <domain>-routing.module.ts should have sub-menus. This will use the lazy loading feature of Angular. Don’t add sub menu entries 
to app-routing.module.ts.
  
```typescript
  const DOMAIN1_ROUTER: Routes = [{ 
    path: '', 
    component: Domain1Component 
  },{ 
    path: 'domain-menu-1/sub1-menu1', 
    component: Sub1Domain1Component
    },{ 
    path: 'domain-menu-2/sub1-menu1', 
    component: Sub1Domain1Component 
 }];

```  
4. Create authentication and authorization guards using Angular guard feature.
Below is sample code for Authentication Guard
```typescript
@Injectable()
export class AuthGuard implements CanActivate {

  constructor(
    private securityInitService: SecurityInitService,
    private userService: UserService,
    private router: Router
  ) { }

  canActivate(next: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<any> {
    return this.securityInitService.authenticationComplete.map(e => {
        if (e === 'SUCCESS' && this.userService.getLoggedInStatus())
            return true;
    }).catch(() => {
         this.router.navigate(['error-401']);
         return Observable.of(false);
    });
  }
}
```
Below is sample code for Authorization Guard
```typescript
@Injectable()
export class RoleGuard implements CanActivate, CanActivateChild {
    menuItems: any;
    route: ActivatedRouteSnapshot;
    state: RouterStateSnapshot;

    constructor(
        private securityInitService: SecurityInitService,
        private userService: UserService,
        private router: Router,
        private loggerService: LoggerService,
        private MenuService: NavigationService
    ) { }

    canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<any> {
      const browserUrl: string = state.url;
      this.route = route;
      this.state = state;

      return this.securityInitService.authenticationComplete.map(e => {
        if (e) {
          if (e === 'SUCCESS' && this.userService.getLoggedInStatus()) {
          }
        }
      }).catch(() => {
        this.router.navigate(['error-403']);
        return Observable.of(false);
      });
    }
    
    canActivateChild(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<any> {
      return this.canActivate(route, state);
    }
    
```
5. Use authentication guards on default route canactivate, this ensures that all routes are authenticated using guard. Use authorization guards to ensure child routes have the necessary roles.
```typescript
  const routes: Routes = [{
    path: '', 
    canActivate: [AuthenticationGuard], 
    canActivateChild: [AuthorizationGuard], 
    component: HomeComponent,
    children: [];
```  
6. Optionally decorate routes within each domain module (in <domain>-routing.module.ts) with guards specific to handle activation/deactivation.
```typescript
  const DOMAIN1_ROUTER: Routes = [{ 
    path: '', 
    component: Domain1Component 
  },{ 
    path: 'domain-menu-1/sub1-menu1', 
    canActivate: [Domain1Guard], 
    component: Sub1Domain1Component, 
    canDeactivate: [Domain1DeactivateGuard] 
  }];
``` 
8. Shared Module – All common components should go to shared module. These should include new shared components, directives, pipes, 
validators.
9. Create logging component to log to server side in case of unhandled exception.
```typescript
  export class GlobalErrorHandlerService extends ErrorHandler {

  constructor(
    private loggingService: LoggerService,
    private errorService: ErrorService
  ) {
    //super(false);
    super();
  }

  handleError(error): void {
    
  }
```  
10. Logs – Feel free to add tons of console.logs to your code. These can be removed for production as a part of build process.
11. Create BaseComponent which you your components inherit fromExtend all business(not the shared ones) components from baseComponent. 
This injects 
  * GrowlService - For error message display on top right hand side
  * ConfirmationProgressService - Shown on components for confirmation activities 
  * PreloaderService - Shown during loading
12. Create your own HttpClient class. Added Angular interceptor to this custom HttpClient class for 
  * Timeout – It times out services if Java APIs are taking too long
  * There is retry functionality, so if first service fails, it will make subsequent call.
  * Error handling for authentication, authorization, api unavailable.
  * Any custom header needed for every request
13. Use Linting to fix common style error.
14. Use Codelyzer for static code analysis.
15. Adhere to [Angular Style Guide] (https://angular.io/guide/styleguide)
16. No inline styles please(style html attribute).So any inline styles are going to impact responsive design. You can create styles 
for component in your .component.css, No !important tags to be used for styles
17. Constants - Each module should have its own constants Eg. <domain>.constants.ts
18. Global constants: Should go in app.constants.ts

Below is sample folder structure:

```
app
|__core //this where core infra components/services are kept
|__home //home components – side menu/top level menu
|__domain 1(functionality-root level menu)
|__domain 2 //Keep component, services, model, html, css specific to domain. 
|    |    |    //If it is common it should be moved to app/shared/services and app/shared/models. Create subfolders if required.
|    |    |__domain2.component.ts
|    |    |__domain2.component.html            
|    |    |__domain2.component.css
|    |    |__domain2.service.ts
|    |    |__domain2.model.ts
|    |__domain2-module.ts 
|    |__domain2-routing.module.ts //add menu/navigation here
|__domain3
|__domain3
|__shared
|    |__components 
|    |    |__confirmation //common reusable infra components used across complete applications – eg. confirmation, growl, nav-bar. Keep all models, services, html, css, components in single folder
|    |    |    |__confirmation-progress
|    |    |    |__confirmation-progress.component.css
|    |    |    |__confirmation-progress.component.html
|    |    |    |__confirmation-progress.component.spec.ts
|    |    |    |__confirmation-progress.component.ts
|    |    |    |__confirmation-progress.model.ts
|    |    |    |__confirmation-progress.service.spec.ts
|    |    |    |__confirmation-progress.service.ts
|    |    |__growl
|    |__models //folder for only common models, this is for shared models which are commonly used. This to promote reuse
|    |    |__some.model.ts 
|    |    |__common-abc //group common models  
|    |        |__abc1.model.ts
|    |        |__abc2.model.ts       
|    |__services //folder for only common business services, this is for shared services which are commonly used. This to promote reuse
|    |    |__some.service.ts           
|    |    |__common-abc //group common service for patient using folder  
|    |        |__abc1.service.ts
|    |        |__abc2.service.ts     
|    |__directives
|    |__validators
|    |__pipes
|__app-routing.module.ts //keep only base level menu items – submenus should go to individual folder structure
|__app.module.ts //keep only base level modules – submodules should go to individual folder structure   
```
