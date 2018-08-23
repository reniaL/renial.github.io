---
layout: post
title:  "CORS! Boom!"
tags: [code]
---

CORS! Boom!


https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials
https://medium.com/@buddhiv/what-is-cors-or-cross-origin-resource-sharing-eccbfacaaa30
https://mortoray.com/2014/04/09/allowing-unlimited-access-with-cors/
https://www.haorooms.com/post/cors_requestheaders
https://segmentfault.com/a/1190000003710973

In addition, this flag is also used to indicate when cookies are to be ignored in the response. The default is false. XMLHttpRequest from a different domain cannot set cookie values for their own domain unless withCredentials is set to true before making the request. The third-party cookies obtained by setting withCredentials to true will still honor same-origin policy and hence can not be accessed by the requesting script through document.cookie or from response headers.