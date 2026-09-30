# Commands to Copy
  This document is meant to make copying commands easier for workshop participants.

## Initialization Steps
1. Build the docker instance
   ```
   docker build -t rails2hanami .
   ```

1. Run the docker instance
   ```
   docker run -it --name rails2hanami --publish 3001:3000 --publish 2301:2300 --volume .:/usr/src/app rails2hanami
   ```
 
   1. If your container was running and then stopped restart the container by running
      ```
      docker start rails2hanami
      ```

1. Setup the databases
   ```
   docker exec -it rails2hanami bundle exec rails db:migrate
   ```

## Run the Rails tests

   ```
   docker exec -it rails2hanami bundle exec rspec
   ```

## Look at the Rails application
  **Note** we are running Rails on 3001 to avoid anything you may already have running
   http://localhost:3001

## Workshop steps

### Initial Application Setup

1. Generate the app
   ```
   docker exec -w /usr/src/app -it rails2hanami bash
   ```

   ```
   hanami new bookshelf
   ls bookshelf
   exit
   ```

1. Setup and run the new Hanami application
   install the dependencies
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   ```
   
   run the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

1. Look at the [Hanami application](http://localhost:2301)

1. Seed the Rails database and copy it over to Hanami
   ```
   docker exec -it rails2hanami bundle exec rails db:seed
   docker exec -w /usr/src/app/bookshelf -it rails2hanami cp ../rails_bookshelf/storage/development.sqlite3 db/
   docker exec -w /usr/src/app/bookshelf -it rails2hanami cp ../rails_bookshelf/storage/test.sqlite3 db/
   ```

1. In **bookshelf/.env** replace all with the following
   ```
   DATABASE_URL=sqlite://db/development.sqlite3
   ```

1. Generate the Hanami relation and repository
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate relation books
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate repo book
   ```

1. Examples of the syntax for Hanami can be found in the docs and specifically the web app tutorial https://hanakai.org/learn/hanami/v3.0/getting-started/building-a-web-app

1. Add The book interface methods to **bookshelf/app/repos/book_repo.rb** inside the Class 
   ```
   def all = books.to_a

   def create(attributes)
     attributes[:created_at] = Time.now
     attributes[:updated_at] = Time.now
     books.changeset(:create, attributes).commit
   end

   def count = books.count

   def delete(id)
     books.by_pk(id).changeset(:delete).commit
   end

   def get(id) = books.by_pk(id).one!

   def last = books.last

   def update(id, attributes)
     attributes[:updated_at] = Time.now
     books.by_pk(id).changeset(:update, attributes).commit
   end
   ```

1. View the Rails database in your Hanami application
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami console
   ```

   ```
   puts Bookshelf::Repos::BookRepo.new.all.map(&:inspect)
   exit
   ```

#### Working Database Connection

1. check out the completed code branch
   ```
   mv bookshelf bookshelf_db_connect
   git reset --hard origin/db-connect
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami run 'puts Bookshelf::Repos::BookRepo.new.all.map(&:inspect)'
   ```

### Hanami Routes and Actions

1. In **bookshelf/config/routes.rb** add the following line to configure all routes for our book
   ```
   resources :books
   ```

1. View the routes created by the resource
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami routes
   ```

1. Generate the actions we just defined routes for utilizing the Hanami command line generators 
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.show --skip-route --skip-tests
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.index --skip-route --skip-tests
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.new --skip-route --skip-tests
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.create --skip-route --skip-tests
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.edit --skip-route --skip-tests
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.update --skip-route --skip-tests
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate action books.destroy --skip-route --skip-tests --skip-view
   echo "**** Created Actions ****"
   ls bookshelf/app/actions/books
   echo "**** Created Views ****"
   ls bookshelf/app/views/books
   echo "**** Created Templates ****"
   ls bookshelf/app/templates/books
   ```

#### Working Routes and Actions

1. check out the completed code branch
   ```
   mv bookshelf bookshelf_routes
   git reset --hard origin/routes
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami routes
   ```

### Rails Views and Specs

1. Copy over the system specs and the Rails views
   ```
   docker exec -it rails2hanami cp -r spec/system ../bookshelf/spec/
   docker exec -it rails2hanami cp -r app/views/books ../bookshelf/app/templates/
   ```

1. run all the tests
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec
   ```
