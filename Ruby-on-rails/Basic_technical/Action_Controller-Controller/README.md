# Action Controller

## Controller
Action Controller is the C in MVC.
Link document [action_controller_overview](https://guides.rubyonrails.org/action_controller_overview.html)

*  Cố gắng gọt giũa nội dung của controller. Trong controller chỉ nên thực hiện việc lấy những data mà bên view cần, không code business logic ở đây. (Những cái đó nên viết trong model)

* Mỗi action trong controller thì lý tưởng là 1 method initialize model , 1 method seach , 1 method thực hiện tác vụ gì đó.

* Không chia sẻ giữa controller và view từ 2 biến instance trở lên.

* Đối với biến instance biểu thị resource chính ở Controller, hãy gán vào object của resource đó. Ví dụ, đối với `@article` ở bên trong ArticlesController thì gán instance của class `Article` vào. Với `@articles` thì gán collection của nó vào.

```ruby
# Không tốt
class ArticlesController < ApplicationController
  def index
    @articles = Article.all.pluck [:id, :title]
  end

  def show
    @article = "This is an article."
  end
end

# Tốt
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find params[:id]
  end
end
```

* Controller cần xử lý ngoại lệ xuất hiện tại model. Cần phải thông báo việc xuất hiện ngoại lệ bằng cách gửi đến client code 400 trở lên.

* Để tham số của render là symbol.

```ruby
render :new
```

* Không lược bỏ action ngay cả khi action đó không xử lý gì cả mà chỉ để hiển thị view

```ruby
class HomeController < ApplicationController

  def index
  end

end
```

* Đối với những action được truy cập bằng những method HTTP khác GET thì nhất định sau khi đã xử lý xong phải redirect đến một action được truy cập bằng phương thức GET. Tuy nhiên, những trường hợp mà không truy cập trực tiếp, ví dụ như chỉ là API để trả về json thì điều này không cần thiết.

**Lý do**

Ngăn chặn việc phát sinh nhiều xử lý khi mà người dùng thao tác refresh.

* Trong hàm callback nên sử dụng tên method hoặc ```lamda```. Không được sử dụng block ở đây.


```ruby
# cách viết không tốt

  before_action{@users = User.all} # brock

# cách viết tốt

  before_action :methodname # method name

# cách viết cũng tốt

  before_action ->{@users = User.all} # lambda
```