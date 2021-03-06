STARTING OUT YOUR APPS
============

## Node APPS

### Starting off

* mkdir firstapp (get the party started)
* touch app.js
* touch .gitignore
* git init

Add the text 'node_modules' in our .gitignore

* echo "node_modules" >> .gitignore

Start the project and start adding dependencies that you might need!

* npm init
* Ex) npm install --save express ejs pg lodash sequelize sequelize-cli bcrypt

If your going to incoporate database create one and sequelize the project

* createdb firstapp
* sqlize init

### Setup - config, models and migrations

change config file to

```
{
  "development": {
    "database": "firstproject",
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
  "test": {
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
  "production": {
    "use_env_variable": "DATABASE_URL"
  }
}


```
Your routes.js file should look like this
```
var routeMiddleware = {
  checkAuthentication: function(req, res, next) {
    if (!req.user) {
      res.render('login', {message: "Please log in first"});
    }
    else {
     return next();
    }
  },

  preventLoginSignup: function(req, res, next) {
    if (req.user) {
      res.redirect('/home');
    }
    else {
     return next();
    }
  }
};
module.exports = routeMiddleware;

```

Depending on how many dependices you have a normal app.js might look like this

```
var express = require('express'),
    app = express(),
    bodyParser = require('body-parser'),
    methodOverride = require('method-override'),
    db = require("./models/index"),
    passport = require("passport"),
    request = require("request"),
    passportLocal = require("passport-local"),
    cookieParser = require("cookie-parser"),
    session = require("cookie-session"),
    flash = require("connect-flash");
    var routeMiddleware = require("./config/routes");

app.set('view engine', 'ejs');
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.urlencoded({extended: true}));
app.use(methodOverride('_method'));

app.get('/', function(req, res){
  res.send('Hello World');
});

var server = app.listen(process.env.PORT || 3000, function() {
    console.log('Listening on port %d', server.address().port);
});
```




### Creating Models

EX) `sqlize model:create --name User --attributes username:string, password:string'

with your user migration look like
```
"use strict";
module.exports = {
  up: function(migration, DataTypes, done) {
    migration.createTable("Users", {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: DataTypes.INTEGER
      },
      username: {
        type: DataTypes.STRING,
        unique: true,
        allowNull: false
      },
      password: {
        type: DataTypes.STRING
      },
      createdAt: {
        allowNull: false,
        type: DataTypes.DATE
      },
      updatedAt: {
        allowNull: false,
        type: DataTypes.DATE
      }
    }).done(done);
  },
  down: function(migration, DataTypes, done) {
    migration.dropTable("Users").done(done);
  }
};
```

and your user model looking like
```
"use strict";

var bcrypt = require("bcrypt");
var passport = require("passport");
var passportLocal = require("passport-local");
var salt = bcrypt.genSaltSync(10);

module.exports = function (sequelize, DataTypes){
   var User = sequelize.define('User', {
     username: {
        type: DataTypes.STRING,
        unique: true,
        allowNull: false,
        validate: {
          len: [6, 30]
        }
    },
    password: DataTypes.STRING
    },

  {
    classMethods: {
      associate: function(models) {
        User.hasMany(models.Food,{onDelete: "NO ACTION"});
    },
      encryptPass: function(password) {
        var hash = bcrypt.hashSync(password, salt);
        return hash;
    },
      comparePass: function(userpass, dbpass) {
      // don't salt twice when you compare....watch out for this
        return bcrypt.compareSync(userpass, dbpass);
    },
      createNewUser:function(username, password, err, success ) {
        if(password.length < 6) {
          err({message: "Password should be more than six characters"});
        }
        else{
        User.create({
            username: username,
            password: this.encryptPass(password)
          }).done(function(error,user) {
            if(error) {
              console.log(error)
              if(error.name === 'SequelizeValidationError'){
              err({message: 'Your username should be at least 6 characters long', username: username});
            }
              else if(error.name === 'SequelizeUniqueConstraintError') {
              err({message: 'An account with that username already exists', username: username});
              }
            }
            else{
              success({message: 'Account created, please log in now'});
            }
          });
        }
      },
      } // close classMethods
    } //close classMethods outer

  ); // close define user

  passport.use(new passportLocal.Strategy({
      usernameField: 'username',
      passwordField: 'password',
      passReqToCallback : true
    },

    function(req, username, password, done) {
      // find a user in the DB
      User.find({
          where: {
            username: username
          }
        })
        // when that's done,
        .done(function(error,user){
          if(error){
            console.log(error);
            return done (error, req.flash('loginMessage', 'Oops! Something went wrong.'));
          }
          if (user === null){
            return done (null, false, req.flash('loginMessage', 'Username does not exist.'));
          }
          if ((User.comparePass(password, user.password)) !== true){
            return done (null, false, req.flash('loginMessage', 'Invalid Password'));
          }
          done(null, user);
        });
    }));

  return User;
}; // close User function

