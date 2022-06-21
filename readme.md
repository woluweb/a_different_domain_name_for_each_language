# How to set up a multilingual Joomla 4 website with a different domain name for each language

## Introduction

Suppose you have a multilingual Joomla 4 website with a different domain name for each language.
Here is a nice and easy solution to implement that.

But atm and afaik there is no solution for Joomla 4

In practice there are only 3 little Steps
1. add a few lines in your .htaccess file (so obviously it only works with Apache servers. If your server runs Nginx, [give a try to this tool to convert the rules](https://winginx.com/en/htaccess))
2. add a first Regular Labs ReReplacer rule to adapt the <head> section of your page
3. add a second Regular Labs ReReplacer rule to adapt the <body> section of your page
   
Steps 2 and 3 make sure that all links *within Joomla* are correct (note that they can be combined if you wish [by using some regex](https://github.com/woluweb/a_different_domain_name_for_each_language/issues/1)). But this does not prevent a human being or a robot to reach manually a "wrong" combination of domain name and language tag.

In that case, Joomla would simply display the page keeping the "wrong" domain name... but obviously we want to *force the right domain name* (ao to avoid hurting your SEO for different reasons like Duplicate Content or [Redirections](http://www.thesempost.com/google-dont-link-hreflang-redirecting-urls/)). That is what Step 1 does.
  
### Our example

Let's take a real life example, where 3 domain names of the same site are respectively:
- EN: healthybelgium.be
- FR: belgiqueenbonnesante.be
- NL: gezondbelgie.be

In our `System - Language Filter` plugin
- the Option `Remove URL Language Code` for the default language is set to No. With other words, what we want to get as url for the Homepage in each language is the following:
   - EN: healthybelgium.be/en
   - FR: belgiqueenbonnesante.be/fr
   - NL: gezondbelgie.be/nl
- the Option `Alternate Meta Tags` is set to Yes, which is better for SEO. If you set it to No, obviously you don't need Step 2

### Alternative solutions for Joomla 3

In Joomla 3 there were a couple of plugins allowing to do that: 
- 'Language2Domain' https://github.com/pe7er/plg_system_language2domain by Peter Martin
   - it is based on 'Language Domains' https://github.com/yireo-joomla/plg_system_languagedomains by Jisse Reitsma aka Yireo
   - IIRC it takes care of everything, namely the equivalent of the 3 above-mentioned steps
   - by default it removes the language tag after the domain name
   - a Joomla 4 version should be released one day but not in the short term
- 'GJ Multilanguages Domains'
   - it is not available anymore on the Joomla Extensions Directory
   - IIRC it was only taking care of step 1 (so if you use it on some site you should definitely reproduce the steps 2 & 3 described here if you don't want to hurt your SEO)
   - it does not remove the language tag after the domain name

## The 3 steps in detail

### Step 1 - forcing the correct domain name in .htaccess

```
# REMINDER
# - RewriteCond applies only to the RewriteRule that immediately follows 
# - [NC] makes the test case-insensitive
# - [OR] combines rule conditions with a local OR instead of the implicit AND
# - 301 means that a new page has taken over permanently and 302 redirect means that the page has been moved temporarily.
#   If this is intended to be permanent then change the 302 (temporary) redirect to 301 (permanent) only once you have confirmed that everything works as intended, so as to avoid potential caching issues.
# - L stands for last
# - in Regex ^blabla(.*)$ means that what you search
#   - is between ^ and $
#   - and is therefore blabla(.*) where (.*) means literally *any chain of characters*
#   With other words, ^blabla(.*)$ simply means that you search for *something starting with blabla followed by anything else*
   
<IfModule mod_rewrite.c>

RewriteEngine On

# FORCE HTTPS
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# FORCE WWW
RewriteCond %{HTTP_HOST} !^www\.
RewriteRule ^ https://www.%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# FOR MULTILINGUAL WEBSITES HAVING A DIFFERENT DOMAIN NAME FOR EACH LANGUAGE
# The Conditions and Rules hereafter allow to have a different domain name for each language of a Joomla website (where SEF is enabled obviously)
# It corrects the domain name if it does not match the language tag
# Note: on top of that, we also need to use Regular Labs ReReplacer (Free or Pro) within the website to put the correct domain name
# - in the <body> because all links are Relative
#   - for all the links in the Language Switcher. Example: for each language
#     - change <a href="/en/
#     - into   <a href="https://www.healthybelgium.be/en/
#   - and elsewhere in the content (there could be direct links to Articles or Menu items from another language)
# - in the <head> because all the links are Absolute
#   - for all the links to the Associated pages like <link href="https://www.healthybelgium.be/en/" rel="alternate" hreflang="en-GB" />. Example: for each language
#     - change https://www.belgiqueenbonnesante.be/en/
#     - into   https://www.healthybelgium.be/en/

# switch domain name to English if language tag is en
RewriteCond %{HTTP_HOST} ^(.*)belgiqueenbonnesante.be$ [OR,NC]
RewriteCond %{HTTP_HOST} ^(.*)gezondbelgie.be$ [NC]
RewriteRule ^en(.*)$ https://www.healthybelgium.be/en$1 [R=301,L]

# switch domain name to French if language tag is fr
RewriteCond %{HTTP_HOST} ^(.*)healthybelgium.be$ [OR,NC]
RewriteCond %{HTTP_HOST} ^(.*)gezondbelgie.be$ [NC]
RewriteRule ^fr(.*)$ https://www.belgiqueenbonnesante.be/fr$1 [R=301,L]

# switch domain name to Dutch if language tag is nl
RewriteCond %{HTTP_HOST} ^(.*)healthybelgium.be$ [OR,NC]
RewriteCond %{HTTP_HOST} ^(.*)belgiqueenbonnesante.be$ [NC]
RewriteRule ^nl(.*)$ https://www.gezondbelgie.be/nl$1 [R=301,L]

</IfModule>
```

### Step 2 - fixing the domain name in the `<head>`

In [Regular Labs ReReplacer](https://regularlabs.com/rereplacer), create a new rule, limit the Search Areas to the Head and replace
```
https://www.belgiqueenbonnesante.be/nl/,
https://www.healthybelgium.be/nl/,
https://www.belgiqueenbonnesante.be/en/,
https://www.gezondbelgie.be/en/,
https://www.healthybelgium.be/fr/,
https://www.gezondbelgie.be/fr/
```
by
```
https://www.gezondbelgie.be/nl/,
https://www.gezondbelgie.be/nl/,
https://www.healthybelgium.be/en/,
https://www.healthybelgium.be/en/,
https://www.belgiqueenbonnesante.be/fr/,
https://www.belgiqueenbonnesante.be/fr/
```

Explanation: we have to put the correct domain name in the `<head>` because all the links are Absolute
- for all the links to the Associated pages like `<link href="https://www.healthybelgium.be/en/" rel="alternate" hreflang="en-GB" />`. Example: for each language
   - change `https://www.belgiqueenbonnesante.be/en/`
   - into   `https://www.healthybelgium.be/en/`

### Step 3 - fixing the domain name in the `<body>`

In [Regular Labs ReReplacer](https://regularlabs.com/rereplacer), create a new rule, limit the Search Areas to the Body and replace
```
<a href="/nl/,
<a href="/fr/,
<a href="/en/
```
by
```
<a href="https://www.gezondbelgie.be/nl/,
<a href="https://www.belgiqueenbonnesante.be/fr/,
<a href="https://www.healthybelgium.be/en/
```

Explanation: we have to put the correct domain name in the `<body>` because all links are Relative
- for all the links in the Language Switcher. Example: for each language
  - change `<a href="/en/`
  - into   `<a href="https://www.healthybelgium.be/en/`
- and elsewhere in the content (there could be direct links to Articles or Menu items from another language)
