---
title: Setup reactjs for web development  
layout: post
date: 2016-12-06
image: 
headerImage: false
tag:
- reactjs
- javascript
- bootstrap
blog: true
author: gdf
description: how to setup reactjs for web development, include the essential npm for reactjs
---

# Essential NPM Packages

|name|functions|
| --- | --- |
|[Browserify](http://browserify.org/)|Allows the browser to use NPM modules|
|[Babel](http://babeljs.io/)|Compiles ES6/JSX into browser-readable Javascript|
|[Babel-React-Preset](https://babeljs.io/docs/plugins/preset-react/)|Tells Babel how to compile JSX|
|[Babelify](https://github.com/babel/babelify)|Allows you to use Babel with Browserify|
|[Watchify](https://github.com/substack/watchify)|Watches for changes and instantly starts compiling|

# Create the react skeleton project

1. create a folder name `react-skeleton`
2. `cd` into the `react-skeleton` folder
3. create `public` and `src` folders
4. run `git init` and push your project to github
5. run `npm init` to initilize the `package.json` file (indicate that this project is a node project)
6. install the essential npm packages
  - npm install -g browserify
  - npm install --save babelify 
  - npm install --save watchify
  - npm install --save babel-preset-react
  - npm install --save react
  - npm install --save react-dom
7. create a `components` folder inside `src` folder
8. create file `src/main.jsx`
9. create file `src/components/List.jsx` 
10. create file `src/components/ListItem.jsx`
11. create a `js` folder inside `public` folder
12. create file `public/js/main.js`

Project [Link](https://github.com/devslopes/react-skeleton)

# Use Bootstrap(3.X) and Jquery for styling

1. add [Bootstrap](http://getbootstrap.com/getting-started/#download) content delivery network (CDN) to html file
2. add [Jquery](https://code.jquery.com/) CDN to html file

example html:

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- Latest compiled and minified CSS -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <!-- Optional theme -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous">
  </head>
  <body>
    <button class="btn btn-primary">Click Me!</button>
    <script src="https://code.jquery.com/jquery-3.1.1.min.js" integrity="sha256-hVVnYaiADRTO2PzUGmuLJr8BLUSjGIZsDYGmIJLv2b8=" crossorigin="anonymous"></script>
    <!-- Latest compiled and minified JavaScript -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
  </body>
</html>
```

