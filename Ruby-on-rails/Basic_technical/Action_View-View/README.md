# Action View

## View View and View
Link document [action_view_overview](https://guides.rubyonrails.org/action_view_overview.html#what-is-action-view-questionmark)

* Không gọi trực tiếp model trong view, mà phải sử dụng thông qua controller hoặc helper.
* Tuy nhiên vẫn có ngoại lệ cho phép trực tiếp gọi Model trong View như một master cho các thẻ như Select.
* Không viết các xử lý phức tạp trong view. Những xử lý phức tạp thì nên đặt trong các view helper hoặc trong model.
* Sử dụng partial hoặc layout để tránh viết lại code
* Sử dụng helper để biết form. Các đoạn code có chưa logic hoặc thiết lập như đọc file ngoài hoặc link thì cũng phải sử dụng helper.
* Không sử dụng form_tag khi mà có thể sử dụng form_for.
* Thêm 1 space bên trong các ``` <% ``` , ``` <%= ``` と ``` %> ```.

```ruby
#Cách viết không tốt
<%foo%>
<% bar%>
<%=bar%>
<%=bar %>

#Cách viết tốt
<% foo %>
<%= bar %>
```

* Cân nhắc sử dụng [client side validation](https://github.com/bcardarella/client_side_validations). Cách sử dụng như dưới đây.
    * Khai báo custom validator kế thừa ``` ClientSideValidations::Middleware::Base ```

```ruby
module ClientSideValidations::Middleware
  class Email < Base
    def response
      if request.params[:email] =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
        self.status = 200
      else
        self.status = 404
      end
      super
    end
  end
end
```

  * Tạo file ``` public/javascripts/rails.validations.custom.js.coffee ```、và thêm tham chiếu đến file vừa tạo vào ``` application.js.coffee ```.

```coffee
# app/assets/javascripts/application.js.coffee
#= require rails.validations.custom
```

  * Thêm validator của bên client.

```coffee
#public/javascripts/rails.validations.custom.js.coffee
clientSideValidations.validators.remote['email'] = (element, options) ->
  if $.ajax({
    url: '/validators/email.json',
    data: { email: element.val() },
    async: false
  }).status == 404
    return options.message || 'invalid e-mail format'
```
