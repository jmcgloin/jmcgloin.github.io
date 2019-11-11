---
layout: post
title:      "Activerecord Scope: "
date:       2019-11-11 03:44:36 +0000
permalink:  activerecord_scope
---

## Harness the power of Activerecord with custom class methods and scope 

Let's say you're making an app that lists various kinds of animals up for adoption.  In this rails app, you have an Animals class.  On your index view, you want to list all of the animals that are up for adoption.  However, you know your users will want to only see certain kinds of animals at a time.  This isn't really a very difficult problem to solve using Activerecord querying.  Something like:
```
Animal.where("animal_type = ?", "Dog")
```
Not bad at all, really.  But lets say you need this  query to be done multiple times, in multiple places in your code.  Not only will all that typing add up, but you increase the chances of needing to hunt down those super pesky bugs like typing "dog" instead of "Dog". 

If you could just call something like: `Animal.all_dogs` it would really simplify things.  This is where class methods come in.  Really it's as simple as defining a class method containing the code written above.
```
class Animal < ApplicationRecord
  def self.all_dogs
	  where("animal_type = ?", "Dog")
	end
end
```

Now you have access to the all_dogs method whenever you need it.  You can further extend the convenience of this  class method by passing in a variable, removing the need to write a method for each kind of animal.
```
class Animal < ApplicationRecord
  def self.all_of_kind(kind_of_animal)
	  where("animal_type = ?", kind_of_animal)
	end
end
```
Simply calling this method and passing in the kind of animal you want will give you the  results you're looking for.

"So, J..." you say, "this is cool and useful stuff.  But I'm not seeing anything about scope like your title suggests."  Well, here you go.  The above method can also be written like this:
```
class Animal < ApplicationRecord
  scope :all_of_kind, -> (kind_of_animal) { where("animal_type = ?", kind_of_animal) }
end
```
What's the difference?  Syntactic sugar, baby.  Its just a more elegant way of writing it: fewer lines to write and you can spot your scoped queries right away without them getting lost in a sea of class methods.  Doing it either way (custom defined class method vs. scope) will yield the same `ActiveRecord::Relation` object.  

As with class methods, all of your scopes can  be chained with other scopes and class methods.  For instance, lets say we have another scope within the Animal class:
```
  scope :with_temperament, ->(temperament) {where("animal_temperament = ?", temperament) }
```
Now we can chain these two together to find just the right pet.  For example: `Animal.all_of_kind('Cerberus').with_temperament('bloodthirsty')` would show  us a  list of animals perfect for guarding a trapdoor where we could be hiding a certain precious stone.

Lastly, let's look at using a scope method that relies on another model.  For this, we'll create a Shelter class that will describe the shelters where these animals are residing while awaiting adoption.  These two models have an association between them.  It would look like this:
```
class Shelter < ApplicationRecord
  has_many :animals
end

class Animal < ApplicationRecord
  belongs_to :shelter
end
```
When we show the animals, we want to show where they can be found.  Again, this isn't a difficult thing to do.  Using the methods the associations create for us, we could say `animal.shelter`.  But if we want to look at animals from a certain shelter we must either iterate over all the animals.  We could, of course, go through the Shelter class, i.e. `Shelter.animals`.  But what if we want to search for all the gerbils at One of a Kind Pets shelter?  This grows in complexity and length of code quickly and really isn't something we want to type repeatedly.  Class methods and scoping makes this very simple.  In the Animal class we can write:
```
scope :all_of_kind, -> (kind_of_animal) { where("animal_type = ?", kind_of_animal) }
scope :from_shelter_called, -> (shelter_name) { joins(:shelters).where("shelter_name = ?", shelter_name) }
```
Now when we want to search for local gerbils, its as easy as: `Animal.all_of_kind('Gerbil').from_shelter_called('One of a Kind Pets')`.  Two nice and tidy lines in our class (assuming the associations and table columns match what we've used) and we can get whatever information we require.