1. Replace in bookshelf/spec
   ```
   require "rails_helper"
   ```
   With
   ```
   require "spec_helper"
   ```

1. Replace in bookshelf/spec
   ```
   Book.create(
   ```
   With
   ```
   Bookshelf::Repos::BookRepo.new.create(
   ```

   **Technically we could stop here to get to our next failing test, but for time's sake we will fix up the syntax in all the specs now...**

1. Replace in bookshelf/spec (fixes test for delete exercise)
   ```
   Book.last
   ```
   With
   ```
   Bookshelf::Repos::BookRepo.new.last
   ```

1. Replace in bookshelf/spec ( Hanami does not have the change helpers. fixes test for delete exercise)
   ```
   change(Book, :count)
   ```
   With
   ```
   change { Bookshelf::Repos::BookRepo.new.count }
   ```

1. Replace in bookshelf/spec (reload no longer works as objects don't change in functional programming. Fixes test for update exercise)
   ```
   book.reload
   ```
   With
   ```
   book = Bookshelf::Repos::BookRepo.new.get(book.id)
   ```

1. Replace in bookshelf/spec (Hanami form helpers create ids with `-` instead of `_`. fixes tests for update exercise)
   ```
   book_title
   ```
   With
   ```
   book-title
   ```

1. Replace in bookshelf/spec (fixes tests for update exercise)
   ```
   book_author
   ```
   With
   ```
   book-author
   ```

1. Replace in bookshelf/ (fixes remaining @book references)
   ```
   @book
   ```
   With
   ```
   book
   ```

1. We no longer have syntax errors in our specs...  They are now running and telling us our code is not yet working!

#### Working Specs

1. check out the completed code branch
   ```
   mv bookshelf bookshelf_specs
   git reset --hard origin/running-specs
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec
   ```

### Setting Up Flash Notices
1. Replace in bookshelf/app/templates/books/
   ```
   notice
   ```
   With 
   ```
   flash[:notice]
   ```

1. Allow cookies in **bookshelf/config/app.rb**
   ```
   config.actions.sessions = :cookie, { key: "bookshelf.session", secret: settings.session_secret, expire_after: 60*60*24*365 }
   ```

1. Add the secret in **bookshelf/config/settings.rb**
   ```
   setting :session_secret, constructor: Types::String, default: "____local_development_secret_only____local_development_secret_only___"
   ```


#### Working Flash Notices

1. check out the completed code branch
   ```
   mv bookshelf bookshelf_flash
   git reset --hard origin/flash
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec
   ```

### Book Show

1. Run the Show spec (keep running this until it passes)
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_show_spec.rb
   ```

1. Add to **bookshelf/app/views/books/show.rb**
   ```
   include Deps["repos.book_repo"]

   expose :book do |id:|
     book_repo.get(id)
   end
   ```

1. Replace in bookshelf/app/templates/books/show.html.erb
   ```
   <%= render book %>
   ```
   With 
   ```
   <%= render "book", book: book %>
   ```

1. Create a Book struct
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami generate struct book
   ```

1. Add to **bookshelf/app/structs/book.rb**
   ```
   def dom_id
     "book_#{id}"
   end
   ```

1. Replace in bookshelf/app/templates/books/_book.html.erb
   ```
   <%= dom_id book %>
   ```
   With
   ```
   <%= book.dom_id %> 
   ```

1. Replace in bookshelf/app/templates/books/show.html.erb
   ```
   edit_book_path(book)
   ```
   With 
   ```
   routes.path(:edit_book, id: book.id)
   ```

1. Replace in bookshelf/app/templates/books (will replace in 3 templates)
   ```
   books_path
   ```
   With 
   ```
   routes.path(:books)
   ```

1. Replace in bookshelf/app/templates/books/show.html.erb
   ```
   <%= button_to "Destroy this book", book, method: :delete %>
   ```
   With
   ```
   <%= form_for :book, routes.path(:book, id: book.id), method: :delete do |f| %>
     <%= f.submit "Destroy this book" %>
   <% end %>
   ```
1. Restart the Hanami server by going to the terminal and hitting `ctrl-c` and then rerunning the Hanami command
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

1. Look at the [show page in the application](http://localhost:2301/books/2)

#### Working Show Page

1. Stop the Hanami server by going to the terminal and hitting `ctrl-c`
1. check out the completed code branch
   ```
   mv bookshelf bookshelf_my_show
   git reset --hard origin/book-show
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_show_spec.rb
   ```
1. Start the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

### Books Index

1. run the Index spec (keep running this until it passes)
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_index_spec.rb
   ```

 1. Add to **bookshelf/app/views/books/index.rb**
    ```
    include Deps["repos.book_repo"]

    expose :books do
      book_repo.books.to_a
    end
    ```

1. Replace in bookshelf/app/templates/books/index.html.erb
   ```
   new_book_path
   ```
   With 
   ```
   routes.path(:new_book)
   ```

1. Look at the [index page in the application](http://localhost:2301/books)
   **NOTE** the show link does not work

1. Replace in bookshelf/app/templates/books/index.html.erb
   ```
   <%= link_to "Show this book", book %>
   ```
   With
   ```
   <%= link_to "Show this book", routes.path(:book, id: book.id) %>
   ```

1. Look at the [index page in the application](http://localhost:2301/books)

#### Working Index

1. Stop the Hanami server by going to the terminal and hitting `ctrl-c`
1. check out the completed code branch
   ```
   mv bookshelf bookshelf_my_index
   git reset --hard origin/book-index
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_index_spec.rb
   ```
1. Start the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

### Application Layout

1. Copy over the Rails layout
   ```
   docker exec -it rails2hanami cp app/views/layouts/application.html.erb ../bookshelf/app/templates/layouts/app.html.erb
   docker exec -it rails2hanami cp app/assets/stylesheets/application.css ../bookshelf/app/assets/css/app.css
   ```
1. Hanami does Content Security by default so the CSP tag is not needed. Replace the following in bookshelf/app/templates/layouts/app.html.erb
   ```
   <%= csrf_meta_tags %>
       <%= csp_meta_tag %>
   ```
   With
   ```
   <%= csrf_meta_tags&.html_safe %>
   ```

1. Replace the following in bookshelf/app/templates/layouts/app.html.erb
   ```
   <%= stylesheet_link_tag :app, "data-turbo-track": "reload" %>
       <%= javascript_importmap_tags %>
   ```
   With
   ```
   <%= stylesheet_tag "app", "data-turbo-track": "reload" %>
   ```

1. We will utilize the Hanami icon so we can tell our tabs apart. Replace the following in bookshelf/app/templates/layouts/app.html.erb
   ```
   <link rel="icon" href="/icon.png" type="image/png">
       <link rel="icon" href="/icon.svg" type="image/svg+xml">
       <link rel="apple-touch-icon" href="/icon.png">
   ```
   With
   ```
   <%= favicon_tag %>
   ```

1. Look at the [index page in the application](http://localhost:2301/books)

   If there server has an error
   
   1. Compile the assets
      ```
      docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
      ```
   1. Restart the Hanami server by going to the terminal and hitting `ctrl-c` and then rerunning the hanami command
      ```
      docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
      ```

#### Working Layout

1. Stop the Hanami server by going to the terminal and hitting `ctrl-c`
1. check out the completed code branch
   ```
   mv bookshelf bookshelf_my_layout
   git reset --hard origin/app-layout
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_index_spec.rb
   ```
1. Start the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

### Book New

1. Run the Create test (keep running it until it passes)
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_create_spec.rb
   ```

1. Add to **bookshelf/app/views/books/new.rb**
   ```
   include Deps["repos.book_repo"]

   expose :form_submit, default: "Create Book"
   expose :book do |context:|
     context.request.params[:book]
   end
   ```

1. Replace the following in bookshelf/app/templates/books/_form.html.erb
   ```
   <%= form_with(model: book) do |form| %>
   ```
   With
   ```
    <%= form_for :book, routes.path(:books), method: "POST" do |form| %>
   ```

1. Replace in bookshelf/app/templates/books/_form.html.erb
   ```
   <%= form.submit %>
   ```
   With
   ```
   <%= form.submit form_submit %>
   ```

1. Replace in bookshelf/app/templates/books/new.html.erb
   ```
   <%= render "form", book: book %>
   ```
   With
   ```
   <%= render "form", book: book, form_submit: form_submit %>
   ```

1. Copy the Rails logic into **bookshelf/app/actions/books/create.rb** handle method from rails_bookshelf/app/controllers/books_controller.rb#create
   Handle method will look like:
   ```
   def handle(request, response)
     @book = Book.new(book_params)
     if @book.save
       redirect_to @book, notice: "Book was successfully created."
     else
       render :new, status: :unprocessable_entity
     end
   end
   ```

1. Add in **bookshelf/app/actions/books/create.rb** inside `class Create < Bookshelf::Action`
   ```
   include Deps["repos.book_repo"]

   params do
      required(:book).hash do
         required(:title).filled(:string)
         required(:author).filled(:string)
      end
   end
   ```

1. Change the handle code to validate parameters and utilize the book repo
   ```
   def handle(request, response)
     if request.params.valid?
        book = book_repo.create(request.params[:book])
        response.flash[:notice] = "Book was successfully created"
        response.redirect_to routes.path(:book, id: book[:id])
     else
       response.flash.now[:alert] = "Could not create book"
     end
   end
   ```

1. Restart the Hanami server by going to the terminal and hitting `ctrl-c` and then rerunning the Hanami command
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

1. Look at the [index page in the application](http://localhost:2301/books) and make a new book

#### Working New

1. Stop the Hanami server by going to the terminal and hitting `ctrl-c`
1. check out the completed code branch
   ```
   mv bookshelf bookshelf_my_new
   git reset --hard origin/book-new
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_create_spec.rb
   ```
1. Start the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

### Exercise 1

1. Run the delete test (keep running it until it passes)
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_delete_spec.rb
   ```

#### Exercise 1 Hints
1. There is an example in the [Hanami docs for deleting a Book](https://hanakai.org/learn/hanami/v3.0/getting-started/building-a-web-app#deleting-a-book)

1. All elements are already visible to the user.  The link for deletion is on the show page

1. Code to delete a book has already been added to the repository.  You could call it with some code like...
   ```
   include Deps["repos.book_repo"]
   ...
      # in handle method
      result = book_repo.delete(request.params[:id])
   ```

1. you can require an integer parameter via
   ```
   params do
      required(:id).filled(:integer)
   end
   ```

1. You can redirect to the books index with the following code
   ```
   response.redirect_to routes.path(:books)
   ```

#### Working Delete

1. Stop the Hanami server by going to the terminal and hitting `ctrl-c`
1. check out the completed code branch
   ```
   mv bookshelf bookshelf_my_delete
   git reset --hard origin/book-delete
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_delete_spec.rb
   ```
1. Start the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```

### Exercise 2

1. Run the edit test (keep running it until it passes)
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_edit_spec.rb
   ```

#### Exercise 2 Hints

1. There is an example in the [Hanami docs for updating a Book](https://hanakai.org/learn/hanami/v3.0/getting-started/building-a-web-app#updating-a-book)

1. The new and edit templates display the same form template.  You can expose the book submit wording, form method and form path in the update view **views/edit.rb** with code like
   ```
   include Deps["repos.book_repo"]

   expose :book do |context:, id:|
      book_repo.get(id)
   end

   expose :form_submit, default: "Update Book"
   expose :form_method, default: "PATCH"
   expose :form_path do |context:, id:|
      context.routes.path(:book, id: id)
   end
   ```
   You will also need to expose the form method and path in the new view **views/new.rb**
   ```
   expose :form_method, default: "POST"
   expose :form_path do |context:|
      context.routes.path(:books)
   end
   ```

1. You can pass the exposed vars to the form partial in new.html.erb and edit.html.erb
   ```
   <%= render "form", book: book, form_submit: form_submit, form_path: form_path, form_method: form_method %>
   ```

1. You can utilize all the form vars with code like
   ```
   <%= form_for :book, form_path, method: form_method do |form| %>
   ```

1. The create and update actions take almost all the same parameters. You may want to make a copy of create to start with for update.

1. the update action will need both the id and the parameters from the create
   ```
   params do
      required(:id).filled(:integer)
      ...
   end 
   ```

1. you can update the book by calling something like
   ```
   book = book_repo.update(request.params[:id], request.params[:book])
   ```
#### Working Edit

1. Stop the Hanami server by going to the terminal and hitting `ctrl-c`
1. check out the completed code branch
   ```
   mv bookshelf bookshelf_my_edit
   git reset --hard origin/book-edit
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami npm install
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami assets compile
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec rspec spec/system/book_edit_spec.rb
   ```
1. Start the hanami dev server
   ```
   docker exec -w /usr/src/app/bookshelf -it rails2hanami bundle exec hanami dev
   ```
