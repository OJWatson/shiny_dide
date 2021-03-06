## DIDE shiny server

### To deploy

```
ssh shiny
```

Then set everything up

```
./scripts/init
./scripts/pull_images
./scripts/vault_auth
./scripts/provision_all
./scripts/sync_server
./scripts/configure_apache
```

Do the actual deployment with

```
docker-compose up -d --scale shiny=12
./scripts/register_workers 12
```

### To add a new application


**First, set up your repository so it can be automatically provisioned**.  To do this, add a file `provision.yml` to your repository so that dependencies can be automatically added.  This should live in the directory that your application lives in (i.e., alongside `app.R` or `server.R`/`ui.R`) and that might not be the root of the repository.  There are two probable types here.

1. if your repository is just a shiny app you will need something like [this](https://github.com/arranhamlet/POLICI/blob/cb97ebad7264e6d77206382b826303b769171d5c/provision.yml) with a list of packages:

```yaml
packages:
  - package1
  - package2
```

if this is alphabetical that's easier to keep up to date.  All packages used anywhere in your application need to be included (i.e., everything used by `library`, `require`, `::`, `:::`, `loadNamespace`, `requireNamespace`).  Don't worry about dependencies of those packages though, they'll be installed automatically.

2. If your repository is actually a package and the package is at the root of the repository, then you might want to use

```
self: true
packages:
  - package1
  - package2
```

Which will install your package (and its dependencies).  The `packages` section is optional and installs any additional packages required (packages that your package only `Suggests` are not included by default).

**Modify _this_ repository so that we know about your application**.  First, fork or branch this repository.  Edit the [`site.yml`](site.yml) to add your new application.  In most cases this will look like

```yaml
  <path-for-your-app>:
    type: github
    spec: <github-account>/<repo-name>
    admin: <username>
```

The keys are:

- `<path-for-your-app>` where the application will be hosted.  If you use `superhampster` then your application will be hosted at https://shiny.dide.imperial.ac.uk/superhampster
- `<github-account>/<repo-name>` is similar to what is used for `install_github` - where to find the repo.  This is `mrc-ide/typochallenge` for https://github.com/mrc-ide/typochallenge - but if your application lives at a subdirectory within your repo and you want a specific branch, tag, or commit you might want to use the full spec such as `mrc-ide/superhampster/inst/app@release` which would use the application within the directory `inst/app` on the `release` branch of the repository `mrc-ide/superhampster` (see `remotes::parse_github_repo_spec` to play with the parseing of these specs)
- `<username>`: a username that you'd like on the server.  This will be used by the (currently in development) administration interface.  This does not need to be your DIDE username.

**Is your repository private**: If so please add

```yaml
    auth:
      type: deploy_key
```

to the yaml above.  As part of the process Rich will send you a public key to add to your repo (if the repo is hosted on `mrc-ide` he will do this directly).

**Do you want a private application**: If so please add

```yaml
    groups:
      - group1
      - group2
```

to your yaml.  You should add a group to the bottom of the file too if none of the other groups look like what you want.  Just list usernames that you'd like and we can sort out the rest later.

**Let Rich know**: Create a pull request with your changes and tag `@richfitz` in the request, and let Rich know on slack too.  Join the `#shiny-server` channel on slack so that you are notified of changes.

**What happens next?**: Rich will try and deploy your application and may request a few changes so that it deploys smoothly.

**This seems like a giant faff**: I am trying to get this a bit automated so that once things are up and running you'll be able to administer your applications yourself, redeploying changes, checking logs, etc.  If you have access to docker you should also be able to test things more easily locally to make it more likely that we know things will work out the box.

### How does this work

See [`twinkle`](https://github.com/mrc-ide/twinkle) for the details.

### Other information

There is a `#shiny-server` channel on slack for status updates/problems etc.
