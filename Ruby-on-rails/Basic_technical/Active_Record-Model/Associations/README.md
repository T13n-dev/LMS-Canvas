# Active record 

## Một vài thứ để bắt đầu
Link document [active_record_basics](https://github.com/cgriego/active_attr)
* Active Record là M trong MVC - the model 
* Có thể sử dụng model không cần dựa trên ActiveRecord
* Cố gắng đặt tên ngắn, dễ hiểu nhưng không giản lược quá mức.
* Sử dụng gem [ActiveAttr](https://github.com/cgriego/active_attr) khi cần có những thao tác của ActiveRecord giống như validation trong model.

```ruby
class Message
  include ActiveAttr::Model

  attribute :name
  attribute :email
  attribute :content
  attribute :priority

  attr_accessible :name, :email, :content

  validates_presence_of :name
  validates_format_of :email, :with => /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i
  validates_length_of :content, :maximum => 500
end
```
### Naming Conventions 
Rails sẽ pluralize tên class để tìm bảng database tương ứng. ( class Book, bảng database được gọi books).

| Model/ Class  | Table/ Schema |
| --- | --- |
| Article  | articles  |
| LineItem  | line_items  |
| Deer | deers |
| Mouse | mice |
| Person | people |

### Schema Conventions
Active Record sử dụng các quy ước đặt tên cho các cột trong bảng cơ sở dữ liệu, tùy thuộc vào mục đích của các cột này.

* __Foreign keys__ - Các trường này phải đặt tên theo mẫu `singularized_table_name_id` ( Ví dụ: item_id, order_id ). Đây là các trường mà Active Record sẽ tìm kiếm khi bạn tạo liên kết giữa các mô hình của mình.  
* __Primary keys__ - Bởi mặc định, Active Record sẽ sử dụng một cột số nguyên tên id làm khóa chính của bảng ( bigint co PostgreSQL và MySQL, integer cho SQLite). Khi sử dụng [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) để tạo bảng, cột này sẽ tự động được tạo.

Phải sử dụng những database có sẵn, nếu không có lý do chính đáng thì không thay đổi những thứ mặc định của ActiveRecord như tên bảng hay là primary key.

```ruby
# Không tốt - không làm thế này nếu như có thể thay đổi thay đổi schema
class Transaction < ActiveRecord::Base
  self.table_name = 'order'
  ...
end
```

* Nhóm các macro lại, Và đặt các constant của class lên đầu tiên. Các macro cùng loại ( như là belongs_to và has_many ) hoặc là cùng macro nhưng tham số khác nhau ( ví dụ như validates ) thì sắp xếp theo thứ tự từ điển. Các callback thì sắp xếp theo thứ tự thời gian.

* Viết các macro theo thứ tự như sau:
  * các constant
  * các macro liên quan đến attr_
  * các macro quan hệ
  * validation
  * các macro callback
  * những macro khác

#### Scope
* Đặt tên scope thể hiện việc lấy một tập hợp con trong tập hợp bản ghi cha. 
* Hãy đặt tên scope sao cho có thể hiểu được một cách tự nhiên như sau 
`[số nhiều của tên model] có đặc tính [tên scope]`. 
(Ví dụ: với việc đặt tên scope là `active` trong model user có thể hiểu 
            Hãy lấy ra các `[users] có đặc tính [active]`.)
* Trong trường hợp có đối số, hãy kết hợp tên scope và đối số sao cho thật tự nhiên và dễ hiểu. 
* Cố gắng tránh việc đặt tên scope có bao gồm tên model. 

```ruby
# Không tốt
class User < ActiveRecord::Base
  scope :active_users, ->{where activated: true}
end
class Post < ActiveRecord::Base
  scope :by_author, ->author{where author_id: author.id}
end

# Tốt
class User < ActiveRecord::Base
  scope :active, ->{where activated: true}
end
class Post < ActiveRecord::Base
  scope :posted_by, ->author{where author_id: author.id}
end
```

* scope thì viết theo cách ngắn gọn của lambda. Nếu trong 1 dòng mà quá 80 kí tự thì nên cắt xuống dòng mới thích hợp để cho 1 dòng chỉ nên ít hơn 80 kí tự.

```ruby
class User < ActiveRecord::Base
  # các constants thì cho lên đầu
  GENDERS = %w(male female)

  # tiếp theo là các macro liên quan đến attr_
  attr_accessor :formatted_date_of_birth

  attr_accessible :login, :first_name, :last_name, :email, :password

  # rồi đến các macro quan hệ
  belongs_to :country

  has_many :authentications, dependent: :destroy

  # các validation macro
  validates :email, presence: true
  validates :password, format: {with: /\A\S{8,128}\z/, allow_nil: true}
  validates :username, format: {with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/}
  validates :username, presence: true
  validates :username, uniqueness: {case_sensitive: false}
  # theo thứ tự : email -> password -> username, format -> presence -> uniqueness

  # tiếp đến là các callback macro. Nên viết theo thứ tự thời gian: before -> after
  before_save :cook
  before_save :update_username_lower

  after_save :serve

  # scopes
  scope :active, ->{where(active: true)}

  # tiếp theo là các macro khác (ví dụ như macro của devise chẳng hạn)

  ...
end
```

* Không dùng ``` default_scope ``` ngoài việc liên quan đến xoá logic. Ngoài ra trong trường hợp này không được dùng ``` order ```.

* Một khi đã dùng `has_many` hoặc `has_one` đối với một model thì nhất định phải khai báo `belongs_to` với model tương ứng.

### Validation 
Active Record cho phép bạn xác thực trạng thái của một model trước khi nó được ghi vào cơ sở dữ liệu. Có một số phương pháp mà bạn có thể sử dụng để kiểm tra các model và xác thực rằng giá trị thuộc tính không trống, là duy nhất và chưa có trong cơ sở dữ liệu, tuân theo một định dạng cụ thể và nhiều định dạng khác.
Validation is a very important issue to consider when persisting to the database, so the methods save and update take it into account when running: they return false when validation fails and they don't actually perform any operations on the database. All of these have a bang counterpart (that is, save! and update!), which are stricter in that they raise the exception ActiveRecord::RecordInvalid if validation fails. A quick example to illustrate:

```ruby
class User < ApplicationRecord
  validates :name, presence: true
end
 
user = User.new
user.save  # => false
user.save! # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```
Xem thêm tại [Active Record Validations guide](https://guides.rubyonrails.org/active_record_validations.html)