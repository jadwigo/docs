---
title: The Bolt JS and CSS assets
level: advanced
---
Developing and Compiling the Bolt JS and CSS assets
===================================================

Bolt's backend has a lot of JavaScript and CSS, in order to make it look good,
and work smoothly for the editors that use this environment. Meanwhile, it can
be daunting to developers who need to either debug something in the Bolt
backend, or who wish to use the available code to extend the backend in their
extensions.

This page will give some insights into how it is structured, and how you can
compile the assets in order to work with them.

For starters, Bolt's backend build process builds both the JavaScript as well
as the CSS. Both of these consist of external packages we leverage, as well as
our own code.


Quickstart
----------

To get started, you'll need to have [Node.js][node] and [Yarn][yarn] on your
system. The build process works best using Node 6+ and Yarn 1.0+.

To get the required modules, and pull in other dependencies, change into the
`assets/` directory and run:

```
yarn install --strict-semver
```

You can then build or rebuild assets files by running:

```
yarn run grunt build
```

Alternatively, you can start the Grunt "watch" task that will update the built
asset files while you're working on them by running:

```
yarn run grunt watch
```


CSS - Bootstrap and custom SCSS
-------------------------------

The CSS is based on [Bootstrap 3.3.7][bs], with our own theming and custom
styles added to that. It is built using sass. The main Bootstrap files are
found at `assets/node_modules/bootstrap-sass`. Our custom SCSS can
be found in the `assets/sass/` folder.

This folder contains the `.scss` files for the development of the back-end
theme. The `.scss` files will be compiled into normal stylesheets in the
`public/bolt/css/` folder.

There is one main `.scss` file in the `sass/` directory: `app.scss`


### Folders

Within the `modules/` folder you'll find all sass partials that should be
included in the main app `.scss` files. In `_base.scss` we set all variables,
mixins, extends and other sass statements that do not generate any styles in
the final built file.


### Media Query Handling

In `_base.scss` we have the Media Query handling. Here's the summary:

Put your code for a specific Media Query in this mixin:

```
@include respond-to('query_name') {}
```

For instance:

```
/* MQ for medium-screens and up ( min-width >=768 ) */
@include respond-to(medium-screens) {
    /*
     * some styles, e.g.
        color:$green; //gotta have greens!
     */
}
```

`$medium-screens` is coupled to a certain 'em' based width for the viewport.

As we develop 'mobile first' the base styles are not in a Media Query, so we set
the following: (assuming our base em size of `1em` is `16px`)

 - `$mid-mobile`
 - `$medium-screens`
 - `$large-screens`
 - `$wide-screens`

See `_base.scss` for the current values.


JavaScript - Bootstrap, jQuery, CKEditor and more
-------------------------------------------------

Our JavaScript source files can be found in `assets/js`. Some of the
files are included in our git repository, others are pulled in using either NPM
or our Grunt build process.

The final JavaScript files are built int `public/bolt/js/`.


Available Grunt tasks
---------------------

- **`grunt`**<br> Starts the watch task that monitors the JavaScript and Sass source files and
  automatically rebuilds `bolt.js`, `bolt.css` and `liveeditor.css` when changes are detected.

- **`grunt updateBolt`**<br> Manually starts a rebuild of `bolt.js`, `bolt.css` and
  `liveeditor.css`.

- **`grunt updateLib`**<br> Updates everything that depends on external resources, either
  provided by npm or embedded in the `lib` folder. This command mainly builds `lib.js` and
  `lib.css` from external libraries, installs fonts, CKEditor and library locale files. It has to
  be run after any update to one of the external resources.

- **`grunt prepareCkeditor`**<br> Does some cleanup on CKEditor files in `lib/ckeditor` after
  updating it. Update process:

    * Get the latest version with URL extracted from `lib/ckeditor/build-config.js`.
    * Empty the folder `lib/ckeditor` and unpack the newer version there.
    * Run `grunt prepareCkeditor` to get files prepared.
    * Run `grunt updateLib` to get everything in place.

- **`grunt docJs`**<br> Generates documentation of Bolt's own JavaScript modules in the folder
  `docs/js`.

- **`grunt docPhp`**<br> Generates documentation of Bolt source files in the folder `docs/php`.

- **`grunt lintHtml`**<br> Downloads the Bolt backend pages defined in `grunt-local/pages.js` and
  checks them for html errors and problems.

- **`grunt lintBoot`**<br> Downloads Bolt backend pages defined in `grunt-local/pages.js` and
  checks them for Bootstrap errors and problems.


