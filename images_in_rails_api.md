# Uploading and serving images with Rails API, PostgreSQL, and carrierwave

## Goal
In this case, I had an existing Rails API project that interfaces with a mobile app, and wanted to allow the user of that mobile app to upload a profile picture (and then, of course, see that profile picture later on).

## Client/Server interaction:
* Client requests => server to set the profile pic (The image data will be sent as in data-uri format, as base64 image data).
* The server will save the image to the DB, and return a URL (another endpoint on this same server) where the image can be found (in our case, the "profile_pic url".
* If the client requests the profile_pic url, the server will return that file (simply a stream of the image file, this is not JSON).

## Prereqs/Assumptions
* You have a rails app (I used rails 5 w/ API option)
* You have a `User` model
* You are using PostgreSQL as your DB
* (Kind of) you have RSpec. You definitely don't need it, but my test example uses it.

## The Good Stuff:
Here are the steps we will take!

Step 1: Migration
---
First, we want to actually add the `profile_pic` attribute to the `User` model, with the data type `oid` (we will be using [PG's large object data type](https://www.postgresql.org/docs/9.2/static/largeobjects.html)):
> NOTE: the column `profile_pic` has type `oid`, which is only a reference to the file, not the file itself. The file itself is stored in a different table, but for the purposes of this tutorial, this is not really important.

So, run this migration:

```bash
rails g migration add_profile_pic_to_users profile_pic:oid
rails db:migrate # rake db:migrate for older versions of rails
```

Step 2: Endpoint to upload the profile pic
---
We are going to create an endpoint that uploads the profile pic. In our case, `POST users/:id/set_profile_pic`, but it can easily be part of the typical `PUT users/:id` update method.

To do this, we are going to leverage two awesome gems:

* [carrierwave-base64](). This is built on [carrierwave](), but we don't need to include carrierwave directly.
* [carrierwave-postgresql](https://github.com/diogob/carrierwave-postgresql). This allows us to easily store our uploads in postgres using the large object storage.

So, add these to our Gemfile:

```ruby
# Gemfile
gem 'carrierwave-base64'
gem 'carrierwave-postgresql'
```

We also need to require some carrierwave things on app initialization, so add this in your preferred initialization spot (some put it in `environment.rb`, I added a new file: `config/initializers/carrierwave.rb`:

```ruby
# config/initializers/carrierwave.rb
require 'carrierwave'
require 'carrierwave/postgresql'
require 'carrierwave/orm/activerecord'
```
Then, create a file `app/uploaders/profile_pic_uploader.rb`, and add this to it:

```ruby
class ProfilePicUploader < CarrierWave::Uploader::Base
  storage :postgresql_lo
end
```
In our User model:

```ruby
mount_base64_uploader :profile_pic, ProfilePicUploader
```
in the users_controller:

```ruby
# app/controllers/users_controller.rb
...

def set_profile_pic
  if user.update(profile_pic_param)
    render json: user.reload
  else
    render json: user_errors, status: 400
  end
end

private

def profile_pic_param
  params.require(:user).permit(:profile_pic)
end

def user
  @user ||= User.find(params[:id])
end
...
```
config/routes.rb:

```ruby
# config/routes.rb

resources :users do
  member do
    post :set_profile_pic
  end
end
```

And finally, a test (note that I added a small .png file to `spec/support/member.png` for testing)

```ruby
# spec/controllers/users_controller_spec.rb

...
# (inside the describe UsersController block)
# Also, there is some setup that I don't reference (specific to the JSON
# response for this particular application), but this is not what this tutorial
# is about, and hopefully you can figure that part out.
...

  describe '#set_profile_pic' do
    let(:profile_pic_data) do
      image_file = Rails.root.join('spec', 'support', 'member.png')
      base64_image = Base64.strict_encode64(File.read(image_file))
      "data:image/png;base64,#{base64_image}"
    end

    before do
      params = {
        id: user.id,
        user: {
          profile_pic: profile_pic_data
        }
      }

      post :set_profile_pic, params: params
    end

    it 'sets the profile_pic of the user' do
      file = user.reload.profile_pic.file
      expect(file).not_to be_nil
      expect(user_response[:profile_pic][:url]).to eq(file.url)
    end
  end
```

Nice! That's it for that step. Almost there...

Step 3: Endpoint to get the image back from the server:
---

lkj adlf kj.  
adf asdf a
shoulda done better here...



---
![image in chrome](https://rpl.cat/uploads/v7mBdhyVqg9-8CRWnLREvQNTp5rkt6G9blTon__PB9c/public.png)

---

## Acknowledgements/Resource Links:
This post is mainly a consolidation of resources and other articles that I read in order to figure this stuff out:

* [This post](http://brewhouse.io/2016/05/20/documented-json-api-image-upload.html) by Paulo Ancheta helped immensly as a starter point for my understanding. This article is in many ways adapted from it.
* [carrierwave-base64](https://github.com/lebedev-yury/carrierwave-base64)
* [carrierwave-postgresql](https://github.com/diogob/carrierwave-postgresql)
* [postgresql\_lo\_streamer](http://diogob.github.io/postgresql_lo_streamer)