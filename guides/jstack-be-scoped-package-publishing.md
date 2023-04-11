# Publishing `@jstack-be:______` scoped packages to GitHub packages

- [Publishing `@jstack-be:______` scoped packages to GitHub packages](#publishing-jstack-be______-scoped-packages-to-github-packages)
  - [Configuring a node project for publishing to GitHub packages](#configuring-a-node-project-for-publishing-to-github-packages)
    - [Publishing through Github Actions](#publishing-through-github-actions)
    - [Publishing through the command line](#publishing-through-the-command-line)
  - [Setting package visibility](#setting-package-visibility)
  - [Creating Github Personal Access Token (PAT)](#creating-github-personal-access-token-pat)
  - [Making the PAT value available in your environment](#making-the-pat-value-available-in-your-environment)


## Configuring a node project for publishing to GitHub packages

### Publishing through Github Actions

First, add a publish config to your `package.json` file:

```json
  "publishConfig": {
    "jstack-be:registry": "https://npm.pkg.github.com"
  },
```

Then, create a workflow to perform the publish.  

This example yaml creates a Github Actions workflow that triggers when a release is created. You can use any trigger you want. Be careful to not over-publish your package. Publishing on each push to each branch would be overkill. A good practice would be to only publish on releases or on merge to main.  
The `GITHUB_TOKEN` secret is available in all workflows and already has the `write:packages` scope for packages in the jstack-be organization. It is a good idea to still specify the `permissions` key in your workflow file, to make it clear what scopes your workflow needs.  
You can add any others steps you want to this workflow, like running tests, linting, etc.
```yaml
# .github/workflows/publish.yml
name: publish on release
on:
  release:
    types: [created]
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          registry-url: https://npm.pkg.github.com/
      - run: npm install
      - run: npm run build
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Publishing through the command line

First, you'll need a PAT to publish from your local machine. Refer to [this section](#creating-github-personal-access-token-pat) on how to get one and [this section](#making-the-pat-value-available-in-your-environment) on how to make it available in your environment.

Add a publish config to your `package.json` file:

```json
  "publishConfig": {
    "jstack-be:registry": "https://npm.pkg.github.com"
  },
```

Add a `.npmrc` file to your project, in the same directory as your `package.json` file, probably at the root of your repository or package.  
Use the following contents:

```
# Configure npm to look for @jstack-be scoped packages in the GitHub packages registry, 
# or to publish them to it
@jstack-be:registry=https://npm.pkg.github.com

# Tell npm to use the value of the GH_PKGS_READ_TOKEN
# environment variable as an auth token for accessing the registry
//npm.pkg.github.com/:_authToken=${GH_PKGS_RW_TOKEN}
always-auth=true
```

This should work for npm, yarn (v1), and pnpm.  
If your project uses yarn 2+, use [yarn’s instructions](https://yarnpkg.com/configuration/yarnrc) for converting `.npmrc` to `.yarnrc.yml`.  
  
You should now be able to run `npm publish`, `yarn publish`, or `pnpm publish` to publish your package to GitHub packages.

## Setting package visibility

@jstack-be scoped packages [are listed here](https://github.com/orgs/jstack-be/packages).  
Github documentation on package access control and visibility [can be found here](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility).  
  
You can probably figure out how to achieve the specific visibility you want from these docs, but if you simply want your package to be visible to all jstack-be org members, you can either (by order of preference):
- Set the package's visibility to `internal` in the package's settings. This way, everyone in the jstack-be organization can see the package.
- Go to your package's settings, click 'Invite teams or people' and add the `jstack-be/developers` team as a member with `Role: Read`.
- Add the `jstack-be/developers` team as a collaborator to the repository the package's codebase is hosted in.


## Creating Github Personal Access Token (PAT)

To delegate publishing `jstack-be` scoped packages to GitHub packages you will need a PAT with at least the `write:packages` scope.

To get one, go to [https://github.com/settings/tokens](https://github.com/settings/tokens), click ‘Tokens (classic)’ and ‘Generate new token (classic)’.

![PAT instruction 1](https://github.com/jstack-be/.github/blob/main/assets/guides/package-publishing/pat-1.png)
![PAT instruction 2](https://github.com/jstack-be/.github/blob/main/assets/guides/package-publishing/pat-2.png)

Give your new PAT a descriptive name like `gh packages jstack-be rw' and choose an expiration period.  
It's okay to select 'no expiration', but the choice is up to you. You will have to regenerate your token after your selected period has passed.

![PAT instruction 3](https://github.com/jstack-be/.github/blob/main/assets/guides/package-publishing/pat-3.png)

Be sure to enable the `write:packages` and/or `delete:packages` scopes based on your needs.

![PAT instruction 4](https://github.com/jstack-be/.github/blob/main/assets/guides/package-publishing/pat-4.png)

Click ‘Generate token’. copy the value of your token and store it in a secure place.  
❗Github will show you the value of your token only one time.  
❗Store your token somewhere safe where you can access it again later.  

## Making the PAT value available in your environment

If you set an environment variable with the value of your token, you facilitate sharing `.npmrc` files among the team an among projects (see a further step in this guide).  
The recommended name for this variable is `GH_PKGS_ACCESS_TOKEN`.  
The way you set a global and permanent environment variables depends on your shell of choice.

```bash
# bash
# add the following line to your ~/.bashrc
export GH_PKGS_ACCESS_TOKEN="PAT value here"

# zsh
# add the following line to your ~/.zshrc
export GH_PKGS_ACCESS_TOKEN="PAT value here"

# fish
set -Ux GH_PKGS_ACCESS_TOKEN "PAT value here"
```