# Good night

## Description
An application that lets you record your sleeping time and share it with you friends.
[API Documentation](API_DOC.md)

## User Stories (How a user uses Good night app)
* As a user I can login by inputing my name
* As a user I can click `Clock in` button to record the time when I sleep.
* As a user I can click `Clock out` button to record the time when I wake up.
* As a user I can see a list of users by name and id.
* As a user I can click `Follow` on the user that I want to follow.
* As a user I can see all user that I'm following.
* As a user I can see all user that follows me (my followers).
* As a user I can see my friend's (user I follow) sleep records in past week. [link](API_DOC.md#get-usersid)
* As a user I can see my friend's sleep records past week order by sleep_time [link](API_DOC.md#get-usersidlast_week_order_sleep_time)

## API Endpoints
* [/auth (GET)](API_DOC.md#get-auth)
* [/sleeps/clockin (GET)](API_DOC.md#get-sleepsclockin)
* [/sleeps/:id/clockout (GET)](API_DOC.md#get-sleepsidclockout)
* [/users (GET)](API_DOC.md#get-users)
* [/users/:user_id (GET)](API_DOC.md#get-usersid)
* [/users/:id/last_week_order_sleep_time (GET)](API_DOC.md#get-usersidlast_week_order_sleep_time)
* [/users/:user_id/follows (POST)](API_DOC.md#post-usersidfollows)
* [/users/:user_id/follows (GET)](API_DOC.md#get-usersidfollows)
* [/users/:user_id/followers (GET)](API_DOC.md#get-usersidfollowers)
* [/follows/:follow_id (DELETE)](API_DOC.md#delete-followsid)

## Database
* users :id, :name, :auth
* sleeps :id, :user_id, :date, :clockin_at, :clockout_at
* follows :id, :from_user_id, :to_user_id

## Sample of Frontend screen usage of API endpoint
* login screen:
    - Input name field and press enter
        + call GET /auth?name=Trung
            + save current_user: [id, name, auth] from response to localstorage
            + Go to current_user dashboard screen
* My dashboard screen
    - Display
        + GET /users/:id with id as param, auth as header from localstorage
* User list screen
    - Display
        + GET /users
            + each user's followed_by_me_id is null `follow` button else `unfollow`
    - Click follow button on each users
        + POST /users/:id/follows with id from corresponding user and header from localstorage
        + save follow_id from response to localstorage
    - Click unfollow button on each users 
        + DELETE /follows/:id with followed_by_me_id from corresponding

* My Follows screen
    - Display
        + GET /users/:user_id/follows with id, auth from localstorage
    - Click on each follows
        + GET /users/:id with id from corresponding follow.to_user.id
        + Go to Friend screen

* Friend screen
    - Display
        + GET /users/:id with id from corresponding follow.to_user.id
            + show `last_week_sleeps` with graph x-axis: date, y-axis: sleep_time
        + GET /users/:id/last_week_order_sleep_time with id from corresponding follow.to_user.id
            + show list order by sleep_time

* Clockin screen
    - Display a "Clockin" button
    - Click on button
        + POST /sleeps/clockin
        + save current_clockin_time: [id, checkin_at] from response to localstorage
        + "Clockin" button change to "Clockout" button
    - Click on "Clockout" button
        + POST /sleeps/:id/clockout from localstorage (saved above)

## Setup
### Docker way
* Run docker-composer to setup db & api services
```
docker-composer up
```

### Normal way
* Check ruby version `2.4.1` & rails version `5.2.0`
* have db postgre install with version `10.3`
    - ~or use db sqlite by using `database.sqlite.yml`~
* Run commands
```
bundle install
rails server -p 3000
```

## Development Process Log
* New rails app with only api & no bundle install
```
rails new good-night-api --api-only --skip-bundle
```

* Add docker-composer & dockerfiles
    - db usage change to postgres

* Add User Stories & API endpoints from specfication

* Add Rspec & TDD related gems

* Add rubocop (cuz i like coding standards)

* Add model, migration, rspec-model for [:users, :sleeps, :follows]

* Add controller, rspec-request for :users

* Add authentication for GET/users, GET /users/:id
    - add seed_fu gem and Seed 7 users for authentication testing.
    - add simple `authenticate!` with no encyption. (Could implemnt JWT later)
    - add rspec's `ShareExampleHelper`, `Request::AuthenticationHelper`
    - add rspec test cases with error message

* Add login with GET /auth
    - add rspec 2 error cases: no parameter name, user doesnt exists

* Add GET /sleeps/clockin & GET /sleeps/clockout
    - add rspec error cases: already checkin or checkout

* Add POST&GET /users/:id/follows & GET /users/:id/follower
    - add rspec

* Add DELETE /follows/:id
    - add rspec 2 cases: follow_id not found, not allowed operation

* Add Postgre gem and connection

* Add `last_week_sleeps` to GET /users/:id
    - add fixture 02_sleeps data.
    - group by for correct database select.

* Improve `last_week_sleeps`
    - if clockin_at, clockout_at is not recorded return sleep_time to not show.
    - if a date is not recorded it will still display. Always show 7 days.

* Add GET /users/:id/last_week_order_sleep_time
    - use sql `age` to calculate sleep_time
    - remove group by since its not nessary
    - order by sleep_time straight from sql
    - rspec for order check

* Change POST /sleeps/clockin & POST /sleeps/:id/clockout
    - clockin will input clockin time of today
    - clockout with sleep_id to be more accurate

* Add sample frontend screen usage

* Add left_join_followed_by_me to GET /users
    - for giving follow_id for `unfollow` feature

## Todo
- Add time_zone to user.
- better clockin clockout responses
- rspec for last_week_sleeps in GET /users
- calculate sleep_time and save to `sleeps.sleep_time`
- unsolved cases:
    + if user sleep at 1am the next day, he will skip one day sleep.
    + if user clockin and only clockout 2 days after.