```
THIS SHOULD GET YOU STARTED ON MAKING AN AWESOME NODE APPLICATION
* Refer back to your moonwalk app to get you started Ryan....


## Heroku - Start it up!
Once you finish making your app lets
Make a app on your heroku dashboard

### Setup
* touch Procfile
* `echo "web: node app.js" >> Procfile`
* `heroku config:set PORT=80 --app YOUR_APPLICATION_NAME`
* `heroku git:remote -a YOUR_APPLICATION_NAME`
* `git push heroku master`
* 'heroku ps:scale web=1 --app YOUR_APPLICATION_NAME'

### Connect a DB with sequelize (this should be done ONLY after you have set up your local database and ran all migrations successfully):

1. In terminal, install the add-on for postgres
  ``` heroku addons:add heroku-postgresql:dev```
2. This should create a DATABASE_URL config variable for you. If not, run ``` heroku config:set DATABASE_URL=(THE OTHER DB URL HEROKU HAS GIVEN) --app YOUR_APPLICATION_NAME```
3. Set your NODE_ENV variable to 'production' by running this command in terminal: ```heroku config:set NODE_ENV='production' --app YOUR_APPLICATION_NAME```
4. Make sure your production variables in config.json are set like this

  ```
  "production": {
      "use_env_variable": "DATABASE_URL"
  }
  ```

5. Add and commit your changes using `git commit -am "adding production db"` and then push your changes to heroku using `git push heroku master`
6. Now run your migrations by typing in terminal ``` heroku run node_modules/.bin/sequelize db:migrate``` and you should have all your tables set up in a heroku hosted database


### Connect to your heroku DB using psql

1. In terminal, type in heroku pg:psql and it should connect you to your DB

### Connect to your heroku DB using PG Commander:

1. In the bottom left corner of PG Commander click New Favorite
2. Put in the following information from your heroku database URL (to see this again just run in terminal: ``` heroku config```)
3. Here is the pattern for the URL and the information you need to put into PG Commander ___do not include : or @ or / when inputting information___:
  - __postgres://USERNAME:PASSWORD@HOSTNAME:PORT/DATABASE__
4. Once connected it should alert you that it cannot verify the identity of the server, just click connect anyway and you should be in

### I broke it...what do I do?

Always, always, always start by looking at the heroku logs (in terminal, type ```heroku logs -t```). This will tell you any node/db errors you are having (you can think of heroku logs like your terminal, whenever you `console.log` something, you will see it here) and remember...___It happens___, here are some things to double check:

1. Make sure you have named the file "Procfile" and that it is NOT in any sub-folders (it should be in the same folder as your app.js file)
2. Make sure you typed this __exactly__: ```web: node app.js``` (if your main file is named app.js, otherwise change it to whatever you have named your starting file)
3. Did you miss a dependency? Check your package.json file to make sure you have all the modules you need to run the application and if you're missing something, run ```npm install --save MODULE_NAME```
4. Check your config variables and make sure you have at least a PORT, NODE_ENV and DATABASE_URL variable set. Is your PORT set to 80? Is your NODE_ENV set to 'production' (make sure this is a string)
5. Have you created a remote to push to heroku? Check with ```heroku remote -v```
6. Check to see if for some reason you have a PORT variable declared in your .zshrc or .bashrc file by running echo $PORT (if you see anything, that's not good and you need to go in your .zshrc or .bashrc file and remove it). This sometimes overwrites the heroku variable and heroku doesn't get too happy about that...

### But....things are still breaking!

1. Did you __commit__ your most recent changes and push them to heroku? If you made any changes or installed any new modules, heroku will not know about it until you run ```git push heroku master```
2. Is your `config.json` file is set up correctly? Make sure the NODE_ENV is set to 'production' and then check to see that the information from your DATABASE_URL variable match what is in the `config.json`
3. We can't use the alias sqlize anymore, so when you run your migrations make sure to run `heroku run node_modules/.bin/sequelize db:migrate` or `sequelize db:migrate`
4. Did you accidentally forget to create config variables for your keys? Remember to add your keys using `heroku config:set VARIABLE_NAME=VALUE`
5. Is there anything in your `.gitignore` file that heroku needs?

### Heroku best practices

- Store your secret information in config variables (this includes the password to your database and secret keys!)
  - To create a new config variable run this in terminal: `heroku config:set VARIABLE_NAME=VALUE --app YOUR_APPLICATION_NAME`
  - To remove a variable name run in terminal: `heroku config: unset VARIABLE_NAME --app YOUR_APPLICATION_NAME`
  - To see all of your config variables run `heroku config`
  - To reference your heroku variable in your code use  `process.env.VARIABLE_NAME`
  - Start fresh with a production database, but if you REALLY want your development database info to transfer to the production one use  `heroku pg:push NAME_OF_YOUR_LOCAL_DATABASE NAMEOF_HEROKU_CONFIG_DB --app YOUR_APPLICATION_NAME` (note, your heroku DB __must__ be empty for this to run)




## RAILS APP

### Starting off

* Start with a new rails app `rails new something -TBd postgresql
* In your gemfile, comment out the bcrypt-ruby gem and add pry-rails
* bundle install

