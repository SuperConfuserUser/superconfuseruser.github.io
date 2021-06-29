---
layout: post
title:      "Creating Custom API Data - Rails Serializer "
date:       2018-06-25 00:14:55 -0400
permalink:  rails_serializer
---

You can add more than just ActiveRecord model attributes created in the database schema and associated models to the Rails active model serializer.

<!--more-->

## Background
The [Travelogger](https://github.com/SuperConfuserUser/travelogger/blob/javascript-front-end/app/assets/javascripts/trips.js) site keeps track of trips. Users have many trips. And a trip belongs to a user.

The project uses a Rails backend API to send JSON data to the JavaScript frontend. You can serialize Rails data into a JSON format very easily with [Active Model Serializer](https://github.com/rails-api/active_model_serializers).

I want an individual trip to know it's user's total trip count. One method is to add the user and all of their trips to the single trip's API data and have JavaScript count them. But that's total overkill...

Custom API data can be created on the Rails side instead and then sent to JavaScript. Much nicer.
## Requirements

1. Set up the Rails general backend, especially models and associations.
2. Add active_model_serializer gem.
3. Generate serializers.
4. Generate any explicit serializers.

## Very Basic Set Up
### Schema
{% highlight ruby %}
create_table "trips", force: :cascade do |t|
    t.string "name"
    t.integer "user_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["user_id"], name: "index_trips_on_user_id"
end

create_table "users", force: :cascade do |t|
    t.string "username"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
end

{% endhighlight %}

### Models
{% highlight ruby %}
# User Model

class User < ApplicationRecord
  has_many :trips
end
{% endhighlight %}

{% highlight ruby %}
# Trip Model

class Trip < ApplicationRecord
  belongs_to :user
end
{% endhighlight %}

### Serializers
{% highlight ruby %}
#Trip Serializer

class TripSerializer < ActiveModel::Serializer
  attributes :id, :name, :user_id
  belongs_to :user, serializer: TripUserSerializer
end
{% endhighlight %}

{% highlight ruby %}
#UserSerializer
class UserSerializer < ActiveModel::Serializer
  attributes :id, :username
  has_many :trips
end
{% endhighlight %}

### Explicit Serializer
{% highlight ruby %}
# Trip User Serializer

class TripUserSerializer < ActiveModel::Serializer
  attributes :username
end
{% endhighlight %}

##  Custom API Data
Create a new function in your model that returns data.

{% highlight ruby %}
# User Model

class User < ApplicationRecord
  has_many :trips

  def trip_count
    self.trips.count
  end
end
{% endhighlight %}

Add the name of the new method to your attributes.

{% highlight ruby %}
# Trip User Serializer

class TripUserSerializer < ActiveModel::Serializer
  attributes :username, :trip_count
end
{% endhighlight %}

Another way is to create the new function in the serializer itself.

{% highlight ruby %}
# Trip User Serializer

class TripUserSerializer < ActiveModel::Serializer
  attributes :username, :trip_count
	
  def trip_count
    object.trips.count
  end
end
{% endhighlight %}

And that's it! Super simple.

## Conclusion
This is a great way to control exactly what info each API serves and how it can be accessed. API calls are minimized and streamlined. 

