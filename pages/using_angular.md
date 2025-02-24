---
layout: default
title: Using Angular
permalink: /using-angular/
sitemap:
    priority: 0.7
    lastmod: 2015-01-29T23:41:00-00:00
---

# <i class="fa fa-html5"></i> Using Angular

## Tooling

Angular is using TypeScript instead of JavaScript, and as a result some specific tooling is necessary to work efficiently with it. Our [development]({{ site.url }}/development/) workflow for an Angular 2+ application is as below, use `npm` instead of `yarn` if you prefer that.

1. When you generate an application the files are created and at the end of generation `npm install` task is triggered.
2. Once `npm install` is complete it calls the `postInstall` script in `package.json`, this step triggers the `webapp:build` task.
3. Now you should have all files generated and compiled into the `www` folder inside the `target` or `build` folder based on the build tool (Maven or Gradle) selected.
4. Now run `./mvnw` or `./gradlew` to launch the application server and it should be available at [localhost:8080](localhost:8080) this also serves the client side code compiled from the above steps.
5. Now run `npm start` or `yarn start` in a new terminal to launch Webpack dev-server with BrowserSync. This will take care of compiling your TypeScript code, and automatically reloading your browser.

If you start making changes to the client side code without having `npm start` or `yarn start` running, nothing will be reflected as the changes are not compiled so you need to either run `npm run webapp:build` manually after changes or have `npm start` or `yarn start` running.

You can also force Maven to run the `webapp:dev` task while starting by passing the `webapp` profile like `./mvnw -Pdev,webapp`.

**Note** Gradle automatically runs webpack compilation in `dev` profile if front end has changed (only at start up, for live reload use `npm start` or `yarn start`).

Other available yarn/npm commands can be found in the `scripts` section of your project's `package.json` file.

