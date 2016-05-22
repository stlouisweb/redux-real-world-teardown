# Redux Real-World Example Project Teardown

## Project set up
### Boilerplate

At the route of the project are the following "boilerplate" files that can be used in your own React-Redux projects in more-less the same form.

#### ./package.json

```json
{
  "name": "redux-real-world-example",
  "version": "0.0.0",
  "description": "Redux real-world example",
  "scripts": {
    "start": "node server.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/reactjs/redux.git"
  },
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/reactjs/redux/issues"
  },
  "homepage": "http://redux.js.org",
  "dependencies": {
    "babel-polyfill": "^6.3.14",
    "humps": "^0.6.0",
    "isomorphic-fetch": "^2.1.1",
    "lodash": "^4.0.0",
    "normalizr": "^2.0.0",
    "react": "^0.14.7",
    "react-dom": "^0.14.7",
    "react-redux": "^4.2.1",
    "react-router": "2.0.0",
    "react-router-redux": "^4.0.0-rc.1",
    "redux": "^3.2.1",
    "redux-logger": "^2.4.0",
    "redux-thunk": "^1.0.3"
  },
  "devDependencies": {
    "babel-core": "^6.3.15",
    "babel-loader": "^6.2.0",
    "babel-preset-es2015": "^6.3.13",
    "babel-preset-react": "^6.3.13",
    "babel-preset-react-hmre": "^1.1.1",
    "concurrently": "^0.1.1",
    "express": "^4.13.3",
    "redux-devtools": "^3.1.0",
    "redux-devtools-dock-monitor": "^1.0.1",
    "redux-devtools-log-monitor": "^1.0.3",
    "webpack": "^1.9.11",
    "webpack-dev-middleware": "^1.2.0",
    "webpack-hot-middleware": "^2.9.1"
  }
}
```

#### ./.babelrc

```
{
  "presets": ["es2015", "react"],
  "env": {
    "development": {
      "presets": ["react-hmre"]
    }
  }
}
```

#### ./webpack.config.js

```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = {
  devtool: 'cheap-module-eval-source-map',
  entry: [
    'webpack-hot-middleware/client',
    './index',
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/static/',
  },
  plugins: [
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
  ],
  module: {
    loaders: [
      {
        test: /\.js$/,
        loaders: ['babel'],
        exclude: /node_modules/,
        include: __dirname,
      },
    ],
  },
};
```

#### ./index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Redux real-world example</title>
  </head>
  <body>
    <div id="root">
    </div>
    <script src="/static/bundle.js"></script>
  </body>
</html>
```

#### ./index.js

```javascript
import 'babel-polyfill';
import React from 'react';
import { render } from 'react-dom';
import { browserHistory } from 'react-router';
import { syncHistoryWithStore } from 'react-router-redux';
import Root from './containers/Root';
import configureStore from './store/configureStore';

const store = configureStore();
const history = syncHistoryWithStore(browserHistory, store);

render(
  <Root store={store} history={history} />,
  document.getElementById('root')
);

```

#### server.js

```javascript
var webpack = require('webpack');
var webpackDevMiddleware = require('webpack-dev-middleware');
var webpackHotMiddleware = require('webpack-hot-middleware');
var config = require('./webpack.config');

var app = new (require('express'))();
var port = 3000;

var compiler = webpack(config);
app.use(webpackDevMiddleware(compiler, { noInfo: true, publicPath: config.output.publicPath }));
app.use(webpackHotMiddleware(compiler));

app.use(function (req, res) {
  res.sendFile(__dirname + '/index.html');
});

app.listen(port, function (error) {
  if (error) {
    console.error(error);
  } else {
    console.info('==> 🌎  Listening on port %s. Open up http://localhost:%s/ in your browser.',
    port, port);
  }
});

```

#### routes.js

```javascript
import React from 'react'
import { Route } from 'react-router'
import App from './containers/App'
import UserPage from './containers/UserPage'
import RepoPage from './containers/RepoPage'

export default (
  <Route path="/" component={App}>
    <Route path="/:login/:name"
           component={RepoPage} />
    <Route path="/:login"
           component={UserPage} />
  </Route>
)

```

You'll see some project specific routes in routes.js.

---

### Directory Structure

```
real-world/
 |---- actions/
 |---- components/
 |---- containers/
 |---- middleware/
 |---- reducers/
 |---- store/
 |---- .babelrc
 |---- index.html
 |---- index.js
 |---- package.json
 |---- routes.js
 |---- server.js
 |---- webpack.config.js

```

---

### Configuring the store

Redux uses a single "Store", heres how they set it up:

```
real-world/
...
|---- reducers/
|---- store/
      |---- configureStore.js
      |---- congifureStore.dev.js
      |---- configureStore.prod.js
...
```

The configureStore.js file simply checks if the environment is labeled production, and if so requires the configureStore.prod.js configuration, otherwise it loads configureStore.dev.js. The primary difference between prod and dev is more logging and hot module replacement in dev.

#### configureStore.js

```javascript
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./configureStore.prod')
} else {
  module.exports = require('./configureStore.dev')
}
```

#### configureStore.dev.js

```javascript
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import createLogger from 'redux-logger'
import api from '../middleware/api'
import rootReducer from '../reducers'
import DevTools from '../containers/DevTools'

export default function configureStore(initialState) {
  const store = createStore(
    rootReducer,
    initialState,
    compose(
      applyMiddleware(thunk, api, createLogger()),
      DevTools.instrument()
    )
  )

  if (module.hot) {
    // Enable Webpack hot module replacement for reducers
    module.hot.accept('../reducers', () => {
      const nextRootReducer = require('../reducers').default
      store.replaceReducer(nextRootReducer)
    })
  }

  return store
}
```

#### configureStore.prod.js

```javascript
import { createStore, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import api from '../middleware/api'
import rootReducer from '../reducers'

export default function configureStore(initialState) {
  return createStore(
    rootReducer,
    initialState,
    applyMiddleware(thunk, api)
  )
}

```