Lets create a user model
* rails g model User username password password_digest

Your User.rb in models folder should look like
```
class User < ActiveRecord::Base
  has_secure_password

  validates :username,
    uniqueness: true,
    presence: true

end
```
and your migration should look like
this includes a password reset token :3
```
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :username
      t.string :password
      t.string :password_digest
      t.integer :reset_token

      t.timestamps
    end
  end
end
```


Your home page can do cool stuff with Users!
Here is an example!!! Of login and password resets

```
class AccessController < ApplicationController
before_action :confirm_logged_in, except: [:new, :create, :attempt_login, :password_reset, :reset, :reset_password, :logout, :home]
before_action :prevent_login_signup, only: [:home]



# HOME PAGE
  def home
    @user = User.new
  end

  def create
    @user = User.create(user_params)
    if(@user.save)
      UserMailer.signup_confirmation(@user).deliver
      session[:user_id] = @user.id
      flash[:success] = "You are now logged in!"
      redirect_to index_path
    else
      flash[:alert] = "Something went wrong. Try again"
      redirect_to root_path
    end

  end

  def attempt_login

    if params[:username].present? && params[:password].present?
      found_user = User.where(username: params[:username]).first
      if found_user
        authorized_user = found_user.authenticate(params[:password])
      end
    end

    if !found_user
      flash.now[:alert] = "Invalid Username"
      @user = User.new
      render :home
    elsif !authorized_user
      flash.now[:alert] = "Invalid Password"
      @user = User.new
      render :home
    else
      session[:user_id] = authorized_user.id
      flash[:success] = "You are now logged in."
      redirect_to index_path
    end
  end

  def password_reset

    if User.where(username: params[:username]).present?
      @user = User.where(username: params[:username]).first
      @user.update_attributes(:reset_token => Random.rand(100))
      UserMailer.password_reset(@user, root_url).deliver
      redirect_to root_path
    else
      redirect_to root_path
    end
  end

# RESET PAGE
  def reset
    puts "RESET ACTION!!!!"
    if User.find_by_reset_token(params[:user_reset_token]).present?
      @user = User.find_by_reset_token(params[:user_reset_token])
    else
      redirect_to root_path
    end
  end


  def reset_password
    @user = User.find_by_reset_token(params[:user_reset_token])
    @user.update_attributes(user_params)
    @user.update_attributes(:reset_token => nil)
    if(@user.save)
      session[:user_id] = @user.id
      flash[:success] = "You're profile is updated"
      redirect_to root_path
    else
      render :reset
    end
  end


  def logout
    # mark user as logged out
    session[:user_id] = nil
    flash[:notice] = "You are now logged out"
    redirect_to root_path
  end


# APP PAGE
  def index
    @current_user = current_user
  end

  private

  def user_params
    params.require(:user).permit(:username, :password, :password_confirmation)
  end
end

```

### Mailers

Create a mailer instace

```
rails g mailer user_mailer signup_confirmation password_reset

```

Inside teh mailers folder
It can look like this!

```
class UserMailer < ActionMailer::Base
  default from: "moonwalk2015@gmail.com"

  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.user_mailer.signup_confirmation.subject
  #
  def signup_confirmation(user)
    @user = user
    mail to: user.username, subject: "Sign Up Confirmation"
  end

  def password_reset(user, root_url)
    @user = user
    @url = root_url+"reset/#{user.reset_token}"
    mail to: user.username, subject: "Password Reset"
  end
end

```

You can then change your template how you like!
EX)

```
<!-- <h1 style="text-align:center">Moonwalk</h1><br>

Hi <%= @user.username %>,<br><br>

Thank you for signing up!  We appreciate your interest in our application and look forward to hearing your feedback.  Please let us know if you need help navigating our site or just have a general question.  Check us out at <a href="http://moonwalk.herokuapp.com/">Moonwalk.Heroku.com!</a><br>

<img style="height:150px; width: 250px;"src="https://s3.amazonaws.com/uploads.hipchat.com/39979/1252803/PUPLibnyzYmw3pM/Moonwalk7.gif" alt=""> -->
```

