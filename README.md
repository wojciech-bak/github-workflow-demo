Steps needed to start working on this project locally:

    git clone git@github.com:wojciech-bak/github-workflow-demo.git
    cd github-workflow-demo
    npm install


# Quality first! 5 easy steps to lint your code with GitHub workflows in Node project

Code quality is a crucial part of development process, especially when you want to work efficiently in a long-term manner. There are many approaches and best practices, including whole agile methodologies stuff, but most of them relate to some big, enterprise project conducted by at least 6 persons.

What should we do, when the project is small or the customer still doesn’t know if it’s worth investing more? Obviously, at MVP stage of the project, code styling or unit tests are not the top priority stuff. Investors usually want to have good product and c’mon - if it works, it doesn’t need testing, right?

Actually, I have some experiences in building apps from scratch, even without using best practices on the first place. Some business circumstances forced me to look for the compromise between investor’s budget plans and developer’s „nice-to-have” list. Thankfully, if you use GitHub, most of common issues related to code quality can be solved in few minutes.

**In this article I’m gonna show you how to use GitHub workflows in Node.js environment to standardise your codebase.**

---
Few assumptions before we start:
- You are familiar with NPM and Linux console.
- You have some experience with styles preprocessors, module loaders, bundlers etc.
- You know what linters are for and really want to use them in your projects.
---


## 1. Typical Node project structure
If you ever used some JS frameworks like Vue or React, you can easily spot some common things between them, e.g.:
-  `/src` directory with all your JS logic and components,
-  `/test` directory for unit and e2e tests,
-  `/assets` directory for styles, images etc.

Obviously, as we’re talking about Node environment, there should be also some Node stuff like `package.json`, `package-lock.json` and `/node_modules` catalogue in our root directory.

All these things are on their place - that's what we call __convention__. Frameworks are invented to provide some reasonable conventions, so usually we don't even need to care about the initial design pattern. As in this example I want to explain some approach, I won't apply any ready-to-use solutions like `Vue CLI`.

__Time to understand what's lying under all these magic `lint` scripts!__


## 2. Extending typical Node project
To provide high-quality solutions, linters are the first thing we should start with while setting up new project. Let’s focus on two linters - Stylelint for styles (`*.scss`) and ESLint for source files (`*.js`). Both of these linters are available on NPM and pretty easy to config. Using linters requires going through installation process, adding config files and defining project scripts. Let’s do it step-by-step.

