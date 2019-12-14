[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)


## Rails Has and Belongs to Many Relationship

Let's build a recipes app where a recipe has and belongs to many ingredients and an ingredient has and belongs to many recipes.

<br>

## Create New Rails App

1. `rails new recipe_app -d postgresql`
1. `rails g model recipe name`
1. `rails g model ingredient name`
1. `rails g migration CreateJoinTableRecipesIngredients recipe ingredient` 
 
   > check out the `db/migrate` folder  

1. `rails db:setup`
1. `rails db:migrate`
1. `rails s`

<br>

## Setup the model associations
 
`models/recipe.rb`

  ```ruby
  class Recipe < ApplicationRecord
    has_and_belongs_to_many :ingredients
  end
  ```

`models/ingredient.rb`

  ```ruby
  class Ingredient < ApplicationRecord
    has_and_belongs_to_many :recipes
  end
  ```

<br>

## Create some seed data

`db/seeds.rb`

  ```ruby
  Recipe.destroy_all
  Ingredient.destroy_all

  pizza = Recipe.create(name: "Pizza")
  grilled_cheese = Recipe.create(name: "grilled cheese")

  grilled_cheese.Ingredients.create(name: "Pickles")

  pizza.ingredients.create(name: "tomato sauce")
  pizza.ingredients.create(name: "pepperoni")

  cheese = pizza.ingredients.create(name: "cheese")
  mushrooms = pizza.ingredients.create(name: "mushrooms")

  grilled_cheese.ingredients << [cheese, tomato]
  ```

1. `rails db:seed`
1. `rails c`
1. `pizza = Recipe.first`
1. `pizza.ingredients`
1. `pizza.ingredients.create(name: "peppers")`

<br>

## Create Routes

```ruby
Rails.application.routes.draw do
  root 'recipes#index'
  resources :recipes, :ingredients
end
```

<br>

## Recipes Controller

`rails g controller recipes`

```ruby
class RecipesController < ApplicationController
  def index
    @recipes = Recipe.all
    render json: @recipes
  end

  def show
    @recipe = Recipe.find(params[:id])
    # This code below has a quick demo of how to form a json object with nested ingredients on a recipe.
    render json: @recipe, include: :ingredients
  end
end
```

<br>

## Recipe Show View

```html
<h1><%= @recipe.name %></h1>

<ul>
<% @recipe.ingredients.each do |i| %>
  <li><%= i.name %></li>
  <% end %>
</ul>
```

<br>

## Recipe New View

```html
<%= form_for(@recipe) do |f| %>
  <p>
    <%= f.label :name, "Recipe Name" %>
    <%= f.text_field :name %>
  </p>

  <div>
  <%= f.collection_check_boxes(:ingredient_ids, Ingredient.all, :id, :name) do |i| %>
    <%= i.label %>
    <%= i.label { i.check_box } %>
  <% end %>
 </div>

  <p>
    <%= f.submit 'Create' %>
  </p>
<% end %>

<%= link_to 'Back', recipes_path %>
```

<br>

## Recipes Controller Create Method

The key here is to add `:ingredient_ids => []` to the `recipe_params` method.

```ruby
class RecipesController < ApplicationController
  def index
    @recipes = Recipe.all
    render json: @recipes
  end

  def show
    @recipe = Recipe.find(params[:id])
    # render json: @recipe, include: :ingredients
  end

  def new
    @recipe = Recipe.new
  end

  def create
    puts params
    @recipe = Recipe.create(recipe_params)
    redirect_to @recipe
  end

  private

    def recipe_params
      params.require(:recipe).permit(:name, :ingredient_ids => [])
    end
end
```

<br>

<details>

## You Do

Using this lesson as a guide, create a new app where a `person` has and belongs to many `groups`.

<br>

## Additional Resources

- [Rails Guides - Has and Belongs to Many](https://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association)
- [Collection Check Boxes Helper](https://apidock.com/rails/ActionView/Helpers/FormOptionsHelper/collection_check_boxes)
- [Devise](http://devise.plataformatec.com.br/)
