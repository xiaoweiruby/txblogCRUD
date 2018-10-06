现在我们开始学习 Ruby on Rails 的知识体系，现在主要是从知识体系叠加的角度来思考问题。


一、刚刚入门的孩子的学习方式：
# 1、基础 POST 的 CRUD 功能体系；
![CRUD 展示.gif](https://upload-images.jianshu.io/upload_images/7680238-80aecc22ae3370b1.gif?imageMogr2/auto-orient/strip)

https://github.com/shenzhoudance/txblogCRUD

# 2、用户 COMMENT 评论的体系；
![comment 展示.gif](https://upload-images.jianshu.io/upload_images/7680238-82e52ba12840fdca.gif?imageMogr2/auto-orient/strip)

# 3、页面 bulma 美化体系；
![bulma 展示.gif](https://upload-images.jianshu.io/upload_images/7680238-e568ea9e828f6606.gif?imageMogr2/auto-orient/strip)

# 4、用户 devise 的对接体系；
![devise 展示.gif](https://upload-images.jianshu.io/upload_images/7680238-2281f045f8adbd79.gif?imageMogr2/auto-orient/strip)

# 5、用户 heroku 上传体系
![heroku 上传3.gif](https://upload-images.jianshu.io/upload_images/7680238-b2147961e0436ed6.gif?imageMogr2/auto-orient/strip)

#第一部分：基本的功能体系；
git checkout -b posts
rails g controller posts

rails g model post title:string body:text

rake db:migrate

config/routes.rb

```
Rails.application.routes.draw do
  resources :posts
  root 'posts#index'
end
```
app/controllers/posts_controller.rb
```
class PostsController < ApplicationController
  def index
    @posts = Post.all.order('created_at DESC')
  end


  def new
    @post = Post.new
  end

  def create
    @post = Post.new(post_params)

      if @post.save
        redirect_to @post
      else
        render 'new'
      end
    end

  def show
   @post = Post.find(params[:id])
  end

  def edit
   @post = Post.find(params[:id])
  end

  def update
		@post = Post.find(params[:id])

		if @post.update(params[:post].permit(:title, :body))
			redirect_to @post
		else
			render 'edit'
		end
	end

  def destroy
     @post = Post.find(params[:id])
     @post.destroy

     redirect_to posts_path
  end



private
   def post_params
     params.require(:post).permit(:title, :body)
   end
end

```

touch app/views/posts/index.html.erb
touch app/views/posts/new.html.erb
touch app/views/posts/show.html.erb
touch app/views/posts/edit.html.erb
touch app/views/posts/-form.html.erb

app/views/posts/index.erb
```
<% @posts.each do |post| %>
  <h2 class="card"><%= link_to post.title, post %></h2>
  <p class="date"><%= post.created_at.strftime("%B, %d, %Y") %> </p>
<% end %>
<%= link_to "New Post", new_post_path, class:"button" %>
<%= link_to "Back", posts_path, class:"button" %>

```
app/views/posts/new.erb
```
<h1>New Post</h1>
<%= render 'form' %>
```
app/views/posts/show.erb
```
 <%= @post.title %>
<%= time_ago_in_words(@post.created_at) %> Age
 <%= @post.body %>

<%= link_to "New Post", new_post_path, class:"button" %>
<%= link_to "Back", posts_path, class:"button" %>
<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Delete', post_path(@post), method: :delete, data: { confirm: 'Are you sure?' } %>
```
app/views/posts/edit.erb
```
<h1>edit Post</h1>

<%= render 'form' %>
```
app/views/posts/-form.erb
```
<%= form_for @post do |f| %>
	<%= f.label :title %><br>
	<%= f.text_field :title %>
</div>
</div>
	<%= f.label :body %><br>
	<%= f.text_field :body %><br>
	<br>
	<%= f.submit %>
<% end %>
```
---

#第二部分：用户评论体系
git checkout -b comment
rails g model  comment name:string body:text post:references
rake db:migrate
rails g controller comments
touch app/views/comments/-comment.html.erb
touch app/views/comments/-form.html.erb
app/controllers/comments_controller.rb
```
class CommentsController < ApplicationController
	def create
		@post = Post.find(params[:post_id])
		@comment = @post.comments.create(params[:comment].permit(:name, :body))

		redirect_to post_path(@post)
	end

	def destroy
		@post = Post.find(params[:post_id])
		@comment = @post.comments.find(params[:id])
		@comment.destroy

		redirect_to post_path(@post)
	end
end

```
app/views/comments/-comment.html.erb
```
<%= comment.name %>
<%= comment.body %>
<%= time_ago_in_words(comment.created_at) %>

<%= link_to 'Delete', [comment.post, comment],
								  method: :delete,
								  class: "button",
								 	data: { confirm: 'Are you sure?' } %></p>
```
app/views/comments/-form.html.erb
```
<%= form_for([@post, @post.comments.build]) do |f| %>
	<%= f.label :name %><br>
	<%= f.text_field :name %><br>
	<br>
	<%= f.label :body %><br>
	<%= f.text_area :body %><br>
	<br>
	<%= f.submit %>
<% end %>

```
app/views/posts/show.html.erb
```
 <%= @post.title %>
</h1>

<p class="date"></p>
submitted<%= time_ago_in_words(@post.created_at) %> Age

<h1 class="body">
 <%= @post.body %>
</h1>

<div class="comments">
  <h2><%= @post.comments.count %></h2>
  <%= render @post.comments %>
</div>

<h3>add a comments</h3>
  <%= render "comments/form" %>

<%= link_to "New Post", new_post_path, class:"button" %>
<%= link_to "Back", posts_path, class:"button" %>
<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Delete', post_path(@post), method: :delete, data: { confirm: 'Are you sure?' } %>
```
对接 post 和 comments 的对应关系；
app/models/post.rb
```
class Post < ApplicationRecord
  has_many :comments
end

```
app/models/comment.rb
```
class Comment < ApplicationRecord
     belongs_to :post
end

```
config/routes.rb
```
Rails.application.routes.draw do
  resources :posts do
    resources :comments
  end
  root 'posts#index'
end

```
#第叁部分：页面美化体系
git checkout -b bulma
https://bulma.io/documentation/
gem 'bulma-rails', '~> 0.7.1'
gem 'simple_form', '~> 4.0', '>= 4.0.1'
bundle install

app/assets/stylesheets/application.scss
@import "bulma";

app/views/layouts/application.html.erb
```
<!DOCTYPE html>
<html>
  <head>
    <title>DemoBlog</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
  	<section class="hero is-primary is-medium">
	  <!-- Hero head: will stick at the top -->
	  <div class="hero-head">
	    <nav class="navbar">
	      <div class="container">
	        <div class="navbar-brand">
	          <%= link_to 'Demo Blog', root_path, class: "navbar-item" %>
	          <span class="navbar-burger burger" data-target="navbarMenuHeroA">
	            <span></span>
	            <span></span>
	            <span></span>
	          </span>
	        </div>
	        <div id="navbarMenuHeroA" class="navbar-menu">
	          <div class="navbar-end">
	            <%= link_to "Create New Post", new_post_path, class:"navbar-item" %>
	          </div>
	        </div>
	      </div>
	    </nav>
	  </div>

	  <!-- Hero content: will be in the middle -->
	  <div class="hero-body">
	    <div class="container has-text-centered">
	      <h1 class="title">
	        <%= yield :page_title %>
	      </h1>
	    </div>
	  </div>
	</section>
    <%= yield %>
  </body>
</html>

```
app/views/posts/-form.html.erb
```
<div class="section">
<%= simple_form_for @post do |f| %>
  <div class="field">
    <div class="control">
      <%= f.input :title, input_html: { class: 'input' }, wrapper: false, label_html: { class: 'label' } %>
    </div>
  </div>

  <div class="field">
    <div class="control">
      <%= f.input :body, input_html: { class: 'textarea' }, wrapper: false, label_html: { class: 'label' }  %>
    </div>
  </div>
  <%= f.button :submit, class: "button is-primary" %>
<% end %>
</div>

```
app/views/posts/edit.html.erb
```
<% content_for :page_title, "Edit Post" %>
<%= render 'form' %>

```
app/views/posts/index.html.erb
```
<% content_for :page_title,  "Index" %>

<div class="section">
	<div class="container">
		<% @posts.each do |post| %>
			<div class="card">
		  <div class="card-content">
		    <div class="media">
		      <div class="media-content">
		        <p class="title is-4"><%= link_to post.title, post  %></p>
		      </div>
		    </div>
		    <div class="body">
		     	<%= post.body %>
		    </div>
		    <div class="comment-count">
		    	<span class="tag is-rounded"><%= post.comments.count %> comments</span>
		    </div>
		  </div>
		</div>
		<% end %>
	</div>
</div>
```
app/views/posts/new.html.erb
```
<% content_for :page_title, "Create a new post" %>
<%= render 'form' %>

```
app/views/posts/show.html.erb
```
<% content_for :page_title, @post.title %>

<section class="section">
	<div class="container">
		<nav class="level">
		  <!-- Left side -->
		  <div class="level-left">
		    <p class="level-item">
          <div class="title">
            <%= @post.title %>
          </div>
		    </p>
		  </div>

		  <!-- Right side -->
		  <div class="level-right">
		  	<p class="level-item">
		    	<%= link_to "Edit", edit_post_path(@post), class:"button" %>
		  	</p>
		  	<p class="level-item">
				<%= link_to 'Delete', post_path(@post), method: :delete, class: "button is-danger", data: { confirm: 'Are you sure?'  } %>
		  </div>
		</nav>
		<hr/>

    <div class="body">
      <%= @post.body %>
    </div>


	</div>
</section>


<section class="section comments">
	<div class="container">
		<h2 class="subtitle is-5"><strong><%= @post.comments.count %></strong> Comments</h2>
		<%= render @post.comments %>
		<div class="comment-form">
			<hr />
			<h3 class="subtitle is-3">发表评论</h3>
	 		<%= render 'comments/form' %>
		</div>
	</div>
</section>

```
app/views/comments/-comment.html.erb
```
<div class="box">
  <article class="media">
    <div class="media-content">
      <div class="content">
        <p>
          <strong><%= comment.name %>:</strong>
          <%= comment.body%>
        </p>
      </div>
    </div>
     <%= link_to 'Delete', [comment.post, comment],
                  method: :delete, class: "button is-danger", data: { confirm: 'Are you sure?' } %>
  </article>

</div>

```
app/views/comments/-form.html.erb
```
<%= simple_form_for([@post, @post.comments.build]) do |f| %>

<div class="field">
  <div class="control">
    <%= f.input :name, input_html: { class: 'input' }, wrapper: false, label_html: { class: 'label' } %>
  </div>
</div>

<div class="field">
  <div class="control">
    <%= f.input :body, input_html: { class: 'textarea' }, wrapper: false, label_html: { class: 'label' }  %>
  </div>
</div>
<%= f.button :submit, 'Leave a reply', class: "button is-primary" %>
<% end %>

```
# 第四部分：添加用户功能模块
（1）只有登录才可以评论
（2）只有登录才可以修改
（3）只能修改自己的数据
（4）没有登录跳转到登录


gem 'devise', '~> 4.3'
rails generate devise:install
config/environments/development.rb:
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
rails generate devise:views
rake db:migrate
rails g devise User
rake db:migrate

config/routes.rb
```
  devise_for :users, controllers: { registrations: 'registrations' }
```
touch app/controllers/registrations_controller.rb
```
# https://jacopretorius.net/2014/03/adding-custom-fields-to-your-devise-user-model-in-rails-4.html
class RegistrationsController < Devise::RegistrationsController

 private

  def sign_up_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation)
  end

  def account_update_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation, :current_password)
  end

end
```


rails g migration AddFieldsToUsers name:string
```
def change
  add_column :users, :name, :string
  add_column :users, :username, :string
  add_index :users, :username, unique: true
 end
end
```
rake db:migrate
```

app/models/user.rb
```
has_many :posts
```
app/models/post.rb
```
belongs_to :user

```
<!DOCTYPE html>
<html>
  <head>
    <title>Twittter</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= stylesheet_link_tag "https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
  	<% if flash[:notice] %>
  		<div class="notification is-primary global-notification">
  			<p class="notice"><%= notice %></p>
  		</div>
  	<% end %>
  	<% if flash[:alert] %>
  		<div class="notification is-danger global-notification">
  			<p class="alert"><%= alert %></p>
  		</div>
  	<% end %>
  	<nav class="navbar is-info">
  		<div class="navbar-brand">
  		<%= link_to root_path, class: "navbar-item" do %>
  			<h1 class="title is-5">blog</h1>
  		<% end %>
			<div class="navbar-burger burger" data-target="navbarExample">
					<span></span>
					<span></span>
					<span></span>
  		</div>
  	 </div>

			<div id="navbarExample" class="navbar-menu">
				<div class="navbar-end">
          <div class="navbar-item">
					<div class="field is-grouped">
						<p class="control">
							<%= link_to 'New Post', root_path, class: "button is-info is-inverted" %>
						</p>
            <% if user_signed_in? %>
              <p class="control">
                <%= link_to current_user.name, edit_user_registration_path, class: "button is-info" %>
              </p>
              <p>
                <%= link_to "Logout", destroy_user_session_path, method: :delete, class:"button is-info" %>
              </p>
            <% else %>
            <p class="control">
              <%= link_to 'Sign In', new_user_session_path, class: "button is-info" %>
            </p>
						<p class="control">
              <%= link_to 'Sign Up', new_user_registration_path, class: "button is-info" %>
            </p>
            <% end %>
            </div>
					</div>
				</div>
			</div>
  	</nav>

    <%= yield %>
  </body>
</html>

```
app/views/devise/registrations/edit.html.erb
```
<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-4">

      <h2 class="title is-2">Edit <%= resource_name.to_s.humanize %></h2>
      <%= simple_form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
        <%= f.error_notification %>

          <div class="field">
            <div class="control">
              <%= f.input :name, required: true, autofocus: true, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
            </div>
          </div>

          <div class="field">
            <div class="control">
              <%= f.input :username, required: true,  input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
            </div>
          </div>

          <div class="field">
            <div class="control">
              <%= f.input :email, required: true, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
            </div>
          </div>

          <div class="field">
          <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
            <p>Currently waiting confirmation for: <%= resource.unconfirmed_email %></p>
          <% end %>
          </div>

          <div class="field">
            <div class="control">
            <%= f.input :password, autocomplete: "off", hint: "leave it blank if you don't want to change it", required: false, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
            </div>
          </div>

          <div class="field">
            <div class="control">
            <%= f.input :password_confirmation, required: false, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
            </div>
          </div>

          <div class="field">
            <div class="control">
              <%= f.input :current_password, hint: "we need your current password to confirm your changes", required: true, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
            </div>
        </div>

        <%= f.button :submit, "Update", class:"button is-info" %>

      <% end %>

        <hr />
        <h3 class="title is-5">Cancel my account</h3>
        <p>Unhappy? <%= link_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?" }, method: :delete %></p>

      </div>
    </div>
  </div>
</section>

```
app/views/devise/registrations/new.html.erb
```
<section class="section">
	<div class="container">
		<div class="columns is-centered">
			<div class="column is-4">
				<h2 class="title is-2">Log in</h2>
				<%= simple_form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>

				  <div class="field">
				  	<div class="control">
				    	<%= f.input :email, required: false, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
				    </div>
				  </div>

				  <div class="field">
				  	<div class="control">
				    <%= f.input :password, required: false, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
						</div>
					</div>

					<div class="field">
				  	<div class="control">
				    <%= f.input :remember_me, wrapper: false, as: :boolean if devise_mapping.rememberable? %>
				  	</div>
					</div>

				  <%= f.button :submit, "Log in", class:"button is-info is-medium" %>
				<% end %>
				<br/>
				<%= render "devise/shared/links" %>
			</div>
		</div>
	</div>
</section>

```
app/views/devise/sessions/new.html.erb
```
<section class="section">
	<div class="container">
		<div class="columns is-centered">
			<div class="column is-4">
				<h2 class="title is-2">Log in</h2>
				<%= simple_form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>

				  <div class="field">
				  	<div class="control">
				    	<%= f.input :email, required: false, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
				    </div>
				  </div>

				  <div class="field">
				  	<div class="control">
				    <%= f.input :password, required: false, input_html: { class: "input"}, wrapper: false, label_html: { class: "label" } %>
						</div>
					</div>

					<div class="field">
				  	<div class="control">
				    <%= f.input :remember_me, wrapper: false, as: :boolean if devise_mapping.rememberable? %>
				  	</div>
					</div>

				  <%= f.button :submit, "Log in", class:"button is-info is-medium" %>
				<% end %>
				<br/>
				<%= render "devise/shared/links" %>
			</div>
		</div>
	</div>
</section>

```
rails g migration AddUserIdToPosts user_id:integer
rake db:migrate
```
class PostsController < ApplicationController
  before_action :authenticate_user!, except: [:index, :show]
  def index
    @posts = Post.all.order('created_at DESC')
  end

  def new
    @post = current_user.posts.build
  end

  def create
    @post = current_user.posts.build(post_params)

      if @post.save
        redirect_to @post
      else
        render 'new'
      end
    end

  def show
   @post = Post.find(params[:id])
  end

  def edit
   @post = Post.find(params[:id])
  end

  def update
        @post = Post.find(params[:id])

        if @post.update(params[:post].permit(:title, :body))
            redirect_to @post
        else
            render 'edit'
        end
    end

  def destroy
     @post = Post.find(params[:id])
     @post.destroy

     redirect_to root_path
  end



private
   def post_params
     params.require(:post).permit(:title, :body)
   end
end

```
```
git checkout -b heroku
---
Gemfile
---
group :development, :test do
  gem 'sqlite3', '1.3.13'
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '~> 2.13'
  gem 'selenium-webdriver'
end

group :production do
  gem 'pg', '0.20.0'
end
---
bundle install
rails server
---
git add .
git commit -m "Update Gemfile for Heroku"
```
```
heroku create xiaoweiblog
git push heroku heroku:master
rake assets:clean
heroku run rake db:migrate
heroku open
```

# 二、中级知识体系的学习方式
接下来的升级版本就是：
1、购物车的知识体系；
2、支付对接的知识体系；
3、阿里云服务器的对接知识体系；

#三、 高级知识体系的学习方式
1、功能体系的逐渐的制作；
2、各种小型和大型知识体系的掌握；
3、各种疑难问题的解决；
