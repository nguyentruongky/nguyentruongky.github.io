---
layout: post
title:  Different way to do multi languages in iOS
subtitle: Save your time and effort to do multi languages in iOS 
date:   2018-3-29
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fi18n%2Fmulti_lang.jpg?alt=media&token=946311d8-57d0-492d-9f56-faa3f8706c4c'
---

Google about multi languages in iOS, I find some step-by-step tuts to do multi languages. That's a good way, I did that 2 years ago. It shows me to use string files for multi languages. 

This note I want to take note about another way I found in my recent projects. I use plist files to save languages. Do it now. 

### Steps
- New project. Design UI like this, connect outlets

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fi18n%2Fi18n_demo.png?alt=media&token=0e0a0ce4-103c-48fe-b8ef-f9f6c292c8e4" width="350px"/>
- Add 2 plist files, named Lang_en.plist, Lang_vi.plist
- Add some codes for localization.

```

    class knI18n {
        static let shared = knI18n()
        
        private let knCurrentLanguageKey = "knCurrentLanguageKey"
        private let fileNameBase = "Lang_"
        lazy var localizableDictionary: NSDictionary! = self.getLanguageFile()
        func getLanguageFile() -> NSDictionary {
            let language = currentLanguage ?? "en"
            if let path = Bundle.main.path(forResource: fileNameBase + language, ofType: "plist") {
                return NSDictionary(contentsOfFile: path)!
            }
            fatalError("Localizable file NOT found")
        }
        
        func localize(string: String) -> String {
            guard let localizedString = localizableDictionary.value(forKeyPath: string) as? String else { return string }
            return localizedString
        }
        
        func setLanguage(_ language: String) {
            currentLanguage = language
            localizableDictionary = getLanguageFile()
        }
        
        private var currentLanguage: String? {
            get { return UserDefaults.standard.value(forKeyPath: knCurrentLanguageKey) as? String }
            set { UserDefaults.standard.setValue(newValue, forKeyPath: knCurrentLanguageKey) }
        }
    }

    extension String {
        var i18n: String {
            return knI18n.shared.localize(string: self)
        }
    }

```

Hope code clean enough to understand. The main idea is to load the plist file into a dictionary and get the value for key I want to change language. 

- Add some code into controller

```

    @objc func handleChangeToEn() {
        knI18n.shared.setLanguage("en")
        refreshUI()
    }

    @objc func handleChangeToVi() {
        knI18n.shared.setLanguage("vi")
        refreshUI()
    }

    func refreshUI() {
        languageLabel.text = "lang".i18n
        helloLabel.text = "hello".i18n
        vietnameseButton.setTitle("change_vi".i18n, for: .normal)
        englishButton.setTitle("change_en".i18n, for: .normal)
    }
    
```

- Run the app, and try. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fi18n%2Ffinish_demo_i18n.gif?alt=media&token=4eda2910-d8c5-4de8-b257-100042c030f6" width="320px"/>

### What is the advantages in this way? 

First, I can easily add new text, without worries about key duplication. XCode will tell key exists, and I never to face to error on key duplication. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fi18n%2Fkey_exist.png?alt=media&token=3bb069ff-ca96-4885-9b52-9146232a1f3b" width="350"/>

Second, no need to worry about `;` and the end of text. Once, I missed a `;` and faced to error, took me much time to fix. XCode was not kind enough to tell me this problem 2 years ago. 

Third, very easy to add new languages, just new plist files with name start with `Lang_`. 

Minor note: XCode can be very slow anytime, so, I like to shorten the name to `i18n` and `knI18n` to easier to type without suggestion. 

### But still have disadvantages in this way

I can't send these plist files to translator, difficult for them to open and do their work. These attached files can help. 

- `GenerateText` is used to export text from language file to text, then I can easily send to translators. 

- `GeneratePlist` is used to convert translated text file to plist. And can import directly to project. 

> How to use? 

Put `GenerateText` app in the same place with plist file, run Terminal at that folder, type `./GenerateText`. New file text is generated and ready to send to translators. 

Put translated file in the same folder with `GeneratePlist` app, type `./GeneratePlist` in Terminal to generate new plist file. 

My app can backup your plist file before generate new one. But should backup manual will be more secure. 

### Conclusion

Hope this solution can help iOS developer work with i18n easier. Download project at [https://github.com/nguyentruongky/i18n_Demo](https://github.com/nguyentruongky/i18n_Demo)