# Introduction

Suppose you have a multilingual Joomla 4 website with a different domain name for each language.
Here is a nice and easy solution to implement that.

Indeed in Joomla 3 there were a couple of plugins allowing to do that: 
- 'Language Domain' https://github.com/pe7er/plg_system_language2domain by Peter Martin (based on https://github.com/yireo-joomla/plg_system_languagedomains by Jisse Reitsma aka Yireo)
- 'GJ Multilanguages Domains' which is not available anymore on the Joomla Extensions Directory any more

But atm and afaik there is no solution for Joomla 4

There are 3 Steps
1. add a few lines in your .htaccess file (so obviously it only works with Apache servers. If your server runs Nginx, give a try to https://winginx.com/en/htaccess)
2. add a first Regular Labs ReReplacer rule to adapt the <head> section of your page
3. add a second Regular Labs ReReplacer rule to adapt the <body> section of your page

# Our example

Let's take a real life example, where 3 domain names of the same site are respectively:
- EN: healthybelgium.be
- FR: belgiqueenbonnesante.be
- NL: gezondbelgie.be

# Step 1

```
# REMINDER
# - RewriteCond applies only to the RewriteRule that immediately follows 
# - [NC] makes the test case-insensitive
# - [OR] combines rule conditions with a local OR instead of the implicit AND
# - 301 means that a new page has taken over permanently and 302 redirect means that the page has been moved temporarily
# - L stands for last

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

# Step 2

In Regular Labs ReReplacer, create a new rule, limit the Search Areas to the Head and replace
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

# Step 3

In Regular Labs ReReplacer, create a new rule, limit the Search Areas to the Body and replace
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