Local options
-------------

Add JS options files to the folder `assets/grunt-local/`, in which
you can put the options you want to overwrite. The content of these files look
like:

```javascript
    module.exports = {
        value: "The value"
    };
```
These files will automatically be ignored by git.


### Sourcemaps

If it doesn't yet exist, create the file `assets/grunt-local/sourcemap.js`.

A sample file to enable generation of sourcemaps looks like this:

```javascript
    module.exports = {
        css: true,
        js: true
    };
```


### Pages

For the linting tasks you have to define a list of pages to download to the
`tmp/pages` folder. If it doesn't yet exist, create the file `assets/grunt-local/pages.js`.

A sample file to enable this task looks like this:

```javascript
    module.exports = {
        baseurl: "http://bolt.localhost/bolt/",
        requests: { … }
    };
```

The key of the `requests` part is the filename and the value defines the page
to download.

- If no extension is given on the request key `.html` is automatically appended.
- If the value is a string it is handled as a GET request with that value a relative url.
- If the value is an empty string the key is used as value.
- If the value is an object it is used as request configuration (see https://github.com/request/request).
- If the key is `@login` it is handled as not saved login request.
  The value has to be `{u: "<username>", p: "<password>"}` then.
- If the key is `@logout` it is handled as not saved logout request. The value has to be `{}` then.


#### Example: Key handling

Three requests save the same page to file `login.html`.
```javascript
    module.exports = {
        baseurl: "http://bolt.localhost/bolt/",
        requests: {
                "login": "",
                "login": "login",
                "login.html": "login"
            }
        }
    };
```


#### Example: POST request

Issue a manual login (same as `@login`, only page is saved as `dashboard.html`):

```javascript
    module.exports = {
        baseurl: "http://bolt.localhost/bolt/",
        requests: {
            dashboard: {
                url: "login",
                method: "POST",
                form: {
                    username: "<enter username here>",
                    password: "<enter password here>",
                    action: "login"
                }
            }
        }
    };
```


#### Example: "Full" interface check

```javascript
    module.exports = {
        baseurl: "http://bolt.localhost/bolt/",
        requests: {
            "login": "",
            "@login": {"u": "<enter username here>", "p": "<enter password here>"},

            // Dashboard
            "dashboard": "/",

            // Configuration: Users & Permissions
            "config-users": "users",
            "config-users-new": "users/edit",
            "config-users-edit": "users/edit/1",
            "config-roles": "roles",
            "config-permissions": "file/edit/config/permissions.yml",

            // Configuration: Main configuration
            "config-main": "file/edit/config/config.yml",

            // Configuration: Contenttypes
            "config-contenttypes": "file/edit/config/contenttypes.yml",

            // Configuration: Taxonomy
            "config-taxonomy": "file/edit/config/taxonomy.yml",

            // Configuration: Menu
            "config-menu": "file/edit/config/menu.yml",

            // Configuration: Routing
            "config-routing": "file/edit/config/routing.yml",

            // Configuration: Check database
            "config-dbcheck": "dbcheck",
            "config-prefill": "prefill",

            // Configuration: Clear the cache
            "config-clearcache": "clearcache",

            // Configuration: Change Log
            "config-changelog": "changelog",

            // Configuration: System Log
            "config-systemlog": "systemlog",

            // File Management
            "files-files": "files",
            "files-theme": "files/theme",

            // Translations
            "translations-messages": "tr",
            "translations-long-messages": "tr/infos",
            "translations-contenttypes": "tr/contenttypes",

            // Extras
            "extras-view-install": "extend",
            "extras-configure": "files/config/extensions",

            // Main Menu
            "profile": "profile",

            "@logout": {},
        }
    };
```


### Bootlint

If it doesn't yet exist, create the file `assets/grunt-local/bootlint.js`.

You can override Bootlint options, e.g.:

```javascript
    module.exports = {
        relaxerror: ["W012"],
        showallerrors: false,
        stoponerror: true,
        stoponwarning: false
    };
```


### Htmllint

If it doesn't yet exist, create the file `assets/grunt-local/htmllint.js`.

You can override Htmllint options, e.g.:

```javascript
    module.exports = {
        ignore: [
            "Element “link” is missing required attribute “property”.",
            /^Duplicate ID/
        ]
    };
```

[node]: https://nodejs.org/en/download/package-manager/
[yarn]: https://yarnpkg.com/en/docs/install
[bs]: https://github.com/twbs/bootstrap/releases/tag/v3.3.7
