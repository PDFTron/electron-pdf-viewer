# How to build a desktop PDF viewer using Electron and PDF.js

[Electron](https://electronjs.org/) is an open-source framework that allows you to create cross platform desktop applications using web technology. This post will go over how to implement [PDF.js](https://mozilla.github.io/pdf.js/) into an Electron app to build your own desktop PDF viewer.

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


## Set Up Project Structure
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

## Create the Viewer
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

Thats it! Run `npm start` to run the Electron app in development mode. You should see a UI that looks like this:

![]() image here

When you click the button, the native file picker will pop up and allow you to select a PDF file. Once selected, PDF.js viewer will appear with the PDF you selected open.

So far, we are able to view a PDF in our Electron app, but that's about it. If we wanted more functionality, such as annotating, PDF manipulation, and opening other file types, then open source software just won't cut it.

That's where PDFTron's [WebViewer](https://www.pdftron.com/webviewer) comes in! It provides all this functionality out of the box, with no configuration. We can implement it into our Electron app (for free) just as easily as PDF.js.

## Implementing with WebViewer
To implement viewer, first start by [signing up for a free trial](https://www.pdftron.com/form/download-trial/). Once you complete this step, [download the WebViewer package](https://www.pdftron.com/documentation/web/guides/run-samples) and extract the contents into the 'public' folder.

Now we need to require the WebViewer files. In `src/index.html`, add the following line in the `<head>`:

```html
<script src='../public/WebViewer/lib/webviewer.min.js'></script>
```

This will load WebViewer into the global context for use.

Now we just need to change our buttons onclick handler to use WebViewer instead of PDF.js. For the sake of demonstration, we will create a new file called `renderer-webviewer.js` and insert the following code:

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

    // create an instance of WV.
    new window.PDFTron.WebViewer({
      path: '../public/WebViewer/lib',
      l: 'YOUR_KEY_HERE,
      initialDoc: filePath
    }, viewerEle)

  })
})
```

This code is the same as the code we used in the PDF.js step, except it creates an instance of WebViewer instead of creating an iframe.

Now we need to reference the new js file. In `src/index.html` change this:

```js
 require('./renderer.js');
```

to this:

```js
 require('./renderer-webviewer.js');
```

And that's it!

You can run `npm start` to start the app. The flow is the exact same, except now you will see WebViewer instead of PDF.js. You are now also able to annotate the document, and much more!

## Building for Production
The steps involved in building an Electron app for production use varies by system and OS, so please visit [this guide](https://electronjs.org/docs/development/build-instructions-gn) if you are interested in doing so.

## Conclusion
This post showed how easy it is to get a basic PDF viewing Electron app up-and-running using open source software. We also went over how to create a more feature-packed PDF experience by implementing [WebViewer](https://www.pdftron.com/webviewer).

The full source code for this article can be found [here](https://github.com/PDFTron/electron-pdf-viewer).

You can view a full demo of WebViewer [here](https://www.pdftron.com/webviewer/demo/). Feel free to compare it to the [PDF.js viewer!](https://mozilla.github.io/pdf.js/web/viewer.html)

If you have any questions about implementing Webviewer in your project, please feel to [contact us](https://www.pdftron.com/company/contact-us/) and we will be happy to help!





