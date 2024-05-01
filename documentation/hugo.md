# hugo

This site is built on Azure Static Apps using [hugo](https://gohugo.io/)(v0.125.3). The theme is [coder](https://themes.gohugo.io/themes/hugo-coder) and the initial configuration was copied from [the maintainer docs](https://github.com/luizdepra/hugo-coder/blob/main/docs/configurations.md#complete-example) following the [hugo quickstart guide](https://gohugo.io/getting-started/quick-start/). The init config was converted from toml to yaml using [transform.tools](https://transform.tools/toml-to-yaml).

## Folders removed from default

`hugo new site` autogenerates a number of folders without content. These are not included in the git index pushed to the GitHub remote. at present these "hidden" folders are:
- static/
- layouts/
- i18n/
- data/
- assets

## content/ directory

The content/ directory is the default target for the [`new content` quickstart command](https://gohugo.io/getting-started/quick-start/#add-content).

```sh
hugo new content posts/my-first-post.md
```

Unfortunately this command does not create the directory if it is missing. I manually deleted the folder at the same time as 23db905 and needed to recreate it to bypass the error message:

> Error: no existing content directory configured for this project.

Note the archetypes/ directory does not appear to be a requisite for the "my-first-post" step.