### Mail config

Inside ` /config/environments/development.rb `

Add these lines
```
config.action_mailer.raise_delivery_errors = true
config.action_mailer.delivery_method = :sendmail
config.action_mailer.perform_deliveries = true

```
Create a new intializer file

` /config/initializers/setup_mail.rb `

And add an action mailer
Ex)

```
ActionMailer::Base.smtp_settings = {
  :address              => "smtp.gmail.com",
  :port                 => 587,
  :domain               => "gmail.com",
  :user_name            => "moonwalk2015@gmail.com",
  :password             => "moonwalk24",
  :authentication       => "plain",
  :enable_starttls_auto => true
}

```


### Setting up Heroku

Make sure your using inside your Gemfile
`gem 'pg'`

and add

```
group :production do
  gem 'rails_12factor'
end

```

then run a bundle install --without production

add changes to your deploy and commit them :3

You can then
```
$heroku create my_app_name
then
git push heroku master
```

then migrate your stuff

heroku run rake db:migrate

if your having problems inside your production.rb file add

` config.assets.compile = true`



## ANGULAR

Look at following rails app setup to get started

* Start with a new rails app `rails new something -TBd postgresql
* inside gemfile 'pry-rails'

```
bundle
rake db:create
rake db:migrate
rake db:seed
rails s
```

DELETE TURBOLINKS
```
REMOVE THE Gem turbolinks
remove them in your javascipt application
and remove the two
"data-turbolinks-track" => true
inside your view application

```

ADD ANGULAR
```
gem 'angular-gem'
gem 'angularjs-rails-resource', '~> 1.1.1'

inside your app javascript
//= require angular
//= require angularjs/rails/resource

```

your application view page can look like
````
<!-- <!DOCTYPE html>
<html>
<head>
  <title>Whatever</title>
  <%= stylesheet_link_tag    'application', media: 'all' %>
  <%= javascript_include_tag 'application' %>
  <%= csrf_meta_tags %>
</head>
<body ng-app="app">
  <div class="container">
<%= yield %>
  </div>
</body>
</html>
 -->
 ```

 Turn your rails into an api server

 ```
 rails g controller main index

 (Models are Singular, Controllrers are Plural)

 rails g model player name:string winner:boolean rating:integer
 rails g controller players index show create update destroy

 then in your playerscontroller

class PlayersController < ApplicationController
  # controller supports json only, it can't render pages
  respond_to :json

  def index
    # For a given controller action,
    # respond_with generates an appropriate
    # response based on the mime-type requested
    # by the client.
    respond_with Player.all

  end

  def show
    respond_with Player.find(params[:id])
  end

  def create
    respond_with Player.create(player_params)
  end

  def update
    respond_with Player.update(params[:id], player_params)
  end

  def destroy
    respond_with Player.delete(params[:id])
  end

  private

  def player_params
    params.require(:player).permit(:name, :winner, :rating)
  end
end

```

Add player resource routes:
```
resources :players

  root to: 'main#index'
end
```

You can remove the players view folder
and also the player js folder

inside your main.js.cofeee
it can look like with couple examples of calling the Player model

```
# Place all the behaviors and hooks related to the matching controller here.
# All this logic will automatically be available in application.js.
# You can use CoffeeScript in this file: http://coffeescript.org/
app = angular.module "app", ['rails']

# Define Config for CSRF token
app.config ["$httpProvider", ($httpProvider)->
  $httpProvider.defaults.headers.common['X-CSRF-Token'] = $('meta[name=csrf-token]').attr('content')
]

# Grab the Player routes
app.factory "Player", (railsResourceFactory) ->
  resource = railsResourceFactory(
    url: "/players"
    name: "player"
  )
  return resource

app.controller "IndexCtrl", ['$scope','$http','Player', ($scope, $http, Player ) ->

  $scope.test = 123;

  # GETS ALL THE PLAYERS INSIDE THE DATABASE
  Player.query().then (result) ->
    $scope.players = result

  # ADD THAT PLAYER
  $scope.addPlayer = () ->
    console.log "it's here"
    newPlayer = new Player(name: $scope.newName, rating: 5, winner: false)
    newPlayer.create().then (newlyCreatedPlayer) ->
      $scope.players.push newlyCreatedPlayer
      $scope.newName = ""

  # DELETE THAT PLAYER
  $scope.deleteItem = (player) ->
    player.delete().then () ->
      select = $scope.players.indexOf(player)
      $scope.players.splice(select, 1)

  # PICK THAT WINNER
  $scope.pickWin = () ->
    unwinners = $scope.players.filter (human) ->
      !human.winner

    if unwinners.length is 0
      $scope.players.forEach (human) ->
        human.winner = false;
        human.update()
    else
      item = Math.floor(Math.random() * unwinners.length)
      console.log item
      person = unwinners[item];
      person.winner = true;
      person.update()

]
```

