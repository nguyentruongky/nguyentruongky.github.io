---
layout: post
title:  "Contact Picker"
subtitle: "Pick phone number from ABAddressBook"
date:   2016-1-19
background: '/img/posts/contact_list.jpg'
---
I have a feature in my app: pick a phone number. I code this feature in Objective C and now I want to parse to Swift. The feature requires to load all phone numbers from all contacts, search contacts, contacts are grouped into first character of the name. Here what I did

![](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ffullcontact.png?alt=media&token=0225e273-c25a-46f1-a053-2a29ce26aed2) ![](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fsearch_contact.png?alt=media&token=63aa4ae0-4149-4ecf-a649-059940d17210)

I use intention in this function so that this picker can be use anywhere easily. Read more about intention at my posts: [1000+ lines of code, good work?“ (Cont)](https://truongky.wordpress.com/2016/01/19/1000-lines-of-code-good-work-cont/) and [PageViewController: How can I reuse it?](https://truongky.wordpress.com/2016/01/12/pageviewcontroller-how-can-i-reuse-it/)

Let's begin.

-   Create new project.
-   Design what you want. Two controls must have: a UITableView and a UISearchBar.
-   Add a UITableViewCell and name it `contactCell` with style is `Subtitle`.
-   Add a new file `ContactPickerIntention.swift` and implement NSObject class. It's our intention.
-   Add a NSObject in IB and change its subclass is `ContactPickerIntention.swift`.
-   Connect controls to the intention. I named them `searchBar` and `tableView`.
-   Connect the intention to ViewController.
-   The intention has to conform some delegate and datasource.
    -   UISearchBarDelegate
    -   UITableViewDelegate
    -   UITableViewDataSource

You have to ask permission to access to contact.

```
ABAddressBookRequestAccessWithCompletion(addressBook) { granted, error in
	// warn the user that because they just denied permission, this functionality won't work
	// also let them know that they have to fix this in settings
	if granted == false { return }
	// load contact 
}
```

Load all contacts

```
let allContacts = ABAddressBookCopyArrayOfAllPeople(addressBook).takeRetainedValue() as Array
```

All contacts are here. Run a loop and load every contact, find the phone number and consider this contact with every number is a contact item.

```
private func getNameFromContact(currentContact: ABRecordRef) -> String {
	let firstName = ABRecordCopyValue(currentContact, kABPersonFirstNameProperty)
	let lastName = ABRecordCopyValue(currentContact, kABPersonLastNameProperty)
	var currentName = ""
	if firstName == nil && lastName == nil { // prevent anonymous contact
            currentName = ""
	} else {
		currentName = ABRecordCopyCompositeName(currentContact).takeRetainedValue() as String
	}
	return currentName
}
```

The `ABRecordCopyCompositeName` function can get the contact's full name for you, but if there is a contact without first name or last name, your app crashes. Take care some anonymous friends with this func.

Get the phone numbers.

```
private func getPhoneNumbersFromContact(currentContact: ABRecordRef) -> [String]? {
    var phoneNumberList = [String]()
    let phones:ABMultiValueRef = ABRecordCopyValue(currentContact, kABPersonPhoneProperty).takeRetainedValue()
    for var j: CFIndex = 0; j < ABMultiValueGetCount(phones); j++ {
        let mobileLabel = ABMultiValueCopyLabelAtIndex(phones, j).takeRetainedValue()
        if mobileLabel == kABPersonPhoneMobileLabel ||
            mobileLabel == kABHomeLabel ||
            mobileLabel == kABPersonPhoneMainLabel ||
            mobileLabel == kABPersonPhoneIPhoneLabel ||
            mobileLabel == kABOtherLabel ||
            mobileLabel == kABWorkLabel {
                let phone = ABMultiValueCopyValueAtIndex(phones, j).takeRetainedValue() as! String
                phoneNumberList.append(phone)
        }
    }
    return phoneNumberList
}
```

There are a few phone number types such as Mobile, Home, Work. And those types are marked with the Label. We have to get the phones multi value and find the phone number conform to the labels.

I created struct `Contact` to keep the name and the phone number. And a dictionary with key is the first character of the name and the value is an array of Contact to store all contacts.

If you want to get email, try this func

```
func getEmail(currentContact: ABRecordRef) -> String {
    let email:ABMultiValueRef = ABRecordCopyValue(currentContact, kABPersonEmailProperty).takeRetainedValue()
    var emailString = ""
    if ABMultiValueGetCount(email) > 0 {
        emailString = ABMultiValueCopyValueAtIndex(email, 0).takeRetainedValue() as! String
    }
    return emailString
}
```

You can download my complete project at [my github](https://github.com/nguyentruongky/ContactPicker)
You can use this intention anywhere you want with simple actions, copy the file and use it in ViewController.
ViewController

```
contactPickerIntention.getAllContacts()
```

### Conclusion
This project just demonstrate how I did my feature. And you can do a lot with this, add delegate to pass data from picker to ViewController, show alert and open setting when access denied and user want to accept it.
Intention is extremely easy and powerful. Understand it clearly, your code grows to a higher level.
