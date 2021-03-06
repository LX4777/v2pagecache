# pagecache

v2pagecache - Fast, free, open source page cache for opencart v2.X
 

## VERSION

Version 1.3

##IMPORTANT NOTE

This is beta quality software, with VERY LITTLE support.  Opencart has made numerous changes to things that are important to a pagecache.  For example, $registry and $response used to be global...that changed in OpenCart 2.3.  Several other changes to areas like session, cart, etc, also happened.  We have done testing, but not extensively.  You are "ON YOUR OWN" to test this module.

Also, if you're running 3rd party modules or templates that could cause a single url to output different content, the pagecache will likely not work well. We cannot, an will not help, with issues encountered with 3rd party modules.  However, if you find and fix an issue, please feel free to submit it as a pull request to http://github.com/budgetneon/v2pagecache.

In short, we are very unlikely to be to able to help with issues. We are not a software development shop.  We made this module for our own use, and released it open source so others could use it as well.  We don't have the resources, however, to offer support.

## DESCRIPTION

This page level cache for opencart version 2.x is a port of our very similar page cache for opencart 1.5.x, which you can find [on github](https://github.com/budgetneon/pagecache). Please see [octurbo.com](http://octurbo.com) for the full story behind both of these opencart modules.

This page cache is  simple, but it's very fast and also includes an administration panel. Installation is straightforward, using the [ocmod installer](https://github.com/opencart/opencart/wiki/Modification-System) that is built into opencart since version 2.X.

It has been tested so far only in an apache/mod_php environment, but should also work under fastcgi, provided you are using PHP 5.4 or greater. 

This is a new piece of software, and unlike our page cache for earlier versions of opencart, we aren't running this in production.  Thus, we recommend you test it thoroughly before deploying to production.  See the CAVEATS section at the bottom for more detail. 

## Requirements

- Opencart 2.X
- PHP 5.4 or greater

## Installation

First, you need to install a separate opencart mod that fixes problems with the stock opencart installer:

- Install this ["Quick Fix"](http://www.opencart.com/index.php?route=extension/extension/info&extension_id=18892) module that fixes the opencart extension loader so that it doesn't require you to enable ftp  
- Go to Extensions -> Modifications, and refresh the modification cache:

![Refreshing the modification cache](https://i.imgur.com/8s7gy23.png)

Now, you can install the page cache:
- Download the v2pagecache-1.0.1.ocmod.zip file
- Log into your opencart admin panel
- Go to Extensions -> Extension Installer, and upload this extension
- Now, go to Extensions -> Modules, and click the green button install the module, then the blue button to enable and configure it.

![Install the Module](https://i.imgur.com/2kMrkyz.png)

- Now that you're on the module's config page, enable the page cache.

![Enabling the Page Cache](https://i.imgur.com/0v74TNv.png)

## Manual Installation
This extension has to make changes to your main index.php before it will work.  There is an "Enable Cache" button in the extension's admin page that can make those changes.  However, in some environments, this button may not be able to do that, due to file permissions, a customized installation of opencart, etc.  If that's the case, you'll have to make the changes to the index.php manually.

NOTE: THESE MANUAL INSTALLATION INSTRUCTIONS ARE ONLY FOR PEOPLE HAVING ISSUES WITH THE "ENABLE CACHE" BUTTON WITHIN THE PAGECACHE ADMIN PANEL.

There's two changes that need to be made.

1. At the top of your main index.php file, find this section of code:

```
    if (is_file('config.php')) {
        require_once('config.php');
    }  
```

Then, add these 6 lines, exactly as shown below, just after the section of code above:

    require_once(DIR_SYSTEM . 'library/v2pagecache.php');         //V2PAGECACHE
    $pagecache = new V2PageCache();                               //V2PAGECACHE
    if ($pagecache->ServeFromCache()) {                           //V2PAGECACHE
        // exit here if we served this page from the cache        //V2PAGECACHE
        return;                                                   //V2PAGECACHE
    }                                                             //V2PAGECACHE

2. At the very bottom of your main index.php, find this section of code:


For opencart 2.0.x and 2.1.x:

```
    // Output
    $response->output();
```

For opencart 2.2.x:

```
    // Application
    require_once(DIR_SYSTEM . 'framework.php');
```

For opencart 2.3.x
```
    // Startup
    require_once(DIR_SYSTEM . 'startup.php');
    start('catalog');
```


Then, add these 3 lines of code, just after the section of code above:
For opencart 2.0.x through opencart 2.2.x:

    if ($pagecache->OkToCache()) {                                //V2PAGECACHE
        $pagecache->CachePage($response);                         //V2PAGECACHE
    }                                                             //V2PAGECACHE

For opencart 2.3.x or greater:

    if ($pagecache->OkToCache()) {                                //V2PAGECACHE
        $pagecache->CachePage();                                  //V2PAGECACHE
    }                                                             //V2PAGECACHE


## OVERVIEW
Very early in opencart's main index.php file, some lines of code are added when you enable the page cache.  This section of code looks to see if there's a previously cached file for the url that's currently being requested.  If there is, and it's not expired, the request is served from the cache.  Because this is very early in the index.php file, and because it exits if served from the cache (via the return() call), it skips over almost all of the processing that opencart normally does.  No database calls, etc..so it's very fast.

Later, towards the end of the index.php, a few more lines of code are also added.  This is very near the end of the file, after whatever page was requested has already been generated by opencart, and served to the end user.  At this point, the code is checking if we should take the page that opencart just generated and save it to the cache.  This adds a small amount of overhead because it writes out the content twice (once to the browser, and once to the cache file).  That is, however, the only notable overhead added, and it's very small.
## ADMIN PANEL

This extension includes an easy to use admin panel that allows you to:

- Enable and disable the cache. (adds and removes code from your index.php file)
- Purge cached files (either all of them, or just the expired ones)
- View statistics on the number of cache files, as well as consumed disk space, for both valid and expired items
- View, but not change, the settings (you have to manually edit system/library/v2pagecache.php to change settings).

Here's a screenshot of the admin panel:

![V2Pagecache Admin Panel Screenshot](https://i.imgur.com/xJknPBN.jpg)

## SETTINGS

In order to keep this page cache lightweight, we do not store settings in the opencart database like a normal opencart extension would.  This allows a cached page to be served with almost no opencart code running, and no database calls at all.  Therefore, to make changes to settings, you have to hand edit the v2pagecache.php file.  Here's the settings you can safely change:

     
    // These are all near the top of the v2pagecache.php file  
    private $expire='14400'  ; // expire time, in seconds 14400 = 4 hours
    private $lang='en'       ; // default language for site
    private $currency='USD'  ; // default currency for site

    private $addcomment=true ; // set to true to add a comment to the bottom
                                // of cached html pages with info+expire time
                                // only works where headers_list() works

    private $wrapcomment=true; // if this is set to true (and $addcoment is
                               // also set to true), we will use a comment
                               // that most html minifiers won't remove, like:
                               // <!--[if IE]><!--comment--><![endif]-->


    private $end_flush=false ; // set to true to do an ob_end_flush() before
                               // serving a cached page. Slightly faster
                               // "first byte received" times, but it creates
                               // issues in some environments, hence, it's off
                               // by default

    // If using a 3rd party theme that serves different html to
    // mobile/tablet/desktop, you can enable "cachebydevice"
    // For the standard opencart theme, use false
    // For journal theme, use 'mobiledetect';
    // For Opencart Mobile Framework, use 'categorizr'
    // For Omtex Mobile Framework, use 'categorizr'
    //
    // make sure only one of the three lines below is uncommented
    private $cachebydevice=false;
    //private $cachebydevice='mobiledetect';
    //private $cachebydevice='categorizr';


    private $skip_urls= array(
        '#checkout/#',
        '#product/compare#',
        '#/captcha#',
        '#account/#',
        '#register/#'
    );


A new notes on these settings:

- $expire : Because it's in the declaration section of the class, your options to set this are limited...you can't, for example, do $expire=24*24*60.  Just figure out the number of seconds you want cached pages to exist before they expire. We used 14400, which is 4 hours.
- $lang and $currency : We cache pages for different languages and currencies in different files.  These two settings control what the default language and/or currency is if we get an http request that does not have the session variable(s) for each respective setting already set.
- $addcomment : Set to true if you want the page cache to append an html comment at the end of the stored cache file that notes the url cached and it's expiry time.  This can be helpful since you can see it with live pages via your browser's "view source" functionality.  Set to false if you don't want this.  We only add the comment if headers_list() is available and indicates that the cached resource is an html page.  This keeps us from adding an html comment to a cached JSON response, for example. 
- $wrapcomment : If true, wraps the comment (see above) with <!-[if IE]><![endif]--> so that it doesn't get stripped out by an html minifier (like Cloudflare's, for example)
- $end_flush : Set to true to run ob_end_flush() before serving a cached page. This gives you a slightly faster "first byte to browser" time, but creates issues (errors, blank pages) in some environments, so we've defaulted it to false.
- $skip_urls : This is an array of PCRE patterns for urls that you do not want to be cached.  The default settings prevent caching for checkout pages, product comparisons, captchas, and the JSON country list.  Note that this is not the only check done to decide when a page shoudn't be cached.  You shouldn't have to, for example, mark the 'account/' pages here, because we already disable caching when a user is logged in.
- $cachebydevice: Set to false if you have a truly responsive theme that serves the exact same html to all devices.  If you, however, have a theme that serves different html to mobile, tablets, and desktops, you will have to select either mobiledetect or categorizr. It is best to select the one that your theme actually uses.  For example, the Opencart Mobile Framework uses categorizr.


Also, the cached pages are kept in a directory named 'v2pagecache', under the existing directory opencart uses for it's more general cache.  In most installations, this would be /your_opencart_root/system/cache/v2pagecache.  I

## LOCALES AND CHARSETS

This module does not set either the charset encoding or the locale.  This can be a source of issues, as many people are depending solely on their third party opencart themes to set the correct charset and encoding, rather than setting it globally, at either the php or webserver (apache, for example) level.

If you're having trouble with odd characters when cached pages are served, you're probably seeing this issue.  To fix it, you'll need to correctly set up your php environment, and t
hen delete any existing cached pages.

There are many ways to set up the default locale and charset.  You could set that up within apache (or nginx, etc), or you could set it within your php.ini file.  One method that wo
rks with this module is to set it up within opencart's config.php file.  Since the opencart config.php file is read very early within opencart's index.php file, our cache module wil
l inherit anything set there.  Here's an example of what to add to the opencart config.php file:

    # put this at the bottom of the opencart config.php file
    # example to set locale and encoding for Turkish
    ini_set('intl.default_locale','tr_TR');
    ini_set('default_charset','windows-1254');


## DEMO

We created a demo site at [octurbo.com](http://octurbo.com) for our original pagecache that's used for opencart 1.5.X.  The home page has links to both a stock/vanilla installation of opencart 1.5.x, and one with our page cache.  Note that you may have to load a page from the pagecache enabled site twice...once to prime the cache, and a second time to see the performance improvement.

A page cache makes a much bigger difference on an opencart site that has a lot or products, categories, and other functionality.  So, for the demo site, we uploaded a large number of movies from [an Amazon AWS Cloudsearch dataset](https://aws.amazon.com/developertools/9131774809784850). You can also try our production site at [budgetneon.com](http://budgetneon.com/).  It has lots of products and nested categories.  If you view the source of a cached page, you'll see the html comment at the bottom.

## CAVEATS

- This extension has been tested, but not in a rigorous way.  Please test it thorougly before deploying on a production server.

- Using the "Enable Cache" and "Disable Cache" buttons in the admin panel makes live changes to your main index.php file.  Make sure you have a backup of the main index.php file in case anything goes wrong. 

- The "Output Compression Level" within opencart (System->Settings->Edit->Server->Output Compression Level) is disabled when a cached page is served.  If you want cached pages to be compressed, disable output compression within opencart, and use apache's mod_deflate instead. It's a better solution anyway.
 
- The page cache does not check the sanity of url parameters. So, it will happily cache '/index.php?foo=1', '/index.php?foo=2', and so on...all as separate urls and cache file.  This could result in a very large number of cached pages, and in extreme circumstances (like a robot crawling invalid pages), it could potentially fill up your hard drive.

- We did try to put in sufficient logic such that dynamically created pages that should not be cached...aren't.  However, it's possible we missed some.  Also, if you've added extensions to opencart that have url's that shouldn't be cached, that will likely be missed.  See the "$skip_urls" setting to remedy that. 

- A page cache can be a terrible crutch, that effectively hides important performance problems.  For example, if your home page is really slow, the first person to request the page when it's not cached (or when the cache expires) will see that horrible performance.  Subsequent visits will get the benefit of the page cache, which is nice...but not if that's hiding the true issue from the site owner.  In short, it's not a substitute for proper performance tuning.

## TROUBLESHOOTING

If you are using a theme other than the opencart default, and the page looks "messed up", here are some things to try:

- Change the addcomment setting to false.  Some themes serve up json, css, or javascript without changing the content header from the default of text/html.  I
f the addcomment setting is true, it will add an html comment to that file, which only works for html...it will break json and javascript.
- Look into the cachebydevice setting.  If your theme serves different html to mobile devices than it does to desktops, you will need to specifically set the 
cachebydevice setting.
- Read the section on this page regarding charsets and locales.
- If you have some technical ability, look at the cached files that have been created in the cachefolder.  Is there something there that should not be cached?
 Add it to the skip_urls setting.

Also, be sure to clear the cache after changing settings.

## AUTHOR

Kerry Schwab, `<sales at budgetneon.com>`

We run a busy ecommerce site, here: [BudgetNeon.com](http://budgetneon.com/).  Our website uses the opencart platform, and this code originated from a need we had. We sell [open neon signs](http://budgetneon.com/open-signs), neon for [restaurants](http://budgetneon.com/restaurant) and [businesses](http://budgetneon.com/business), as also [custom neon signs](http://budgetneon.com/custom).  Our current plan is to release more of our local opencart additions, especially those that improve performance.  Lazy image loading for product thumbnails is the most likely one we'll release next.

## LICENSE AND COPYRIGHT

Copyright (c) 2014 Kerry Schwab & BudgetNeon.com, All rights reserved.

This program is free software; you can redistribute it and/or modify it under the terms of the the FreeBSD License . You may obtain a copy of the full license at:

[http://www.freebsd.org/copyright/freebsd-license.html](http://www.freebsd.org/copyright/freebsd-license.html)


Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

- Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
- Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
- Neither the name of the organization nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.

IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;

LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
