# micro-cacheable
[![NPM version](https://img.shields.io/npm/v/micro-cacheable.svg)](https://www.npmjs.com/package/micro-cacheable)
[![Build Status](https://travis-ci.org/fmiras/micro-cacheable.svg?branch=master)](https://travis-ci.org/fmiras/micro-cacheable)
[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)
[![XO code style](https://img.shields.io/badge/code_style-XO-5ed9c7.svg)](https://github.com/xojs/xo)
[![Join the community on Spectrum](https://withspectrum.github.io/badge/badge.svg)](https://spectrum.chat/micro)
[![Greenkeeper badge](https://badges.greenkeeper.io/fmiras/micro-cacheable.svg)](https://greenkeeper.io/)

micro-cacheable is an utility for data caching focused on [micro framework](https://github.com/zeit/micro). The problem this package solves is to save the already requested data in-memory to have it available for a configurated time without processing it again.

## Usage

```bash
cd my-micro-project/
npm install --save micro-cacheable
```

and add use the package like this:

```javascript
// index.js
const cache = require('micro-cacheable')

const microFn = (req, res) => {
  return new Date()
}

module.exports = cache(60 * 60 * 1000, microFn)
```

then just run your microservice normally and it will return the same result for an hour (first param as miliseconds) unless that you change the request url.

#### A more useful example:

Let's say that we need a microservice that receives a name (string) and search data of a person on 3 or 4 APIs:
```javascript
const { parse } = require('url')
const fetch = require('node-fetch')

module.exports = async (req, res) => {
  const { searchName } = parse(req.url, true).query
  const facebookData = await fetch('https://someapi1.com')
  const githubData = await fetch('https://someapi2.com')
  const financialData = await fetch('https://someapi3.com')
  return { facebookData, githubData, financialData }
}
```

This microservice would fetch 3 APIs every time it receives a request. Probably, in some cases, if the microservice receive the same name it will return the same data, at least for the same day, so you can just add micro-cacheable like this:
```javascript
const { parse } = require('url')
const fetch = require('node-fetch')
const cache = require('micro-cacheable')

const microFn = async (req, res) => {
  const { searchName } = parse(req.url, true).query
  const facebookData = await fetch('https://someapi1.com')
  const githubData = await fetch('https://someapi2.com')
  const financialData = await fetch('https://someapi3.com')
  return { facebookData, githubData, financialData }
}

module.exports = cache(24 * 60 * 60 * 1000, microFn) // One day data caching
```
### Mongo Support
micro-cacheable supports Mongo (and hopefully Redis in the future) because in-memory data cache can't scale horizontally and if you work with microservices you will loose that advantage. So you can avoid that problem setting the `MONGO_URL` and `MONGO_DB` enviroment variables so all your microservice's instances use the same cache.

## Why?
I worked on a project with micro using it for making web-scrapping workers that take too long the first time to get the data, and users requested often the same data so with this I can save a lot of requests, processing and time making requests of +5000ms only take 50ms.

## Contributing

1. [Fork](https://help.github.com/articles/fork-a-repo/) this repository to your own GitHub account and then [clone](https://help.github.com/articles/cloning-a-repository/) it to your local device
2. Link the package to the global module directory: `npm link`
3. Within the module you want to test your local development instance of micro-cacheable, just link it to the dependencies: `npm link micro-cacheable`. Instead of the default one from npm, node will now use your clone of micro-cacheable!

## Credits

Thanks to [ZEIT](https://zeit.co) Team for giving us [micro](https://github.com/zeit/micro) to make our life easier!
