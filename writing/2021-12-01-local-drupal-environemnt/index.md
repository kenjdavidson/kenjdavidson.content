---
title: "Getting Drupal 8 Up and Running"
description: Lucky me got tasked with having to setup a local Drupal 8 environment provided by a vendor we chose to do our migration.. let's just say it wasn't fun.
date: 2021-12-01
tags: [Technology, PHP, Drupal 8]
slug: "/2021/12/01/getting-drupal-8-up-and-running"
---

> First off, just let me say that I'm not the biggest fan of PHP - something about it just doesn't fit right for me.  Which made this task even more difficult!  So if something is up with this or I've made some poor decisions or poorer executions please let me know.

So, we recently got a vendor (which won't be named here due to how absolutely terrible they are) to migrate our Drupal 6 site to Drupal 8.  Given that I literally knew (and probably still don't) anything about Drupal I assumed it would be a fairly straigh forward process.  I've worked with and as a vendor before, I get how things go - the games you play trying to milk more mony out of clients.

> First of all it's a super shady (borderline assholish) practice that I couldn't stand then - when I was working for a vendor providing great service - it's even worse now working with a terrible vendor providing terrible service.

Due to the lack of knowledge and training that we got transferred to us in terms of development environments, I was tasked with getting things setup myself.  Since I generally try not to do things half assed I decided to jump all in and get everything I could think of setup.

## Cloning the Repository

The one shining star in this is that we were responsible for maintaining the git repository where the vendor would put their changes.  First things first let's get things cloned:

```
$ git clone <repsoitory>
```

which went great.

## Deciding on a Development Environment

