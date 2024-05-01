# hugo

This site is built on Azure Static Apps using [hugo](https://gohugo.io/)(v0.125.3). The theme is [coder](https://themes.gohugo.io/themes/hugo-coder) and the initial configuration was copied from [the maintainer docs](https://github.com/luizdepra/hugo-coder/blob/main/docs/configurations.md#complete-example) following the [hugo quickstart guide](https://gohugo.io/getting-started/quick-start/). The init config was converted from toml to yaml using [transform.tools](https://transform.tools/toml-to-yaml).

## Folders removed from default

`hugo new site` autogenerates a number of folders without content. These are not included in the git index pushed to the GitHub remote. at present these "hidden" folders are:
- static/
- layouts/
- i18n/
- data/
- content/
- assets
