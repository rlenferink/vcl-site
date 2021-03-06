---
title: Themes
---

{{% toc %}}

## Overview
Custom themes can be created in VCL for each affiliation. This allows a multi-site 
installation of VCL to have themes that match each affiliation's primary web site
theme. When a user logs in, the site will be displayed using the theme specified
for that user's affiliation. Additionally, when using [LDAP 
authentication](/docs/ldapauth.html), the login page for the affiliation will be 
displayed with the theme matching that affiliation.

## File system structure
The themes are in a **themes** directory in the web code. If you installed your web
frontend under a directory named **vcl**, you would look in vcl/themes. Each VCL 
release comes with a **default** theme, which you should see at vcl/themes/default. 
Typically, you would set up the following directory structure for a new theme:

* vcl/themes/mynewtheme
* vcl/themes/mynewtheme/page.php
* vcl/themes/mynewtheme/images
* vcl/themes/mynewtheme/js
* vcl/themes/mynewtheme/css

The page.php file and css directory are the only ones that are required. After creating 
the directory structure, you will need to go to the themes directory and run the
copydojocss.sh script, passing it the name of the directory containing your new theme as
an argument:

    cd vcl/themes
    ./copydojocss.sh mynewtheme

## VCL HTML generation
There are a few parts to how VCL generates the HTML for any given page. There are some
important functions of which to be aware. The first two must be in the page.php file:

* getHeader - generates all content up to and including the opening of a div element 
that will contain the main page content (typically what is to the right of the 
navigation menu); this div element must have an **id** of **content**; must have 
$refresh passed to it as an argument, which is then passed to getDojoHTML
* getFooter - closes the div element containing the main page content and generates any 
remaining content to close out the HTML for the page

These are functions that exist in utils.php that are available to be called from 
getHeader and getFooter:

* getDojoHTML - returns a string of Dojo related style and javascript; needs to be 
called in the &lt;head&gt; part of the page; needs to be passed $refresh as an argument
* getExtraCSS - returns an array of css files expected to exist in the vcl/css directory
that need to be included for the current value of $mode; a link tag needs to be created
for each of these files
* getSelectLanguagePulldown - returns the html for a form that has a select element for
changing the locale of the site
* getNavMenu - returns a string of list items that make up the navigation part of the
site; needs to be wrapped in &lt;ul&gt; tags; accepts 3 arguments:
   * $inclogout - 0/1 - should a **Logout** item be at the bottom of the navigation list
   * $inchome - 0/1 - should a **HOME** item be at the top of the navigation list
   * $homeurl - (optional) - if $inchome is 1, use this as the href of the HOME link;
if $inchome is 1 and $homeurl is not specified, HOME from conf.php is used

## Understanding page.php
As noted above, this file must contain getHeader and getFooter. **getHeader** must 
contain these elements, all of which can be seen in the default theme's page.php file:

* opening html and head tags
* stylesheet link to "css/vcl.css"
* script tag with src="js/code.js"
* script block setting the javascript variable *cookiedomain* to the php define 
*COOKIEDOMAIN* and setting usenls = 0
* a call to getDojoHTML($refresh) in the head section of the HTML
* a call to getExtraCSS() with a foreach to add a stylesheet link for each file in the 
array returned by getExtraCSS()
* closing head tag
* body tag with its class set to the name of the theme
* an unordered list wrapping the string returned by a call to getNavMenu() - this should
be inside an if conditional checking for $authed to be true
* must end with an opening div tag with an id of *content*

**getFooter** must contain these elements:

* closing div tag for the content div element
* closing body and html tags

## Path references
There are a few things to keep in mind with respect to path references. From a web 
browser's perspective, all HTML generated by getHeader and getFooter will be part of the
vcl/index.php file. So, any paths referenced in getHeader and getFooter need to be
relative to that - example: &lt;img src="themes/mynewtheme/images/someimage.png"&gt;. Any
files referenced in css files in the theme's css directory will be relative to that css 
file. So, if you were to include a reference to an image in the theme's image directory 
in a css file in the css directory, you would reference it relative to the css directory 
- example: url(../images/someimage.png)

