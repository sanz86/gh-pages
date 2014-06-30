# gh-pages

Publish files to a `gh-pages` branch on GitHub (or any other branch anywhere else).

## Getting Started

```shell
npm install gh-pages --save-dev
```

This module requires Git `>=1.7.6`.

## Basic Usage

```js
var ghpages = require('gh-pages');
var path = require('path');

ghpages.publish(path.join(__dirname, 'dist'), function(err) { ... });
```


## `publish`

```js
ghpages.publish(basePath, callback);
// or...
ghpages.publish(basePath, options, callback);
```

Calling this function will create a temporary clone of the current repository, create a `gh-pages` branch if one doesn't already exist, copy over all files from the base path, or only those that match patterns from the optional `src` configuration, commit all changes, and push to the `origin` remote.

If a `gh-pages` branch already exists, it will be updated with all commits from the remote before adding any commits from the provided `src` files.

**Note** that any files in the `gh-pages` branch that are *not* in the `src` files **will be removed**.  See the [`add` option](#optionsadd) if you don't want any of the existing files removed.


### <a id="optionsbase">`basePath`</a>
* type: `string`

The base directory for all source files (those listed in the `src` config property).

Example use of the `basePath`:

```js
/**
 * Given the following directory structure:
 *
 *   build/
 *     index.html
 *     js/
 *       site.js
 *
 * The usage below will create a `gh-pages` branch that looks like this:
 *
 *   index.html
 *   js/
 *     site.js
 *
 */
ghpages.publish(path.join(__dirname, 'build'), callback);
```


### Options

The default options work for simple cases cases.  The options described below let you push to alternate branches, customize your commit messages, and more.


#### <a id="optionssrc">options.src</a>
 * type: `string`
 * default: `'**/*'`

The [minimatch](https://github.com/isaacs/minimatch) pattern used to select which files should be published.


#### <a id="optionsdotfiles">options.dotfiles</a>
 * type: `boolean`
 * default: `false`

Include dotfiles.  By default, files starting with `.` are ignored unless they are explicitly provided in the `src` array.  If you want to also include dotfiles that otherwise match your `src` patterns, set `dotfiles: true` in your options.

Example use of the `dotfiles` option:

```js
/**
 * The usage below will push dotfiles (directories and files)
 * that otherwise match the `src` pattern.
 */
ghpages.publish(path.join(__dirname, 'dist'), { dotfiles: true }, callback);
```


#### <a id="optionsadd">options.add</a>
 * type: `boolean`
 * default: `false`

Only add, and never remove existing files.  By default, existing files in the target branch are removed before adding the ones from your `src` config.  If you want the task to add new `src` files but leave existing ones untouched, set `add: true` in your options.

Example use of the `add` option:

```js
/**
 * The usage below will only add files to the `gh-pages` branch, never removing
 * any existing files (even if they don't exist in the `src` config).
 */
ghpages.publish(path.join(__dirname, 'build'), { add: true }, callback);
```


#### <a id="optionsrepo">options.repo</a>
 * type: `string`
 * default: url for the origin remote of the current dir (assumes a git repository)

By default, `gh-pages` assumes that the current working directory is a git repository, and that you want to push changes to the `origin` remote.

If instead your script is not in a git repository, or if you want to push to another repository, you can provide the repository URL in the `repo` option.

Example use of the `repo` option:

```js
/**
 * If the current directory is not a clone of the repository you want to work
 * with, set the URL for the repository in the `repo` option.  This usage will
 * push all files in the `src` config to the `gh-pages` branch of the `repo`.
 */
ghpages.publish(path.join(__dirname, 'build'), {
  repo: 'https://example.com/other/repo.git'
}, callback);
```


#### <a id="optionsbranch">options.branch</a>
 * type: `string`
 * default: `'gh-pages'`

The name of the branch you'll be pushing to.  The default uses GitHub's `gh-pages` branch, but this can be configured to push to any branch on any remote.

Example use of the `branch` option:

```js
/**
 * This task pushes to the `master` branch of the configured `repo`.
 */
ghpages.publish(path.join(__dirname, 'build'), {
  branch: 'master',
  repo: 'https://example.com/other/repo.git'
}, callback);
```


#### <a id="optionstag">options.tag</a>
 * type: `string`
 * default: `''`

Create a tag after committing changes on the target branch.  By default, no tag is created.  To create a tag, provide the tag name as the option value.


#### <a id="optionsmessage">options.message</a>
 * type: `string`
 * default: `'Updates'`

The commit message for all commits.

Example use of the `message` option:

```js
/**
 * This adds commits with a custom message.
 */
ghpages.publish(path.join(__dirname, 'build'), {
  message: 'Auto-generated commit'
}, callback);
```


#### <a id="optionsuser">options.user</a>
 * type: `Object`
 * default: `null`

If you are running the `gh-pages` task in a repository without a `user.name` or `user.email` git config properties (or on a machine without these global config properties), you must provide user info before git allows you to commit.  The `options.user` object accepts `name` and `email` string values to identify the committer.

Example use of the `user` option:

```js
ghpages.publish(path.join(__dirname, 'build'), {
  user: {
    name: 'Joe Code',
    email: 'coder@example.com'
  }
}, callback);
```


#### <a id="optionsclone">options.clone</a>
 * type: `string`
 * default: temporary directory inside the `gh-pages` directory

Path to a directory where your repository will be cloned.  If this directory doesn't already exist, it will be created.  If it already exists, it is assumed to be a clone of your repository.

Example use of the `clone` option:

```js
/**
 * If you already have a temp directory, and want the repository cloned there,
 * use the `clone` option as below.  To avoid re-cloning every time the task is
 * run, this should be a directory that sticks around for a while.
 */
ghpages.publish(path.join(__dirname, 'build'), {
  clone: 'path/to/tmp/dir'
}, callback);
```


#### <a id="optionspush">options.push</a>
 * type: `boolean`
 * default: `true`

Push branch to remote.  To commit only (with no push) set to `false`.

Example use of the `push` option:

```js
ghpages.publish(path.join(__dirname, 'build'), { push: false }, callback);
```


#### <a id="optionssilent">options.silent</a>
 * type: `boolean`
 * default: `false`

Suppress logging.  This option should be used if the repository URL or other information passed to git commands is sensitive and should not be logged.  With silent `true` log messages are suppressed and error messages are sanitized.

Example use of the `silent` option:

```js
/**
 * This configuration will suppress logging and sanitize error messages.
 */
ghpages.publish(path.join(__dirname, 'build'), {
  repo: 'https://' + process.env.GH_TOKEN + '@github.com/user/private-repo.git',
  silent: true
}, callback);
```


#### <a id="optionslogger">options.logger</a>
 * type: `function(string)`
 * default: `function(){}`

Logger function.  The default logging function is a no-op, allowing you to provide a custom logging implementation.

Example use of the `logger` option:

```js
/**
 * This configuration will log to the console
 */
ghpages.publish(path.join(__dirname, 'build'), {
  logger: function(message) {
    console.log(message);
  }
}, callback);
```


#### <a id="optionsgit">options.git</a>
 * type: `string`
 * default: `'git'`

Your `git` executable.

Example use of the `git` option:

```js
/**
 * If `git` is not on your path, provide the path as shown below.
 */
ghpages.publish(path.join(__dirname, 'build'), {
  git: '/path/to/git'
}, callback);
```

## Dependencies

Note that this plugin requires Git 1.7.6 or higher (because it uses the `--exit-code` option for `git ls-remote`).  If you'd like to see this working with earlier versions of Git, please [open an issue](https://github.com/tschaub/gh-pages/issues).

[![Current Status](https://secure.travis-ci.org/tschaub/gh-pages.png?branch=master)](https://travis-ci.org/tschaub/gh-pages)
