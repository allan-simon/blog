+++
date = "2015-02-10T18:19:22+08:00"
draft = true
title = "Solve composer github api limitation"

+++

### Could not fetch X enter your GitHub credentials to go over the API rate limit

The problem is that composer is retrieving nearly all the packages from github
using the anonymous access github API. In order to solve it you need to generate
a token 

  1. you need a github account (or ask a friend)
  2. go on your setting page [here](https://github.com/settings/applications)
  3. click on `Generate new token`


now two possibilities 

<!--more-->

#### 1 - Put this token in one specific project


for this simply run 

```
composer config -g github-oauth.github.com YOUR_TOKEN
```

or 

```
php composer.phar config -g github-oauth.github.com YOUR_TOKEN
```

(depending on how you installed composer)

it will add that to your project's composer.json file


####  2 -  Put this token in your global composer configuration

If your project is an open-source or company project you may not want
to commit your token on the git of your project/company

for this edit the file `~/.composer/config.json` and put in it:


```
{
    "config": {
        "github-oauth": { "github.com": "YOUR_TOKEN" }
    }
}
```

that's all folks!

Source:

  * [coderwall.com](https://coderwall.com/p/kz4egw/composer-install-github-rate-limiting-work-around)
  * [composer's github](https://github.com/composer/composer/issues/3542) 
  * [github's blog](https://github.com/blog/1509-personal-api-tokens) More details about Github token
