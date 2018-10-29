---
layout: post
title: Angular Internationalization
tags:
- angular
- internationalization
- i18n
- localization
- globalization
---
**NOTE**: The source code is available at <https://github.com/NadeemAfana/angular-internationalization>

### Introduction
There are different ways to globalize an Angular application. One way, as mentioned by the official Angular documentation, is to compile 
multiple instance of the application and server them under a different folder (such as `/en/` and `/es/`).  In this post, I am going to take a different approach 
that has the following features:
- Single instance of application running instead of multiple ones. This saves comilation time, space and deployment time.
- The locale Id does not need to be part of the url. It can be stored in `localStorage` for exmaple.
- Strongly typed localization. The localization text is supplied via declared types.
- Detection of the default browser language and localizes the app based on it.
- `LOCALE_ID` is updated so that other parts of Angular such a `DatePipe` will render values for the current locale.
- Right to left languages are supported by setting `dir='rtl'` automatically on the `html` tag.
- Support for both neutral locales (ie `en`) and specific ones (ie `en-US`).
- This approach does not use [ICU Message Format](http://userguide.icu-project.org/formatparse/messages). However, it is easy 
to mimic the same behavior in the component itself or in a global helper class.


### Globalization and Localization
Internationalization involves Globalization and Localization. Globalization is the process of designing applications that support different cultures. Localization is the process of customizing an application for a given culture.

The format for the locale name is `<languagecode2>-<country/regioncode2>`, where `<languagecode2>` is the language code and `<country/regioncode2>` is the subculture code. Examples include `es-CL` for Spanish (Chile) and `en-US` for English (United States).

Anyway, Internationalization is often abbreviated to `I18N`. The abbreviation takes the first and last letters and the number of letters between them, so 18 stands for the number of letters between the first `I` and the last `N`. The same applies to Globalization (`G11N`), and Localization (`L10N`).

Now, let’s review the terms used so far:

*    **Globalization** (G11N): The process of making an application support different languages and regions.
*    **Localization** (L10N): The process of customizing an application for a given language and region.
*    **Internationalization** (I18N): Describes both globalization and localization.
*    **Culture**: It is a language and, optionally, a region.
*    **Locale**: A locale is the same as a culture.
*    **Neutral locale**: A locale that has a specified language, but not a region. (e.g. "en", "es")
*    **Specific locale**: A locale that has a specified language and region. (e.g. "en-US", "en-GB", "es-CL")

### Why do we need a region? Isn't a language alone enough?

You might not need a region at all. It is true that English in the United States is not the same as English in the United Kingdom but if your application just shows English text readable to people from these English-speaking countries, you will not need a region. The problem arises when you need to deal with numbers, dates, and currencies. For example, compare the following output for two different Spanish-speaking regions (Chile, Mexico)

```
26-07-2011  // Date in es-CL, Spanish (Chile)
$5.600,00   // Currency in es-CL, Spanish (Chile)
 
26/07/2011  // Date in es-MX, Spanish (Mexico)
$5,600.00   // Currency in es-MX, Spanish (Mexico)
```

You can notice the difference in date and currency format. The decimal separator in each region is different and can confuse people in the other region. If a Mexican user types one thousand in their culture "1,000", it will be interpreted as 1 (one) in a Chilean culture website. We mainly need regions for this type of reasons.


### Globalizing Angular Apps
We want to localize the following form with three locales
- `en-US` Default locale
- `es` Spanish (Neutral locale)
- `ar` Arabic (Neutral locale)
![](/images/posts/archived/angular-i18n-1.png)

Thus a global class is needed to hold the localized text for each locale. The app will use the following class to access the localized text.

```typescript
export abstract class Resources {
    public static RightToLeft = 'ltr';

    public static DateIs: string = null;
    public static FirstName: string = null;
    public static LastName: string = null;
    public static Age: string = null;
    public static AgeRange: string = null;
    public static AgeRequired: string = null;
    public static Create: string = null;
    public static FirstNameLong: string = null;
    public static FirstNameRequired: string = null;
    public static LastNameLong: string = null;
    public static LastNameRequired: string = null;
}
```
Next, we need a localized version for each locale. Each locale will have its own `.js` file and can be placed inside the `assets` folder:

**`resources.en-us.js`**
```javascript
export const resources = {
        'DateIs': 'The date is: ',
        'FirstName': 'First name',
        'LastName': 'Last name',
        'Age': 'Age',
        'AgeRange': 'Must be between 10 and 130',
        'AgeRequired': 'Age is required',
        'Create': 'Create',
        'FirstNameLong': 'Must be less than 50 characters',
        'FirstNameRequired': 'First name is required',
        'LastNameLong': 'Must be less than 50 characters',
        'LastNameRequired': 'Last name is required'
}
```
**`resources.es.js`**
```javascript
export const resources = {
        'DateIs': 'La fecha es: ',
        'FirstName': 'Primer nombre',
        'LastName': 'Apellido',
        'Age': 'Edad',
        'AgeRange': 'Debe ser entre 10 y 130',
        'AgeRequired': 'Debe ingresar su edad',
        'Create': 'Crear',
        'FirstNameLong': 'Debe ser menos de 50 caracteres',
        'FirstNameRequired': 'Debe ingresar su nombre',
        'LastNameLong': 'Debe ser menos de 50 caracteres',
        'LastNameRequired': 'Debe ingresar su apellido'
}
```
**`resources.ar.js`**
```javascript
export const resources = {
        RightToLeft: 'rtl',

        'DateIs': 'التاريخ الآن :',
        'FirstName': 'الإسم',
        'Age': 'العمر',
        'AgeRange': 'يجب أن يكون بين 10 و 130',
        'AgeRequired': 'أدخل العمر',
        'Create': 'إنشاء',
        'FirstNameLong': 'يجب أن يكون أقل من 50 حرف',
        'FirstNameRequired': 'أدخل الإسم',
        'LastName': 'إسم العائلة',
        'LastNameLong': 'يجب أن يكون أقل من 50 حرف',
        'LastNameRequired': 'أدخل إسم العائلة'        
}
```

Now we need a helper class `LocaleHelper` that allows us to get and update the current locale and other related information.
```typescript
export abstract class LocaleHelper {
    public static defaultLocaleId = 'en-US';
    public static implementedLocales = ['ar', 'es', LocaleHelper.defaultLocaleId];


    public static setCurrentLocale(localeId: string) {
        localStorage.setItem('__localeId', localeId);
    }

    public static isDefaultLocaleSet(): boolean {
        return LocaleHelper.getCurrentLocale() === this.defaultLocaleId;
    }

    public static getCurrentLocale(): string {
        // Retrieve localeId from `localStorage` if any; otherwise, default to 'en-US'.
        // The first time the app loads, check the browser language.
        const storedLocaleId = <string>localStorage.getItem('__localeId');
        if (storedLocaleId == null) {
            let partialLocaleMatch = null;
            // tslint:disable-next-line:forin
            for (const id in LocaleHelper.implementedLocales) {
                const implemetedLocaleId = LocaleHelper.implementedLocales[id];
                if (navigator.language === implemetedLocaleId) {
                    // Exact match, return.
                    return implemetedLocaleId;
                } else if (navigator.language.startsWith(implemetedLocaleId)) {
                    // For example, browser has `es-CL` and the implemented locale is `es`.
                    partialLocaleMatch = implemetedLocaleId;
                } else if (implemetedLocaleId.startsWith(navigator.language)) {
                    // For example, browser has `es` and the implemented locale is `es-CL`.
                    partialLocaleMatch = implemetedLocaleId;
                }
            }
            if (partialLocaleMatch != null) { return partialLocaleMatch; }
        }
        return storedLocaleId || this.defaultLocaleId;
    }
}
```
The locale will be stored in a field called `__localeId` in `localStorage`. This storage mechanism can be changed easily as required.
The first time a user visits the app, we try to guess the locale from their browser instead of having them choose the locale explcitly, assuming their browser locale is implemented or has a close match. For example, if the browser's locale is `es-CL`, we could still use the implemented `es` neutral locale instead of the default `en-US` one. This makes the app more user friendly.

The Angular `LOCALE_ID` token needs to be updated accordingly to reflect the current locale. This can be done inside the `app.module.ts` under the `providers` section:
```typescript
providers: [{
    provide: LOCALE_ID, useFactory: () => LocaleHelper.getCurrentLocale()
  }],
```
Updating this token is important for Angular i18n pipes (eg `DatePipe` and `CurrencyPipe`).

Angular has some locale files that need to be imported for each of the implemented locales. Additionally, the localized text files need to be fetched. All this can be done inside `AppModule`:

```typescript
export class AppModule {
  constructor() {
    // Pre-load all the needed locales.
    registerLocaleData(localeEs, 'es', localeEsExtra);
    registerLocaleData(localeAr, 'ar', localeArExtra);

    // There are other ways to loads a module dynamically.
    import( `../assets/resources.${LocaleHelper.getCurrentLocale().toLowerCase()}.js`).then((r) => {

      // Load `Resources` with values.
      for (const key in r.resources) {
        if (r.resources.hasOwnProperty(key)) {
          Resources[key] = r.resources[key];
        }
      }

      // Is the current language right to left?
      document.documentElement.dir = Resources.RightToLeft;
    });
  }
}
```
The last piece is having a base component that provides access to the localized resources

```typescript
export class LocalizedComponent {
    public resources = Resources;
    public localeId: string = null;

    constructor() {
        this.localeId = LocaleHelper.getCurrentLocale();
      }
}
```

### Try It Out
We could use `AppComponent` for a proof of concept:

```typescript

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent extends LocalizedComponent {
  public now = Date.now();

  public firstName: string = null;

  // Implemented languages
  public languages: Language[] = [{ name: 'English', localeId: 'en-US' },
                                  { name: 'Español', localeId: 'es' },
                                  { name: 'العربية', localeId: 'ar' }];
  profileForm: FormGroup;

  constructor() {
    super();

    this.profileForm  = new FormGroup({
      'firstName': new FormControl('', [
        Validators.required,
        Validators.maxLength(50)
      ]),
      'lastName': new FormControl('', [
        Validators.required,
        Validators.maxLength(50)
      ]),
      'age': new FormControl('', [
        Validators.required,
        Validators.max(130),
        Validators.min(10)
      ])
    });
  }

  public createClicked($event): void {
    Object.keys(this.profileForm.controls).forEach(field => {
      const control = this.profileForm.get(field);
      control.markAsTouched({ onlySelf: true });
    });
  }

  public languageSelected($event, language: Language): void {
    // Set the new language.
    LocaleHelper.setCurrentLocale(language.localeId);

    // Reload page.
    window.location.reload();
  }
}

interface Language {
  name: string;
  localeId: string;
}
```

and its template 
```
<div class="col-md-4 col-md-offset-4">
  <h4>{{ resources.DateIs }} {{ now | date:'fullDate' }}</h4>
  <hr/>
  <h4>Choose your language</h4>
  <form>
    <div class="form-group">
      <div *ngFor="let language of languages">
        <label for="language_{{language.localeId}}">
          <input id="language_{{language.localeId}}" [value]='language.localeId' type="radio" name="language" [checked]="( localeId == language.localeId )"
            (change)="languageSelected($event, language)"> {{language.name}}
        </label>
      </div>
    </div>
  </form>

  <form>
    <div class="form-group" [formGroup]="profileForm">
      <label for="firstName">{{ resources.FirstName}}</label>
      <input type="text" class="form-control" id="firstName" name="firstName" required formControlName="firstName" />

      <div *ngIf="profileForm.controls.firstName.invalid && (profileForm.controls.firstName.dirty || profileForm.controls.firstName.touched)"
        class="alert alert-danger">
        <div *ngIf="profileForm.controls.firstName.errors.required">
          {{ resources.FirstNameRequired }}
        </div>
        <div *ngIf="profileForm.controls.firstName.errors.maxlength">
          {{ resources.FirstNameLong }}
        </div>
      </div>

      <label for="lastName">{{ resources.LastName}}</label>
      <input type="text" class="form-control" id="lastName" required formControlName="lastName" />
      <div *ngIf="profileForm.controls.lastName.invalid && (profileForm.controls.lastName.dirty || profileForm.controls.lastName.touched)"
        class="alert alert-danger">
        <div *ngIf="profileForm.controls.lastName.errors.required">
          {{ resources.LastNameRequired }}
        </div>
        <div *ngIf="profileForm.controls.lastName.errors.maxlength">
          {{ resources.LastNameLong }}
        </div>
      </div>


      <label for="age">{{ resources.Age}}</label>
      <input type="text" class="form-control" id="age" required formControlName="age" [formControl]="profileForm.controls.age" />
      <div *ngIf="profileForm.controls.age.invalid && (profileForm.controls.age.dirty || profileForm.controls.age.touched)"
        class="alert alert-danger">
        <div *ngIf="profileForm.controls.age.errors.required">
          {{ resources.AgeRequired }}
        </div>
        <div *ngIf="profileForm.controls.age.errors.max || profileForm.controls.age.errors.min">
          {{ resources.AgeRange }}
        </div>
      </div>      
    </div>

    <button type="submit" (click)="createClicked($event)" class="btn btn-success">{{ resources.Create }}</button>

  </form>
</div>
```

Notice how the expression `{{ "{{" }} now | date:'fullDate' }}` renders the localized version of the date.

**Spanish**
![](/images/posts/archived/angular-i18n-2.png)

**Arabic**
![](/images/posts/archived/angular-i18n-3.png)

### How to Store Locale in the Url instead ?
It is possible to store the locale in the url instead of `localStorage`, so that a url like `http://localhost:4200/profile` will look like `http://localhost:4200/es/profile` and `http://localhost:4200/ar/profile`.

Let's say we have the following routes

```typescript
const appRoutes: Routes = [
  { path: 'profile', component: ProfileComponent },
  { path: '',  redirectTo: 'profile', pathMatch: 'full' },
  { path: '**', component: PageNotFoundComponent },

];
```
where `profile` takes us to the form we built in the above steps.

**NOTE**: The source code for this change is fully available in the `store-locale-in-url` branch at 
<https://github.com/NadeemAfana/angular-internationalization>

The implementation is a little tricky. First, `LocaleHelper` needs to support that:

```typescript
export abstract class LocaleHelper {
    public static defaultLocaleId = 'en-US';
    public static implementedLocales = ['ar', 'es', LocaleHelper.defaultLocaleId];


    public static setCurrentLocale(localeId: string) {
        // Set the new locale. Assume localeId is valid.
        const urlLocaleId = LocaleHelper.getCultureFromCurrentUrl();
        if (urlLocaleId) {
            // Replace current locale in url if any.
            if (localeId !== LocaleHelper.defaultLocaleId) {
                window.location.href = window.location.href.replace(`/${urlLocaleId}/`, `/${localeId.toLowerCase()}/`);
            } else {
                window.location.href = window.location.href.replace(`/${urlLocaleId}/`, '/');
            }

        } else {
            // If there is no locale in the url, add one.
            // Do not add one if it is the default locale.
            if (localeId !== LocaleHelper.defaultLocaleId) {
                const newUrl = window.location.href.replace(window.location.pathname,  `/${localeId}` + window.location.pathname);
                if (newUrl !== window.location.href) {
                    window.location.href = newUrl;
                }
            }
        }
    }

    public static isDefaultLocaleSet(): boolean {
        return LocaleHelper.getCurrentLocale() === this.defaultLocaleId;
    }

    private static getCultureFromCurrentUrl(): string {
        // Retrieve localeId from the url if any.
        const matches = window.location.pathname.match(/^\/[a-z]{2}(\-[a-z]{2})?\//gi);
        let urlLocaleId = null;
        if (matches) {
            urlLocaleId = matches[0].replace(/\//gi, '');
        }
        return urlLocaleId;
    }

    public static getCurrentLocale(): string {
        // Retrieve localeId from the url if any; otherwise, default to 'en-US'.
        // The first time the app loads, check the browser language.
        const storedLocaleId = LocaleHelper.getCultureFromCurrentUrl();
        if (storedLocaleId == null) {
            let partialLocaleMatch = null;
            // tslint:disable-next-line:forin
            for (const id in LocaleHelper.implementedLocales) {
                const implemetedLocaleId = LocaleHelper.implementedLocales[id];
                if (navigator.language === implemetedLocaleId) {
                    // Exact match, return.
                    return implemetedLocaleId;
                } else if (navigator.language.startsWith(implemetedLocaleId)) {
                    // For example, browser has `es-CL` and the implemented locale is `es`.
                    partialLocaleMatch = implemetedLocaleId;
                } else if (implemetedLocaleId.startsWith(navigator.language)) {
                    // For example, browser has `es` and the implemented locale is `es-CL`.
                    partialLocaleMatch = implemetedLocaleId;
                }
            }
            if (partialLocaleMatch != null) { return partialLocaleMatch; }
        }
        return storedLocaleId || this.defaultLocaleId;
    }
}
```

Now we need to override the `APP_BASE_HREF` token in `app.module.ts`:

```typescript
provide: APP_BASE_HREF, useFactory: () => {
      // Suppress the default locale from the url
      return LocaleHelper.isDefaultLocaleSet() ? '/' :  `/${LocaleHelper.getCurrentLocale()}/`;
    }
```
I suppressed the default locale (ie `en-US`) from the url on purpose, but that behavior can be changed easily.

![](/images/posts/archived/angular-i18n-4.png)

![](/images/posts/archived/angular-i18n-5.png)

![](/images/posts/archived/angular-i18n-6.png)