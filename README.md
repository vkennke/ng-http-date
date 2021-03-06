# NgHttpDate

  [![NPM Version][npm-image]][npm-url]
  [![NPM Downloads][downloads-image]][downloads-url]
  
[npm-image]: https://img.shields.io/npm/v/ng-http-date-core.svg
[npm-url]: https://npmjs.org/package/ng-http-date-core
[downloads-image]: https://img.shields.io/npm/dm/ng-http-date-core.svg
[downloads-url]: https://npmjs.org/package/ng-http-date-core  

## What?

NgHttpDate is an [Angular](https://github.com/angular/angular) library that detects **dates** in HTTP JSON reponses and modifies the responses to contain `Date` objects (or Day.js or Moment via Plugin) instead of `String` objects.

## Installation

Install `ng-http-date` via npm or yarn or any other compatible packet manager:

```shell script
npm i ng-http-date
```

If you're not using the current Angular version you should use an older version of `ng-http-date`:

| Angular      | 11.x | 10.x | 9.x | 8.2.x | older         |
|--------------|------|------|-----|-------|---------------|
| ng-http-date | 11.x | 10.x | 9.x | 8.x   | not supported |


Import the `NgHttpDateModule` in your root module:

```typescript
import {NgHttpDateModule} from 'ng-http-date-core';

@NgModule({
  declarations: [...],
  imports: [
    ...
    NgHttpDateModule.forRoot(),
    ...
  ],
  providers: [...],
  bootstrap: [...]
})
export class AppModule {
  ...
}
```

## Why?

Angular is not converting dates in JSON to EcmaScript `Date` objects. Even if you declare properties of your HTTP call return type as `Date` they're just `String`s at runtime:

```typescript
export class ResultOfHttpCall {
  someProperty: Date;
}

...

this.http.get<ResultOfHttpCall>('someURL').subscribe((result: ResultOfHttpCall) => {
  typeof result.someProperty; // string
  result.someProperty.toISOString(); // ERROR: someProperty.toISOString is not a function
}
```

## How?

NgHttpDate registers an `HttpInterceptor` which tests each property of the response via an ISO8601 regex. 
When a date is detected, the response is modified to create a `Date` object for that property.

## Configuration

Per default only ISO8601 date strings are converted. 
In addition, all strings that correspond to an ISO8601 date are converted, which means that e.g. '2000' or '0001' also become a date.
To configure this behavior you can pass a configuration object to the modules forRoot method:

```typescript
import {NgHttpDateModule} from 'ng-http-date-core';

@NgModule({
  declarations: [...],
  imports: [
    ...
    NgHttpDateModule.forRoot({iso8601Only: false}), // converts RFC 2822 timestamps
    ...
  ],
  providers: [...],
  bootstrap: [...]
})
export class AppModule {
  ...
}
```

You have the following configuration options:
- `iso8601Only` (default `false`) - pass `true` to convert every string that can be converted to a `Date` (or `moment` or `Dayjs`) object. This has a negative impact on performance.
- `customRegex` (default `undefined`) - pass a custom `RegExp` which decides if a string is converted or not. This will set `iso8601Only` to false.
- `debug` (default `false`) - pass `true` to print some more or less helpful debug messages to the browser console.

## Performance

The core library is 2 kB gzipped. The gzipped size of the plugins is 1 kB each. 
The detection and creation of `Date` objects has nearly no performance impact. For example, modifying a [small size nested JSON object](https://github.com/vkennke/ng-http-date/blob/master/projects/ng-http-date-core/src/lib/ng-http-date.interceptor.spec.ts#L82-L104) 1.000.000 times is done in ~ 8 seconds.
That's 0.008 ms per object.

## Plugins

If you want NgHttpDate to use Day.js objects or Moment objects instead of EcmaScript Dates you can install the appropriate plugin:

[Installation of ng-http-date-dayjs](https://github.com/vkennke/ng-http-date/tree/master/projects/ng-http-date-dayjs)

[Installation of ng-http-date-moment](https://github.com/vkennke/ng-http-date/tree/master/projects/ng-http-date-moment)

## What's next?

Currently only ISO8601 compliant dates are recognized and converted. It's planned for the next minor release to provide a way to configure this behavior. 
