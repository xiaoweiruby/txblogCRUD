
# 一、刚刚入门的孩子的学习方式：
1、基础 POST 的 CRUD 功能体系；

2、用户 COMMENT 评论的体系；
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
