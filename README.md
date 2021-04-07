# GitStats, slightly enhanced

This is a fork of [gitstats](https://github.com/hoxu/gitstats). It adds the following fixes/enhancements:

- Ability to read a gitstats config file in source repo
- Fixed the inability to specify `commit_begin`
- Ability to specify `commit_begin` param in the config file
- Ability to specify files and paths to ignore in the config file
- Remove the "fatal: Needed a single revision" error
- Add this README


## Background

The greatly appreciated but ignored gitstats has been abandoned since 2012 (only 2 out of 56 pull requests were merge, last was in 2012).

If you want to use a well maintained fork/version of gitstats, I highly recommend [repostat](https://github.com/vifactor/repostat). I would have as well if only it had included ignore paths. If one day repostats has added this, please let me know by raising an issue.  


## Documentation

gitstats does have a README, it is hidden under [doc](doc) and [doc/gitstats.pod](doc/gitstats.pod). Please read to learn about gitstats.


## Issues

Please raise issues only related to this fork (or if repostats has added ignore paths).
Please raise issues related to gitstats in [gitstats/issues](https://github.com/hoxu/gitstats/issues). There are people there who can help you.


## Config File

To specify `commit_begin` and the file paths to ignore, add `gitstatsCfg.js` at the root of the repo to run gitstats on.

The config file is a javascript file since both js and non-js sources (like this repo) can easily read and make use of it.

The config file requires the following:

1. 1st line must define the file paths ignore regex. Define it as an empty string "" if just want to define the next `commit_begin` line.
2. 2nd line if provided, must define the `commit_begin` string.
3. Both values must be delimited by double quotes.

These are the only rules. The variable names do not matter. The file does not even need to be a valid js file. 

Specifying the `ignore` file paths is most likely not enough since the LOC per commit is done via line additions and subtractions, and there is no easy way to filter the deltas based on file paths. If you are lucky enough to have removed large files early in your history, specifying the `commit_begin` will help.

Here's the actual `gitstatsCfg.js` I am using in my project.

```javascript
const ignore="/static|public/|.json|.txt|.gitignore|.yml|.editorconfig|.lock|chart/config.js"
const commit_begin="2add5d6"

export { // module.exports = { // if will import with require
  ignore,
  commit_begin,
}

/**
Rules:
1. 1st line must define the file paths ignore regex. Define it as an empty string "" if just want to define the next `commit_begin` line.
2. 2nd line if provided, must define the `commit_begin` string.
3. Both values must be delimited by double quotes.
*/
```

### Other Uses for the Config File

Here is an example of using the config file in your own repo.

```javascript
import exec from 'child_process'
import ignore from './gitstatsCfg'
// const { ... } = require(...) // if using require

exec('git ls-files | grep -Ev "'+ignore+'" | xargs wc -l | grep " total"', (err, stdout, stderr) => {
  if (err || stderr) {
    console.error(`Error: ${err} ${stderr}`);
    return
  }
  const loc = +stdout.replace('total', '')
  console.log(`Lines of codes in repo: ${loc}`)
})
```

You can modify this further and have it write to a LOC log file per commit via a commit hook.


## Bugs Fixed

- Specifying `commit_begin` is ignored, fixed.
- Version info for gitstats should use `HEAD`, but it used your repo's `commit_begin` and `commit_end` instead, causing "fatal: Needed a single revision" error, fixed.