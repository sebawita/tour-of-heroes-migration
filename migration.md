<!--
Issues to talk about:
- the default RouteReuseStrategy between web (ngOnInit triggers every time) and {N} (ngOnInit doesn't trigger on navigating back).
-->

<h1>Migrating the Tour of Heroes into a web+mobile Code-Sharing project</h1>

> Need some intro here
> 
> Point to the docs: [Migrating a Web Project](https://docs.nativescript.org/angular/code-sharing/migrating-a-web-project)

# Preparation

This tutorial is based on [the Tour of Heroes tutorial](https://angular.io/tutorial).

In order to save the time, here is the completed version of the project: [github.com/sebawita/tour-of-heroes](https://github.com/sebawita/tour-of-heroes).

You should start by cloning the project.

```bash
git clone https://github.com/sebawita/tour-of-heroes/
cd tour-of-heroes
npm install
```

> Please note that any Angular project that you want to convert to a code-sharing project, should be using:
> 
> - **Angular 6.0** or newer,
> - the **Angular CLI 6.1** or newer.
>
> This is already the case for the above repo, so there is no need to make any changes to it.

## NativeScript setup

For the purposes of this tutorial we will be using the **NativeScript preview workflow**. For this you will need:

The **NativeScript CLI 5.0.0** or newer
> To install the NativeScript CLI, run the following command:
> 
```bash
npm install -g nativescript@latest
```

The companion apps, installed on a real device:

| app | Android | iOS |
|-----|---------|-----|
| NativeScript Preview    | [Google Play](https://play.google.com/store/apps/details?id=org.nativescript.preview)   | [App Store](https://itunes.apple.com/us/app/nativescript-preview/id1264484702) |
| NativeScript Playground | [Google Play](https://play.google.com/store/apps/details?id=org.nativescript.play) | [App Store](https://itunes.apple.com/us/app/nativescript-playground/id1263543946) |

<!--
> To install NativeScript, follow the [installation instructions](https://docs.nativescript.org/angular/start/quick-setup).
> -->


# Converting the project to a code-sharing structure

The first step is to convert the project into a Web+NativeScript Code-Sharing project structure.

This is done with the **ng add** command. Run the following from inside the project:

```bash
ng add @nativescript/schematics
```

The easiest way to validate that the migration worked is to build and run your apps.

## Validate the web project
Run **ng serve --open** from your terminal or command prompt, and you should get the same app as before running the **ng add** command.

## Validate the NativeScript project
Next run the **preview** command in the NativeScript CLI, like this:

```
tns preview --bundle
```

This should present you with a QR code. Open the **NativeScript Playground** app on your phone and press the **Scan QR code** button and scan the QR Code.

This should open the **NativeScript Preview** app, which at first will display a white screen (just be patient the NativeScript CLI is just building the project for you). After a little you should be presented with a big button saying **AUTO-GENERATED WORKS!**.

### Make the first change
You don't need to scan the QR code each time you want to update the app. The NativeScript CLI will push all saved changes automatically.

Let's put that to a test.
Open **src/app/auto-generated/auto-generated.component.tns.html** and change the `text` attribute to `text="hello world"` and save the file.

Now you should see an app with a big button saying **Hello World** 💭🌍.

### Troubleshooting

If the app cannot scan the QR code.
You might need to change the colour theme of your terminal to one with more contrast, and try again.


## Navigation

In your migrated project you can find:

* **app-routing.module.ts** - a web-specific Routing Module file
* **app-routing.module.tns.ts** - a NativeScript-specific Routing Module file

> Note, that the code-sharing project uses a special naming convention, where it marks NativeScript-specific files with a `.tns` extension. You can learn more about it in the NativeScript docs on [code-splitting](https://docs.nativescript.org/angular/code-sharing/code-splitting)

If you open **app-routing.module.tns.ts**, you will notice that NativeScript imports **NativeScriptRouterModule**, which provides the same functionality as **RouterModule**. This is how NativeScript handles native navigation for **iOS** and **Android**.

Additionally, you will find a **routes** property with a single path to the **HomeComponent** (a sample NativeScript component).

It is also possible to merge the **routes** configuration into a single shared object. This is covered in the [Shared Navigation section ---add link---]().

<h1>Migrating the Project Content</h1>
The **ng add @nativescript/schematics** command converts the project to a code-sharing structure, but it doesn't convert your app contents.

The next steps from here are to:

* migrate the **AppModule**
* migrate the other **modules**
* *(optional)* unify **navigation**

# Migrate the AppModule

Often a migration of the AppModule involves finding equivalents for all of its imported modules and migrating all declared components. 

## Migrating AppModule Imports

Transforming module **Imports** usually means moving the imports from `app.module.ts` to `app.module.tns.ts`. For each import we need to check:

* if this is a **generic import** - meaning it works for both web and {N} - then just add it to the **{N} AppModule**

* if this is a web specific import - then we need to find a **{N} equivalent** and add it to the **{N} AppModule**

Let's have a look at the imports in **app.module.ts** of our Tour of Heroes app, and go through them one by one.

```TypeScript
imports: [
  BrowserModule,
  FormsModule,
  AppRoutingModule,
  HttpClientModule,

  HttpClientInMemoryWebApiModule.forRoot(
    InMemoryDataService, { dataEncapsulation: false }
  )
],
```

**BrowserModule**

The **BrowserModule** is **web specific**, the equivalent module in {N} is **NativeScriptModule**. You can import it from: 

```TypeScript
import { NativeScriptModule } from 'nativescript-angular/nativescript.module';
```

Your **app.module.tns.ts** will already contain an import of **NativeScriptModule**, as this is done by the `ng add` command, so no need to do anything.

**FormsModule**

The **FormsModule** is also **web specific**, the equivalent module in {N} is **NativeScriptFormsModule**. You can import it from:

```TypeScript
import { NativeScriptFormsModule } from 'nativescript-angular/forms';
```

Your **app.module.tns.ts** will already contain a commented out import of **NativeScriptModule**. Just uncomment it and it to the **NgModule imports**.

**AppRoutingModule**

At this stage you will have two versions of the **AppRoutingModule**:

* Web - **app-routing.module.ts** - which will contain the **routes** to navigate to all your web components,
* {N} - **app-routing.module.tns.ts** - which will contain the **routes** to navigate to the sample **HomeComponent**.

We will cover the navigation in more details later on and we will discuss the options to bring both configurations in one place.
However, at this stage of the migration process it is better to keep them separately.

Your **app.module.tns.ts** will already contain an import of the **{N} AppRoutingModule**, so no need to do anything.

**HttpClientModule**

The **HttpClientModule** is **web specific**, the equivalent module in {N} is **NativeScriptHttpClientModule**. You can import it from:

```TypeScript
import { NativeScriptHttpClientModule } from 'nativescript-angular/http-client';
```

Your **app.module.tns.ts** will already contain a commented out import of **NativeScriptModule**. Just uncomment it and it to the **NgModule imports**.

**HttpClientInMemoryWebApiModule**

The **HttpClientInMemoryWebApiModule** is platform-agnostic, so is the **InMemoryDataService**, which means that you don't need a {N} version of it. Just copy the import of it from **app.module.ts** to **app.module.tns.ts** and we are good to go.

## Module Imports Summary

This is how your imports in **app.module.tns.ts** should look like:

```TypeScript
import { NativeScriptModule } from 'nativescript-angular/nativescript.module';
import { NativeScriptFormsModule } from 'nativescript-angular/forms';
import { NativeScriptHttpClientModule } from 'nativescript-angular/http-client';

import { AppRoutingModule } from './app-routing.module.tns';

import { HttpClientInMemoryWebApiModule } from '../../node_modules/angular-in-memory-web-api';
import { InMemoryDataService } from './in-memory-data.service';
...

@NgModule({
  ...
  imports: [
    NativeScriptModule,
    NativeScriptFormsModule,
    AppRoutingModule,
    NativeScriptHttpClientModule,

    HttpClientInMemoryWebApiModule.forRoot(
      InMemoryDataService, { dataEncapsulation: false }
    ),
  ],
  ...
})
export class AppModule { }
```

Here is a quick table with all the imports we had to look into:

| Name | Same Module | {N} Name | import from  |
|---------------|---|---|---|
| BrowserModule | No | NativeScriptModule | 'nativescript-angular/nativescript.module' |
| FormsModule | No | NativeScriptFormsModule | 'nativescript-angular/forms' |
| AppRoutingModule | Yes/No | AppRoutingModule | './app-routing.module.tns' |
| HttpClientModule | No | NativeScriptHttpClientModule | 'nativescript-angular/http-client' |
| HttpClientInMemoryWebApiModule | Yes | --- | --- |
| InMemoryDataService | Yes | --- | --- |

## Migrating AppModule Components

Your next task is to migrate the **AppModule Components** into a code-sharing structure.

Often the migration of each component consists of three steps:

1. create a **{N}** version of the template file (i.e. **name.component.tns.html**)
	* add the **{N} UI Code** to it
1. (optional) create a **{N}** version of the stylesheet file (i.e. **name.component.tns.css**)
	* add the **{N} Styling** to it
1. add the component to the **Declarations** of the **{N}** AppModule (`app.module.tns.ts`)
1. (optional) add the component to the {N} navigation

This task can be helped with the [component migration schematic](https://docs.nativescript.org/angular/code-sharing/migrating-a-web-project#schematic-migrate-component), which helps you with the first three steps:

```bash
ng g migrate-component --name=component-name
```

### Migrate HeroesComponent

First run the component migration for the **HeroesComponent**.

Run:

```bash
ng g migrate-component --name=heroes
```

This will:

* generate **heroes.component.tns.html** with the commented out html from **heroes.component.html**
* generate an empty **heroes.component.tns.css**
* add the **HeroesComponent** to the **Declarations** of **app.module.tns.ts**

Like this:

```
src
└── app
    ├── heroes
    |   ├── heroes.component.html
    |   ├── heroes.component.tns.html <= create
    |   └── heroes.component.ts
    |   └── heroes.component.tns.css  <= create
    |   └── heroes.component.css
    ├── app.module.tns.ts             <= update
    └── app.module.ts           
```

#### Add the component to the navigation

The **HeroesComponent** for {N} will have some simple content, which is enough to display a simple message on the screen.

<!--Add a screenshot of it-->

You should update the NativeScript navigation **routes**, to make it navigate to the **HeroesComponent** when the app starts.

**app-routing.module.tns.ts**

```TypeScript
import { HeroesComponent } from './heroes/heroes.component';

export const routes: Routes = [
  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
  { path: 'heroes', component: HeroesComponent },
];
```

Rebuild the app:

```
tns preview --bundle
```

You should see a simple screen with a message: *Heroes Component works*.
<!--TODO: add screenshot and make sure the above message is correct.-->

#### Update the Template

Now, you need to update the {N} UI template, to match the content from the web app.

I don't want to dive too deep into building the UI using {N} components, as there are plenty of materials covering the subject. Like:

* `TODO: need some links here and a better explanation for each item`
* You can't use divs in NativeScript, but you can use [NativeScript Layouts](https://docs.nativescript.org/angular/ui/layouts/layouts).
* There is also an [interactive tutorial for {N} layouts](https://www.nslayouts.com/)
* [UI Widgets](https://docs.nativescript.org/angular/ui/ng-ui-widgets/action-bar)

Let's go quickly through the web template and convert it to {N}.

```html
<h2>My Heroes</h2>

<div>
  <label>Hero name:
    <input #heroName />
  </label>
  <!-- (click) passes input value to add() and then clears the input -->
  <button (click)="add(heroName.value); heroName.value=''">
    add
  </button>
</div>

<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <a routerLink="/detail/{{hero.id}}">
      <span class="badge">{{hero.id}}</span> {{hero.name}}
    </a>
    <button class="delete" title="delete hero"
      (click)="delete(hero)">x</button>
  </li>
</ul>
```


First, you have:

```html
<h2>My Heroes</h2>
```

Which is clearly the title of the page. You can replace it with an **ActionBar**.

```html
<ActionBar title="My Heroes" class="action-bar">
</ActionBar>
```

Then you have a simple container with a label, an input field and a button.

```html
<div>
  <label>Hero name:
    <input #heroName />
  </label>
  <!-- (click) passes input value to add() and then clears the input ->
  <button (click)="add(heroName.value); heroName.value=''">
    add
  </button>
</div>
```

You can convert it to the {N} template by replacing:

* the `<div>` container with a `<GridLayout>` that is arranged in three columns
* the `<label>` with a `<Label>` - with **Hero name:** provided as a text property
* the `<input>` with a `<TextField>`
* the `<button>` with a `<Button>` - where you pass `heroName.text` (instead of `heroName.value`)
  * use the `(tap)` event instead of the `(click)` event 

Like this:

```html
<GridLayout rows="auto" columns="auto, *, auto">
  <Label text="Hero name:" class="h2 m-l-5"></Label>
  <TextField col="1" hint="enter hero name..." #heroName class="m-x-5"></TextField>
  <Button col="2" text="add" (tap)="add(heroName.text); heroName.text=''" class="btn btn-primary"></Button>
</GridLayout>
```

Next, you have the list of heroes:

```html
<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <a routerLink="/detail/{{hero.id}}">
      <span class="badge">{{hero.id}}</span> {{hero.name}}
    </a>
    <button class="delete" title="delete hero"
      (click)="delete(hero)">x</button>
  </li>
</ul>
```

You can convert it by replacing:

* the unordered list (`<ul>`) with a `<ListView>`
* each item (`<li>`) with an `<ng-template>`
* the navigation link (`<a>`) with a `GridLayout`, also swap `routerLink` for `nsRouterLink`
* the `<span>` with a `<Label>`
* the `<button>` with a `<Button>`

Like this:

```html
<ListView [items]="heroes" class="list-group" height="100%">
  <ng-template let-hero="item">
    <GridLayout columns="auto, *, auto" nsRouterLink="/detail/{{hero.id}}">
      <Label col="0" [text]="hero.id" class="list-group-item"></Label>
      <Label col="1" [text]="hero.name" class="list-group-item"></Label>
      <Button col="2" text="X" (tap)="delete(hero)" class="btn btn-primary"></Button>
    </GridLayout>
  </ng-template>
</ListView>
```

Finally, you need to wrap the `GridLayout` and the `ListView` in a `StackLayout`.

> Please note that besides the **ActionBar**, a NativeScript template (this also applies to **ng-template**) can only contain one component. This is why you need a Layout component, which will contain all the other components.

The full template should look something like this:

**heroes.component.tns.html**

```TypeScript
<ActionBar title="My Heroes" class="action-bar">
</ActionBar>

<StackLayout>
  <GridLayout rows="auto" columns="auto, *, auto">
    <Label text="Hero name:" class="h2 m-l-5"></Label>
    <TextField col="1" hint="enter hero name..." #heroName class="m-x-5"></TextField>
    <Button col="2" text="add" (tap)="add(heroName.text); heroName.text=''" class="btn btn-primary"></Button>
  </GridLayout>

  <ListView [items]="heroes" class="list-group" height="100%">
    <ng-template let-hero="item">
      <GridLayout columns="auto, *, auto" nsRouterLink="/detail/{{hero.id}}">
        <Label col="0" [text]="hero.id" class="list-group-item"></Label>
        <Label col="1" [text]="hero.name" class="list-group-item"></Label>
        <Button col="2" text="X" (tap)="delete(hero)" class="btn btn-primary"></Button>
      </GridLayout>
    </ng-template>
  </ListView>
</StackLayout>
```

<!--#### Update the Styling

Then, update the {N} styling.

**heroes.component.tns.css**

```CSS
// need to get the style here
```-->

#### Heroes Component Ready

This is how you migrate the **HeroesComponent**.
Your mobile app should look something like this:
<!--TODO: prepare the GIFs -->
> Android GIF
> 
> iOS GIF


## Migrate HeroDetailComponent

Next up you should migrate the **HeroDetailComponent**.

The steps are the same as before.

1. migrate the component:
	
	```bash
	ng g migrate-component --name=hero-detail
	```
	
1. update the navigation

	Add a path for the **HeroDetailComponent** to the routes:

	**app-routing.module.tns.ts**

	```bash
	import { HeroDetailComponent } from './hero-detail/hero-detail.component';
	
	export const routes: Routes = [
	  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
	  { path: 'heroes', component: HeroesComponent },
	  { path: 'detail/:id', component: HeroDetailComponent },
	];
	```

1.  update the NativeScript template

	<!--
	```html
	<div *ngIf="hero">
	  <h2>{{hero.name | uppercase}} Details</h2>
	  <div><span>id: </span>{{hero.id}}</div>
	  <div>
	    <label>name:
	      <input [(ngModel)]="hero.name" placeholder="name"/>
	    </label>
	  </div>
	  <button (click)="goBack()">go back</button>
	  <button (click)="save()">save</button>
	</div>
	```
-->
	**hero-detail.component.tns.html**

	```html
	<ActionBar title="{{hero?.name | uppercase}} Details" icon="" class="action-bar">
	</ActionBar>
	
	<StackLayout *ngIf="hero">
	  <Label text="ID: {{hero.id}}" class="h1 text-center"></Label>
	  <TextField [(ngModel)]="hero.name" hint="name" class="input input-border"></TextField>
	  
	  <Button text="Go Back" (tap)="goBack()" class="btn btn-primary"></Button>
	  <Button text="Save" (tap)="save()" class="btn btn-primary"></Button>
	</StackLayout>
	```
	
	<!--1. Update the stylesheet

	**hero-detail.component.tns.css**
	
	```css
	//TODO: need the CSS here
	```
-->

1. Test the component

	Run the app and navigate to a hero, navigate back and then navigate to another hero. This should look something like this:
	
	> TODO: add a demo GIF 

## Migrate DashboardComponent and HeroSearchComponent

The **DashboardComponent** uses the the **HeroSearchComponent**, which means that you should migrate both of the components at the same time.

1. migrate both components

	```bash
	ng g migrate-component --name=dashboard
	ng g migrate-component --name=hero-search
	```
	
1. update the navigation

	Add a path for the **DashboardComponent** to the routes.
	However, you don't need a path for the **HeroSearchComponent**, as it is used directly  via the **<app-hero-search>** selector.

	**app-routing.module.tns.ts**

	```bash
	import { DashboardComponent } from './dashboard/dashboard.component';
	
	export const routes: Routes = [
	  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
	  { path: 'heroes', component: HeroesComponent },
	  { path: 'detail/:id', component: HeroDetailComponent },
	  { path: 'dashboard', component: DashboardComponent },
	];
	```

1.  update the NativeScript templates

	**dashboard.component.tns.html**

	```html
	<ActionBar title="Top Heroes" icon="" class="action-bar">
	</ActionBar>
	
	<GridLayout rows="*, *">
	  <ListView row="0" [items]="heroes" class="list-group">
	    <ng-template let-hero="item">
	      <Button [text]="hero.name" nsRouterLink="/detail/{{hero.id}}" class="btn btn-primary"></Button>
	    </ng-template>
	  </ListView>
	  <StackLayout row="1">
	    <app-hero-search row="1"></app-hero-search>
	  </StackLayout>
	</GridLayout>
	```

	**hero-search.component.tns.ts**

	```html
	<StackLayout>
	  <Label text="Hero Search" class="h1 text-center"></Label>
	
	  <TextField #heroName
	    hint="enter hero-name"
	    autocorrect="false"
	    class="m-x-5"
	    (loaded)="heroName.text = ''"
	    (textChange)="search($event.value)">
	  </TextField>
	  <ListView [items]="heroes$ | async" class="list-group">
	    <ng-template let-hero="item">
	      <Label [text]="hero.name" nsRouterLink="/detail/{{hero.id}}" class="list-group-item"></Label>
	    </ng-template>
	  </ListView>
	</StackLayout>
	```
	
	> Here is the interesting part. You can use the `<app-hero-search>` selector for both the web and mobile apps. As the **HeroSearchComponent** is platform agnostic.
	
	<!--1.  update the NativeScript stylesheets-->

1. Test the components

	At this point, your mobile app doesn't have a way to navigate to the Dashboard. The easiest way to test it, is to change the default **redirectTo** path, to `'/dashboard'`. Like this:

	**app-routing.module.tns.ts**
	
	```TypeScript
	export const routes: Routes = [
	  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
	  { path: 'heroes', component: HeroesComponent },
	  { path: 'detail/:id', component: HeroDetailComponent },
	  { path: 'dashboard', component: DashboardComponent },
	];
	```
	
	But, make sure to put it back, once you finished testing the app.
	
	Your app should look something like this:
	> TODO: add a demo GIF

## SideDrawer Navigation

What you are missing now is a nice way to navigate between the Heroes page and the Dashboard. This usually comes down to the choice between a  **TabView** or a **SideDrawer**.

For the purpose of this tutorial, you will use a **SideDrawer** with a set of buttons to navigate to the Heroes page and to the Dashboard page.

### Adding nativescript-ui-sidedrawer to your project

To install the **NativeScript SideDrawer**, run the following command:

```bash
tns plugin add nativescript-ui-sidedrawer
```

### Update the {N} AppModule

Then you need to add the **NativeScriptUISideDrawerModule** to the **NativeScript AppModule**, like this:

**app.module.tns.ts**

```TypeScript
import { NativeScriptUISideDrawerModule } from 'nativescript-ui-sidedrawer/angular';

@NgModule({
  imports: [
    ...
    NativeScriptUISideDrawerModule,
  ]
  ...
})
export class AppModule { }
```

### Add the SideDrawer

Adding the SideDrawer to your project is really easy. Just like the navigation for the web project is defined in **app.component.html**, you can add the SideDrawer to **app.compoment.tns.html**, like this:

**app.component.tns.html**

```html
<RadSideDrawer>
  <GridLayout tkDrawerContent rows="*" class="sidedrawer sidedrawer-left">
    <StackLayout class="sidedrawer-content">
      <Button text="heroes" nsRouterLink="/heroes" (tap)="closeDrawer()" clearHistory="true" class="btn btn-primary"></Button>
      <Button text="dashboard" nsRouterLink="/dashboard" (tap)="closeDrawer()" clearHistory="true" class="btn btn-primary"></Button>
    </StackLayout>
  </GridLayout>

  <page-router-outlet tkMainContent class="page page-content"></page-router-outlet>
</RadSideDrawer>
```

Note, that the `<GridLayout>` contains the SideDrawer content (this is where you need to add all your navigation links). While, the `<page-router-outlet>` is where the page content is loaded.

Additionally, each button uses the combination of `nsRouterLink='/path'` and `clearHistory="true"`. This is to stop iOS from adding a back button in the ActionBar, or to stop Android from navigating back, when the user hits the back button on the device.

Finally, each button calls the `closeDrawer()` function, so that after the app navigates, the SideDrawer will close itself gracefully, thus giving a nice UX experience for your users.

```html
<Button
	text="heroes"
	nsRouterLink="/heroes"
	clearHistory="true"
	(tap)="closeDrawer()"
	class="btn btn-primary">
</Button>
```

However, you need to add the `closeDrawer()` function to the **app.component.tns.ts**, like this:

```TypeScript
import { Component } from '@angular/core';
import * as app from 'tns-core-modules/application';
import { RadSideDrawer } from 'nativescript-ui-sidedrawer';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})

export class AppComponent {
  closeDrawer() {
    const sideDrawer = <RadSideDrawer>app.getRootView();
    sideDrawer.closeDrawer();
  }
}
```

### Test the SideDrawer

Now you should be able to slide with your finger from the left of the screen, which will show the SideDrawer. When you tap on any of the buttons, it should close the drawer and navigate to that page.

> TODO: add a demo GIF

> ...... TO BE USED LATER ...... TO BE USED LATER ...... TO BE USED LATER ...... TO BE USED LATER

# Shared Navigation

## Split Routes
In this state the routes are split into two separate configurations. Which allows you to work with different navigation **routes** for web and mobile.

This is why when you run a NativeScript project now, it will start by showing the **HomeComponent**.

> This is usually useful, while you are in the process of migrating your web components into a code-sharing structure. Once this is complete, you could switch to a singe routes configuration for both web and mobile.

> Also, if you expect your web app to have a different set of pages to your mobile app, then you could keep the routes separate. For example, your web app could have an admin screen, which might not be required in the mobile app. In this case it makes sense to have two sets of navigation configurations.

The process here is that you should add navigation paths to routes array in app-routing.module.tns.ts, as you migrate each of your page components.

## Shared Routes
However, for this project you can easily work with a single **routes** configuration.

To achieve that, you need to move the **routes** configuration to a shared file, and move, import and use the shared routes.

### Step 1 - Create a shared file routes file.

Create a new file called **app.routes.ts** and copy the **routes** configuration from **app-routing.module.ts**.

Make sure to **export** routes, and also import all the components

**app.routes.ts**

```
import { Routes } from '@angular/router';
import { HeroesComponent } from './heroes/heroes.component';
import { DashboardComponent } from './dashboard/dashboard.component';
import { HeroDetailComponent } from './hero-detail/hero-detail.component';

export const routes: Routes = [
  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
  { path: 'heroes', component: HeroesComponent },
  { path: 'dashboard', component: DashboardComponent },
  { path: 'detail/:id', component: HeroDetailComponent },
];
```

### Step 2 - Update both **app-routing** with the shared **routes** property

Replace the **routes** property definition with the imported one from **'./app.routes'** in both **app-routing** files. Then clean up any unused imports.

**app-routing.module.ts**

```
import { NgModule } from '@angular/core';
import { RouterModule } from '@angular/router';
import { routes } from './app.routes';

@NgModule({
  imports: [
    RouterModule.forRoot(routes)
  ],
  exports: [
    RouterModule
  ],
})
export class AppRoutingModule { }
```

**app-routing.module.tns.ts**

```
import { NgModule } from '@angular/core';
import { NativeScriptRouterModule } from 'nativescript-angular/router';
import { routes } from './app.routes';

@NgModule({
  imports: [NativeScriptRouterModule.forRoot(routes)],
  exports: [NativeScriptRouterModule]
})
export class AppRoutingModule { }
```

Now you have a single shared navigation configuration for both your web and mobile apps.




<!--
 this part might not be necessary for this part of the tutorial
 TODO: move this for when we need to add a side-drawer navigation
-->
<!--
> **Partial differences**
> 
> In the case where the component class contains **web-specific** code, you will need to extract the **platform-specific** code into a set of **helper files** of **services** and keep only the shared code. You can read more on handling [partial differences here]([read more here ](https://docs.nativescript.org/angular/code-sharing/code-splitting#partial-differences)).-->