If you want to add filter, directive pages that's not in your main controller you can do something like this!

Inside your application.js folder
you can require your main js file with your app = module thingy.

`//= require main`

then inside your main.js file

`window.app = angular.module "app", []`

add that window so you can make it global

now you can make filter.js files and directive.js files and all would be good

ex) Inside my filter.js.coffee file!
`
app.filter("reverse", () ->
  (input) ->
    console.log input
    for title in input
      title.split("").reverse().join("")
)
`

ADDING BOOTSTRAP TO RAILS


Inside application.css
add
```
*= require bootstrap-3.2.0.min
```

Go get the bootstrap file from somewhere and put it inside your vendors stylesheet folder.






### Refactoring your angular Code

We want to Refactor our code so we can have a more organized look for our angular app. This can get a bit confusing so lets get started!!! It's in javascript but I'll do coffeee script later :3

We need to install the underscore gem because if you notice
the "_.find" is an udnerscore method so go inside your gem file `gem 'underscore-rails'`
bundle and restart

Go slow ryan You can do this!!!

Inside your javascript folder
Lets create an app folder

Inside this app folder
* lets create a controllers folder
* app.js
* factories.js

Then lets go to our application.js file
and add the required stuff

```
//= require angular
//= require angular-route
//= require angularjs/rails/resource
//= require underscore
//= require app/app
//= require_tree .
```
Should look like this through thick and thin :3


Now inside your app.js file lets get the needed things :)

```
angular.module "app.controllers", []
angular.module "app.factories", []

app = angular.module("app", [
  "rails"
  "app.controllers"
  "app.factories"
])

```

This will allow us to use our controllers folder and factories file because it goes through this javascript file first.

Now inside your factory folder you can get your resources like this

```
angular.module("app.factories").factory "Player", (railsResourceFactory) ->
  resource = railsResourceFactory(
    url: "/players"
    name: "player"
  )
  resource


```
Grabbing from your players controller/route


Inside yoru controllers folder you can now setup your controler like this!

Continuing on our main.js file
```
angular.module("app.controllers").controller "IndexCtrl", [
  "$scope"
  "Player"
  ($scope, Player) ->

    Player.query().then (result) ->
      $scope.players = result
      return

    # ADD THAT PLAYER
    $scope.addPlayer = () ->
      console.log "it's here"
      newPlayer = new Player(name: $scope.newName, rating: 5, winner: false)
      newPlayer.create().then (newlyCreatedPlayer) ->
        $scope.players.push newlyCreatedPlayer
        $scope.newName = ""

    # DELETE THAT PLAYER
    $scope.deleteItem = (player) ->
      player.delete().then () ->
        select = $scope.players.indexOf(player)
        $scope.players.splice(select, 1)

    # PICK THAT WINNER
    $scope.pickWin = () ->
      unwinners = $scope.players.filter (human) ->
        !human.winner

      if unwinners.length is 0
        $scope.players.forEach (human) ->
          human.winner = false;
          human.update()
      else
        item = Math.floor(Math.random() * unwinners.length)
        console.log item
        person = unwinners[item];
        person.winner = true;
        person.update()
]
```

You gotta add that angular.modlule now



### Moving onto multiple angular routing

Require your `//= require angular-route`

and inside your app.js file

```
angular.module "app.controllers", []
angular.module "app.factories", []

app = angular.module("app", [
  "rails"
  "app.controllers"
  "app.factories"
  "ngRoute"
])

```
We got to add it!


Lets say we want ot have three routes/pages in our app:

/ - home page

/movies -> Goes to a page that lists the top 25 movies

/movie/:movie_id -> Goes to a page that plays requested movie trailers

You can do whatever you want :3

Your views will now be inside the public/templates page now

So inside lets create
* index.html
* movies.html
* movie.html

Your index.html file can now look like

```
<div ng-controller="IndexCtrl">
  <div class="container">

  <h1>Southpark Raffle</h1>

  <form ng-submit="addPlayer()">
    <input type="text" ng-model="newName">
    <input type="submit" value="Add">
  </form>

  <button ng-click="pickWin()" class="btn btn-lg">WINNER</button>

  <ul>
    <li ng-repeat="player in players" ng-class="{fun: player.winner}">
      {{player.name}}
      <button ng-click="deleteItem(player)" class="btn btn-sm btn-danger">Delete</button>
    </li>
  </ul>

  <br>
    <a ng-href="/movies">I'm tired of this, lets watch movie trailers ... </a>
  </div>
</div>
```


