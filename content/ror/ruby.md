---
title: Ruby
draft: false
date: 2024-03-04
tags:
  - ruby
  - language
---

Ruby là một ngôn ngữ lập trình **hướng đối tượng**, **động** (dynamic), **thông dịch**

Tính hướng đối tượng
- Mọi giá trị đều là Object và Class. Root Class là **Object**
  - Class cũng là Objects kế thừa từ **Class** Class.
  - Tất cả các methods đều cần được gọi trên một Object. Do đó, **class methods** chỉ là instance methods trên Object của **Class**.
- Modules giống như Class, có các Methods, nhưng không thể tạo instance.
  - Module kế thừa từ **Module** Class
  - Module có thể nhúng (**mixed in** - mixin) vào trong Class và toàn bộ các Methods của Module cũng được nhúng vào Class đó.
  - Module giúp triển khai cơ chế **multiple inheritance**

Tính động
- Mọi thứ đều có thể thay đổi lại - **malleable**
- Dễ dàng mở rộng **nội dung** của các Class hay thậm chí các thư viện - **monkey patching**
  - **Singleton Methods** là methods chỉ xác định trên một Object duy nhất, tức là chỉ object đó có thể gọi methods đó.
- Có thể mở rộng nội dung cho Classes _ngay khi chương trình đang thực thi_, tính năng này gọi là **Metaprogramming**
- Kiểu dữ liệu động: Một biến có thể chứa bất kỳ giá trị nào, và có thể thay đổi giá trị tùy ý
  - Khi thực thi method trên Object, Ruby chỉ thực hiện tìm kiếm trong object đó dựa trên tên method (object không cần chứng minh nó được định nghĩa một method tương ứng) - [**duck typing**](https://stackoverflow.com/a/40434829)


Thông dịch
- Chương trình Ruby không được biên dịch thành tệp thực thi
- Các lệnh trong Ruby được thực thi lần lượt theo thứ tự từ trên xuống dưới
- Kể cả các lệnh bên trong một Class


- Mọi lệnh đều là biểu thức (control structures, loop,...)

# Installation, Tools and Configuration

## Installation

[rbenv](https://github.com/rbenv/rbenv) is a version manager tool for the Ruby programming language on Unix-like systems.

- Install rbenv: `git clone https://github.com/rbenv/rbenv.git ~/.rbenv`
- Config rbenv: `echo 'eval "$(~/.rbenv/bin/rbenv init - bash)"' >> ~/.bashrc` [Bash]
  - Set PATH: `echo 'export PATH="~/.rbenv/bin:$PATH"' >> ~/.bashrc` [Bash]
- List supported Ruby stable versions: `rbenv install -l`
- Install Ruby: `rbenv install <version>`
- Set Ruby default version for this machine: `rbenv global <version>`
- Remove Ruby: `rbenv uninstall <version>` or remove the specified directory in **~/.rbenv/versions** directory.

## Tools

- **ruby**: Interpreter. `-e` execute inline
- **irb**: Interactive Ruby
- **ri**: Ruby Document
- **gem**: Package Management

## Development Environment

- VS Code Extensions:
  - Rufo: Formatter
    - `gem install rufo`
    - [Config](https://github.com/ruby-formatter/rufo/blob/master/docs/settings.md)
  - Ruby Solargraph: Intellisense
    - `gem install solargraph`
  - Endwise: Auto add `end`
  - Rails DB Schema: Completion for DB Schema
  - Auto Rename Tag: Rename HTML Tag on typing
  - ERB Helper Tags: Snippets for ERB Tag 

# Variables and Scope

Trong Ruby, không cần khai báo các biến và phạm vi truy cập của chúng. 

Phạm vi truy cập của biến được xác định dựa trên Name Convention:
- `name`: Biến cục bộ, chỉ truy cập trong khối lệnh định nghĩa nó
- `$name`: Biến toàn cục
- `@name`: Biến instance, có thể truy cập qua **instance** hoặc **bên trong Class**
  - Nếu muốn truy cập bên ngoài cần khai báo các Attributes Accessors (GET, SET)
  - **Chú ý**: Nếu biến `@name` được khai báo ở Class scope thì nó là instance variables ứng với Class object, không phải cho instance của Class đó - **class instance variables**
- `@@name`: Có thể truy cập qua **Class** hoặc **bên trong Class**, bên trong và ghi đè trong các **Classes kế thừa**.
- `Name`: Hằng số
  - Hằng số có thể thay đổi giá trị nhưng sẽ nhận được warning.
  - Nếu truy xuất hằng số chưa được gán giá trị sẽ sinh lỗi **NameError**
  - Các **Class** object thực chất là các hằng số.

Sử dụng **`defined?`** để kiểm tra một biến có thể truy cập từ context hiện tại không.

## Pseudo Variables

Các biến giả là các biến không thể thay đổi giá trị

- **`self`**: Xác định context cho method hiện tại.
- **`nil, false, true`**
- **`$1` -> `$9`**: Các Captured Groups từ kết quả của Regex

# Methods


- Class methods: Các Methods ở mức Class
  - Có thể truy cập từ các Class object
  - Có thể truy cập tới các **class variables** và **instance class variables**

Mặc định các Methods trong Class là **public** (ngoại trừ **`initialize`** và các Global Methods - định nghĩa cho **Object**)

Phạm vi truy cập được xác định qua **`public`, `private`, `protected`**, chúng đều là các methods, nhận vào định danh của các instance methods muốn xác định phạm vi truy cập
- Nếu không xác định đối số, toàn bộ phần code phía sau chúng sẽ được coi là có phạm vi truy cập tương ứng

Có sự khác biệt giữa **private** trong Ruby so với một số ngôn ngữ khác:
- **Các `private` methods có thể truy cập từ bên trong Class nhưng không thể xác định cụ thể đối tượng gọi, kể cả đó là `self`**.
- **Các `private` method có thể truy cập từ Class kế thừa**
- **Điểm khác biệt giữa `protected` và `private` là có thể truy cập `protected` methods từ instances**

```ruby
class AccessPrivate
  def a; end
  private :a # a is private method

  def accessing_private
    a # sure!
    self.a # nope! private methods cannot be called with an explicit receiver at all, even if that receiver is "self"
    other_object.a # nope, a is private, you can‚t get it (but if it was protected, you could!)
  end
end
```

Sử dụng `private_class_method` để khai báo các Class methods cần **private**

# Module

Module là một cách để gom nhóm các chức năng lại: Biến, Methods, Class,...

Module kế thừa từ **Module** Class nhưng không thể khởi tạo instance và cũng không có `self`

Khi `include` một Module bên trong Class, Class đó có thể truy cập toàn bộ các biến, methods và Classes được định nghĩa bên trong Module đó.

## EigenClass

EigenClass của một Class là một Class ánh xạ tới Class đó, ở đó các Class Methods của Class gốc chính là các instance methods của EigenClass

EigenClass của Class `A` được truy cập qua `class << A`

Như vậy, các Class Methods có thể được khai báo như sau:

```ruby
class A
  def self.class_method_1; end

  class << self
    def class_method_2; end

    def class_method_3; end
  end
end
```

# Block

Block là một đối tượng đặc biệt trong Ruby, nó là một khối lệnh, nhưng khác với khối lệnh trong C/C++, Block có thể nhận vào đối số.

Block cho phép một method có thể gọi thực thi một khối lệnh bên ngoài. Bên trong một method, ta có thể sử dụng mệnh đề `yield` để chuyển quyền thực thi sang Block. Quyền thực thi chỉ được trả lại cho Method khi Block hoàn thành.
  - Đối số cho `yield` là đối số cho Block
  - Kết quả trả về từ Block là kết quả trả về của mệnh đề `yield`

Block được truyền cho method với cú pháp `{}` hoặc `do...end` (`{` và `do` phải nằm trên cùng dòng với method)
  - Danh sách tham số của Block được bao bởi `||` (phía sau `do` hoặc `{`) và phân tách bởi dấu `,`
  - Khi `yield` không cung cấp đủ đối số, các đối số phía sau sẽ nhận `nil`
  - Giá trị trả về của Block có thể sử dụng `next` - Nếu sử dụng `return` nó sẽ kết thúc cả method gọi Block.
  - Ta có thể định nghĩa biến cục bộ cho Block ở sau danh sách tham số, phân tách với danh sách tham số bằng `;`

Ex:
```ruby
1.upto(3) {|x| puts x } # Parens and curly braces work
1.upto 3 do |x| puts x end # No parens, block delimited with do/end
1.upto 3 {|x| puts x } # Syntax Error: trying to pass a block to 3
```

# Proc and Lambda

Block, Proc và Lambda là các cơ chế triển khai **higher-order function** (**closures**) trong Ruby.

Block không phải là một Object và nó luôn cần một đối tượng khác để tồn tại.

Ta có thể tạo các objects cho một Block, chúng gọi là _proc_ và _lambda_, cả hai đều là **`Proc`** object. **`Proc`** là một object bao đóng các biến cục bộ (**closure**) và các biến này vẫn có thể truy cập ở các lần gọi khác nhau.
- **_proc_** được tạo qua `Proc.new`
  - Sử dụng `return` sẽ kết thúc cả context bên ngoài _proc_
- **_lambda_** được tạo qua `Kernel.lambda` hoặc qua Literal (cú pháp của Block kết hợp với method `->` cho tên hàm). Ex: `succ = ->(x) { x + 1 }`
  - Nếu sử dụng `return` sẽ chỉ kết thúc context của _lambda_.
  - Kiểm tra danh sách đối số đầu vào

Khi truyền một Block cho một method, Block đó tự động chuyển về một **Proc** và có thể gọi bên trong method qua:
- Sử dụng mệnh đề `yield`
- Gọi `call` trên đối số Proc truyền vào.
  - Hoặc `[], ()`

Việc sử dụng `&` cho đối số cuối cùng chỉ rằng:
- Method này có thể nhận vào một Block.
- Block đó sẽ được chuyển về Proc và gán cho đối số cuối cùng
- Có thể gọi `yield` vì nó là một Block
- Có thể gọi `call` vì nó là Proc object.
- Tuy nhiên khi gọi Method, ta không thể truyền một Proc object trực tiếp.
- Nếu muốn truyền Proc object, cần chuyển nó về Block bằng `&`

# Symbol

Mỗi khi tạo một String object, Ruby sẽ khởi tạo một khối nhớ mới để lưu giá trị của String đó. Ta có thể kiểm tra **`object_id`**, nó chỉ ra hai Objects có cùng ở một vị trí ô nhớ hay không.

Symbol là một kiểu dữ liệu đặc biệt trong Ruby, nó mô tả một chuỗi nhưng có thể so sánh, và nếu hai Symbols là giống nhau thì chúng luôn nằm ở cùng một vị trí ô nhớ (tức là cùng `object_id`)

Symbol là các chuỗi bắt đầu bằng `:` (và không cần bao trong `'` hay `"`)

# Some Operators

Một số Operators đặc biệt:
- **`*`**: [Splat] Cho phép Pack/Unpack một danh sách đối số thành **Array** và ngược lại
- **`&`**: Dùng để chuyển từ **Proc** sang **Block** và ngược lại.
  - Nếu tham số cuối của một Method sử dụng `&` thì method nó mong muốn truyền vào một **Proc** object. Khi truyền vào một **Block**, nó tự động được đóng gói thành **Proc**
- **`::`**: Truy cập hằng số hoặc 


# Syntax

- true, false, nil, self, __FILE__, __LINE__, __ENCODING__
- Only `false` and `nil` are False.

## Comments

- \# starts a comment
- `\n=begin .... \n=end` define a Embedded Document.

## DataTypes and Literals

- Numeric: _Immutable_
    - Integer: Fixnum (32 bits) <-> Bignum
    - Float:
    - Complex, BigDecimal (real, use decimal unit 1/10 instead of 1/2), Rational
    - **Literals**:
        - 0x, 0X, 0b, 0B, 0
        - ., e, E
    - **Operators**:
        - -7/3 -> -3 (round to -Inf)
        - -7/0 -> Error
        - -7.0/0 -> Infinity
        - -7%3 = 2
        - 12[0] -> 0 (bit 0)
- String: _Mutable_
    - `%q[...]` (thay cho `'`), `%Q[...]` (thay cho `"`)
    - String Interpolation: Chỉ dùng với Double Quotes
      - `#{}` (expression) or `#$name` (variables)
    - `` and `%x[...]` for System Command
    - `?` for a single character
    - **Operators**:
        - `+` and `<<` to append, but the later is in-place. `*`
        - `[,]` or `[Range]` for indexing and substring (retrieve, replace, append, delete)
- Array: `a = [3, 2, 1]`
    - `%w[...]` and `%W[...]` for Word Extract (Literal Array).
    - **Operators**:
        - `+` and `<<` to append, but the later is append element. `*`, `-`
        - `|`, `&` for Set Operations
        - `[,]` or `[Range]` for indexing and subarray (retrieve, replace, append, delete)
- Hash: `h = {:one => 1, :two => 2}`
- Range:
    - `...` (exclude end) and `..` (include end). Ex: `1..3` (1, 2, 3) and `1...3` (1, 2)
    - All Types to define `.succ`
    - 2 types of membership test: `.member?`, `.include?` and `.cover` (Always continuous test)
        - _Discrete Test_: Match exact the element
        - _Continuous Test_: Use comparator <=>
- Symbol: A unique String, same memory for same value
    - Identifier or String start with `:`
    - `%s[...]` for Literal Symbol
    - Used to refer to a method name. [All classes, methods, variables are stored in a Symbol Table]
    - String2Symbol: `intern`, `to_sym`
    - Symbol2String: `to_s`, `id2name`
- Object:
    - All operations are based on reference. Except Fixnum and Symbol.
    - `object_id` to retrieve ID.
    - `class` to retrieve Class. `instance_of?`, `is_a?`, `===`
    - Explicit Conversions: `to_s`, `to_i`, `to_f`, `to_a`
    - Implicit Conversions: Override `to_str`, `to_int`, `to_ary`

## Punctuation and Convention

- Special Operators
    - `::` to access Variables or Constants in Object, Module,...
    - `.` to invoke Method on Object (or default `self` if not specify the Object)
    - `*` is splat operator, used to pack and unpack
    - `()` used to grouping and unpack
    - `<=>` return -1 for less than, 0 for equal and 1 for greater than
    - `=~` is pattern-matching operator, used for matching String with REGEX. `!~` is inverse version.
    - `defined?` tests whether its operand is defined or not.
- Suffixes and Prefixes:
    - `name?` indicate Predicate method (return Boolean)
    - `name=` indicate Assignment method (can be used as `name = value` or `name=(value)`)
    - `name!` indicate in-place method


- Conventions:
    - Constants: LIKE_THIS or LikeThis
    - Variables: 

## Expression

- All statements are expression, have a value.
- Parallel Assignments: return Array of right handside value

|   |   |
|---|---|
| `x, y, z = 1, 2, 3`  | `x = 1, y = 2, z = 3`  |
| `x = 1, 2, 3` | `x = [1, 2, 3]` |
| `x, = 1, 2, 3` | `x = 1` |
| `x, y, z = [1, 2, 3]` | `x = 1, y = 2, z = 3` | 
| `x, y, z, t = 1, 2` | `x = 1, y = 2, z = nil, t = nil` |
| `x, y, z, t = *[1, 2], 3, *[4]` | `x = 1, y = 2, z = 3, t = 4` | 
| `x, *y, z = *[1, 2, 3, 4, 5]` | `x = 1, y = [2, 3, 4], t = 5` |
| `(x, y), z = 1, 2` | `x,y = 1; z = 2` |
| `x, (y, (z, t)) = [1, [2, [3, 4]]]` |  `x = 1, y = 2, z = 3, t = 4` | 

## Control Statements

### if/unless

```ruby
if expression[\n ; then]
  code
elsif expression[\n ; then]
  code
else
  code
end
```

Return value is the last expression in `code` that is executed.

Single-line version:
```ruby
if expression then code end

code if expression
```

In single-line version (modifier), expression must use `and, or, not` instead of `&&, ||, !`

Instead of using `if`, we can use `unless`, but without `elsif`

### case

```ruby
# Same as `if`
case
when expr, expr,... then code
when expr, expr,... then code
else code
end

# Use `===` as compare variable === expr.
# The `variable` evaluate once
case variable
when expr, expr,... then code
when expr, expr,... then code
else code
end
```

`===` used in `case` expression
- Fixnum: Same as `==`
- Class: Same as `instance_of?`
- Range: Same as `include?`
- Regexp: Same as `=~`

### while/until

```ruby
# Loop until the expr becomes `false`
while expr[\n ; do]
  code
end

# Loop until the expr becomes `true`
until expr [\n ; do]
  code
end
```
### for/in

```ruby
# Loop until the expr becomes `false`
for var, var,... in collection[\n ; do]
  code
end
```
- `collection` is a **Enumerable object** (define a `each` [iterator method](#iterators))

### Altering Control Flow

#### return

`return` statement cause the enclosing method to return to its caller. `return` can accepts many expressions, separate by `,` to indicate the method return value. If no expr, return `nil`, if has many, wrap them in Array.

In fact, sometimes `return` does not neccessary, the last expr in method is default the return value of that method. 

`return` used in:
- Not exit method at end.
- Exit on condition.
- Return multiple values

#### break and next

`break` the _loop_ or _exit the block and terminate the iterator using it_.

`break` can return a value for _loop_ or _the iterator_

`next` is like `continue` in other languages. It _continue the loop_ or _exit the block and continue the iterator using it_.

#### redo and retry

`redo` indicate to re-execute _the first statement in loop_ or _the first statement in block_.

`retry` is same as `redo`
- Used with `rescue` for catching error.
- Reevaluate the iterator expression (not just the block like `redo`)
- Used within method to re-execute the first statement of it.

#### throw/catch and raise/rescue

`throw/catch` are used for (general purpose) exception handling, they accept a Symbol to indicate the exception.
- `throw` can return value for `catch` by additional expression.

Ex:
```ruby
for matrix in data do # Process a deeply nested data structure.
 catch :missing_data do # Label this statement so we can break out.
   for row in matrix do
     for value in row do
       throw :missing_data unless value # Break out of two loops at once.
 # Otherwise, do some actual data processing here.
     end
   end
 end
 # We end up here after the nested loops finish processing each matrix.
 # We also get here if :missing_data is thrown.
en
```

`raise/rescue` are used for `Exception` handling.
- `raise` accept an `Exception` name, message,...
- `rescue` commonly used with `begin...end` to wrap codes that may raise Exception.

Ex:
```ruby
begin
 # Any number of Ruby statements go here.
 # Usually, they are executed without exceptions and
 # execution continues after the end statement.
rescue [ExceptionType,... => ex]
 # This is the rescue clause; exception-handling code goes here.
 # If an exception is raised by the code above, or propagates up
 # from one of the methods called above, then execution jumps here.
 # `ex` is Exception object return by `raise`
else
 # This code runs if has no Exception
ensure
 # This code runs in all condition. Like `finally`
end
```


# Advanced

## Iterators and Enumerators

Iterators are special methods in a **Enumerable object**, they accept a **[Block](#block)** and invoke that block with using `yield`.

Value return from block can be access by the return value of `yield` statement.

| Iterator | Object | Description | Example |
|---|---|---|---|
| `loop`  | `Kernel`  | Loop until the block execute a `return`, `break`,... | |
| `upto`, `downto` | `Integer` | Loop from instance to argument | `4.upto(6) {\|x\| print x} # 456` |
| `times` | `Integer` | Loop from 0 to (instance-1) | `3.times {\|x\| print x} # 012` |
| `step` | `Integer` | Loop from instance to argument1 with step is argument2 | `0.step(Math::PI, 0.5) {\|x\| p Math.sin(x)} # Loop through 0, 0.5, 1.0,..., 3.0` |
| `each`, `each_with_index` | [Enumerable](#enumerators) | Loop through each item in collection | `[1,2,3].each {\|x\| print x} # 123`<br>`[1,2,3].each_with_index {\|x, ind\| print x} # 123` |
| `collect`, `map` | [Enumerable](#enumerators) | Loop through each item in collection and return new array collected from return value | `[1,2,3].collect {\|x\| x*x} # [1, 4, 9]` |
| `select` | [Enumerable](#enumerators) | Filter the collection | `[1,2,3].select {\|x\| x%2==1} # [1, 3]` |
| `reject`, `reject!` | [Enumerable](#enumerators) | Filter the collection (reverse) | `[1,2,3].reject {\|x\| x%2==1} # [2]` |
| `inject` | [Enumerable](#enumerators) | Accumulate the collection. Initial value is the argument or the first item | `[1,2,3].inject {\|acc,x\| acc + x} # 6 (1+2+3)`<br>`[1,2,3].inject(4) {\|acc, x\| acc+x} # 10 (4+1+2+3)` |

### Custom Iterators

- Use `yield` to invoke the block after. Append block argument to `yield` separated by `,`.
- Use `block_given?` to check that user pass block for the iterator.
- The iterator can return a value.

```ruby
# This method expects a block. It generates n values of the form
# m*i + c, for i from 0..n-1, and yields them, one at a time, 
# to the associated block.
def sequence(n, m, c)
 i = 0
 while(i < n) # Loop n times
   yield m*i + c # Invoke the block, and pass a value to it
   i += 1 # Increment i each time
 end
end
```

### External Iterators

We can custom how an iterator loop through the element by calling `next` on it, this method return the next element on each call or raise `StopIteration` error when has no more. We can catch the error in `Kernel.loop` or using `rescue`.

### Enumerators

Enumerators are Enumerable objects (`Array, Hash, Range, IO,...`). We can define a Enumerator with `to_enum` (or `enum_for` defined for all `Object`) and pass it with a iterator (default is self-defined `each`).

## Method, Proc, Lambda and Closure

### Method

```ruby
def [object_expr.]name([param = default,...])
  # Method body here
end
```

- All are objects, so all functions are methods. Can use `self` to indicate the object call the method.
- Return value is the last expression or the value from [`return` statement](#return).
- A singleton method is method on just one object. Use expression to evaluate the object prepend the method name. If the object is Class (except `Numeric` and `Symbol`), the method is static method.
- Delete a method or (inherited method) with `undef`.
- Operator overloading can be redefine (not `undef`).
- One method can have many names with `alias [new_name] [original_name]`. Alias can be used in extend existing method avoid recursive
- Arguments do not require `()` for wrapping. But should only when has at most 1 argument
- Default parameters can accept an expression
- Arbitrary parameters can express with `*` (splat)
- Named arguments can be pass as Hash.
- Iterator method can express the block as the final parameter (prepend with `&`) and call it by `call` method on this parameter instead of using `yield` with anonymous block.


### Closure and Binding

Closures are like Partial Functions, they are invocable and can store its own state (variable, not just value). _proc_ and _lambda_ are Closures.

Binding open new way to change the behavior of Closure. If we pass a Closure to second parameter of `eval` method, and a String represent the statements as the first argument. The Binding of Closure will change with.

# OOP and Module

## Class

```ruby
class ClassName
  @@a = 0 # class variable

  # constructor, always private
  def initialize(x, ...)
     @x = x  # instance variable
  end

  # class method, default is public
  def self.method(x,...)
    ....
  end

  # instance method, default is public
  def method(x,...)
    ...
  end
end
```

```ruby
instance = ClassName.new(arguments)
instance.class                      # ClassName
instance.is_a? ClassName   # true
```

- `self` firstly is resolved as Class object.
- When initialize an instance, `self` points to that instance.
- All instance variables are **private**, must create a accessor (use prefix `=` for setter) for them -> All are methods. Easily the process by `attr_accessor :x, :y,...` (getter + setter) or `attr_reader :x, :y,...` (getter).
- All constants are **public**
- `public, protected, private` are all methods, the arguments of them are Symbol are String represent the instance method name.
- `private_class_method` is similar, they make Class Method private.
- `Struct` is immutable version of Class.

```ruby
Struct.new("ClassName", :x, :y,...)
ClassName = Struct.new(:x, :y,...)
```

## Module

Module is much like a Class, but they can not be initialized or inheritance, they are stand-alone, they are methods, classes,... wrapper.

Module is a way to separate Ruby source code in multiple files. These files are connected by `load` and `require`

# Examples

## Class

```ruby
class Book
  attr_accessor :title, :author, :n_pages   # define read + write attributes
  @@count = 0 # define class variable

  def initialize(title, author, n_pages) # Constructor
    @title = title
    @author = author
    @n_pages = n_pages
    @@count += 1
  end

  def to_s()   # Instance method, override Object.to_s
    @title + "-" + @author + ":" + @n_pages + " pages"
  end

  def self.help()
    "This class object describe a book with title, author and number of pages"
  end
end

book = Book.new "Hello World", "Me", 400 # Initialize a Book instance
# p "Book Count" + Book.count
p Book.help
```

###


