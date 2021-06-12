---
title: "Always open Microsoft docs in en-us instead of local language"
description: "How to use the ModHeader browser extension to always open Microsoft docs in EN-US instead of local language."
summary: "Are you sent to the docs.microsoft.com page in your local language when opening a link on Google? In this blog post I will show you how to always use the us-en version."
date: 2021-06-12
draft: false
author: richard
thumbnail: /posts/Microsoft-Logo-icon-png-Transparent-Background.png
featureImage: /posts/Microsoft-Logo-icon-png-Transparent-Background.png

tags: [
    "azure",
    "documentation"
]
---

# The issue

When you open a page on [docs.microsoft.com](http://docs.microsoft) through a Google search, and you are outside of the United States, you often get the localized version of the page. In my case, I am being served with the Dutch nl-nl version of the website. 

Yes, there is a switch on the docs page to view the page in English. But I just want to open the en-us version of [docs.microsoft.com](http://docs.microsoft.com).

# The solution

I use a browser extenstion called [ModHeader](https://bewisse.com/modheader/). It allows me to configure a redirect rule. Everytime I hit the URL [https://docs.microsoft.com/nl-nl/](https://docs.microsoft.com/nl-nl/)* it will automatically redirect me to [https://docs.microsoft.com/en-us/](https://docs.microsoft.com/nl-nl/)*

ModHeader is available for Google Chrome and most other Chromium based browsers, like Brave and Microsoft Edge[.](http://edge.In) As long as the browser can install extensions from the [Google web store](https://chrome.google.com/webstore/category/extensions).

## Setting up the extension

1. In your browser, go to the [Google web store](https://chrome.google.com/webstore/category/extensions).
2. Search for ModHeader and open it. 
3. Click "Add to Chrome" to install the extension. In case you're using a different browser, the button might be called "Add to Edge" or something similar.
4. Click on the ModHeader icon in the extensions section of the browser.
    {{< figure src="modheader-icon.png" >}}
    Note, in Google Chrome the extension icon may be hidden at first. You can open it through the extension menu. 
5. In ModeHeader, click the + symbol, and choose **URL redirect**.
    {{< figure src="modheader-ui1.jpg" >}}
6. A new redirection rule is being presented. Click the two arrows.
    {{< figure src="modheader-ui2.jpg" >}}
7. Fill in your localized Microsoft docs URL in the **Original URLs** field. Fill in the en-us URL in the **Redirect URLs** field. Press Done to complete.
    {{< figure src="modheader-redirect.jpg" >}}

# Conclusion

With this simple trick your browser will always redirect to the en-us version of the Microsoft Docs website. 

If you want, you can also do it the other way around, where you are redirected to your local language instead of the English version.