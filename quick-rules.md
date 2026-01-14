
---
title: Quick Rules for Agents
permalink: /quick-rules/
---

# Quick Rules for Agents (SousLeSensVocables)

These rules are authoritative for AI assistants working on SousLeSensVocables.

## MUST (strict)
- Consult `function-index.md` first (DRY). Reuse existing functions before creating new ones.
- No Promises and no async/await. Use async.js (`async.series`, `async.eachSeries`) with error-first callbacks.
- Module pattern: IIFE returns an object. Put `export default` OUTSIDE the IIFE. Also attach to `window.ModuleName`.
- Code, identifiers, and code comments must be in English only.

## MUST NOT
- Do not invent file names or paths.
- Do not put `export default` inside an IIFE.
- Do not suggest GitHub write operations (push/PR) unless explicitly requested.


## Correct module template (copy/paste)

```javascript
var MyModule = (function () {
  var self = {};

  self.init = function (options, callback) {
    async.series([
      function stepOne(cb) { cb(null); },
      function stepTwo(cb) { cb(null); }
    ], function (err) {
      if (err) return callback(err);
      callback(null);
    });
  };

  return self;
})();

export default MyModule;
window.MyModule = MyModule;
