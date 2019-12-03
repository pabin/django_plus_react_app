# django_plus_react_app

#### git clone mydjangoapp
#### virtualenv -p python3 venv
#### cd mydjangoapp
#### source ../venv/bin/activate
#### ./manage.py runserver
#### npm run watch



## Django React Setup Basic Process

### Django and static files

STATIC_URL = '/static/'

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'assets'),
)

### Configuring the Django project

pip install django djangorestframework django-webpack-loader
pip freeze > requirements.txt
django-admin startproject mydjangoapp
mkdir assets templates

### Configuring project/settings.py

INSTALLED_APPS = [
  ...
  'rest_framework',
  'webpack_loader',
  ...
]

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        ...
    },
]

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'assets'),
)

WEBPACK_LOADER = {
    'DEFAULT': {
        'BUNDLE_DIR_NAME': 'dist/',
        'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats.json'),
    }
}

### The project will have the following structure:
├── node_modules
├── project
├── package.json
├── templates
├── babelrc
├── manage.py
├── package.json
├── requirements.txt
├── webpack-stats.json
├── webpack.config.js
└── assets
    └── dist
    └── src


### Creating the template for the front-end application
mkdir templates/frontend
touch templates/frontend/index.html

### templates/frontend/index.html
{% load render_bundle from webpack_loader %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Django React Webpack</title>
    <!-- Webpack rendered CSS -->
    {% render_bundle 'main' 'css' %}
</head>
<body>
    <div id="react-app"></div>
    <!-- Webpack rendered JS -->
    {% render_bundle 'main' 'js' %}
</body>
</html>

### project/urls.py
...
from django.views.generic import TemplateView

urlpatterns = [
    ...
    path('', TemplateView.as_view(template_name='frontend/index.html')),
]

### Initializing the front-end project

npm init
mkdir -p assets/src/js
touch assets/src/js/index.js

### Configuring Webpack
npm install --save-dev webpack webpack-bundle-tracker
touch webpack.config.js

### webpack.config.js

var path = require('path');
var webpack = require('webpack');
var BundleTracker = require('webpack-bundle-tracker');
module.exports = {
  entry:  path.join(__dirname, 'assets/src/js/index'),
  output: {
    path: path.join(__dirname, 'assets/dist'),
    filename: '[name]-[hash].js'
  },
  plugins: [
    new BundleTracker({
      path: __dirname,
      filename: 'webpack-stats.json'
    }),
  ],
}

### package.json

...
"scripts": {
  ...
  "start": "./node_modules/.bin/webpack --config webpack.config.js",
  "watch": "npm run start -- --watch"
},
...


### Configuring React
npm install --save react react-dom


### Configuring babel

npm install --save-dev babel-cli @babel/core @babel/preset-env babel-preset-es2015 @babel/preset-react babel-register babel-loader

touch .babelrc

{
    "presets": [
      "es2015", "react"
    ]
}

### webpack.config.js

module.exports = {
    ...
    module: {
      rules: [
        {
          test: /\.jsx?$/,
          loader: 'babel-loader',
          exclude: /node_modules/,
        },
      ],
    },
}

### Organizing the application
mkdir -p assets/src/js/components/Title
touch assets/src/js/components/Title/index.js

### assets/src/js/components/Title/index.js

import React from 'react';
const Title = ({ text }) => {
  return (
    <h1>{text}</h1>
  )
}
export default Title;

### assets/src/js/containers/App/index.js

import React from 'react';
import Title from '../../components/Title';
class App extends React.Component {
  render () {
    const text = 'Django + React + Webpack + Babel = Awesome App';
    return (
      <Title text={text} />
    )
  }
}
export default App;

### assets/src/js/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './containers/App';
ReactDOM.render(<App />, document.getElementById('react-app'));

### Adding styles
npm install --save-dev css-loader style-loader sass-loader node-sass extract-text-webpack-plugin

npm install extract-text-webpack-plugin@4.0.0-alpha.0

### webpack.config.js

...
var ExtractText = require('extract-text-webpack-plugin');
module.exports = {
    ...
    plugins: [
      ...
      new ExtractText({
        filename: '[name]-[hash].css'
      }),
    ],
    module: {
      rules: [
        ...
        {
          test: /\.css$/,
          loader: ['style-loader', 'css-loader'],
        },
        {
          test: /\.scss$/,
          use: ExtractText.extract({
            fallback: 'style-loader',
            use: ['css-loader', 'sass-loader']
          })
        },
      ],
    },
}

### CSS files
mkdir assets/src/scss
touch assets/src/scss/index.scss assets/src/scss/title.scss

### assets/src/scss/index.scss
@import "./title";

### assets/src/scss/title.scss

.title {
  color: red;
}

### assets/src/js/index.js
...
import '../scss/index.scss';
...

### assets/src/js/components/Title/index.js

...
    <h1 className="title">{text}</h1>
...
