# GitStats, slightly enhanced

This is a fork of [gitstats](https://github.com/hoxu/gitstats). It adds the following fixes/enhancements:

- Add this README
- Ability to read a gitstats config file in the target source repo
- Fixed the "fatal: Needed a single revision" error
- Fixed the inability to specify `commit_begin`
- Ability to specify `commit_begin` param in the config file
- Ability to specify files and paths to ignore for file counts in the config file
- Ability to specify timestamp and line count delta pairs to correct the miscounts
- Ability to remove the Lines of Code by Author chart
- Enhanced Activity Page:
  - Commits by Month chart is now full width bars instead of thin single pixel bars 
  - Simpler right-hand side tables
  - Show last 52 weeks instead of just 32
- Enhanced Lines of Code charts:
  - Bigger with average lines of code line 
  - Added Lines of Code by Month chart:
    - with average LOC per month count shown with line and background
    - averge LOC per month count excludes first month if it is < 50% of average  
    - averge LOC per month count excludes most recent month if < 50% of average and is < 10 days into the month
- Reduce noise of Activity page by:
  - Removing LOC counts in activity tables
  - Removing "Hour of Day" section

## Background, Other Forks

The greatly appreciated but ignored gitstats has been abandoned since 2012 (only 2 out of 56 pull requests were merge, last was in 2012).

If you want to use a well maintained fork/version of gitstats, I highly recommend [repostat](https://github.com/vifactor/repostat). I would have as well if only it had included ignore paths or delta abilities. If one day repostat has these, please let me know by replying to [my fork thread](https://github.com/hoxu/gitstats/issues/99).

If you want to develop futher on gitstats, here are some interesting forks worth taking a look or to fork from:

- [KaivnD's](https://github.com/KaivnD/gitstats/commits/master) - Refactored to split the big file into multiple files, good fork to based off from to understand the code and make your own changes.
- [matt-chalmer's](https://github.com/matt-chalmers/gitstats/commits/master) - more changes on top of [laserb's](https://github.com/laserb/gitstats/commits/master) fork which added changes around authors.

If you know of other good forks with great features added, let me know and I will add to the list.

## Documentation

gitstats does have a README, it is hidden under [doc](doc) and [doc/gitstats.pod](doc/gitstats.pod). Please read to learn about gitstats.


## Issues

Issues is not enabled since this is a fork.
Please raise issues related to gitstats in [gitstats/issues](https://github.com/hoxu/gitstats/issues). There are people there who can help you.
For issues related to this fork, please reply to [my fork thread](https://github.com/hoxu/gitstats/issues/99).

## Config File

To specify `commit_begin` and the file paths to ignore, add `gitstatsCfg.js` at the root of the repo to run gitstats on.

The config file is a javascript file since both js and non-js sources (like this repo) can easily read and make use of it.

The config file requires the following:

1. 1st line must define `ignore` -- the file paths ignore regex **.
2. 2nd line if provided, must define the `commit_begin` string.
3. 3rd line if provided, must define `commit_delta` -- array of commit timestamp and line count delta pairs.
  The pair is separated by a comma and pairs are separated by a `|`.
  Earlier commit deltas must come first.
4. 4th line if provided, must define `show_author_loc_chart=false`, to not show the useless/buggy* author LOC charts.
5. All values must be delimited by double quotes.
6. Earlier lines can not be skipped. To define later lines, define the earlier lines with empty strings.

* The author LOC charts only showed lines added, which is useless. I have fixed it by adding removed lines to it but it will not match LOC chart for single author repos if `commit_delta`'s are added. This option allows the author charts to be skipped.

** Specifying the `ignore` will only ignore files in file reports, not LOC since the LOC per commit is done via line additions and subtractions, and there is no easy way to filter the deltas based on file paths. If you are lucky enough to have removed large files early in your history, specifying the `commit_begin` will help. To have the LOC be in similar to the LOC reported by `git ls-files ...` (see the sample javascript below), you need to specify `commit_delta` pairs. If you commit history is short, you should be able to find the deltas easily. If not, you can probably write a one-liner or simple script to find the top commits by lines changed.

The commit timestamp can be obtained via 

```bash
git show -s --format=%ct <commit
```

Here's the actual `gitstatsCfg.js` I am using in my project.

```javascript
const ignore="static/|public/|.json|.txt|.md|.gitignore|.yml|.editorconfig|.lock|chart/config.js"
const commit_begin="2add5d6"
const commit_delta="1616997336,-600|1619306440,1120" 
const show_author_loc_chart=false

export { // module.exports = { // if will import with require
  ignore,
  commit_begin,
}

/**
Rules:
1st line must define `ignore` -- file paths ignore regex (only for files count, not LOC count).
2nd line if provided, must define the `commit_begin` string.
3rd line if provided, must define `commit_delta` -- array of commit timestamp and line count delta pairs.
  The pair is separated by a comma and pairs are separated by a `|`.
  Earlier commit deltas must come first.
4th line if provided, must define `show_author_loc_chart=false` to not show the buggy author LOC charts.

* All values must be delimited by double quotes.
* Earlier lines can not be skipped. To define later lines, define the earlier lines with empty strings.


To get timestamp of a commit:
git show -s --format=%ct <commit>

See https://github.com/fuzzthink/gitstats for details.
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