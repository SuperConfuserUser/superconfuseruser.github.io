---
layout: post
title:      "Rails Only "Dynamic" Form Fields "
date:       2018-06-26 00:14:55 -0400
permalink:  rails_only_dynamic_form_fields
---


Dynamic form fields are generally built with JavaScript, but there's a way to fake this in Rails. The page isn't actually updated dynamically, but it *seems* dynamic.

## Background

This is something I experimented with while working the Learn Rails Portfolio Project. The project is Travelogger where users can create trips with multiple locations. 

How many trips though? It could be anything. Something dynamic would be perfect where the use can choose. The Rails Guide has [this](http://guides.rubyonrails.org/form_helpers.html#adding-fields-on-the-fly) to say on the matter.

So we're on our own! A happy accident caused interesting behavior (i. e. an error) during development.  I just pushed it further to see if it was possible. This is a pared down version with the important bits.

## Basic MVC

### Models

The Trip model has_many Locations. Location belongs_to Trip. 

```
class Trip 
  has_many :locations
end
```

```
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

The create route gets a little more interesting.  Define a private method for trip_params. Trip has a name attribute with a string datatype. 

```
class TripsController
	def create
		@trip = Trip.new(trip_params) 

		if !@trip.save
			render :new
		else
			redirect_to user_trip_path(@trip.user, @trip)
		end
	end
		
private

	def trip_params
		params.require(:trip).permit(:name)
	end
end
```
View 