It won't work we have to do one more step
we need to add our routes and location provider inside our app.js

for this example it would like this

```
angular.module "app.controllers", []
angular.module "app.factories", []

app = angular.module("app", [
  "rails"
  "app.controllers"
  "app.factories"
  "ngRoute"
])

app.config ["$httpProvider", ($httpProvider)->
  $httpProvider.defaults.headers.common['X-CSRF-Token'] = $('meta[name=csrf-token]').attr('content')
]

app.config ($routeProvider, $locationProvider) ->
  $locationProvider.html5Mode
    enabled: true
    requireBase: false

  $routeProvider.when("/",
    templateUrl: "/templates/index.html"
    controller: "IndexCtrl"
  ).when("/movie/:movie_id",
    templateUrl: "/templates/movie.html"
    controller: "MovieController"
  ).when("/movies",
    controller: "MoviesController"
    templateUrl: "/templates/movies.html"
  ).otherwise redirectTo: "/"
  return
```

and one last step
Inside our application file for our views
```
<!DOCTYPE html>
<html ng-app="app">
<head>
  <base href="/">
  <title>Raffler</title>
  <%= stylesheet_link_tag    'application', media: 'all' %>
  <%= javascript_include_tag 'application'%>
  <%= csrf_meta_tags %>
</head>
<body>

<div ng-view>
</div>

</body>
</html>
```

Finally if we can to add other pages like /movies
inside your routes file
```
Rails.application.routes.draw do
  resources :players

  root to: 'main#index'
  match '*any' => "main#index", :via => [:get, :post]

end

```


### Next lets create controllers and factory

Lets create a MovieController and MoviesController

Ex) movie.js
```
angular.module("app.controllers").controller "MovieController", [
  "$scope"
  "$routeParams"
  "$sce"
  "YouTube"
  ($scope, $routeParams, $sce, YouTube) ->
    console.log $routeParams
    YouTube.getTop25Movies().then (result) ->
      movies = result
      console.log result
      $scope.movie = _.find(movies, (v) ->
        v.youtubeId is $routeParams.movie_id
      )

      $scope.movie.youtubeUrl = $sce.trustAsResourceUrl("http://www.youtube.com/embed/" + $scope.movie.youtubeId + "?rel=0")

]
```

and movies.js
```
angular.module("app.controllers").controller "MoviesController", [
  "$scope"
  "YouTube"
  ($scope, YouTube) ->
    YouTube.getTop25Movies().then (result) ->
      $scope.movies = result

]
```

finally lets create a new factory called Youtube
```
angular.module("app.factories").factory "YouTube", ($http, $q)->
  api = {}

  api.testFunction = ->
    "Jaws from a function in YouTube factory"

  api.getTop25Movies = ->
    url = "http://gdata.youtube.com/feeds/api/charts/movies/most_popular?v=2&max-results=25&paid-content=true&hl=en&region=us&alt=json&callback=JSON_CALLBACK"
    d = $q.defer()
    $http.jsonp(url).then (response)->
      movies = response.data.feed.entry.map((movie)->
        youtubeId: movie["media$group"]["yt$videoid"]["$t"]
        title: movie["media$group"]["media$title"]["$t"]
      )
      d.resolve movies


    d.promise

  api

```
This includes ajax call.
It ges the youtubeId from the movie and also the title
hence youtubeId: and title:

so now in our MoviesController
we can get that data.

```
angular.module("app.controllers").controller "MoviesController", [
  "$scope"
  "YouTube"
  ($scope, YouTube) ->
    YouTube.getTop25Movies().then (result) ->
      $scope.movies = result

]
```

With this new data we can now correspond with our movies.html file

```
 <div class="container">
 <div class="row">
    <div ng-repeat="movie in movies" class="col-md-4 col-sm-6 col-xs-12">
        <a href="/movie/{{movie.youtubeId}}">
          <div class="title">
            {{movie.title}}
          </div> <!-- end "title"-->
        </a>
    </div> <!-- end movie -->

  </div>
  <br>
  <br>
  <a ng-href="/">I'm tired of this, lets play raffler </a>
</div>
```

the link leads to the movie page
The page is like this
Were workign with videos awesome :3
```
<div class="container">
  <div class="row">
    <div class="col-xs-12">
      <h2> {{movie.title}} </h2>
      <div class="movie-player-container flex-movie widescreen">
        <iframe width="853" height="480" ng-src="{{movie.youtubeUrl}}" frameborder="0" allowfullscreen></iframe>
      </div>
      <br>
      <a ng-href="/movies">I'm tired of this, lets watch movie trailers ... </a>
    </div>
  </div>
</div>
```

