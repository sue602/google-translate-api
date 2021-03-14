# google-translate-api
[![Actions Status](https://github.com/plainheart/google-translate-api/workflows/autotests/badge.svg)](https://github.com/plainheart/google-translate-api/actions)
[![NPM version](https://img.shields.io/npm/v/@plainheart/google-translate-api.svg)](https://www.npmjs.com/package/@plainheart/google-translate-api)
[![NPM Downloads](https://img.shields.io/npm/dm/@plainheart/google-translate-api.svg)](https://npmcharts.com/compare/@plainheart/google-translate-api?minimal=true)
[![License](https://img.shields.io/npm/l/@plainheart/google-translate-api.svg)](https://www.npmjs.com/package/@plainheart/google-translate-api)

A **free** and **unlimited** API for Google Translate :dollar: :no_entry_sign: for Node.js.

## Features 

- Auto language detection
- Spelling correction
- Language correction 
- Fast and reliable – it uses the same servers that [translate.google.com](https://translate.google.com) uses
- Multiple endpoints

## Why this fork?
This fork of original [vitalets/google-translate-api](https://github.com/vitalets/google-translate-api) contains several improvements:

- Added support for specifying the endpoints to be used.
- Added support for random endpoint and endpoint fallback.  
- Added two new endpoints `dictExt`(dict-chrome-ex) & `api`(translate.googleapis.com).

## Install 

```
npm install @plainheart/google-translate-api
```

## Usage

From automatic language detection to English:

```js
const translate = require('@plainheart/google-translate-api');

translate('Ik spreek Engels', {to: 'en'}).then(res => {
    console.log(res.text);
    //=> I speak English
    console.log(res.from.language.iso);
    //=> nl
}).catch(err => {
    console.error(err);
});
```

> Please note that maximum text length for single translation call is **5000** characters. 
> In case of longer text you should split it on chunks, see [#20](https://github.com/vitalets/google-translate-api/issues/20).

From English to Dutch with a typo:

```js
translate('I spea Dutch!', {from: 'en', to: 'nl', endpoints: ['website']}).then(res => {
    console.log(res.text);
    //=> Ik spreek Nederlands!
    console.log(res.from.text.autoCorrected);
    //=> true
    console.log(res.from.text.value);
    //=> I [speak] Dutch!
    console.log(res.from.text.didYouMean);
    //=> false
}).catch(err => {
    console.error(err);
});
```

Sometimes, the API will not use the auto corrected text in the translation:

```js
translate('I spea Dutch!', {from: 'en', to: 'nl', endpoints: ['website']}).then(res => {
    console.log(res);
    console.log(res.text);
    //=> Ik spea Nederlands!
    console.log(res.from.text.autoCorrected);
    //=> false
    console.log(res.from.text.value);
    //=> I [speak] Dutch!
    console.log(res.from.text.didYouMean);
    //=> true
}).catch(err => {
    console.error(err);
});
```

You can also add languages in the code and use them in the translation:

``` js
translate = require('google-translate-api');
translate.languages['sr-Latn'] = 'Serbian Latin';

translate('translator', {to: 'sr-Latn'}).then(res => ...);
```

## Proxy
Google Translate has request limits. If too many requests are made, you can either end up with a 429 or a 503 error.
You can use **proxy** to bypass them:
```js
const tunnel = require('tunnel');
translate('Ik spreek Engels', {to: 'en'}, {
    agent: tunnel.httpsOverHttp({
    proxy: { 
      host: 'whateverhost',
      proxyAuth: 'user:pass',
      port: '8080',
      headers: {
        'User-Agent': 'Node'
      }
    }
  }
)}).then(res => {
    // do something
}).catch(err => {
    console.error(err);
});
```

## Does it work from web page context?
No. `https://translate.google.com` does not provide [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) http headers allowing access from other domains.

## API

### translate(text, [options], [gotOptions])

#### text

Type: `string`

The text to be translated

#### options

Type: `object`

##### from
Type: `string` Default: `auto`

The `text` language. Must be `auto` or one of the codes/names (not case sensitive) contained in [languages.js](https://github.com/plainheart/google-translate-api/blob/master/languages.js)

##### to
Type: `string` Default: `en`

The language in which the text should be translated. Must be one of the codes/names (case sensitive!) contained in [languages.js](https://github.com/plainheart/google-translate-api/blob/master/languages.js).

##### raw
Type: `boolean` Default: `false`

If `true`, the returned object will have a `raw` property with the raw response (`string`) from Google Translate.

##### client
Type: `string` Default: `"t"`

Query parameter `client` used in API calls. Can be `t|gtx`.

Note that this option only works for `website` endpoint.

##### tld
Type: `string` Default: `"com"`

TLD for Google translate host to be used in API calls: `https://translate.google.{tld}`.

Note that this option only works for `website` endpoint.

##### endpoints
Type: `Array<string>` Default: `['website', 'dictExt', 'api']`

The translation endpoints. Can be `website|dictExt|api`.

##### randomEndpoint
Type: `boolean` Default: `false`

Whether to use a random endpoint.

##### endpointFallback
Type: `boolean` Default: `true`

If `true`, will try the next endpoint automatically when current endpoint failed.

#### gotOptions
Type: `object`

The got options: https://github.com/sindresorhus/got#options

### Returns an `object`:
- `text` *(string)* – The translated text.
- `pronunciation` *(string)* – The pronunciation of translated text.
- `from` *(object)*
  - `language` *(object)*
    - `didYouMean` *(boolean)* - `true` if the API suggest a correction in the source language
    - `iso` *(string)* - The [code of the language](https://github.com/plainheart/google-translate-api/blob/master/languages.js) that the API has recognized in the `text`
  - `text` *(object)*
    - `autoCorrected` *(boolean)* – `true` if the API has auto corrected the `text`
    - `value` *(string)* – The auto corrected `text` or the `text` with suggested corrections
    - `didYouMean` *(boolean)* – `true` if the API has suggested corrections to the `text`
- `raw` *(string|object)* - If `options.raw` is true, the raw response from Google Translate servers..

Note that `res.from.text` will only be returned if `from.text.autoCorrected` or `from.text.didYouMean` equals to `true`. In this case, it will have the corrections delimited with brackets (`[ ]`):

```js
translate('I spea Dutch', { endpoints: ['website'] }).then(res => {
    console.log(res.from.text.value);
    //=> I [speak] Dutch
}).catch(err => {
    console.error(err);
});
```
Otherwise, it will be an empty `string` (`''`).

## License

MIT © [Vitaliy Potapov](https://github.com/vitalets), forked and maintained by [plainheart](https://github.com/plainheart).
