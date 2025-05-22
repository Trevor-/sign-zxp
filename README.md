# sign-zxp

> A JS wrapper for Adobe's extension signer - ZXPSignCmd

[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)
![Dependencies](https://img.shields.io/librariesio/release/npm/sign-zxp)
[![Coverage Status](https://coveralls.io/repos/github/Trevor-/sign-zxp/badge.svg?branch=master)](https://coveralls.io/github/Trevor-/sign-zxp?branch=master)

## Credits

This is an independent fork of <https://github.com/codearoni/sign-zxp-cmd.git> with updated ZXPSignCmd binaries.  
All the hard work was done by <https://github.com/codearoni>

## About

sign-zxp provides a simple interface for Adobe's extension signer. sign-zxp takes care of starting the requires processes, handling errors, and building the output directory if required.

## Installation

```bash
    npm i sign-zxp -D
```

For Yarn

```bash
    yarn add sign-zxp
```

## Usage

```javascript
const zxpSign = require('sign-zxp');
```

There are functions for all 3 operations: sign, selfSignedCert, and verify.
Each function takes an options object as its first parameter. See the property tables for the appropriate function below.
sign-zxp's latest API uses promises and async functions. If you require a callback-style implementation, please revert to the legacy v1.x.x API.
The result will be a string and will contain the stdout provided by ZXPSignCmd (if the task succeeds). In the event that the task fails, an error is thrown.
If you would like to use the await-style implementation, it is recommended to wrap a try/catch and handle errors that way.
However, the API does support .then( ) and .catch( ) configurations.

## Instructions

Make a folder in the root of your repo (outside of your src folder).  
Create a certificate using `signZXP.selfSignedCert()`  
Create a js file in the folder with your build and signing tasks and execute the file using `node buildAnSignExtension.js`  
You can make a vsCode task to execute the file too, if your vsCode is set up properly  
The `buildAnSignExtension.js` (or whatever you call it) can call `yarn build` and update version numbers and perform any other tasks you need before signing.

Sample `buildAnSignExtension.js`

```js
const fs = require("fs/promises");
const path = require("path");
const zxpSignCmd = require("sign-zxp");
const { spawn } = require("child_process");
const signZXP = require("sign-zxp");

async function yarnBuild() {
 return new Promise((resolve, reject) => {
  const child = spawn("yarn", ["build"], {
   stdio: "inherit",
   shell: true,
  });

  child.on("close", (code) => {
   if (code === 0) {
    resolve(true);
   } else {
    reject(new Error(`yarn build exited with code ${code}`));
   }
  });

  child.on("error", (err) => {
   reject(err);
  });
 });
}


const result = await yarnBuild();

const zxpOptions = {
 input: "path/to/the/extension/folder",
 output: "path/and/to/signed/folder/myExtension.zxp",
 cert: path.join(__dirname, "cert.p12"),
 password: "foo",
 timestamp: "<http://timestamp.digicert.com/>",
};   

const zxpResult = await signZXP.sign(zxpOptions);
console.log(zxpResult);
```

`tasks.json`

```json
{
 "version": "2.0.0",
 "tasks": [
  {
   "label": "🏛️ Build an sign extension 🏛️",
   "group": "build",
   "args" : [
    "buildSnippet/buildAnSignExtension.js"
   ],
   "command": "node"
  }
 ]
}
```

### Sign Certificate Example

```js
const signZXP = require("sign-zxp");
// import signZXP from "fs/promises"; // Use import instead of require

const zxpOptions = {
 input: "path/to/the/extension/folder",
 output: "path/and/to/signed/folder/myExtension.zxp",
 cert: path.join(__dirname, "cert.p12"),
 password: "foo",
 timestamp: "<http://timestamp.digicert.com/>",
};
```

const zxpResult = await signZXP.sign(zxpOptions);

### Create Certificate Example

```js
const signZXP = require("sign-zxp");

const zxpCertOptions = {
 country: "US",
 province: "NY",
 org: "My Org",
 name: "cert",
 password: "foo",
 output: path.join(__dirname, "cert.p12"),
 // validityDays: 10 // To limit how long the cert is valid for, something you normally don't want to do!
};

const certResult = await signZXP.selfSignedCert(zxpCertOptions);
```

### Verify Example

```js
const signZXP = require("sign-zxp");

const zxpVerifyOptions = {
 input: "path/to/myExtension.zxp",
 info: true, // optional for a more verbose output
 skipChecks: false, // optional skip online check
}

const verifyResult = await zxpSign.verify(zxpVerifyOptions);
```

### sign

Options

| Property  | Required | Datatype | Purpose |
| --------- | -------- | -------- | ------- |
| input     | yes      | String   | The directory that will be compiled into the packaged zxp file |
| output    | yes      | String   | The path and filename that the zxp will be exported to |
| cert      | yes      | String   | The path and filename of the .p12 certificate that will be used to sign the extension |
| password  | yes      | String   | The password associated with the certificate |
| timestamp | no       | String   | URL for a timestamp server |

Example:

```javascript
const signResult = await zxpSign.sign(options);
```

### selfSignedCert

Options

| Property  | Required | Datatype | Purpose |
| --------- | -------- | -------- | ------- |
| country   | yes      | String   | The country associated with the certificate |
| province  | yes      | String   | The state or province associated with the certificate |
| org       | yes      | String   | The organization associated with the certificate |
| name      | yes      | String   | The commonName for the certificate |
| password  | yes      | String   | The password for the certificate |
| output    | yes      | String   | The path that the certificate will be exported to |
| locality  | no       | String   | The locality for the certificate |
| orgUnit   | no       | String   | Name of the organizational unit |
| email     | no       | String   | An email associated with the certificate |
| validityDays | no    | Number   | The number of days the certificate is valid |

Example:

```javascript
const certResult = await zxpSign.selfSignedCert(options);
```

### verify

Options

| Property  | Required | Datatype | Purpose |
| --------- | -------- | -------- | ------- |
| input     | yes      | String   | The path to the zxp file that will be validated |
| info      | no       | Boolean  | Allows the result to return a more verbose output of the validation |
| skipChecks| no       | Boolean  | The validation will skip onlin revocation checks |
| addCerts  | no       | String   | Additional certificates to validate against |

Example:

```javascript
const verifyResult = await zxpSign.verify(options);
```
