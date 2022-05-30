+++
title = "Hugo"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

{{< toc >}}

## Create Hugo Site

## Install Hugo from [https://gohugo.io/]

### From Mac

> brew install hugo

### From Windows

> choco install hugo -confirm

Reference: [https://gohugo.io/getting-started/installing#chocolatey-windows]

## Create GitHub Page

From GitHub.com -> Create a new Repository, e.g., AlainBouchard.github.io

## Create or Clone exiting Hugo repository

In this example, we'll use project `engineering-notes-site`

### Clone the GitHub repo to publish on the hugo site

```text
engineering-notes-site> git clone git@github.com:AlainBouchard/engineering-notes-site.git
```

### Create a new Hugo site

```text
engineering-notes-site> hugo new site --force
engineering-notes-site> ls
archetypes  config.toml  content  data  layouts  public  static  themes
```

### Clone the Theme directory

Example of theme, many can be found on Hugo site.

Hugo Theme Source: [https://themes.gohugo.io/themes/hugo-theme-relearn/]

```text
cd themes
themes> git clone git@github.com:McShelby/hugo-theme-relearn.git
```

### Update the Hugo config.toml file content with team

```bash
> vi config.toml
```

```toml
baseURL = "https://AlainBouchard.github.io/"
languageCode = "en-us"
title = "Alain Bouchard's Engineering Notes"
theme = "hugo-theme-relearn"

[outputs]
    home = [ "HTML", "RSS", "JSON"]

[module]

  [[module.imports]]
    path = "github.com/alain-bouchard-quality/engineering-notes"

    [[module.imports.mounts]]
      source = "content"
      target = "content"
```

### Build

```bash
> hugo -t hugo-theme-relearn
```

### Verify if the configuration is good

``` bash
> hugo server
```

### Update the logo

- Create statics/logo.png
- Create layouts/partials/logo.html

  ```xml
  <img src="logo.png">
  ```

- Link (git submodule) the documents into the GitHub Page

  ```bash
  > rm -Rf public
  > git submodule add -b master git@github.com:AlainBouchard/AlainBouchard.github.io.git public
  > git remote -v
  ```

- Expect the public directory to get created with the repo content

### Use github documentation repo as content

#### In document source github repo

Source repo example: `github.com/AlainBouchard/engineering-notes`

```bash
> hugo mod init github.com/AlainBouchard/engineering-notes
```

#### In hugo project github repo

Hugo repo example: `github.com/AlainBouchard/engineering-notes-site`

```bash
> hugo mod init github.com/AlainBouchard/engineering-notes-site
```

Verify if the module work:

```bash
> hugo mod get github.com/AlainBouchard/engineering-notes
```

Expect `go.sum` to get created:

```script
github.com/AlainBouchard/engineering-notes v0.0.0-20220518202018-bd023ee889d6 h1:0s4tNjEN0+jGCkNEObTbnW+akRJD5RdjJ8pPsUU5ROU=
github.com/AlainBouchard/engineering-notes v0.0.0-20220518202018-bd023ee889d6/go.mod h1:Z6BTmpjCul+gI1Za7PY2E0LgyfIJpyeII/zcRE3e654=
```

Expect `go.mod` to get updated:

```go
module engineering-notes-site

go 1.18

require github.com/AlainBouchard/engineering-notes v0.0.0-20220518202018-bd023ee889d6 // indirect
```

## Try locally

1. Build

    ```bash
    > hugo
    > hugo server --disableFastRender --ignoreCache
    ```

1. Expect Hugo to run on [http://localhost:1313/]

## Build for GitHub Page

1. build for theme

    ```bash
    > hugo -t hugo-theme-relearn
    > go the `/public`
    > git status
    > git add .
    > git commit -m "xyz"
    > git push
    ```

1. expect the GitHub Page repo to be updated.

## Links and references

### Training

- {{< youtube LIFvgrRxdt4 >}} (10 min)

### References

- ({{< external-link "https://www.hugofordevelopers.com/articles/master-hugo-modules-handle-content-or-assets-as-modules/" "https://www.hugofordevelopers.com/articles/master-hugo-modules-handle-content-or-assets-as-modules/" >}})
- [https://www.thenewdynamic.com/note/develop-hugo-modules-locally/]
- [https://mcshelby.github.io/hugo-theme-relearn/cont/pages/]
- [https://learn.netlify.app/en/shortcodes/children/]
- [https://codingreflections.com/hugo-table-of-contents/]
