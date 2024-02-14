# My Website AKA Questionable experiments in Quarto

This is a work in progress, please come back later :) 

The output/website can be accessed at: https://questionable.quarto.pub/blog/

## Previewing 

The website can be previewed by using the terminal to move into the main quarto website folder and then running `quarto preview`. 

## Publishing 1.0 - it's alive!

Done through: https://quarto.org/docs/publishing/quarto-pub.html 

I have two domains I can publish to: 

 - questionable.quarto.com
 - lisa.quarto.com
 
I run `quarto publish quarto-pub` after cd-ing in to my my_website directory. Answer "Y" to overwrite my previous site and to use the correct account. Alternatively can bypass prompts and render with `quarto publish quarto-pub --no-prompt --no-render`. 

I can now access my account and see my deployments at https://questionable.quarto.pub/blog/. 

The "API error 401" can be resolved by removing and reconnecting the account using `quarto publish accounts` to remove the account. You will be prompted to add an account when `quarto publish quarto-pub` is next run. 

## Implement a broken link checker 

Resources: 

- <https://nanx.me/blog/post/rmarkdown-quarto-link-checker/> 
- <https://github.com/quarto-dev/quarto-cli/issues/1319> 
- <https://github.com/lycheeverse/lychee> 

If using netlify can use something like this (convenient because it can run inside git, and will run after every commit) using [netlify-plugin-quarto
](https://github.com/quarto-dev/netlify-plugin-quarto) and [netlify-plugin-checklinks](https://github.com/Munter/netlify-plugin-checklinks) after installing with `npm install -D netlify-plugin-checklinks`: 

```
[[plugins]]
package = "@quarto/netlify-plugin-quarto"

    [plugins.inputs]
    cmd = "render --site-url $DEPLOY_PREVIEW_URL"
    
[[plugins]]
package = "netlify-plugin-checklinks"

  [plugins.inputs]
  # An array of glob patterns for pages on your site
  # Recursive traversal will start from these
  entryPoints = [
    "*.html",
  ]

  # Recurse through all the links and asset references on your page, starting
  # at the entrypoints
  recursive = true

  # Checklinks outputs TAP (https://testanything.org/tap-version-13-specification.html)
  # by default. Enabling pretty mode makes the output easier on the eyes.
  pretty = true

  # You can mark some check as skipped, which will block checklinks
  # from ever attempting to execute them.
  # skipPatterns is an array of strings you can match against failing reports
  skipPatterns = [
    "https://www.oracle.com/",
    "../../installation"
  ]

  # You can mark some check as todo, which will execute the check, but allow failures.
  # todoPatterns is an array of strings you can match against failing reports
  todoPatterns = []

  # Report on all broken links to external pages.
  # Enabling this will make your tests more brittle, since you can't control
  # external pages.
  checkExternal = false

  # Enable to check references to source maps, source map sources etc.
  # Many build tools don't emit working references, so this is disabled by default
  followSourceMaps = false
```

Or using Lychee with something like: 

.lycheeignore
```
https://twitter.com
https://example.com
```

.github/workflows/links.yml
```
name: Check Links

on:
  repository_dispatch:
  workflow_dispatch:

jobs:
  linkChecker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2
      
      - name: Render site
        uses: quarto-dev/quarto-actions/render@v2

      - name: Restore lychee cache
        uses: actions/cache@v3
        with:
          path: .lycheecache
          key: cache-lychee-${{ github.sha }}
          restore-keys: cache-lychee-

      - name: Link Checker
        id: lychee
        uses: lycheeverse/lychee-action@v1.8.0
        env:
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
        with:
          args: '--cache --max-cache-age 1d _site/**/*.html'

      - name: Create Issue From File
        if: env.lychee_exit_code != 0
        uses: peter-evans/create-issue-from-file@v4
        with:
          title: Link Checker Report
          content-filepath: ./lychee/out.md
          labels: report, automated issue
```

This can be run locally to check for broken links with: 

Create a virtual environment:

```
python3 -m venv utils/venv

source utils/venv/bin/activate

python3 -m pip install -r utils/requirements.txt

source utils/venv/bin/activate
```

Create a justfile that will build the site and run the link checker: 

justfile
```
# build the site locally and test links
linkcheck-local: 
    quarto render
    linkchecker -t 20 -o html _site/index.html > linkchecker.html
```

Build the site and run the link checker:

```shell
just linkcheck-local
```

This will build the site into the `_site` directory and run `linkchecker` against it.

A report will be written to `linkchecker.html`, which you can view in any browser.

Other checks can be added, for example 

main.yml
```
name: lint
on: [pull_request]

jobs:
  vale:
    name: runner / vale
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2

      - name: Render site
        uses: quarto-dev/quarto-actions/render@v2
      
      - name: Lint
        uses: errata-ai/vale-action@reviewdog
        with:
          files: _site/
        env:
          # Required, set by GitHub actions automatically:
          # https://docs.github.com/en/actions/security-guides/automatic-token-authentication#about-the-github_token-secret
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

And github actions can be used to setup a manual link checker action that can be run: 

linkschecker.yml

```
name: Check Links

on:
  repository_dispatch:
  workflow_dispatch:

jobs:
  linkChecker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2
      
      - name: Render site
        uses: quarto-dev/quarto-actions/render@v2

      - name: Restore lychee cache
        uses: actions/cache@v3
        with:
          path: .lycheecache
          key: cache-lychee-${{ github.sha }}
          restore-keys: cache-lychee-

      - name: Link Checker
        id: lychee
        uses: lycheeverse/lychee-action@v1.8.0
        env:
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
        with:
          args: '--cache --max-cache-age 1d _site/**/*.html'

      - name: Create Issue From File
        if: env.lychee_exit_code != 0
        uses: peter-evans/create-issue-from-file@v4
        with:
          title: Link Checker Report
          content-filepath: ./lychee/out.md
          labels: report, automated issue
```

## Publishing 2.0 - now make it automated using github actions

TODO
 
## Troubleshooting

ojs errors: This was resolved by completely uninstalling quarto, uninstalling the RStudio IDE, and then re-installing. 
 

## Inspiration 

There are so many awesome resources out there making building your own website/blog a breeze. Here are some of my favorites that I've stumbled across in my journey that you might enjoy: 

 - [The ultimate guide to starting a Quarto blog](https://albert-rapp.de/posts/13_quarto_blog_writing_guide/13_quarto_blog_writing_guide.html)
 - [Creating a blog with Quarto in 10 steps](https://beamilz.com/posts/2022-06-05-creating-a-blog-with-quarto/en/)
 - [Notes from a data witch](https://blog.djnavarro.net/posts/2022-04-20_porting-to-quarto/)
 - This is rmarkdown, but [Yihui's home page](https://yihui.org/todo/) is beautiful


## Make it pretty 

There are so many different [theme options](https://quarto.org/docs/output-formats/html-themes.html#overview) available to use. You can even have a different theme for [light and dark mode](https://quarto.org/docs/output-formats/html-themes.html#dark-mode) (perfect for anyone like me who is in love with the vapor theme but want to default to something a little less "loud"). 

Some of my unapologetic favorites are: 
 - [quartz](https://bootswatch.com/quartz/) (that gradient!)
 - [vapor](https://bootswatch.com/vapor/) (who isn't into vaporwave/cyberpunk)
 - [sketchy](https://bootswatch.com/sketchy/) (I've always wanted to be an artist)

I had some challenges trying to get the theme to use the secondary colors instead of using the primary everywhere. 

This is where [notes from a data witch](https://blog.djnavarro.net/posts/2022-04-20_porting-to-quarto/#styling-the-new-blog) is a legend. Using that method of creating a custom `.scss` file that copies itself from the theme of my choice. 

We do that by referencing our custom `.scss` file in our yaml with: 
    `theme: `
    `  light: custom_theme.scss`

From the about pages we can learn about the [different templates](https://quarto.org/docs/websites/website-about.html) that come built in to quarto for making it really easy to lay things out. 

## Let's talk about images 

The pain point I have dealt with for so long is now pure magic. 

Read about it: [https://quarto.org/docs/authoring/figures.html#figure-panels](https://quarto.org/docs/authoring/figures.html#figure-panels)

One step further - Have the images "pop up" when clicked using this quarto extension: [https://github.com/quarto-ext/lightbox](https://github.com/quarto-ext/lightbox)


 