So this is where things got tricky, there are so many different ways in which we can create and maintain the environment (looking through a few files I found that they were using MAMP on MacOS), but after talking to some other Drupal-ists (the fact that this is a thing) pointed me in the direction of [Lando](https://docs.lando.dev/).

### Lando 

TLDR; after getting this environment setup, it was unusably slow on Windows so I scratched it, continue to [#mamp](#mamp).

Although we aren't using it, here's the `.lando.yml` file that got things up and running:

```
name: sc-drupal8
recipe: drupal8
excludes:
  - vendor
config:
  webroot: docroot
  composer_version: '1.10.17'
tooling:
  drush:
    service: appserver
    cmd: php /app/vendor/drush/drush/drush --root=/app/docroot      
```

> Would have been solid I think on MacOS.

### MAMP

> I'll be honest, the first time I attempted to set this up I used XAAMP which didn't go well.

Mamp seemed to be pretty straight forward, it came with a little bit tighter installation and configuration with XAAMP and had a bit easier to read documentation.   

#### Installing MAMP

There wasn't really much issue here with installing MAMP, although to make life easier I chose to specify an exact version in the `README` in order to hopefully stop some headaches later on.

[Download MAMP](https://www.mamp.info/en/downloads/)

And run through the installation process, it was pretty straight forward.

#### Configuring MAMP

Configuring mamp was pretty easy as well:

- There was only one Apache
- There was only one Mysql
- We chose to work with `7.4.16` version of PHP

Some minor updates were required:

##### MYSQL

Open up the `C:\MAMP\conf\mysql\my.ini` file and make the following changes:

```
# Need to be able to restore the test/uat database locally
max_allowed_packet = 64M
```

##### PHP

There are two configuration files that need to be updated together, this seems like a specific to Windows thing that I haven't yet found a work around for.  It's not to bad to copy and paste a file, so there wasn't too much effort put in:

Both `C:\MAMP\conf\php\7.4.16\php.ini` and `C:\MAMP\bin\php\php7.4.16\php.ini` need to be modified.  The reasoning for this is that running `composer` and php from the command line uses the version configured in the Environment `Path` while the Apache version uses the one located in the `bin` folder.  Meh.

```

# Get debugging available
[xdebug]
zend_extension = "C:\MAMP\bin\php\php7.4.16\ext\php_xdebug.dll"
xdebug.mode = debug
xdebug.start_with_request=yes

max_execution_time = 600      ; Maximum execution time of each script, in seconds
max_input_time = 60	          ; Maximum amount of time each script may spend parsing request data
;max_input_nesting_level = 64 ; Maximum input variable nesting level
memory_limit = 1024M          ; Maximum amount of memory a script may consume (128MB)
```

> If you follow a lot of the tutorials online for getting XDebug working, you'll run into some issues in your log.  So a heads up to always check it

```
Xdebug: [Config] The setting 'xdebug.remote_enable' has been renamed, see the upgrading guide at https://xdebug.org/docs/upgrade_guide#changed-xdebug.remote_enable (See: https://xdebug.org/docs/errors#CFG-C-CHANGED)
```

##### Apache

Becuase of Drupals cookies and Chrome inability to handle `HttpOnly, Secure=None` cookies anymore, you won't be able to login using Chrome unless you make SSL/HTTPS available for your local environment.  On Lando this was done for you using the proxy, it was glorious, but now that we have to do it manually it's a bit of a process.  There are tons of tutorials around on how to generate local keys, so I'm just going to show the MAMP configuration:

Within `C:\MAMP\conf\apache\httpd.conf`

```
LoadModule ssl_module modules/mod_ssl.so
Include /MAMP/conf/apache/httpd-ssl.conf
```

And the new file `C:\MAMP\conf\apache\httpd-ssl.conf`

```
Listen 443

<VirtualHost *:443>
    ServerName localhost:443	
    SSLEngine on
    SSLCertificateFile "/MAMP/conf/ssl/server.crt"
    SSLCertificateKeyFile "/MAMP/conf/ssl/server.key"
</VirtualHost>
```

## Drupal 8 Configuration

Now that we've got the MAMP environment setup and ready to go, we need to continue down the Drupal 8 path.

### Compser Install

It looks like the vendor followed some standard of best practices (from looking around) so we need to run the `composer install` command to get dependencies.  While doing this I ran into one massive gotcha that blew my mind.  Essentially we were provided with a `composer.json` file that:

- Wasn't locked to specific versions
- Had some dependencies in `dev` version

Both of which had issues of their own and was something that coming from a Maven background just boggled my mind.  It resulted in different versions being on different installs for different developers and provide with a week or so long of attempting to find out why the dev dependencies were missing (which now looking back seems so obvious), but you can read about it here [https://drupal.stackexchange.com/questions/302666/trait-drupal-entity-form-entityduplicateformtrait-not-found-while-setting-up/302959#302959](https://drupal.stackexchange.com/questions/302666/trait-drupal-entity-form-entityduplicateformtrait-not-found-while-setting-up/302959#302959).

TLDR; manually delete all `dev` version dependencies before running `composer install` so that they can be checked out as git submodules correctly.

### Drupal Settings

Another fun thing provided in the project were the setting file(s).  We were given (in git) two files:

- `settings.php` 
- `settings.local.php` 

which required us to copy/overwrite `settings.php` with `settings.local.php` which then turned out to cause some problems with the deployment and development environments of my co-workers.  After looking around I found that the better practice was to:

- Have a `settings.php` file that read `settings.local.php` file when it exists (which had been completely removed)
- Have the `settings.local.php` file not be committed to git so that it can be managed using the `example.settings.local.php` file

Again so much confusion on why these weren't being followed already.  After looking at the original example file it was pretty obvious this was meant to be in there.  I'm assuming due to how Aquia (or whatever the companies called) deploys things it needed to be changed?

```
/**
 * Load local development override configuration, if available.
 *
 * Use settings.local.php to override variables on secondary (staging,
 * development, etc) installations of this site. Typically used to disable
 * caching, JavaScript/CSS compression, re-routing of outgoing emails, and
 * other things that should not happen on development and testing sites.
 *
 * Keep this code block at the end of this file to take full effect.
 */
if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
  include $app_root . '/' . $site_path . '/settings.local.php';
} 
```

## Up and At Them!!

After this you should be able to connect to:

- `http://localhost`
- `https://localhost`

and be able to move around.  

> I skipped the database/install section since it's pretty straight forward

## Debugging

Now that the site is up and running we need to be able to debug the application.  We're kind of split here on the IDE to use so I needed to get debugging working in both VSCode and Eclipse.

### Visual Studio Code

Create the default `launch.json` configuration

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003
        }        
    ]
}
```

and then run the debugger.  You should be able to place a breakpoint anywhere in the application (`index.php` although I learned that was a bad choice in Eclipse) and have the breakpoint hit.

### Eclipse 

Eclipse was a little more painful to setup, although not terrible.

1. Open `Properties > PHP > Debug > Debuggers` 

- Debug Port = `9003`

2. Open `Properties > PHP > Installed PHPs` 

- Add the appropriate PHP
- Select the appropriate Debugger (below)

3. Open `Properties > PHP > Servers`

- Edit the Default PHP Web Server
- Point `docroot` to the appropriate folder
- Ensure the Debugger `XDebug` is selected with the port `9003`

4. Create a Debug Configuration

- Open up `Debug Configurations`
- Select `PHP web Application` and click `New`
- On the Server tab select `Default PHP Web Server` and the `/sc-drupal8/docroot/index.php` file.
- On the Debugger tab confirm `XDebug` is selected

> Eclipse debugging will open a new Debugger thread for each request made (css, images, etc) if you're debugging the `index.php` file this blows things up pretty quickly.  There are a few configuration options with regards to XDebug that I haven't quite figured out how to tone down.

