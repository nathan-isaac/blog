---
title: "Webpack Cypress Component Tests Setup"
date: 2021-05-29T00:53:48-07:00
draft: false
---

I recently installed the cypress component testing library in a custom webpack project. 
In doing so I ran into a few issues. Here are my steps I used to resolve my issues. Here are my notes on what 
I did to solve these issues.

Note that most of these notes can be found on the [cypress component testing introduction](https://docs.cypress.io/guides/component-testing/introduction) documentation.

## Install Node Packages

Depending on your version of webpack you may need to change the `html-webpack-plugin@^4.0` to `html-webpack-plugin@^5.0`.

```shell
yarn add --dev cypress @cypress/react @cypress/webpack-dev-server webpack-dev-server html-webpack-plugin@^4.0
```

## Customize Cypress Plugin Config

Since custom webpack config included the built in `devserver` I needed to remove it from the webpack config only for cypress tests. Otherwize I would randomly get start errors when running `yarn cypress open-ct`. 

Since we were using webpack's function config to retrieve the environment in the webpack config I hard coded `development` as the webpack environment.

```typescript
// cypress/plugins/index.ts
import { startDevServer } from '@cypress/webpack-dev-server';

/**
 * @type {Cypress.PluginConfig}
 */
module.exports = (on, config) => {
  if (config.testingType === 'component') {
    // Your project's Webpack configuration
    const { devServer, ...webpackConfig } = require('../../webpack.config.js')(
      'development'
    );

    on('dev-server:start', options =>
      startDevServer({ options, webpackConfig })
    );
  }

  config.env.reactDevtools = true;

  return config;
};
```

The example below didn't work for me because the `devserver` config was conflicting with cypress's config.

```js
// EXAMPLE ONLY! DIDN'T USE.
// Reference: https://github.com/cypress-io/cypress/blob/develop/npm/react/examples/webpack-file/cypress/plugins/index.js
module.exports = (on, config) => {
  require('@cypress/react/plugins/load-webpack')(on, config, {
    // from the root of the project (folder with cypress.json file)
    webpackFilename: 'webpack.config.js',
  })

  // IMPORTANT to return the config object
  // with the any changed environment variables
  return config
}
```

## Cypress Config

Add the following config to `cypress.json` in the root of the project.

```json
{
  "component": {
    "video": false,
    "componentFolder": "src",
    "testFiles": "**/*spec.{js,jsx,ts,tsx}"
  }
}
```

## Creating first test

```tsx
// src/components/Example/Example.tsx
import React from 'react';

function Example() {
  return <div>Example</div>;
}

export default Example;
```

```tsx
// src/components/Example/Example.spec.tsx
import React from 'react';
import { mount } from '@cypress/react';
import Example from './Example';

it('should see example', () => {
  mount(<Example />);

  cy.get('div').contains('Example');
});
```

## Running tests

To open visual test runner use `open-ct`.

```shell
yarn cypress open-ct
```

To run in CI or headlessly use `run-ct`.


```shell
yarn cypress run-ct
```

## References

- [Cypress Introduction](https://docs.cypress.io/guides/component-testing/introduction)
- [Load Webpack Plugin](https://github.com/cypress-io/cypress/blob/develop/npm/react/plugins/load-webpack/index.js) from the [react examples](https://github.com/cypress-io/cypress/tree/develop/npm/react)

