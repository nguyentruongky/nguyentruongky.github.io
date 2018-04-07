---
layout: post
title: Configuration and User-Defined setting in iOS 
subtitle: Support different products in the same project
date:   2018-4-7
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fuser_setting.jpg?alt=media&token=4bc88186-0901-48b7-9d06-7e34fecef306'
---

I ran into a challange: support different countries with different app's names but same code. It means, when I release to Vietnam market, I have to change name to Red, release to Singapore, app's name is Green. Color theme, text color, font family are different by country. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fuser-defined-setting.gif?alt=media&token=9767c004-c1d6-4788-8ebc-0d9d0fc8c255" width="75%"/>

Cloning to different projects is not my solution. I choose this solution. Every market has its own setting. Do it now. 

### Add configurations
- Access to Project Setting/Info. Rename configuration `Debug` and `Release` to `Debug_Red` and `Release_Red`

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Frename_configuration.png?alt=media&token=abe69701-38e7-4824-9f70-53e28d98627f" width="75%"/>

- Duplicate these 2 configurations and change name to `Debug_Green` and `Release_Green`

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fduplicate_configuration.png?alt=media&token=38bcfbc7-1db0-441b-bb54-ecee23cee2f2" width="75%"/>

### Add Schemes 
- Select current scheme and `Edit Scheme...`

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fedit_scheme.png?alt=media&token=293dc583-d59a-4a8b-b1cb-8ddd627b6e72"/>

- `Duplicate Scheme`
- `Manage Schemes...`
- Select a scheme and press Return key to rename. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Frename_scheme.png?alt=media&token=e436ccc7-1416-4d22-aee4-7ee384d11388" width="75%"/>

- Double click to Scheme Red and check left hand side setting
- Make sure `Debug`, `Test`, `Analyze` is `Debug_Red` configuration and `Profile`, `Release` is `Release_Red`. To change configuration, select `Info` and change in `Build Configuration`.

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fchange_scheme_setting.png?alt=media&token=4ba4f76f-1328-4c7e-899e-86e0f85daa76" width="75%"/>

- Same to Scheme Green

### Add User-Defined Setting
Red and Green have different name, bundle ID, version, build number. I have to add some User-Defined Setting

- Select Project setting/Editor/Add Build Setting/Add User-Defined

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fadd_setting_menu.png?alt=media&token=349c2c95-0e7b-44fa-8c35-0c40fe3bf923" width="75%"/>

- Add some settings: `app_name`, `bundle_id`, `version`, `build_number`
- Add value to new settings
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fnew_setting.png?alt=media&token=eae822e1-389e-4de3-ac55-f531a1d11c22"  width="75%"/>

### Bind User-Defined Setting
- In Tab `Build Setting`, search `Product Bundle Identifier` and enter value `$(bundle_id)`

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fapply_setting.png?alt=media&token=37fd8c6b-d86d-4728-818e-b68a66d40d14" width="75%"/>

- Select Tab `General`
- Enter `$(app_name)` to `Display Name`, `$(version)` to `Version`, `$(build_number)` to `Build`

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Ffinish_apply_setting.png?alt=media&token=35d459c5-979c-42e7-a1df-21ba261fd80e" width="75%"/>

### Result 
- Add some UILabel to Storyboard Main and connect outlets 
- Add some code to show setting of the running app. 

```
appNameLabel.text = Bundle.main.object(forInfoDictionaryKey: "CFBundleDisplayName") as? String
versionLabel.text = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String
buildLabel.text = Bundle.main.infoDictionary?["CFBundleVersion"] as? String
bundleIdLabel.text = Bundle.main.bundleIdentifier
```

- It works

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fred_first.png?alt=media&token=3fa4e33e-3c19-49cf-8cdf-cfe920f98265" width="240px"/> <img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fgreen_first.png?alt=media&token=5293183b-716b-4a33-b4e4-147e90b51d4b" width="240px"/>

### Other setting 
We can apply this solution to Launchscreen, App Icon by adding 2 Launchscreens, 2 App Icons with same name, just different suffix. 

We can set `Launchscreen` same to `Display Name` but App Icon needs to be configured same to `Product Bundle Identifier`

Besides that, we need to change font family, theme color, text color depend on App. Here how to do. 

### Configure in code 

```
protocol Configuration {
    var themeColor: UIColor { get set }
    var textColor: UIColor { get set }
}

struct RedConfiguration: Configuration {
    var themeColor = UIColor.red
    var textColor = UIColor.white
}

struct GreenConfiguration: Configuration {
    var themeColor: UIColor = UIColor.green
    var textColor = UIColor.blue
}
```

In my main setting class

```
let config: Configuration = {
    if let app = Bundle.main.object(forInfoDictionaryKey: "CFBundleDisplayName") as? String {
        if app == "Red" {
            return RedConfiguration()
        }
        else {
            return GreenConfiguration()
        }
    }
    
    return RedConfiguration()
}()
```

### Final 
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fred_final.png?alt=media&token=5429f000-f157-419d-aad4-acc7bc3723a1" width="240px"/> <img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fuser-setting%2Fgreen_final.png?alt=media&token=616338fa-4bd4-4c2a-afaa-fb1aa41dd15a" width="240px"/>


My demo source is available on [Github](https://github.com/nguyentruongky/User-Defined-Setting-iOS)