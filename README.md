现在我们开始学习 Ruby on Rails 的知识体系，现在主要是从知识体系叠加的角度来思考问题。


# 一、刚刚入门的孩子的学习方式：
>1、基础 POST 的 CRUD 功能体系；
![CRUD 展示.gif](https://upload-images.jianshu.io/upload_images/7680238-80aecc22ae3370b1.gif?imageMogr2/auto-orient/strip)

https://github.com/shenzhoudance/txblogCRUD

2、用户 COMMENT 评论的体系；
![comment 展示.gif](https://upload-images.jianshu.io/upload_images/7680238-82e52ba12840fdca.gif?imageMogr2/auto-orient/strip)

3、页面 bulma 美化体系；
4、用户 devise 的对接体系；
5、用户 heroku 上传体系

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
# 二、中级知识体系的学习方式
接下来的升级版本就是：
1、购物车的知识体系；
2、支付对接的知识体系；
3、阿里云服务器的对接知识体系；

#三、 高级知识体系的学习方式
1、功能体系的逐渐的制作；
2、各种小型和大型知识体系的掌握；
3、各种疑难问题的解决；
