---
title: Setup reactjs for web development  
layout: post
date: 2016-12-06
image: 
headerImage: false
tag:
- reactjs
- javascript
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