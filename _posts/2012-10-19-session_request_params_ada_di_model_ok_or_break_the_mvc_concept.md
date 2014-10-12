---
layout:     post
title:      Session, Request, Params Ada Di Model. OK or Break the MVC Concept
date:       2012-10-19 09:45:00
summary:    thought
comment: true
categories: Ruby Rails
---

Code blocks use the [solarized](http://ethanschoonover.com/solarized) theme. Both the light and
dark versions are included, so you can swap them out easily. _Solarized Dark_ is the default.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  around_filter :_controller_for_model

  ....

  protected
  def _controller_for_model
    accessor = instance_variable_get(:@_request)
    kelasess = [ActiveRecord::Base, ActiveRecord::Base.class]

    kelasess.each do |kelas|
      kelas.send(:define_method, "controller", proc {accessor})
    end
    yield
    kelasess.each do |kelas|
      kelas.send :remove_method, "controller"
    end

  end
end
{% endhighlight %}

klo lo modif ApplicationController kaya diatas, Model lo bakal punya method controller yang bisa punya isinya session, request dan params
MVC, Model View Controller. adalah sebuah concept yang ngebantu developer, memisahkan antara logic, design dan Representasi data storage. konsep ini keren bgt, apalagi kalo applikasi yang dikembangkan oleh team yang masing-masing udah punya job desc-nya sendiri - sendiri. Yang designer, urusannya di View, Programmer di Controller dan Model.
Tapi terkadang, buat orang males kaya gue. suka nemuin case (mungkin lo juga sering nemuin). contoh : ada Model User, dan Post. Keduanya saling berelasi

{% highlight ruby %}
#model User
class User < ActiveRecord::Base
  has_many :posts
end
#model Post
class Post < ActiveRecord::Base
  belongs_to :user
end
{% endhighlight %}

session yang merepresentasi kan user yang sudah login udah ada misal, di sebuah Controller kita dah definisikan ini

{% highlight ruby %}
class UserController < ApplicationController
  def auth
      user = User.where({:user_name => params[:user], :password => params[:password]})
      if !user.nil?
          session[:current_user_id] = user.id
      else
          #FTW, Lupa cuyy ??
          redirect_to request.referer
      end
  end
  ....
end
{% endhighlight %}
(Anggep aja, males nulis code nya) kita sudah punya resource Posts. ketika user mau bikin post baru, masalahnya muncul. Mana yang lo pilih, Lo mendefnisikan user(yang udah login) yang belongs_to post lewat resource PostsController

{% highlight ruby %}
#dalam PostController
def create
  @post = post.new(params[:post])
  @post.user =  User.find(session[:current_user_id])
  respond_to do |format|
    ....
  end
end
{% endhighlight %}
atau lo detect di Post (model) pake callback?

{% highlight ruby %}
#dalam model Post
before_save :detect_current_user
def detect_current_user
  user = User.find(controller.session[:current_user_id]) unless session[:current_user_id].nil?
end
#note, method controller ada klo ApplicationController dah di modif kaya snippet yang paling atas
{% endhighlight %}

ini contoh sederhana aja. mungkin lo juga sering nemuin case yang lebih complex lagi yang membuat model harus bisa punya method request, sesssion, params langsung didalam model itu sendiri. tanpa harus passing argument ke model itu lewat controller

Break MVC atau Mempermudah lo ??. gimana bijaksananya lo aja dalam ngoding, dan x-factor yang lo temuin dalam setiap development lo ..