## Example: bare minimal theme
This example shows the bare minimum that must be included. For a more detailed example,
look at the default theme include in each release. We'll call the theme "minimal". First,
we create a directory under the themes directory for it:

```bash
mkdir vcl/themes/minimal
```

Next, we create a css directory and run the copydojocss.sh script:

```bash
mkdir vcl/themes/minimal/css
cd vcl/themes
./copydojocss.sh minimal
```

Now, we create vcl/themes/minimal/page.php with this content:

```php
function getHeader($refresh) {
    global $user, $mode, $authed, $locale, $VCLversion;
    $rt  = "<!DOCTYPE html>\n";
    $rt .= "<html>\n";
    $rt .= "<head>\n";
    $rt .= "<title>VCL</title>\n";
    $rt .= "<link rel=stylesheet type=\"text/css\" href=\"css/vcl.css\">\n";
    $rt .= "<link rel=stylesheet type=\"text/css\" href=\"themes/minimal/css/minimal.css\">\n";
    $rt .= "<script src=\"js/code.js\" type=\"text/javascript\"></script>\n";
    $rt .= "<script type=\"text/javascript\">\n";
    $rt .= "var cookiedomain = '" . COOKIEDOMAIN . "';\n";
    $rt .= "</script>\n";
    $rt .= getDojoHTML($refresh);
    $extracss = getExtraCSS();
    foreach($extracss as $file)
        $rt .= "<link rel=stylesheet type=\"text/css\" href=\"css/$file\">\n";
    $rt .= "</head>\n\n";
    $rt .= "<body class=minimal>\n\n";
    $rt .= "<div id=menulist>\n";
    if($authed) {
        $rt .= "<ul>\n";
        $rt .= getNavMenu(1, 1);
        $rt .= "</ul>\n";
    }
    $rt .= "</div>\n";
    $rt .= "<div id=content>\n";
    return $rt;
}

function getFooter() {
    $year = date("Y");
    $rt  = "</div>\n";
    $rt .= "</body>\n";
    $rt .= "</html>\n";
    return $rt;
}
```

Finally, we create a basic css file at vcl/themes/minimal/css/minimal.css:

```css
#menulist {
    float: left;
}

#content {
    margin-left: 15em;
}
```

## Using a new theme
Themes are assigned on a per-affiliation basis in the affiliation table. This must
be done directly in the database by setting the **affiliation.theme** field for an 
affiliation to the name of a theme in the themes directory. For example, to use our new 
*minimal* theme, you would set **affiliation.theme** to **minimal** for the desired
affiliation.

## Tips and Tricks
The VCL web site has some administrative pages that can generate very large tables. If
you want them to be completely wrapped in your theme, you will probably need to use 
tables as part of you main layout instead of just using div elements and css. (Hopefully,
all of those large tables will be converted to dojo grids like the Manage Groups page by
the 2.4 release.)

Mobile themes - If you would like to use a modified theme for mobile devices, you can add
a section of code to the top of getHeader and getFooter that detects mobile devices
based on $_SERVER['HTTP_USER_AGENT']. If a mobile browser is detected, you can then
call a different function that generates mobile specific HTML and returns it to the
calling function.  Here is an example that could be used in getHeader:

```php
    if(array_key_exists('HTTP_USER_AGENT', $_SERVER) &&
         (eregi('iphone', $_SERVER['HTTP_USER_AGENT']) ||
         eregi('android', $_SERVER['HTTP_USER_AGENT'])))
       return mobileHeader($refresh);
```

mobileHeader would still need to meet all of the requirements of getHeader, but it
would just generate mobile specific HTML. A similar block of code would then be 
added to getFooter, calling a function named something like mobileFooter.
