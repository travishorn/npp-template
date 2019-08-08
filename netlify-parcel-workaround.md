# Netlify Dev Parcel Workaround

Netlify Dev doesn't currently support Parcel. There is an [open pull
request](https://github.com/netlify/netlify-dev-plugin/pull/234) to support it.
But until the pull request is merged, you must use this workaround to get the
template to work.

After running `npm install` to install all dependencies. You must add a new
detector to the `netlify-dev-plugin` package.

Create a new file called `parcel.js` in
`node_modules/netlify-dev-plugin/src/detectors/`. Paste these contents and save:

```javascript
const {
  hasRequiredDeps,
  hasRequiredFiles,
  getYarnOrNPMCommand,
  scanScripts
} = require("./utils/jsdetect");

module.exports = function() {
  /* REQUIRED FILES */
  if (!hasRequiredFiles(["package.json"])) return false;

  /* REQUIRED DEPS */
  if (!hasRequiredDeps(["parcel-bundler"])) return false;
  /* The package "parcel" also works, but the Parcel docs say to use
   * "parcel-bundler" */

  /* Everything below now assumes that we are within parcel */

  const possibleArgsArrs = scanScripts({
    preferredScriptsArr: ["start", "dev", "run"],
    preferredCommand: "parcel"
  });

  if (possibleArgsArrs.length === 0) {
    /* Offer to run it when the user doesnt have any scripts setup! 🤯 */
    possibleArgsArrs.push(["parcel"]);
  }

  return {
    type: "parcel",
    command: getYarnOrNPMCommand(),
    port: 8888,
    proxyPort: 1234,
    env: { ...process.env },
    possibleArgsArrs,
    urlRegexp: new RegExp(`(http://)([^:]+:)${1234}(/)?`, "g"),
    dist: "dist"
  };
};
```