## 3. Adding Stylelint
Installation of Stylelint in Node environment is really simple. According to [official docs](https://stylelint.io/user-guide/get-started), you just need to run: 
    
    npm install --save-dev stylelint stylelint-config-standard

and wait until it’s done. 

`stylelint-config-standard` provides default set of linting rules and can be replaced with any package which suites your needs better (e.g. [Airbnb style](https://www.npmjs.com/package/stylelint-config-airbnb)). Then create a new hidden file `.stylelintrc.json`, which is Stylelint configuration file, responsible for loading our pre-defined rules:

```json
{
    "extends": "stylelint-config-standard"
}
```

Right now, the only missing thing is some NPM script (or scripts) declared in `package.json` file to start linting our SCSS files. Here is my proposition:

```json
"scripts": {
    ...
    "lint:scss": "stylelint '**/*.scss' --syntax scss -f verbose --color",
    "lint:scss:fix": "stylelint '**/*.scss' --syntax scss --fix -f verbose —color"
}
```

As you can see, I’ve declared the script containing `—fix` option - this one is to use before pushing changes to repository.

__Remember - it’s a bad practice to use „—fix” in your CI flow, because then the code you pass to production is not styled correctly.__ That’s why we need both of the scripts.

Let’s test our linter by creating a file `/assets/scss/styles.scss` with some content, like:

```scss
body {
            background-color: #fff;
}
```

Note that I made large indentation. I did it intentionally as linter should catch it. Let's run the command to see if it works.

    npm run lint:scss

You should see in your console something like this:

```bash
> stylelint '**/*.scss' --syntax scss -f verbose --color


assets/scss/styles.scss
 2:21  ✖  Expected indentation of 2 spaces   indentation

1 source checked
 ~/Codest/Projects/github-workflow-demo/assets/scss/styles.scss

1 problem found
 severity level "error": 1
  indentation: 1

npm ERR! code ELIFECYCLE
npm ERR! errno 2
npm ERR! github-workflow-demo@1.0.0 lint:scss: `stylelint '**/*.scss' --syntax scss -f verbose --color`
npm ERR! Exit status 2
```

This actually means that our linter works!

The output shows exactly which line causes an error and describes the issue to solve. Some issues are not fixable automatically as they need developer’s decision, but in majority of cases you just need to run same command with `—fix` option, so let’s run it.

    npm run lint:scss:fix

Now you should see green output with no errors found:

```bash
stylelint '**/*.scss' --syntax scss --fix -f verbose --color


1 source checked
 /Users/wojciechbak/Codest/Projects/github-workflow-demo/assets/scss/styles.scss

0 problems found
```

## 4. Adding ESLint

This step is almost the same as the one before. We're gonna install ESLint, define some default rules set and declare two callable NPM scripts - one for CI, one for pre-push. Let's go through this!

If you use NPM in your daily work, perhaps you'd like to install it globally. If you don't, please check the installation instructions in [official docs](https://eslint.org/docs/user-guide/getting-started).

    npm install -g eslint

When `eslint` command is available on your machine, just run this in your project:

    eslint --init

Following further instructions displayed in your terminal, just make few project decisions like:
- Javascript or TypeScript
- Aibnb style or Google style
- configuration type (JSON file, JS file or inline in `package.json`)
- ES modules (`import`/`export`) or `require` syntax

If it's done, ESlint config file (e.g. `.eslintrc.json`, depending on what you've chosen before) should appear in your root directory, somewhere next to `.stylelintrc.json` created before.

Now we have to define scripts in `package.json` file, same as for SCSS files:
 ```json
 "scripts": {
     ...
    "lint:js": "eslint '**/*.js' --ignore-pattern node_modules/",
    "lint:js:fix": "eslint '**/*.js' --ignore-pattern node_modules/ --fix"
 }
```

Congrats! ESLint is ready to use right now. Let's check if it works correctly. Create `/src/index.js` file with some content:
```javascript
console.log("something");
```

Run linter:

    npm run lint:js

The output should look like this:

```bash
> eslint '**/*.js' --ignore-pattern node_modules/


~/Codest/Projects/github-workflow-demo/src/index.js
  1:1  warning  Unexpected console statement  no-console

✖ 1 problem (0 errors, 1 warning)

```

This warning won't disappear after using `--fix` option, because __linters doesn't affect potentially meaningful code. They are only for code styling__, including white-spaces, newlines, semicolons, quotation marks etc.

## 5. Defining GitHub workflows.

[GitHub workflows](https://help.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow) are pretty well-documented thing. Feel free to dig deeper into this, but for now I'm gonna implement simple workflow to lint our code after `push` to remote repository (obviously, hosted on GitHub).

Create `/.github/workflows` directory and new `code-quality-workflow.yml` file in there as GitHub workflows have to be defined with YAML files.

To run proper workflow, we have to answer few questions:
- When we want to run our workflow (on push, on pull request, on merge etc.)?
- Do we want to exclude some situations (like push to branch `master`)?
- What environment we need to setup to run our commands correctly (in this example - Node)?
- Do we need to install dependencies? If so, how should we cache it?
- What steps we need to make to complete the check?

After some considerations and few hours of work with docs example `.yml` file can look like this:

```yaml
name: Code quality

on: 'push'

jobs:
  code-quality:
    name: Lint source code
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1

    - name: Setup Node
      uses: actions/setup-node@v1
      with:
        node-version: '12.1'

    - name: Cache dependencies
      uses: actions/cache@v1
      with:
        path: ./node_modules
        key: ${{ runner.OS }}-api-deps-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.OS }}-api-deps-${{ env.cache-name }}-
          ${{ runner.OS }}-api-deps-
          ${{ runner.OS }}-

    - name: Install dependencies
      run: |
        npm install

    - name: Lint files
      run: |
        npm run lint
        
```

GitHub provides all environment stuff we need. In the last line we're running command `npm run lint` which was not defined before:

```json
"scripts": {
    ...
    "lint": "npm run lint:js && npm run lint:scss"
}
```

Note that in our workflow I'm not using `:fix` commands.

When all these steps are done, you can enjoy this beautiful output from GitHub before merging your Pull Request:

![GitHub output on Pull Request page](https://i.imgur.com/BXHFSp7.png)
![GitHub "Checks" subpage on Pull Request](https://imgur.com/ocsngb6.png)