- To work on your code in your browser, we recommend using [Angular DevTools](https://angular.io/guide/devtools). Angular DevTools is a browser extension that provides debugging and profiling capabilities for Angular applications (**Note** Angular DevTools supports Angular v12 and later).

## Project Structure

The JHipster client code can be found under `src/main/webapp`, and follows closely the  [Angular style guide](https://angular.io/guide/styleguide). Please read this guide first if you have any question on our application structure, file names, TypeScript conventions...

This style guide is endorsed by the Angular team, and provides best practices that every Angular project should follow.

For Angular routes we follow a dash cased naming convention so that the URLs are clean and consistent.
When you generate an entity the route names, route URLs and REST API endpoint URLs are generated according to this convention, also entity names are automatically pluralized where required.

Here is the main project structure:

    webapp
    ├── app                               - Your application
    │   ├── account                       - User account management UI
    │   ├── admin                         - Administration UI
    │   ├── config                        - Some utilities files
    │   ├── core                          - Common building blocks like configuration and interceptors
    │   ├── entities                      - Generated entities (more information below)
    │   ├── home                          - Home page
    │   ├── layouts                       - Common page layouts like navigation bar and error pages
    │       ├── main                      - Main page
    │           ├── main.component.ts     - Main application class
    │   ├── login                         - Login page
    │   ├── shared                        - Common services like authentication and internationalization
    │   ├── app.module.ts                 - Application modules configuration
    │   ├── app-routing.module.ts         - Main application router
    ├── content                           - Static content
    │   ├── css                           - CSS stylesheets
    │   ├── images                        - Images
    │   ├── scss                          - Sass style sheet files will be here if you choose the option
    ├── i18n                              - Translation files
    ├── swagger-ui                        - Swagger UI front-end
    ├── 404.html                          - 404 page
    ├── favicon.ico                       - Fav icon
    ├── index.html                        - Index page
    ├── robots.txt                        - Configuration for bots and Web crawlers

Using the [entity sub-generator]({{ site.url }}/creating-an-entity/) to create a new entity called `Foo` generates the following front-end files under `src/main/webapp`:

    webapp
    ├── app
    │   ├── entities
    │       ├── foo                                    - CRUD front-end for the Foo entity
    │           ├── foo.component.html                 - HTML view for the list page
    │           ├── foo.component.ts                   - Controller for the list page
    │           ├── foo.model.ts                       - Model representing the Foo entity
    │           ├── foo.module.ts                      - Angular module for the Foo entity
    │           ├── foo.route.ts                       - Angular Router configuration
    │           ├── foo.service.ts                     - Service which access the Foo REST resource
    │           ├── foo-delete-dialog.component.html   - HTML view for deleting a Foo
    │           ├── foo-delete-dialog.component.ts     - Controller for deleting a Foo
    │           ├── foo-detail.component.html          - HTML view for displaying a Foo
    │           ├── foo-detail.component.ts            - Controller or displaying a Foo
    │           ├── foo-dialog.component.html          - HTML view for editing a Foo
    │           ├── foo-dialog.component.ts            - Controller for editing a Foo
    │           ├── foo-popup.service.ts               - Service for handling the create/update dialog pop-up
    │           ├── index.ts                           - Barrel for exporting everything
    ├── i18n                                           - Translation files
    │   ├── en                                         - English translations
    │   │   ├── foo.json                               - English translation of Foo name, fields, ...
    │   ├── fr                                         - French translations
    │   │   ├── foo.json                               - French translation of Foo name, fields, ...

Please note that the default language translations would be based on what you have choosen during app generation. 'en' and 'fr' are shown here only for demonstration.

## Authorizations

JHipster uses [the Angular router](https://angular.io/docs/ts/latest/guide/router.html) to organize the different parts of your client application.

For each state, the required authorities are listed in the state's data, and when the authority list is empty it means that the state can be accessed anonymously.

The authorities are also defined on the server-side in the class `AuthoritiesConstants.java`, and logically the client and server-side authorities should be the same.

In the example below, the 'settings' state is designed to be accessed only by authenticated users who have `ROLE_ADMIN` authority:

    export const settingsRoute: Route = {
        path: 'sessions',
        component: SettingsComponent,
        title: 'global.menu.account.settings',
        data: {
            authorities: ['ROLE_ADMIN'],
        },
        canActivate: [UserRouteAccessService]
    };

Once those authorities are defined in the router, they can be used through `jhiHasAnyAuthority` directive within its 2 variants based on type of argument:

- for a single string, the directive only displays the HTML component if the user has the required authority
- for an array of strings, the directive displays the HTML component if the user has one of the listed authorities

For example, the following text will only be displayed to users having the `ROLE_ADMIN` authority:

    <h1 *jhiHasAnyAuthority="'ROLE_ADMIN'">Hello, admin user</h1>

For example, the following text will only be displayed to users having one of the `ROLE_ADMIN` or `ROLE_USER` authorities:

    <h1 *jhiHasAnyAuthority="['ROLE_ADMIN', 'ROLE_USER']">Hello, dear user</h1>

*Please note* that those directives only show or hide HTML components on the client-side, and that you also need to secure your code on the server-side!

## The ng-jhipster library

The ng-jhipster library is free and OSS, and available on [https://github.com/jhipster/ng-jhipster](https://github.com/jhipster/ng-jhipster).

The ng-jhipster library contains utility functions and common components that are used by Angular 2+ applications. They include:

- Validation directives
- Internationalization components
- Commonly-used pipes like capitalization, ordering and word truncation
- Base64, date and pagination handling services
- A notification system (see below)

### Notification System

JHipster uses a custom notification system to send events from the server-side to the client-side, and has i18n-capable `JhiAlertComponent` and `JhiAlertErrorComponent` components which can be used throughout the generated applications.

By default JHipster will show error notifications when there is an error caught from an HTTP response.

To show a custom notification or alert, use the below methods after injecting the `AlertService` to your controller, directive or service.

The shorthand methods `success`, `info`, `warning` and `error` will have a default timeout of 5 seconds, which can be configured:

    this.alerts.push(
        this.alertService.addAlert(
            {
                type: 'danger',
                msg: 'you should not have pressed this button!',
                timeout: 5000,
                toast: false,
                scoped: true
            },
            this.alerts
        )
    );

## Using Angular CLI

Angular CLI is used to build and test JHipster applications. 
However, we added a custom webpack configuration file in order to improve the developer experience by adding BrowserSync, ESLint (Angular CLI is still on TSLint for now), merging JSON translation files and add notifications when build has completed or failed.

### Overview

[Angular CLI](https://cli.angular.io/) is a tool to develop, scaffold and maintain Angular applications. JHipster generates the Angular CLI configuration file, so the Angular CLI workflows work with JHipster.

This integration is done by generating a `angular.json` file in the application root folder and adding its dependencies in the `package.json` file.

### Usage

```bash
ng help
```

### Building

You can use `ng build` to build your front-end, but we still recommend to use the provided NPM scripts such as `npm start`, `npm run build`, etc. Check our ["using in development" documentation]({{ site.url }}/development/) and our ["using in production" documentation]({{ site.url }}/production/).

### Generating Components, Directives, Pipes and Services

You can use the `ng generate` (or `ng g`) command to generate Angular components:

```bash
ng generate component my-new-component
ng g component my-new-component # using the alias

# Components support relative path generation
# Go to src/app/feature/ and run
ng g component new-cmp
# your component will be generated in src/app/feature/new-cmp
# but if you were to run
ng g component ../newer-cmp
# your component will be generated in src/app/newer-cmp
```
You can find all possible blueprints in the table below:

Scaffold  | Usage
---       | ---
[Component](https://github.com/angular/angular-cli/wiki/generate-component) | `ng g component my-new-component`
[Directive](https://github.com/angular/angular-cli/wiki/generate-directive) | `ng g directive my-new-directive`
[Pipe](https://github.com/angular/angular-cli/wiki/generate-pipe)           | `ng g pipe my-new-pipe`
[Service](https://github.com/angular/angular-cli/wiki/generate-service)     | `ng g service my-new-service`
[Class](https://github.com/angular/angular-cli/wiki/generate-class)         | `ng g class my-new-class`
[Guard](https://github.com/angular/angular-cli/wiki/generate-guard)         | `ng g guard my-new-guard`
[Interface](https://github.com/angular/angular-cli/wiki/generate-interface) | `ng g interface my-new-interface`
[Enum](https://github.com/angular/angular-cli/wiki/generate-enum)           | `ng g enum my-new-enum`
[Module](https://github.com/angular/angular-cli/wiki/generate-module)       | `ng g module my-module`


### Test

For consistency purpose on JHipster application, tests are available through the `npm` command:

```bash
npm test
```

### i18n

JHipster is using the `ngx-translate` dependency for translation purpose. Angular CLI i18n is based on the default Angular i18n support, which is incompatible with JHipster.

### Running the server

If you prefer to use Angular CLI to develop you application, you can run your server directly by using its dedicated command.

```bash
ng serve
```

### Conclusion

For more information about the Angular CLI, please visit the official website [https://cli.angular.io/](https://cli.angular.io/)
