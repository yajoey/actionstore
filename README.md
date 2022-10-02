# ActionStore

ActionStore allows you to push data directly into Svelte stores from your Ruby on Rails backend. Here's an [introductory post](https://dev.to/buhrmi/actionstore-real-time-svelte-stores-for-rails-4jhg)

## Installation

### Backend

Add this line to your application's Gemfile:

```ruby
gem 'actionstore'
```

And then execute:

    $ bundle install

### Frontend

Install the package:

    $ yarn add @buhrmi/actionstore

## Usage

Below are some example usage scenarios. For more details, see the [API documentation](https://www.rubydoc.info/gems/actionstore).

### Subscribe to a database record

In a Svelte component:

```html
<script>
import {subscribe} from '@buhrmi/actionstore'

// you need the signed global id of the record you want to subscribe to
export let user_sgid

// Calling `subscribe` will set up an ActionCable subscription and return a 
// Svelte store which you can push to
const user = subscribe(user_sgid)
</script>

{#if $user}
  Hello {$user.name}
{/if}
```

Now you can populate the store from the backend:

```ruby
class User < ApplicationRecord
  has_actionstore

  def subscribed channel
    push_update name: 'Rich Harris'
  end
end
```

### Multiple stores

You can also push data into stores specified by an id

```js
import {subscribe,store} from '@buhrmi/actionstore'
export let user_sgid
const user = subscribe(user_sgid)

// Use the store() method to get an ActionStore by id
const messages = store('messages')

```

Now you can push into the "messages" store from the backend

```ruby
user.push_append_into 'messages', text: 'hello'
```

### Autopush changes

You can automatically sync record changes to the store.

```ruby
class User < ApplicationRecord
  has_actionstore
  
  # this will only push changed attributes
  after_update_commit -> { push_update saved_changes.transform_values(&:last) }
 end
```

### Perform actions

With ActionStore you can define actions directly on the model and call them from the frontend.

```js
const user = subscribe(user_sgid)
user.perform 'say_hello', 'thomas'
```

```ruby
class User
  def perform_say_hello channel, name
    puts "Hello #{name}"
  end
end
```

### Trigger events

```js
const user = subscribe(user_sgid)
user.on('show_alert', function(data) {
  window.alert(data)
})
```

```ruby
user.push_event 'show_alert', 'Boo!'
```



## API

ActionStore has just a couple of methods that cover a whole spectrum of stuff you can do.

### Frontend

The `@buhrmi/actionstore` package exports the following functions:

`subscribe(sgid, initial=null, storeId=sgid)` - Subscribe to the record with the specified global id

`store(storeId, initial=null)` - Get the store with the specified id

### Backend

Adding `has_actionstore` to your ActiveRecord model will create the following instance methods:

`push_append(data)` - Append data to an array in the default store

`push_update(data)` - Updats fields of an object in the default store

`push_append_into(store_name, data)` - Append data to an array in a specified store

`push_update_into(store_name, data)` - Update fields of an object in a specified store

`push_event(event_name, data)` - Trigger an event on the default store

`push_event_into(store_name, event_name, data)` - Trigger an event on a specified store

