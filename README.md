# ❤ ui5-lib-util *(alpha)*
ui5-lib-util was made to build a production version of UI5 and represents an alternative to [SAPs Grunt based build tool](https://github.com/SAP/openui5/blob/master/docs/developing.md).
The build is responsible for the following tasks:
- Creation of the bundled library.css and library-RTL.css file for all available themes
- Minification of CSS
- Minification of JavaScript
- Combination of JavaScript control files into a single library-preload.js file
- Combination of the most important UI5 core files into sap-ui-core.js

The reason why we created this (from our point of view optimized) alternative is the following statement from SAP:
> IMPORTANT: as we are still migrating our build infrastructure from the old Maven-based one, this new Grunt build does not yet have all desired capabilities. Bear with us as we are adding them. This means that currently the build result is not completely optimized for size and performance! (SAP, 2014)

## Install
Install ui5-lib-util as a development dependency:
```
yarn add ui5-lib-util@alpha --dev
```

## How to use
ui5-lib-util is designed as an agnostic node module and can be used standalone in your custom build script or as part of e.g. a gulp build task.

Example with gulp `4.0.0` (JavaScript ES6):
```js
import gulp from 'gulp'
import tap from 'gulp-tap'
import { ui5Download, ui5Build, ui5CompileLessLib } from 'ui5-lib-util'

// create gulp task
const loadUI5 = gulp.series(downloadOpenUI5, buildOpenUI5)
export { loadUI5 }

// download task
function downloadOpenUI5() {
  const sDownloadURL = 'https://github.com/SAP/openui5/archive/master.zip'
  const sDestinationPath = './dl'
  const sUI5Version = '1.49.0-SNAPSHOT'
  const oDownloadOptions = {
    onProgress(iStep, iTotalSteps, oStepDetails) {
      console.log(`Downloading UI5... [${iStep}/${iTotalSteps}] ${Math.round(
        oStepDetails.progress || 0
      )}%`)
    }
  }

  // the UI5 library will be downloaded and unzipped to ./dl/1.49.0-SNAPSHOT
  return ui5Download(sDownloadURL, sDestinationPath, sUI5Version, oDownloadOptions)
    .then(sSuccessMessage => {
      console.log(sSuccessMessage)
    })
    .catch(sErrorMessage => {
      console.error(sErrorMessage)
    })
}

// build task
function buildOpenUI5() {
  const sSourceCodePath = './dl/1.49.0-SNAPSHOT'
  const sDestinationPath = './ui5'
  const sUI5Version = '1.49.0-SNAPSHOT'
  const oBuildOptions = {
    onProgress(iStep, iTotalSteps, oStepDetails) {
      console.log(`Build UI5... [${iStep}/${iTotalSteps}] (${oStepDetails.name})`)
    }
  }

  // the UI5 library build will be created at ./ui5/1.49.0-SNAPSHOT
  return ui5Build(
      sSourceCodePath,
      sDestinationPath,
      sUI5Version,
      oBuildOptions
    )
    .then(sSuccessMessage => {
      console.log(sSuccessMessage)
    })
    .catch(sErrorMessage => {
      console.error(sErrorMessage)
    })
}

// compile a ui5 less theme library
function compileUi5Theme() {
  const sLibrarySourcePath = './dist/path/to/my/library/themes/sap_belize'

  // create library.css in same directory as library.source.less
  return new Promise((resolve, reject) =>
    gulp
      .src([`${sLibrarySourcePath}/library.source.less`])
      // rename UI5 module (app component) paths and update UI5 resource roots in UI5 bootstrap of index.html
      .pipe(tap(oFile => {
        ui5CompileLessLib(oFile).then(resolve).catch(reject)
      }))
  )
}

```

Furtheremore, in the [OpenUI5 Starter Kit](https://github.com/pulseshift/openui5-gulp-starter-kit) you can find ui5-lib-util integrated in a complete build script.

## Methods
### `ui5Download`
```js
ui5Download(downloadURL, destinationPath, ui5Version, [options])
```

* `downloadURL` (string) The source URL of a single file (archive).
* `destinationPath` (string) Path of destination directory used for the extracted download files.
* `ui5Version` (string) Explicit version of the OpenUI5 library. Used as root folder at destination directory for the downloaded and extracted files.
* `options` (object, optional) The configuration options object.
* `options.onProgress` (function(number, number, object):void, optional) Callback function to track download progress taking as params: current step number, total step number and if available, step details (e.g. name of current step and progress in percent).

### `ui5Build`
```js
ui5Build(sourcePath, destinationPath, ui5Version, [options])
```

* `sourcePath` (string) Path of source directory containing OpenUI5 source code.
* `destinationPath` (string) Path of destination directory for the build OpenUI5 library.
* `ui5Version` (string) Explicit version of the OpenUI5 library. Used as root folder at destination directory for the build. *Note: Only versions  from 1.40.0 or higher are supported.*
* `options` (object, optional) The configuration options object.
* `options.onProgress` (function(number, number, object):void, optional) Callback function to track build progress taking as params: current step number, total step number and if available, step details (e.g. name of current step).

### `ui5CompileLessLib`
```js
ui5CompileLessLib(file)
```

* `file` ([Vinyl](https://github.com/gulpjs/vinyl)) File must be the `library.source.less` file that imports all required less resources of the library.

### Outlook

Here is a brief overview on what we are working right know and what will follow, soon. We are interested to hear your opinion on what should follow next.

Current idea backlog (unordered):
- Optimized OpenUI5 library modules (containing only these controls you used)

### License

This project is licensed under the MIT license.
Copyright 2017 [PulseShift GmbH](https://pulseshift.com/en/index.html)
