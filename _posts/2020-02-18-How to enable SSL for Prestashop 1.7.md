---
layout: post
title: "How to enble SSL for Prestashop 1.7"
date: 2020-02-18 11:10:10 -0000
categories: CMS
---
If you enable SSL for your Prestashop, maybe you get *too many redirect...* error.
This happens because you've not configure Prestashop for SSL connection.
To enable SSL in Prestashop, you must follow these steps:
* Access `Shop parameters-->General`

    ![Screen 1](https://imh01-inmotionhosting1.netdna-ssl.com/support/images/stories/prestashop/prestashop-17-14-settings-general.png)
* Select *Yes* under *Enable SSL* to force HTTPS checkout and account pages and *Enable SSL on all pages* to force HTTPS on all Prestashop pages

    ![Screen 2](https://imh01-inmotionhosting1.netdna-ssl.com/support/images/stories/prestashop/prestashop-17-13-enable-ssl.png)
* You may need to purge your Prestashop cache to apply these changes.