---
layout: post
title:      "Rails Only Dynamic-ish Form Fields "
date:       2018-06-26 03:58:13 -0400
permalink:  rails_only_dynamic-ish_form_fields
---


Dynamic form fields are generally built with JavaScript, but there's a way to fake this in Rails. The page isn't actually updated dynamically, but it *seems* dynamic.

## Background

This is something I experimented with while working on the Learn Rails Portfolio Project. The project is [Travelogger](https://github.com/unenlightened/travelogger) where users can create trips with multiple locations. 

How many trips though? It could be anything. Something dynamic would be perfect where the use can choose. The Rails Guide has [this](http://guides.rubyonrails.org/form_helpers.html#adding-fields-on-the-fly) to say on the matter.

So we're on our own! A happy accident caused interesting behavior (i. e. an error) during development.  I just pushed it further to see what was possible. This is a pared down version with the important bits.

## Basic MVC
The trips and locations tables have a name attribute.

### Models

The Trip model `has_many` Locations. Trip name is validated for presence.

```
#Trip Model

class Trip 
  has_many :locations
  validates :name, presence: true
end
```

Location `belongs_to` Trip. 

```
#Location Model

class Location 
  belongs_to :trip
end
```

### Controller

A new trip for "/trips/new" route. 

```
class TripsController
 def new
  @trip = Trip.new
 end
end
```

The create action gets a little more interesting.  
* Define a private method for `trip_params` allowing `:name`.
* In the create action, make a new trip with `trip_params` submitted from the form.
*  If the trip is invalid and can't save, render the form again.
*  Else, show the new trip.

```
class TripsController
 def create
  @trip = Trip.new(trip_params)

  if !@trip.save
   render :new
  else
   redirect_to trip_path(@trip) 
  end
 end
		
private

 def trip_params
  params.require(:trip).permit(:name)
 end
end
```
### View

Create a form for the @trip object. Rails magic will know that this should be submitted to the TripsController create action.

```
<!-- /trips/new.html.erb -->

<h1>New Trip</h1>

<%= form_for @trip do |f| %>
 <%= f.text_field :name, placeholder: "Name" %>

 <%= f.submit form_submit_text(trip) %>
<% end %>
```

##  Nested Form

Add this to the Trip model. 

* It allows you to create a Trip and its associated Locations in one form.
* Associated objectes can be destroyed with `allow_destroy`. 
* `reject_if` will validate the submission. In this case, none of the fields can be blank.

```
#Trip Model

accepts_nested_attributes_for :locations, :allow_destroy => true, reject_if: :all_blank
```

Create the associated Location with build in the controller. 
* Build one in new.
* When submitted, the validation will reject any that are blank. 
* If none were created, build another to populate the nested form.

```
#TripsController

def new
 @trip = Trip.new(user_id: user_id, start_date: Date.today)
 @trip.locations.build
end

def create
 @trip = Trip.new(trip_params)

 if !@trip.save
  @trip.locations.build if @trip.locations.none?
  render :new
 else
  redirect_to trip_path(@trip) 
 end
end
```

Update the `trip_params` to accept `locations_attributes`.
* Each will have their own id, since there can potentially be multiple.
* Name is an attribute defined in the locations table. 
* Destroy will allow you to delete from Trip.

```
#TripsController

def trip_params
 params.require(:trip).permit(:name, locations_attributes: [:id, :name, :_destroy])
end
```

Add the nested form to the Trip form with `fields_for`.

```
<!-- /trips/new.html.erb -->

<%= form_for @trip do |f| %>
 <%= f.text_field :name, placeholder: "Name" %>
	
 <%= f.fields_for :locations do |location_form| %>
  <%= location_form.text_field :name, placeholder: "Location" %>
 <% end %>

 <%= f.submit form_submit_text(trip) %>
<% end %>
```

Okay, so that will let you create Locations along with Trip. Nested forms can be super powerful. More about them on the [Rails Guide](http://guides.rubyonrails.org/form_helpers.html#nested-forms).

## Fancy Stuff
The order of this section will be reversed from previous examples.

Add a new submit button to the form. I just put it right after the location text_field. Button label text can be anything like "Add Location". The "+" makes it feel more like a button.

```
<!-- /trips/new.html.erb -->

<%= f.submit "+" %>
```

Tell your controller to do something about the new input. I defined a new method to make it more readable. Submit buttons are read in params as `:commit`.

```
#TripsController

private

def added_location?
 params[:commit] == "+"
end	
```

The create action will check if a Location is added and build a new one. We should now get a new Location field! 

By using an OR conditional,  `added_location?` is checked first. If true, it moves straight to the following block and bypasses the `!@trip.save` skipping validations like we want it to.

```
#TripsController

def create
 @trip = Trip.new(trip_params)

 if added_location? || !@trip.save
  @trip.locations.build if @trip.locations.none?
  @trip.locations.build if added_location?
  render :new
 else
  redirect_to trip_path(@trip) 
 end
end
```

## Going Further

Great, so this actually works so far. It works really well if you want to add just two locations.

You'll only be able to have two *blank* fields at most from the two `@trip.locations.build` in the create action. The issue is when you want more. Blank Location fields have to be filled out before being able to add a new one. 

It'd be annoying to fill-in, click, fill-in, click. I want click, click, click. What's going on? 

It's that `reject_if` validation in the Trip model's `accepts_nested_attributes_for`. It was perfect for a standard validation. Now, it's pesky when we want more custom behavior. Nested attributes seem to run whenever the Trip object is touched even we're not at an official validation stage like `@trip.location.build`.

Remove the validation.
```
#Trip Model

accepts_nested_attributes_for :locations, :allow_destroy => true
```

Also, remove `@trip.locations.build if @trip.locations.none?` from create. The one built in the new action will persist now.

```
# TripsController

def create
 @trip = Trip.new(trip_params)

 if added_location? || !@trip.save
  @trip.locations.build if @trip.locations.none?
  @trip.locations.build if added_location?
  render :new
 else
  redirect_to trip_path(@trip) 
 end
end
```

So this will allow you to add as many blank fields as your heart desires. But *now*, there are bunch of blank Locations saved with the Trip. Oof.

Define custom validations in the Trip model. We want to have a bunch of blank fields but don't need to save them. So the best time to do something about it would be right before save.
* Use`'before_save` callback to run a custom method.
* `reject_blank_locations!` will check each of the Trip.locations and delete blank ones.

```
#Trip Model

before_save :reject_blank_locations!

def reject_blank_locations!
 locations.each do |location|
  location.destroy if location.name.blank?
 end
end
```

## Extra Credit
You can use `validates :locations, presence: true` in the Trip model to make sure that it has a Location. But this won't work in conjunction with `:allow_destroy => true`.

```
# Trip Model

accepts_nested_attributes_for :locations, :allow_destroy => true, reject_if: :all_blank
```

Either remove the attribute *or* create a custom validation &#10004;.

## Conclusion
When you click on the "+" submit button, the page does send a new http request. So it's not *truly* dynamic (client-side). The behavior and speed of refresh is very quick and seamless. It *feels* dynamic.

The simple solution would have been to use JavaScript. But working this out allowed me learn so much more about validations, design patterns, and the process of building a custom solution. 


## Location, Location, Location

![](https://lh3.googleusercontent.com/WzyeBnusHLeu3-ma2oH6VRZFqEPwCnFLQvf2n-CANaIh5VxCpwL0P8AbjhU=w2400)





