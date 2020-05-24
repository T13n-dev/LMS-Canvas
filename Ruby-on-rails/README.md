# Sưu tầm Tips trong Ruby on Rails
* [Các quy định về viết code Ruby on Rails (Tập các kiểu chuẩn)](https://github.com/framgia/coding-standards/blob/master/vn/rails/standard.md)

## Thiết lập

* Những thiết lập của ứng dụng đặt trong thư mục ``` config/initializers ```. Những đoạn code được đặt trong này sẽ được chạy khi ứng dụng khởi tạo.

* Đối với những file code khởi tạo của các gem như ``` carierwave.rb ``` hoặc ``` active_admin.rb ``` thì đặt tên file giống tên gem.

* Những thiết lập cho từng môi trường development, test, production thì thiết lập trong các file tương ứng trong thư mục ``` config/environments ```

* Thiết lập cho tất cả môi trường thì viết vào ``` config/application.rb ```

* Trong trường hợp tạo môi trường mới như staging thì cố gắng thiết lập gần giống môi trường production


## ActiveResource

* Trong trường hợp cần trả về response theo định dạng khác ngoài XML hay là JSON thì có thể tự tạo ra định dạng khác theo như dưới đây. Để tạo ra một định dạng khác thì cần phải định nghĩa 4 method là ``` extension ```、``` mime_type ```、``` encode ```、``` decode ```

```ruby
module ActiveResource
  module Formats
    module Extend
      module CSVFormat
        extend self

        def extension
          "csv"
        end

        def mime_type
          "text/csv"
        end

        def encode(hash, options = nil)
          # Encode dữ liệu theo định dạng mới và trả về kết quả
        end

        def decode(csv)
          # Decode dữ liệu từ định dạng mới và trả về kết quả
        end
      end
    end
  end
end

class User < ActiveResource::Base
  self.format = ActiveResource::Formats::Extend::CSVFormat

  ...
end
```

* Trong trường hợp request được gửi không có extension thì chúng ta có thể override 2 method ``` element_path ``` và ``` collection_path ``` của ``` ActiveResource::Base ```, sau đó xoá phần extension đi.

```ruby
class User < ActiveResource::Base
  ...

  def self.collection_path(prefix_options = {}, query_options = nil)
    prefix_options, query_options = split_options(prefix_options) if query_options.nil?
    "#{prefix(prefix_options)}#{collection_name}#{query_string(query_options)}"
  end

  def self.element_path(id, prefix_options = {}, query_options = nil)
    prefix_options, query_options = split_options(prefix_options) if query_options.nil?
    "#{prefix(prefix_options)}#{collection_name}/#{URI.parser.escape id.to_s}#{query_string(query_options)}"
  end
end
```

## Migration

=======
* Quản lý phiên bản của ``` schema.rb ``` （hoặc là ``` structure.sql ```）

* Sử dụng ``` rake db:test:prepare ``` để tạo database phục vụ cho test

* Nếu cần thiết lập giá trị mặc định, thì không thiết lập tại tầng ứng dụng mà thiết lập thông qua migration

```ruby
# không tốt - gán giá trị mặc định tại tầng ứng dụng
def amount
  self[:amount] or 0
end
```

Việc thiết lập giá trị mặc định của các bảng chỉ trong ứng dụng là cách làm tạm bợ, có thể sinh ra lỗi trong ứng dụng. Hơn nữa, ngoại trừ những ứng dụng khá nhỏ thì hầu như các ứng dụng đều chia sẻ database với các ứng dụng khác, thế nên nếu chỉ thiết lập trong ứng dụng thì tính nhất quán của dữ liệu sẽ không còn được đảm bảo.

* Quy định ràng buộc khoá ngoài. Mặc dù ActiveRecord không hỗ trợ điều này nhưng mà có thể dùng gem của bên thứ 3 như [schema_plus](https://github.com/lomba/schema_plus).

* Để thay đổi cấu trúc bảng như thêm column thì viết theo cách mới của Rails 3.1. Tóm lại, không sử dụng ``` up ``` hoặc ``` down ``` mà sử dụng ``` change ```.

```ruby
# cách viết cũ
class AddNameToPerson < ActiveRecord::Migration
  def up
    add_column :persons, :name, :string
  end

  def down
    remove_column :person, :name
  end
end

# cách viết mới tiện hơn
class AddNameToPerson < ActiveRecord::Migration
  def change
    add_column :persons, :name, :string
  end
end
```

* Không sử dụng class của model trong migration. Tại vì model class thì rất dễ bị thay đổi, khi đó xử lý của migration trước đây có thể bị ảnh hưởng.

## Đa ngôn ngữ

* Không đặt các thiết lập phụ thuộc vào ngôn ngữ, quốc gia vào model, controller, view. Những thiết lập này đặt trong ``` config/locales ```.

* Khi cần dịch các nhãn (label) của ActiveRecord model thì viết vào ``` activerecord ``` scope như dưới đây.

```yaml
ja:
  activerecord:
    models:
      user: メンバー
    attributes:
      user:
        name: 姓名
```

Khi đó, ``` User.model_name.human ``` sẽ trả về "メンバー", ``` User.human_attribute_name("name") ``` sẽ trả về "姓名". Kiểu dịch như thế này cũng có thể sử dụng được trong view.

* Chia những đoạn dịch các thuộc tính của ActiveRecord và những đoạn được dùng trong view thành các file riêng biệt. File nào được dùng trong model thì đặt trong thư mục ``` models ```, file nào được dùng trong view đặt trong thư mục ``` views ```.

  * Sửa lại file ``` application.rb ``` để load các file trong locales khi thêm file vào thư mục này.

```ruby
# config/application.rb
config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}').to_s]
```

* Đặt những thứ dùng chung như các định dạng của ngày tháng, tiền tệ ngay bên trong thư mục ``` locales ```.

* Sử dụng những method tên ngắn hơn. Sử dụng ``` I18n.t ``` thay cho ``` I18n.translate ```, ``` I18n.l ``` thay cho  ``` I18n.localize ```.

* Sử dụng Lazy lookup trong view. Ví dụ nếu chúng ta có cấu trúc như sau:
```yaml
ja:
  users:
    show:
      title: "ユーザー情報"
```
thì giá trị của ``` users.show.title ``` có thể lấy được trong ``` app/views/users/show.html.haml ``` bằng cách viết ngắn gọn như sau:
```ruby
= t '.title'
```

* Trong controller và model thì không dùng ``` :scope ``` mà thay vào đấy ta dùng dấu chấm (.) để lấy các giá trị mình muốn. Việc dùng dấu chấm sẽ đơn giản hơn và dễ hiểu hơn.

```ruby
# sử dụng cách viết này
I18n.t 'activerecord.errors.messages.record_invalid'

# thay cho cách viết dưới đây
I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]
```

* Thông tin chi tiết có thể tham khảo tại [RailsGuide](http://guides.rubyonrails.org/i18n.html).

## Asset

Sử dụng asset pipeline

* Stylesheet, javascript, hay image của ứng dụng thì cho vào ``` app/assets ```.

* Những file như thư viện thì cho vào ``` lib/assets ```. Thế nhưng những file thư viện mà đã chỉnh sửa cho phù hợp với ứng dụng của mình thì không cho vào đây.

* Những sản phẩm thirdparty như jQuery hoặc bootstrap thì lưu trong ``` vendor/asstes ```.

* Nếu có thể thì nên sử dụng những gem của asset. (ví dụ：[jquery-rails](https://github.com/rails/jquery-rails)）。

* Trong CSS khi viết url thì dùng asset_url.

## Mailer

* Đối với mailer thì đặt tên giống như ``` SomethingMailer ```. Như thế sẽ hiểu được mail gửi nội dung gì và liên quan đến view nào.

* Viết cả hai loại template  HTML và plaintext.

* Trong môi trường developer thì cho xuất hiện lỗi khi gửi mail không thành công. Thiết lập mặc định là không xuất hiện lỗi nên cần phải chú ý.

```ruby
# config/environments/development.rb
config.action_mailer.raise_delivery_errors = true
```

* Nhất định phải thiết lập host

```ruby
# config/environments/development.rb
config.action_mailer.default_url_options = {host: "localhost:3000"}

# config/environments/production.rb
config.action_mailer.default_url_options = {host: 'your_site.com'}

# trong class mailer của bạn
default_url_options[:host] = 'your_site.com'
```

* Khi mà cần gán link đến trang của mình trong mail thì không dùng method ``` _path ``` mà phải dùng ``` _url ```. Bởi vì ``` _url ``` có chứa hostname nhưng ``` _path ``` thì không.

```ruby
# sai
You can always find more info about this course
= link_to 'here', url_for(course_path(@course))

# đúng
You can always find more info about this course
= link_to 'here', url_for(course_url(@course))
```

* Thiết lập chính xác địa chỉ của From và To. Sử dụng khuôn mẫu như dưới đây.

```ruby
# trong class mail của bạn
default from: 'Your Name <info@your_site.com>'
```

* Đối với môi trường test thì không quên thiết lập `` test ``` cho phương thức gửi mail

```ruby
# config/environments/test.rb
config.action_mailer.delivery_method = :test
```

* Đối với môi trường development hoặc production thì thiết lập ``` smtp ``` là phương thức gửi mail

```ruby
# config/environments/development.rb, config/environments/production.rb
config.action_mailer.delivery_method = :smtp
```

* Bởi vì có những mail client xuất hiện lỗi với external CSS thế nên khi gửi HTML mail thì chỉ định inline css cho tất cả style. Thế nhưng, làm như thế dẫn đến khó bảo trì và việc lặp lại code. Để đối phó với vấn đề này thì chúng ta có thể dùng [premailer-rails3](https://github.com/fphilipe/premailer-rails3) hoặc [roadie](https://github.com/Mange/roadie).

* Cần tránh việc gửi mail khi trang đang được tạo. Bởi vì có thể xảy ra request timeout khi nhiều mail được gửi hoặc độ trễ của việc load trang. Để giải quyết vấn đề đó thì có thể dùng [delayed_job](https://github.com/tobi/delayed_job).

## Bundler

* Những gem mà chỉ dùng trong môi trường development hoặc test thì phải viết trong nhóm tương ứng.

* Chỉ sử dụng những gem thực sự cần thiết. Những gem mà không nổi tiếng thì khi dùng cần phải xem xét kỹ.

* Đối với những gem mà có những phụ thuộc vào loại OS nhất định khi vận hành trên OS khác thì Gemfile.lock sẽ bị sửa lại. Vì vậy nên nhóm các gem cho OS X trong nhóm ``` darwin ```, và các gem cho Linux trong nhóm ``` linux ```

```ruby
# Gemfile
group :darwin do
  gem 'rb-fsevent'
  gem 'growl'
end

group :linux do
  gem 'rb-inotify'
end
```

Để load được gem thích hợp vào môi trường chính xác thì viết như sau vào file ``` config/application.rb ```

```ruby
platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
Bundler.require(platform)
```

* Không xoá ``` Gemfile.lock ``` trong version management system. File này nhằm đảm bảo môi trường phát triển của developer nào cũng chạy gem cùng phiên bản khi ``` bundle install ```.


<%= link_to '<button type="button" class="Button"> Disabled Account </button>'.html_safe, :controller => "accounts", :tab => "disabled" %>

  def active_tab(tab, param = 'tab')
    if request.params[ param ] == tab
      'disabled';
    end
  end
