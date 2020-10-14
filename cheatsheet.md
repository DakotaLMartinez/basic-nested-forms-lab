## Dependencies (Gems/packages)

## Configuration (environment variables/other stuff in config folder)

## Database

## Models
Recipe 
Ingredient
In our Recipe model, we have to add the attribute writer. We can do this using accepts_nested_attributes_for if we don't care about duplicates.
```rb
class Recipe < ActiveRecord::Base
  has_many :ingredients 
  validates :title, presence: true

  accepts_nested_attributes_for :ingredients
end
```
If you need find_or_create_by, you'd do something like this:

```
class Recipe < ActiveRecord::Base
  has_many :ingredients 
  validates :title, presence: true
  
  def ingredients_attributes=(ingredients_attributes)
    ingredients_attributes.values.each do |ingredient_attributes|
      ing = Ingredient.find_or_create_by(name: ingredient_attributes[:name])
    end
  end
end
```
## Views
```
Recipes (index, new, show)
```

New action uses fields_for:

```rb
<%= form_for @recipe do |f| %>
  <p>
    <%= f.label :title, "Title" %>
    <%= f.text_field(:title) %>
  </p>
  <p>
    <%= f.fields_for :ingredients do |ingredient_fields| %>
      <p>
        <%= ingredient_fields.label :name %>
        <%= ingredient_fields.text_field :name %>
      </p>
      <p>
        <%= ingredient_fields.label :quantity %>
        <%= ingredient_fields.text_field :quantity %>
      </p>
    <% end %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```
## Controllers
```
RecipesController (index, show, new, create)
```

In our new action we had to create 2 ingredients that belong to the recipe we're creating a new instance of:
```rb
def new
  @recipe = Recipe.new
  2.times {
    @recipe.ingredients.build
  }
end
```
And then, in strong parameters, we have to make sure that the form data in the fields_for is permitted through. We do this by adding the ingredients_attributes key to the end of the permitted list, and as its value we pass an array of the input names that are nested in the ingredient_fields.

```rb
def recipe_params
  params.require(:recipe).permit(:title, ingredients_attributes: [:name, :quantity])
end
```
## Routes
```rb
resources :recipes, only: [:index, :show, :new, :create]
```
```
class Person < ActiveRecord::Base
  has_many :projects
  accepts_nested_attributes_for :projects
end
This model can now be used with a nested fields_for. The block given to the nested fields_for call will be repeated for each instance in the collection:

<%= form_for @person do |person_form| %>
  ...
  <%= person_form.fields_for :projects do |project_fields| %>
    <% if project_fields.object.active? %>
      Name: <%= project_fields.text_field :name %>
    <% end %>
  <% end %>
  ...
<% end %>
```