but we also need to add the youtube api call into the moviecontroller
But we dont need all the movies on this page. We only need the one specific movie
But how do we know which movie we need to load

Use the Angular's RouteParams service.
It would like this by the end
```
angular.module("app.controllers").controller "MovieController", [
  "$scope"
  "$routeParams"
  "$sce"
  "YouTube"
  ($scope, $routeParams, $sce, YouTube) ->
    console.log $routeParams
    YouTube.getTop25Movies().then (result) ->
      movies = result
      console.log result
      $scope.movie = _.find(movies, (v) ->
        v.youtubeId is $routeParams.movie_id
      )

      $scope.movie.youtubeUrl = $sce.trustAsResourceUrl("http://www.youtube.com/embed/" + $scope.movie.youtubeId + "?rel=0")

]

```

### Authorization with Angular :3

First lets add devise_token_auth to our Gemfile!
```
gem 'devise_token_auth'
gem 'omniauth', '~> 1.2.2'

then
```
$ bundle install
$ rails g devise_token_auth:install
$ rake db:migrate
```
bower init
then Create a bower.json file


```
bower install --save ng-token-auth
```
You should have a bower_components folder
and you want to get the angular-cookie folder
and ng-token-auth folder and put it inside your
vendor folder.

Once you have that you must require it inside your
application.js
looking like
```
//= require jquery
//= require jquery_ujs
//= require angular
//= require angular-route
//= require angularjs/rails/resource
//= require angular-cookie/angular-cookie
//= require ng-token-auth/dist/ng-token-auth
//= require app/app
//= require ui-bootstrap-tpls-0.12.0.min
//= require bootstrap.min
//= require_tree .

```

Then inside your app.js file
you need to add the module it'll look like htis

```
app = angular.module("app", [
  "rails"
  "app.controllers"
  "app.factories"
  "ngRoute"
  'ipCookie'
  "ng-token-auth"
])


```
Create a user_sessions folder inside your templates folder
and crate a new.html file inside that folder.

```
<div class="alert alert-danger" role="alert" ng-show="error"> {{error}}
</div>

<div class="container">
  <div class="row">
    <div class="col-md-6">
      <form ng-submit="submitLogin(loginForm)" role="form" ng-init="loginForm = {}">
        <div class="form-group">
          <label for="email">Email</label>
          <input type="email" name="email" id="email" ng-model="loginForm.email" required="required" class="form-control" autofocus>
        </div>
        <div class="form-group">
          <label for="password">Password</label>
          <input type="password" name="password" id="password" ng-model="loginForm.password" required="required" class="form-control">
        </div>

        <button type="submit" class="btn btn-primary btn-lg">Sign in</button>
      </form>
    </div>
  </div>
</div>



```

Now we need to route to the login page.
It will live at /sign_in and point at UserSessionsCtrl

```
app.config ($routeProvider, $locationProvider) ->
  $locationProvider.html5Mode
    enabled: true
    requireBase: false

  $routeProvider.when("/",
    templateUrl: "/templates/main.html"
    controller: "IndexCtrl"
  ).when("/sign_in",
    templateUrl: "/templates/user_sessions/new.html"
    controller: "UserSessionsCtrl"
  ).when("/sign_up",
    templateUrl: "/templates/user_registrations/new.html"
    controller: "UserRegistrationsCtrl"
  ).when("/home",
    templateUrl: "/templates/home.html"
    controller: "HomeCtrl"
    resolve:
      auth:[
        "$auth"
        ($auth) ->
          $auth.validateUser()

        ]
  ).otherwise redirectTo: "/"
```

Now inside your routes folder make it look like this
```
Rails.application.routes.draw do
  scope '/api' do
    mount_devise_token_auth_for 'User', at: '/auth'
  end

  root to: 'main#index'
  match '*any' => "main#index", :via => [:get, :post]

end
```

and inside yoru application_controller make it look like this
```
class ApplicationController < ActionController::Base
  include DeviseTokenAuth::Concerns::SetUserByToken
  include ActionController::MimeResponds
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
end

```

now inside your user.rb file make it look like this
```
class User < ActiveRecord::Base
  include DeviseTokenAuth::Concerns::User

  before_save -> do
    self.uid = SecureRandom.uuid
    skip_confirmation!
  end

end
```
Create your home page.
home.html!
```
<h1>Home</h1>
<p>This is the home page</p>
```

add this to your new.js controller
```
$scope.$on "auth:login-error", (ev, reason) ->
      $scope.error = reason.errors[0]
