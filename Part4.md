# Part 4: Deploy to the cloud, including the production database

In the Sinatra Hangperson assignment, you already learned how to
deploy a Sinatra app to Heroku.
Deploying a Rails app is very similar, but a few extra steps are
required since most Rails apps use databases.

All apps on Heroku use the PostgreSQL database.  For Ruby/Rails apps
to do so, they must include the `pg` gem.  However, we don't want to
use this gem while developing, since we're using SQLite for that.
Gemfiles let you specify that certain gems should only be used in
certain environments.  Rails apps examine the environment variable
`RAILS_ENV` to determine which environment they're running in, to make
decisions such as which database to use (`config/database.yml`) and
which gems to use.
Heroku
sets this variable to `production` at deploy time; running tests sets
it to `test`; while you're running your app interactively, it's set to
`development`. 

To specify production-specific gems, you must make **two** edits to
your Gemfile.  First, add this: 

```
group :production do
  gem 'pg', '~> 0.21' # for Heroku deployment
  gem 'rails_12factor'
end
```

(If there is already a `group :production` in your Gemfile, just add
those lines to it.)  `rails_12factor` is a gem that has some
additional best-practices for deployment that Heroku recommends
(Rails 5 and later apps don't need it).

Second, find the line that specifies the `sqlite3` gem, and tell the
Gemfile that that gem should **not** be used in production, by moving
that line into its own group like so:

```
group :development, :test do
  gem 'sqlite3', '~> 1.3.0'
end
```

This second step is necessary because Heroku is set up in such a way
that the `sqlite3` gem simply won't work, so we have to make sure it
is _only_ loaded in development and test environments but _not_ production.

As always when you modify your Gemfile, re-run `bundle install` and
commit the modified `Gemfile` and `Gemfile.lock`.
Note that when you run Bundler, it will still compute dependencies and
versions for gems in the `production` group, but it won't install
them.  Heroku will use `Gemfile.lock` to install the matching versions
of the gems when you deploy.

**Don't we have to modify `config/database.yml` as well?**  You'd
think so, but the way Heroku works, it actually _ignores_
`database.yml` and forces Rails apps to use Postgres.  So modifying
the `production:` section of `database.yml` won't have any effect on Heroku.

The first time you
use the Heroku command line tool, you will have to authenticate yourself using your Heroku
username and password, by saying `heroku login --interactive`.

At this point, having committed all changes locally, you should be ready to push your
repository to Heroku for deployment.

Finally, you need to create a new app "container" on Heroku to which
you can deploy.  Since Heroku behaves like a Git repo that you push
to, the app "container" must somehow be associated with your
development repo.  The easiest way to do this is to use the 
Heroku command line tool from within the app's root directory; say `heroku
help create` to see how. The created app container will then be
associated with your app.  You can say `git remote` to show a list of
all remote Git repos associated with your app, among which `heroku`
should now appear.  You can say `git remote show heroku` to verify
that pushing to the `heroku` "repo" will deploy to a URL on Heroku
whose name matches the name you picked for your app.

OK, the moment of truth has arrived.  `git push heroku master` to push
the current head of your master branch to Heroku.  Heroku will try to
install all the Gems in your gemfile and fire up your app!  You should
be able to view your app at `https://your-app-name.herokuapp.com`!
Right?

Almost.  If you try this, you'll see a Heroku error.

<details>
<summary>
  <b>Self-check question:</b> Use the command <code>heroku logs</code> to see the last several lines of error
  log messages from Heroku.  What caused the error?
</summary>
<blockquote>
  As the error message <code>relation "movies" does not exist</code> tells us,
  there is no <code>movies</code> table on Heroku -- in fact there isn't even a  database.
</blockquote>
</details>

(It's worth making sure you understand the self-check question before
charging on to the fix!)

Creating the database locally required 2 steps: first we ran the
initial migration to create the `movies` table (schema), then we
seeded the database with some initial data.  We must do these 2 steps
on Heroku as well, by preceding each with `heroku run`, which just
runs the corresponding command in a shell on Heroku:

```
heroku run rake db:migrate
```
Now go verify that you can visit your app, but no movies are listed...
```
heroku run rake db:seed
```
...and now your app should work on Heroku just as it does locally,
with the seed data.

Voila -- you have created and deployed your first Rails app!


<details>
<summary>
What are the two steps you must take to have your app use a particular
Ruby gem? 
</summary>
<blockquote>
You must add a line to your `Gemfile` to add a gem and re-run `bundle install`.
</blockquote>
</details>

<details>
<summary>
The _first_ time you deploy a particular app on Heroku, what steps do
you need to take (assuming you have adjusted your `Gemfile` correctly
as described above)?
</summary>
<blockquote>
Create the app container on Heroku; push the app to Heroku; run the
initial migration(s) to create the database; and if appropriate, seed
the database with initial data.
</blockquote>
</details>


<details>
<summary>
When you make changes to your app code, including adding new
migrations, what must you do to _update_ the existing Heroku app?
(HINT: try making a simple change to the app, like changing something
in a view, and see if you can deduce the sequence of steps.)
</summary>
<blockquote>
Commit changes to Git, then <code>git push heroku master</code> to
redeploy.  If you created new migrations, you also need to 
<code>heroku run rake db:migrate</code> to apply them on the Heroku side.
</blockquote>
</details>



<div align="center">
<b><a href="Part3.md">&larr; Previous: Part 3</a></b>
</div>
