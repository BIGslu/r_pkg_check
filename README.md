
<!-- README.md is generated from README.Rmd. Please edit that file -->

# RpkgCheck

<!-- badges: start -->

[![R-CMD-check](https://github.com/BIGslu/r_pkg_check/actions/workflows/check-standard.yaml/badge.svg)](https://github.com/BIGslu/r_pkg_check/actions/workflows/check-standard.yaml)
<!-- badges: end -->

### Create package in a new directory (RStudio)

Create new project (right top of RStudio or File -\> New Project) and
select R package in a new directory. This creates package and the
directory structure. Start with deleting `R/hello.R`, `man/hello.Rd` and
`NAMESPACE` files (last one will be automatically recreated when
`devtools::document()` is run).

### Add function

Create R script, write a function with `roxygen` comments
(documentation) and save the file in `R` directory, e.g.`pkg/R/funct.R`.

To render function documentation and update `NAMESPACE` file, run
`devtools::document()`.

### Add unit tests

Recommended testing framework is `testthat`. To initialize test file
structure, run `usethis::use_testthat()`. This also adds `testthat`
package to `Suggests` field in `DESCRIPTION` file.

Create an R script with unit test and save it as
`pkg/tests/testthat/test-<funct_name>.R`

To run full test suite locally, use `devtools::test()`

### Update `DESCRIPTION` file

-   Add authors, maintainers, title and description,
-   update license,
-   bump package version.

### initialize GitHub repo

-   create new repo on GitHub
-   go to your R package directory
-   `git init`
-   `git branch -m "main"` change name from `master` to `main` if that
    was the default
-   `git remote add origin https://github.com/username/reponame`
-   `git add <dir1> <file1> ...`
-   `git commit -m <commit_msg>`
-   `git push origin main`

### GitHub Readme

To create README template run `usethis::use_readme_rmd()`. Update
`README.Rmd`, knit with `Cmd + Shift + Enter` and `commit` `push` both
files.

### Generate GitHub token

Log in to your github.com account and click on your user, then choose
`Settings`. Select `Developer Settings` at the bottom and then
`Personal access tokens`. Click `Generate new token`. Name the token,
set expiration time and tick `workflow` as scope (and others if needed).

Use this token in your credential manager (Windows) or Keychain access
(Mac). Look for Keychain Access app on your Mac and type `git` in
searchbar. Double click on `github.com` and then the token goes to
password field.

### Add GitHub Actions

Run `usethis::use_github_action("check-standard")`

This creates `.github` directory:

    .github
    ???   .gitignore
    ???
    ???????????? workflows
           check-standard.yaml

-   `.gitignore` specifies intentionally untracked files to ignore by
    git. Simply files you don???t want to commit and push to remote
    repository. Each file is in new line and config accepts regex
    patterns, e.g.??`*.html` ignores all html files.
-   `workflows` subdirectory with GitHub Actions workflows configuration
    files,
    -   `check-standard.yaml` - workflow configuration file -
        `R CMD check`. File can be renamed to more informative name,
        e.g.??`R-CMD-check.yaml`. This directory can store multiple
        `.yaml` files for different workflows.

Here are specified branches and events that will trigger the workflow.
Default template is for `main` and `master` only:

      on:
      push:
        branches: [main, master]
      pull_request:
        branches: [main, master]

***Note:*** to trigger `R CMD check` on every branch, substitute
`[main, master]` with `'**'`.

After pushing to remote repo, go to GitHub Actions and see if the
workflow is triggered. You can click on each job to see the its??? console
log.

[Click](https://github.com/r-lib/actions/tree/master/examples) for more
details on GitHub Actions workflows for R.

### GitHub Actions status badge

To generate status badge for your workflow (see the top of this README),
go to GitHub and repo Actions. Select workflow you want to get badge,
click `...` in right top corner and then `Create status badge` and copy
rmarkdown code.

### GitHub actions examples

`.yaml` files are often used as configuration files. Indentations are
part of the syntax and indicate nesting (like Python).

#### Run R expression

You can add the following examples to previously created `.yaml` file to
add more jobs to existing workflow.

    # Run R expressions
      R-expr:
        runs-on: ubuntu-latest
        name: run-R-expr
        steps:
          - uses: actions/checkout@v2
          - uses: r-lib/actions/setup-r@master
          - run: echo $GITHUB_WORKSPACE
          - run: |
              R -e 'print("a");
              # -e flag for "run R expression"
              df <- data.frame(x = 1:10);
              head(df)'

What it does:

-   `R-expr:` - job name in yaml config file
-   `runs-on: ubuntu-latest` - operating system that runs workflow
-   `name: run-R-expr` - name of the workflow that appears in repo???s
    GitHub Actions
-   `steps:` - opens the section for workflow steps
-   `- uses: actions/checkout@v2`
-   `- uses: r-lib/actions/setup-r@master` - environments/containers
    that will check-out to the repository to access it and run GH A
    workflows. The second is used to set up and allow to run `R` code in
    the workflow
-   `- run: echo $GITHUB_WORKSPACE` - commands to be run. E.g.
    `$GITHUB_WORKSPACE` is a repo internal variable, print current
    working directory
-   `- run: |` - note `|` - it allows to write commands in multiple
    runs. To run R code, use `R -e <code`. It opens `R` session inside
    GH A workflow and runs the code. Note the `;` after function calls -
    this is to separate expressions, the same as you would run multiple
    function calls in one line in R studio -
    `print("a"); print("hello")`.

#### Run R expression on multiple versions

This examples lets you run your `R` code on multiple `R` versions.

    # R expressions on multiple R versions
      R-expr-multi-ver:
        runs-on: ubuntu-latest
        strategy:
          matrix:
            R: [ '3.6.1', '4.1.2', '4.2.0' ]
        name: R ${{ matrix.R }} ver-test
        steps:
          - uses: actions/checkout@v2
          - uses: r-lib/actions/setup-r@v2
            with:
              r-version: ${{ matrix.R }}
          - run: Rscript -e 'print(sessionInfo()$R.version$version.string)'

What is new here:

-   creates a version matrix for the job inside `strategy` clause

<!-- -->

    strategy:
      matrix:
        R: [ '3.6.1', '4.1.2', '4.2.0' ]

an object `matrix.R` is created that will the job for each of the
specified versions here. Note: same strategy was used to test
`R-CMD-Check` on multiple operating systems.

-   `name: R ${{ matrix.R }} ver-test` - name is set for each job. In GH
    A, you will see for example `R 3.6.1 ver-test`. This iteratively
    updates the name. We refer to the matrix object using `${{ }}`
-   using `with` lets us use each version from `matrix.R` inside `R`
    container `r-lib/actions/setup-r@v2` we specified.

<!-- -->

    with:
      r-version: ${{ matrix.R }}

-   `- run: Rscript -e` alternative way is to run `Rscript -e`. Also
    useful to run `.R` files, see next sections.

#### Run bash commands

    # Run bash commands in terminal
      Greetings:
        runs-on: ubuntu-latest
        name: hello-where-am-i
        steps:
          - uses: actions/checkout@v2
          - run: echo "Multiple run clause"
          - run: |
             echo "Hello"
             echo "Current workdir is:"
             echo $GITHUB_WORKSPACE
             echo "$(pwd)"
             ls -la

This job simply runs `bash` commands. If you go to GH A and see console
logs for the job, you will see printed current working directory twice:
using `$GITHUB_WORKSPACE` and `"$(pwd)"`. Note: you can simply run bash
commands, e.g. `ls -la` and see the output. We see that as default,
working directory is where `DESCRIPTION` package files is, so package
root directory.

#### Run bash script from `.sh` file

    # Run bash script from inside the repo. Add permission (chmod) before or inside
    # this job
      Bash-script:
        runs-on: ubuntu-latest
        name: run-bash-script
        steps:
          - uses: actions/checkout@v2
          - run: |
                echo "run script $(pwd)/exec/test_script.sh"
                sh ./exec/test_script.sh

You can place a bash script inside your package (here is in `exec`
directory) and run it using GH A workflow. This can be used for example
for testing a typical package workflow using `.sh`.

#### Create another workflow in GH A

If you create another `.yaml` file in `.github/workflows` directory, you
will see another workflow running in your repo.

    on:
      push:
        branches: '**'
      pull_request:
        branches: '**'

    name: R-script
    jobs:

      install-R-pkg:
        runs-on: macOS-latest
        name: install-R-pkg
        steps:
          - uses: actions/checkout@v2
          - uses: r-lib/actions/setup-r@master
            with:
              r-version: '4.1.2'
          - run: |
              Rscript $(pwd)/exec/install_script.R

Remember that creating a new file, requires us to specify events that
trigger job inside `on:` clause. `name: R-script` this is the name that
will appear in GH Actions workflows. Now we???ll see 2: `R-CMD-Check` and
`R-script`.

-   this workflow runs on `macOS`, similarly is set to use `R` and
    version is set to `4.1.2`
-   in `run:` clause we specifiy an `R` script that lives inside `exec`
    directory
-   ***Note:***`R` environment we set here, is not the same environment
    you have locally. This is a remote container with `R` and most
    probably you don???t have many packages (like `dplyr`, `devtools` etc)

### Protected branches

If you want to create a *protected branch*, i.e.??branch that cannot be
updated freely by anyone and prevent unintentionally breaking code
because of not checked changes, do the following:

Go to your repo on GitHub and click on `Settings`. Select `Branches`.

Use branch `main` as your default branch (i.e.??cloning repo and creating
branches will default to `main`).

Next, click `Add branch protection rule` and tick (recommended):

-   `Require a pull request before merging`,
-   `Require approvals`,
-   `Require status checks to pass before merging`,
-   `Require branches to be up to date before merging`.

???and anything else, if you want to be more specific about the repository
rules.

Repository owner can merge branches / pull requests without approvals.
*With great power comes great responsibility* :muscle: :whale2:
