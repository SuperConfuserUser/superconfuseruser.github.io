---
layout: post
title:      "Validations for MVC Sinatra Project"
date:       2018-04-05 12:42:31 -0400
permalink:  validations_for_mvc_sinatra_project
---

# Overview
We will be discussing data validation Sinatra applications at these levels:
1. View 
2. Controller
3. Model

The technologies are:
1. HTML5
2. Sinatra
3. ActiveRecord

# Introduction
For the second Flatiron portfolio project, we must build  an MVC (Model View Controller) application with Sinatra and ActiveRecord. Users should be able to CRUD (Create Read Update Delete) their own content. User input needs to be validated so that bad data isn't added to the database.

I built a database for pens and inks called Ink Well. The user can create a pen with the following attributes:
* brand
* model
* type
* description
* notes
* favorite

Brand, model, and type will be required.

# Validation at View Level
**Client-side validation** happens in the browser before the user submits data. It's very user-friendly and gives instant feedback.

HTML5 includes a new `required` attribute for inputs. All you have to do is add it to your input.

**Before**
```
<label for="brand">Brand</label><br>
<input type="text" name="pen[brand]">
```
	
**After**
```
<label for="brand">Brand</label><br>
<input type="text" name="pen[brand]" required>
```

The input will be invalid if there's no selection or entry and the browser will send an error message. 
![](https://lh3.googleusercontent.com/g-uwIDhMI87_yGuvnezdPaxlAPjx7TVkh--pCNRvGXJp0lyLuACRQp5fxo0=w2400)

For special input types like email and tel, it will be invalid if it doesn't match the correct pattern.

![](https://lh3.googleusercontent.com/7e4aA5QqPhYsc1q_GNncuOHtXwPCLbMGA1cHfAC5op3nyg6JlZvLvGXke_I=w2400)

The error message styling is dependant on the browser. At this time, you *can't* style them. You *can*  use the  :valid and :invalid CSS pseudo-classes to style input. Such as adding a red border to invalid inputs.

Javascript can also be used for validation and is completely customizable.

# Validation at Controller Level

**Server-side validation** happens after the user submits data. It's not as quick as client-side, but is a good secondary defense in case the browser doesn't support HTML5 or turns off Javascript.

If an input is blank, it will be sent through the params as `""`. To validate data with Sinatra, we just need to make sure the required attributes aren't empty. 

**Long form**
```
!params[pen][brand].empty? && !params[pen][model].empty? && !params[pen][type].empty? 
```

**Shorter form**
```
![params[pen][brand], [params[pen][model], [params[pen][type]].any?(&:empty?) 
```

# Validation at Model Level
Hmm, that's a lot to code for every required attribute. Is there a better way? ActiveRecord comes with built-in validation. This is another **server-side validation** that happens after data is submitted. It goes through the controller and into the model as a last line of defense.  Models can validate the data and also check it with the database.

To use these, just add the proper validations.

**Before**
```
class Pen < ActiveRecord::Base
  belongs_to :user
end
```	

**After**
```
class Pen < ActiveRecord::Base
  belongs_to :user

  validates_presence_of :type, :brand, :model
end
```	

Validations are checked when the object is sent to the database at create, save, and update. It won't persist to the database if it's invalid. You can also use `valid?` to check it yourself.

ActiveRecord validations are very customizeable. There are other pre-defined helpers including uniqueness and length. You can customize your requirements and even customize the error messages. Learn more [here](http://guides.rubyonrails.org/active_record_validations.html#when-does-validation-happen-questionmark).
```
class Pen < ActiveRecord::Base
  belongs_to :user

  validates :type, presence: {message: "How dare you!"}
  validates :brand, presence: {message: "Are you trying to break the database?!"}
  validates :model, presence: {message: "Why??"}
end
```

# Summary
There are many methods for form validation at different levels of the application. Using a combination of both client-side and server-side validations would be the safest and most user-friendly. 

For Ink Well, I took advantage of the built-in validations.  So validation happens at the view and model levels.