``
and now in your log in page add this~
```
<div class="alert alert-danger" role="alert" ng-show="error"> {{error}}
</div>
```

Time for the Registration

lets make a user_registrations folder
inside your templates folder
and then a new.html file
```
<div class="alert alert-danger" role="alert" ng-show="error">
  {{error}}
</div>

<div class="container">
  <div class="row">
    <div class="col-md-6">
      <form ng-submit="handleRegBtnClick()" role="form" ng-init="registrationForm = {}">
        <div class="form-group">
          <label for="email">Email</label>
          <input type="email" name="email" id="email" ng-model="registrationForm.email" required="required" class="form-control" autofocus>
        </div>
        <div class="form-group">
          <label for="password">Password</label> <input type="password" name="password" id="password" ng-model="registrationForm.password" required="required" class="form-control">
        </div>
        <div class="form-group">
          <label for="password_confirmation">Password confirmation</label>
          <input type="password" name="password_confirmation" id="password_confirmation" ng-model="registrationForm.password_confirmation" required="required" class="form-control">
        </div>

        <button type="submit" class="btn btn-primary btn-lg">Register</button>

      </form>
    </div>
  </div>
</div>

```

Time to make the controlelr for registration
createa signup.js.coffee file
```
angular.module("app.controllers").controller "UserRegistrationsCtrl", [
  "$scope"
  "$location"
  "$auth"
  ($scope, $location, $auth) ->
    $scope.$on "auth:registration-email-error", (ev, reason) ->
      $scope.error = reason.errors[0]
      return

    $scope.handleRegBtnClick = ->
      $auth.submitRegistration($scope.registrationForm).then ->
        $auth.submitLogin
          email: $scope.registrationForm.email
          password: $scope.registrationForm.password

]

```
lets change our gemfile to this!

gem 'devise_token_auth', git: 'https://github.com/jasonswett/devise_token_auth.git'


bunle install.


IN THE END THIS IS AN EXAMPLE OF WHAT YOUR FILES SHOULD LOOK LIKE.
GEMFILE
```
source 'https://rubygems.org'


# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '4.1.7'
# Use postgresql as the database for Active Record
gem 'pg'
gem 'pry-rails'
gem 'angular-gem'
gem 'angularjs-rails-resource', '~> 1.1.1'
gem 'devise_token_auth', git: 'https://github.com/jasonswett/devise_token_auth.git'
gem 'omniauth', '~> 1.2.2'
# Use SCSS for stylesheets
gem 'sass-rails', '~> 4.0.3'
# Use Uglifier as compressor for JavaScript assets
gem 'uglifier', '>= 1.3.0'
# Use CoffeeScript for .js.coffee assets and views
gem 'coffee-rails', '~> 4.0.0'
# See https://github.com/sstephenson/execjs#readme for more supported runtimes
# gem 'therubyracer',  platforms: :ruby

# Use jquery as the JavaScript library
gem 'jquery-rails'
# Turbolinks makes following links in your web application faster. Read more: https://github.com/rails/turbolinks
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.0'
# bundle exec rake doc:rails generates the API under doc/api.
gem 'sdoc', '~> 0.4.0',          group: :doc

# Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
gem 'spring',        group: :development

# Use ActiveModel has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use unicorn as the app server
# gem 'unicorn'

# Use Capistrano for deployment
# gem 'capistrano-rails', group: :development

# Use debugger
# gem 'debugger', group: [:development, :test]

```
applicaiton.js
```
// This is a manifest file that'll be compiled into application.js, which will include all the files
// listed below.
//
// Any JavaScript/Coffee file within this directory, lib/assets/javascripts, vendor/assets/javascripts,
// or vendor/assets/javascripts of plugins, if any, can be referenced here using a relative path.
//
// It's not advisable to add code directly here, but if you do, it'll appear at the bottom of the
// compiled file.
//
// Read Sprockets README (https://github.com/sstephenson/sprockets#sprockets-directives) for details
// about supported directives.
//
//= require jquery
//= require jquery_ujs
//= require angular
//= require angular-route
//= require angularjs/rails/resource
//= require angular-cookie/angular-cookie
//= require ng-token-auth/dist/ng-token-auth
//= require app/app
//= require ui-bootstrap-tpls-0.12.0.min
//= require bootstrap.min
//= require_tree .
```

application.css
```
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, vendor/assets/stylesheets,
 * or vendor/assets/stylesheets of plugins, if any, can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the bottom of the
 * compiled file so the styles you add here take precedence over styles defined in any styles
 * defined in the other CSS/SCSS files in this directory. It is generally better to create a new
 * file per style scope.
 *
 *= require_tree .
 *= require_self
 *= require bootstrap-3.2.0.min
 */
```











