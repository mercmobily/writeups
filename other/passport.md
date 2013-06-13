# PASSPORT WRITEUP


## TWO WAY AUTH

    passport.use('strategy-name', new SomethingStrategy({
      passReqToCallback: true,  // Always available
      specificParameter: 'something',

      callbackURL: "/auth/strategy-name/callback", // <=== THIS NEEDS TO MATCH THE URL IN app.get LATER
    },
  
    --function(req, login, password, done) {
    --function(req, accessToken, refreshToken, profile, done) {
      // Check that `user` is not false, match profile info against database, etc. and THEN:
      --done( err, false ); // ERROR
      --done( null, userVariable, remoteProfileObject ); // ALL GOOD
      --done( null, false, { message: 'Some problem'} ); // PROBLEM (with info)
    });

    // Common callback: this will redirect to the provider, and will do _all_ of the required
    // communication process to end up with 
    app.get('/auth/strategy-name', passport.authenticate('strategy-name')); 

    // THEN: 

    // Manual way (best one to understand the process)
    //
    app.get('/auth/strategy-name/callback', function( req, res, next) {
      passport.authenticate('strategy-name',  function( err, user, profile) {
        // Redirect/close window/return AJAX/whatever you want to happen after authentication
      });
    });

## Explanation (in simple English rather than code)

This is a 2-way authentication system. So, Passport needs you to set two callbacks. The first one is called without parameters, and it's the one that will redirect to the auth provider and will deal with communicating, receiving tokens back, etc.:

    app.get('/auth/strategy-name', passport.authenticate('strategy-name'));

Passport will do the initial communication with the provider, will likely obtain a secret token, and will then redirect to the  provider (e.g. Facebook).
The provider will then likely redirect to to the callback URL, which is why you need the SECOND callback URL:

    app.get('/auth/strategy-name/callback', function( req, res, next) {
      passport.authenticate('strategy-name',  function( err, user, profile) {
        // Redirect/close window/etc.
      })(req, res. next);
    });


The function passport.authenticate() will return a function that accepts req, res, next. Basically, it will take over the express callback completely with its own returned function. passport.authenticate() accepts the strategy name and an IMPORTANT callback.
passport.authenticate() will:

* Finalise communication with the third party provider, checking for tokens etc.
* Call the authentication callback function supplied to the strategy, passing it the parameters received by the provider
* Call the IMPORTANT callback passed to it, passing it `user` and `profile` (the result  of the details-checking function)


# Notes on authentication

Authentication is all great. However, it's normally only part of the story. Normally, developers need to include sign-up, account recovery, etc. These operations imply different ways to deal existing data in the database (signup will create a user, whereas login will match against an existing entry, etc.). Since datbase interaction happens at password.use() level, you will need to create a named strategy for each "type" of interaction you might want.
for example:

* manager: associate/delete association with a provider
* register: add a user
* signin: sign in (login)
* recover: send account recovery information to recover account

You will likely need four named strategy, with four different ways of handling the database. However, you will probably be bale to re-use the same function as a callback in passport.authenticate('strategy-name', callback ) as the parameters passed are standard (`user` and `profile`). 



