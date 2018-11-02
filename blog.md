# How to build a desktop PDF viewer using Electron and PDF.js

Electron is a platform that allows you to create cross platform desktop applications using web technology. This post will go over how to implement PDF.js into an Electron app to build your own desktop PDF viewer.

## 1. Download and Install Electron
Electron provides a handy starter repo that contains all the boilerplate we need to get started. Get it by running:

```
git clone https://github.com/electron/electron-quick-start

cd electron-quick-start

npm install
```

Once these steps are done, you can make sure everything worked by running 

```
npm start
```

You should see a hello world app pop up on your desktop.


## Set up project structure
In order to keep things clean, lets clean up the boilerplate a bit. Create a folder called `src` and move `index.html` and `renderer.js` into it.

Inside `main/js`, change this line:

```js
mainWindow.loadFile('index.html')
```

to this:

```js
mainWindow.loadFile('src/index.html')
```

Also, in `/src/`, create an `index.css` file that we can use to style our app.

## Download PDF.js
In the root of your project, create a folder called `public`. This is where we will put our PDF.js files.

Download the PDF.js files from [here](https://mozilla.github.io/pdf.js/getting_started/#download) and extract them into the `public` folder. For ease of use, rename the folder it gives you to just `pdfjs`.

You will also need to tell Electron where your static assets are. You can do this by adding the following code to `package.json`

```json
"build": {
  "extraResources": ["./public/**"]
}
```

## Create the viewer
Change the content of `src/index.html` to the following:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
    <link rel='stylesheet' href='./index.css'>
  </head>
  <body>

    <div class='picker'>
      <button id='myButton'>Select PDF to view</button>
    </div>

    <div class='viewer' id='viewer'>

    </div>

    <script>
      require('./renderer.js');
    </script>
  </body>
</html>
```

This will be the UI for our app. We create a button that will be used to show the native file picker, and we create a `viewer` div that will hold our PDF.js viewer. We also link our css styles.

Update `src/index.css` to the following:

```css
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

div.picker {
  width: 100%;
  height: 40px;

  background-color: #222222;
  display: flex;
  justify-content: center;
  align-items: center;
}

div.viewer {
  width: 100%;
  height: calc(100% - 40px);
}

div.viewer iframe {
  width: 100%;
  height: 100%;
}
```

This adds some basic layout for our project, and tells the viewer to fill up as much of the window as it can.

Now, its time to add our JS logic. Update `src/renderer.js` to the following:

```js
const { dialog } = require('electron').remote;
const path = require('path');

// Add an event listener to our button.
document.getElementById('myButton').addEventListener('click', () => {

  // When the button is clicked, open the native file picker to select a PDF.
  dialog.showOpenDialog({
    properties: ['openFile'], // set to use openFileDialog
    filters: [ { name: "PDFs", extensions: ['pdf'] } ] // limit the picker to just pdfs
  }, (filepaths) => {

    // Since we only allow one file, just use the first one
    const filePath = filepaths[0];

    const viewerEle = document.getElementById('viewer');
    viewerEle.innerHTML = ''; // destroy the old instance of PDF.js (if it exists)

    // Create an iframe that points to our PDF.js viewer, and tell PDF.js to open the file that was selected from the file picker.
    const iframe = document.createElement('iframe');
    iframe.src = path.resolve(__dirname, `../public/pdfjs/web/viewer.html?file=${filePath}`);

    // Add the iframe to our UI.
    viewerEle.appendChild(iframe);
  })
})
```

This code uses the Electron `dialog` module to open a file picker when the button in our UI is clicked. Once the user selects a file, we create an iframe that points to the PDF.js viewer. We append a query param to the iframe that tells PDF.js which file we want to open.

## Test the app
Thats it! Run `npm start` to run the Electron app in development mode. You should see a UI that looks like this:

![]() image here

When you click the button, the native file picker will pop up and allow you to select a PDF file. Once selected, PDF.js viewer will appear with the PDF you selected open.



