# Using `jstack-be:______` scoped packages from the GitHub package registry in NodeJS projects

## 1. Github Personal Access Token

To be able to delegate downloading `jstack-be` scoped packages from GitHub packages to your package manager (npm, pnpm, yarn) wou will need a GitHub personal access token (PAT) with the `read:packages` scope.

To get one, go to [https://github.com/settings/tokens](https://github.com/settings/tokens), click ‘Tokens (classic)’ and ‘Generate new token (classic)’.

![PAT instruction 1](https://github.com/jstack-be/.github/blob/main/res/guides/pat-1.png)
![PAT instruction 2](https://github.com/jstack-be/.github/blob/main/res/guides/pat-2.png)

Give your new PAT a descriptive name like `gh packages jstack-be read' and choose an expiration period.  
It's okay to select 'no expiration', but the choice is up to you. You will have to regenerate your token after your selected period has passed.

![PAT instruction 3](https://github.com/jstack-be/.github/blob/main/res/guides/pat-3.png)

Be sure to enable the `read:packages` scope.

![PAT instruction 4](https://github.com/jstack-be/.github/blob/main/res/guides/pat-4.png)

Click ‘Generate token’. copy the value of your token and store it in a secure place.  
❗Github will show you the value of your token only one time.  
❗Store your token somewhere safe where you can access it again later.  

## 2. Making the PAT value available in your environment

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

## 3. Configuring a node project to use a jstack-be scoped package

Add a `.npmrc` file to your project, in the same directory as your `package.json` file.  
Use the following contents:

```
# Configure npm to look for @jstack-be scoped packages 
# in the GitHub packages registry
@jstack-be:registry=https://npm.pkg.github.com

# Tell npm to use the value of the GH_PKGS_READ_TOKEN
# environment variable as an auth token for accessing the registry
//npm.pkg.github.com/:_authToken=${GH_PKGS_READ_TOKEN}
always-auth=true
```

This should work for npm, yarn, and pnpm.  
If your project uses yarn 2+, use [yarn’s instructions](https://yarnpkg.com/configuration/yarnrc) for converting `.npmrc` to `.yarnrc.yml`

## 4. Installing a @jstack-be scoped package in your project

Add a @jstack-be scoped package to your `package.json`

```json
"dependencies": {
    "@jstack-be/retry-wrapper": "^1.0.4"
}
```

If you followed all the steps in this guide correctly, you should now be able to run `npm install` / `yarn` / `pnpm install`
