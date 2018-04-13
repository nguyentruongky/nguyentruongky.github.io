---
layout: post
title: i18n from Server
subtitle: How I support multi languages for Ogenii from server
date:   2018-4-13
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fi18n%2Fi18n-server.png?alt=media&token=5b3313f8-56b1-4ebe-9bb7-96b09cbc66ec'
---

I took a note about multi languages in iOs few days before. But multi languages can't finish from iOS/Android or Web only, but need help from server. Without support from server, users can face to messages in English while using Vietnamese, or read notifications in Vietnamese, while app language is in English. 

In [Ogenii](https://ogenii.com), I fully support i18n from client to server. It's not an easy tasks for me, because I'm just a newbie in Backend - Node JS, we're using Node JS. I'm sure this is not the best way to do but it's a good way I can find and do at this time. Main idea is separation all texts to languages objects and get the values by the keys. 

### Move all texts to language files 
I support English and Vietnamese in [Ogenii](https://ogenii.com), so I add 2 new files `en.js` and `vi.js` and move texts to an object `texts`

`en.js`
```
module.exports.texts = {
    err_401: "Your login is expired. Login again and keep learning.",
    verification_sent: 'Your verification will be processed in 2-4 hours',
    clazz_created: "Your class is created and under review. It'll be live soon."
}
```
`vi.js`
```
module.exports.texts = {
    err_401: "Phiên đăng nhập đã hết hạn. Đăng nhập lại và tiếp tục học thôi",
    verification_sent: 'Thông tin xác thực của bạn sẽ được xử lý xong trong 2 - 4 giờ tới',
    clazz_created: "Lớp của bạn đã được tạo thành công và đang được xem xét. Lớp sẽ sớm online thôi"
}
```

### Add a language center file 
I don't connect languages file directly to others, I make a file as a center to connect all supported language to other files. I named it `LangCenter.js`.

- Import all supported languages to `LangCenter`

```
let En = require('./en')
let Vi = require('./vi')
```

I add all language to an object like this. So don't care how many languages will support. 

```
let supported_langs = {
    "en": En, 
    "vi": Vi
}
```

- Add an object to contain all keys. The values of this object must be exactly **same** to keys of object `texts` in `en.js` and `vi.js`.

```
module.exports.keys = {
    err_401: "401",
    verification_sent: "verification_sent",
    clazz_created: "clazz_created"
}
```

- Add function to get a translated text from a key and requested language from client. By adding languages to object `supported_lang`, we don't need condition to get the requested language texts here, much easier. 

```
/**
 * @desc Get string by language
 * @param key of the text 
 * @param lang If param `lang` is a String, it's a language name. If it's an object, it is a request from client. param `lang` is in the request header. 
 * @return String in selected language
 */
function getText(key, lang) {
    if (typeof lang !== 'string') {
        lang = lang.headers['lang']
    }
    let dict = supported_langs[lang].texts
    return dict[key]
}
module.exports.getText = getText
```

Okay, it's done for all messages from server, success messages, error messages, warning messages, any message we need to send to client, just add to language files and `keys` objects in `LangCenter.js`

This is not fully supported. Notification still is 1 language only. I saved the notification content to database, and it can't be dynamic for multi languages. So I had to change. Just save elements to generate notification content only. Depend on elements, I can get text by language. 

### Generate Notification content by language 

One simple example to generate message.
```
/**
 * @desc Generated text to display to Master when Genius book his class
 * @desc Text structure: Genius `<genius_name>` has just booked your class `<clazz_title>`
 * @param data: `genius_name`, `clazz_title`
 * @param lang
 * @return Meaningful String getting approved
 */
function getClazzBookedToMasterText(data, lang) {
    let booked = Lang.getText(LangCenter.keys.clazz_booked_to_master, lang)
    return `Genius ${data.genius_name} ${booked} ${data.clazz_title}`
}
```

We should make smaller function to generate message like above, because we will need these functions several times. 

NodeJS is not as strict as Swift, so that, I put lot of comments in my code, especially for params. It helps me easier to understand to get back or fix bugs. 

Now, the code is 95% flexible for multi languages. Server now returns notification content by our requested language.

### Multi language in push notification 

Final step is push notification in multi language. Push notification is dynamic from server, when data meets some condition. Genius books a class and server pushes notification to Master, for instance. So that I save user language to database and get it when push notification is needed. When I have the language, I generate content by functions above and push to users. 

### Conclusion 

That's all I do with multi language in Ogenii backend. It's 98% flexible. There is one more edge case I have no solution at this time.  `User has multi devices, and in every device, he selects a different language.` Hope I find solution to solve this edge case soon. Any suggestion to solve it, please drop me a message on Skype. 

Hope this can help. 