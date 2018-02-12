---
title: "8주차 월요일 수업자료(AWS S3)"
tag: material
author: "정다혜"
---

#### **C9**으로 프로젝트를 생성할 경우에는 <b style= "color: red;">`private`</b>로 설정해주세요!

1. c9 프로젝트 생성
2. c9 프로젝트 오른쪽 상단 에 [share]클릭
3.  public 체크를 해제주세요.

** AWS access key**를 사용하므로 반드시 C9을 비공개로 설정해주세요.
 private 설정의 워크스페이스는 계정당 최대 1개까지 만들 수 있습니다.

#### AWS - AWS Access Key 생성하기

프로젝트 안의 멀티미디어 파일을 따로 저장해두는게 좋다. 분리해 저장하면 트래픽 분산과 과금정책에도 유리하다. 우리는 AWS S3를 이용할 것이다.

1. 내 계정 - `My Security Credentials`
2. Access Keys (Access Key ID and Secret Access Key)
3. Create New Access Key
4. ID와 KEY를 따로 기록해두거나 Download Key File을 해주세요.

##### <span style="color: red;">이 키는 노출이 되지않도록 조심해주세요. 노출이 될경우 엄청난 과금이 될 수 있습니다.</span>

#### Gem Carrierwave , for-aws

- `carrierwave` :	레일즈에서 이미지, PDF, 미디어 파일을 쉽게 쓸 수 있게 해준다.  [[ 바로가기 ](https://github.com/carrierwaveuploader/carrierwave)]

- `fog-aws` : Amazon S3를 사용하기 위한 잼	[[ carrierwave AWS S3 바로가기 ](https://github.com/carrierwaveuploader/carrierwave#using-amazon-s3)]

```
# Gemfile
gem 'carrierwave', '~> 1.1'
gem "fog-aws"
```

`bundle install` 하기

```ruby
CarrierWave.configure do |config|
  config.fog_provider = 'fog/aws'
  config.fog_credentials = {
    provider:              'AWS',                        # required
    aws_access_key_id:     '여기에 ID 값',                 # required
    aws_secret_access_key: '여기에 KEY 값',                # required
    region:                'eu-west-1',                  # optional, defaults to 'us-east-1'
    host:                  's3.example.com',             # optional, defaults to nil
    endpoint:              'https://s3.example.com:8080' # optional, defaults to nil
  }
  config.fog_directory  = 'name_of_directory'                          # required
  config.fog_public     = false                                        # optional, defaults to true
  config.fog_attributes = { cache_control: "public, max-age=#{365.day.to_i}" } # optional, defaults to {}
end
```

위의 스크립트 복사 후 `config/initializers/파일명.rb` 파일 생성한 후 전체 복사 붙여넣기

<span style="color: red;">ACCESS KEY, ID값은 github이나 c9에 올릴때 유출되지않도록 주의해야한다.</span>

- [AWS region](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)에서 서울에 해당하는 것을 찾아서 변경
- host는 필요없으니 지워준다.

```ruby
CarrierWave.configure do |config|
  config.fog_provider = 'fog/aws'
  config.fog_credentials = {
    provider:              'AWS',                        # required
    aws_access_key_id:     '여기에 ID 값',                 # required
    aws_secret_access_key: '여기에 KEY 값',                # required
    region:                'ap-northeast-2',
    endpoint:              'https://s3.ap-northeast-2.amazonaws.com' # optional, defaults to nil
  }
  config.fog_directory  = 'name_of_directory'                          # required
  config.fog_public     = false                                        # optional, defaults to true
  config.fog_attributes = { cache_control: "public, max-age=#{365.day.to_i}" } # optional, defaults to {}
end
```

#### AWS S3 생성

1. AWS Storage - S3
2. create bucket
3. Bucket name 설정, Region은 위의 스크립트와 같게 Seoul로 지정해준다.
4. create

```ruby
CarrierWave.configure do |config|
  config.fog_provider = 'fog/aws'
  config.fog_credentials = {
    provider:              'AWS',                        # required
    aws_access_key_id:     '여기에 ID 값',                 # required
    aws_secret_access_key: '여기에 KEY 값',                # required
    region:                'ap-northeast-2',
    endpoint:              'https://s3.ap-northeast-2.amazonaws.com' # optional, defaults to nil
  }
  config.fog_directory  = 'konkuklikelion'                          # required
  config.fog_public     = true                                        # optional, defaults to true
  config.fog_attributes = { } # optional, defaults to {}
end
```
위의 스크립트 처럼 여기서 세팅한 Bucket name을 `config.fog_directory`에 입력,   `config.fog_public` true로 변경하거나 지워준다.`config.fog_attributes`도 지워준다.

#### 파일업로드

```
$ rails generate uploader Avatar
```

`app/uploaders/avatar_uploader.rb` 파일이 생성되는 것을 확인 할 수 있다.

```ruby
class AvatarUploader < CarrierWave::Uploader::Base

  # storage :file
  storage :fog
  
  def store_dir
    # aws에서 s3에 파일이 저장되는 경로 설정
    "uploads/"
  end
end
```

`storage :fog`를 설정해준다.

#### 콘솔창에서 AWS에 올라가는지 확인하기

- `gem install carrierwave`
- `rails c`

이때 이미지파일 아무거나 한개를 프로젝트에 넣어둔다.

```
irb(main):001:0> uploader = AvatarUploader.new
NameError: uninitialized constant AvatarUploader
```
와 같은 에러가 발생하면 밑의 스크립트를 추가해준다.

```ruby
# config/application.rb
require 'carrierwave/orm/activerecord'
```

```
irb(main):001:0> uploader = AvatarUploader.new
=> #<AvatarUploader:0x007fb171027e78 @model=nil, @mounted_as=nil>
irb(main):002:0> file = open("logo.png")
=> #<File:logo.png>
irb(main):003:0> file.inspect
=> "#<File:logo.png>"
irb(main):005:0> uploader.store!(file)
=> [:store_versions!]
```

를 하고 AWS S3를 확인해보면 파일이 올라가 있는 것을 볼 수 있다.

#### 파일 입력 폼 생성

```
rails g controller home index upload
```

```ruby
# routes.rb
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'
  post 'home/upload'
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

```xml
<!-- index.html.erb -->
<h1>Home#index</h1>
<p>Find me in app/views/home/index.html.erb</p>
<!-- 미디어를 받는 경우에는 multipart를 꼭 추가해준다.-->
<form action="/home/upload" method ="post" enctype="multipart/form-data">
<input type="file" name="pic" accept="/image/*">
<input type="submit"> </form>
```

```ruby
# home_controller.rb
class HomeController < ApplicationController
  def index
  end

  def upload
  	file = params[:pic]
	uploader = AvatarUploader.new
    uploader.store!(file)
	redirect_to "/home/index"
  end
end
```

```ruby
# application_controller.rb
class ApplicationController < ActionController::Base
 # 주석처리해준다.
 # protect_from_forgery with: :exception
end
```

파일을 직접 올린 후 AWS S3에 가서 확인해보면 올라와있는 것을 확인 할 수 있다.

#### 파일 이름 바꾸기

파일명이 중복될 경우 덮어쓰기가 될 수 있으므로 파일명을 임의로 지정할 수 있다.

```ruby
# app/uploads/avatar_uploader.rb
class AvatarUploader < CarrierWave::Uploader::Base

 ...

  def filename
    [*('a'..'z')].sample(20).join+"."+ file.extension if original_filename
  end

end
```

여기서 `file.extension`은 원래 파일 확장자로 저장이되도록 설정한 것이다.

#### MiniMagick을 이용한 파일사이즈 변경

- `minimagick`[바로가기](https://github.com/minimagick/minimagick)

```
# Gemfile

gem "mini_magick"
```

`bundle install`을 해준다.

> 만약에 안된다면 bash창에 `sudo apt-get update` 입력 후 끝나면 `sudo apt-get install imagemagick`

```ruby
class AvatarUploader < CarrierWave::Uploader::Base

  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  include CarrierWave::MiniMagick

  ...

  # Create different versions of your uploaded files:
  version :thumb do
    process resize_to_fit: [50, 50]
  end

  version :medium do
    process resize_to_fit: [250, 250]
  end

  ...
end
```
`include CarrierWave::MiniMagick`의 주석을 지워주고, 다른 버전별로 이미지 사이즈를 지정해준다. 그 후 이미지를 저장해서 확인해보면 사이즈별 이미지가 올라간 것을 확인할 수 있다.


## Author

written by [정다혜](https://dh00023.github.io).

![](https://avatars.githubusercontent.com/dh00023?v=2&s=100)

<a href="https://dh00023.github.io" target="_blank" class="btn btn-black"><i class="fa fa-github fa-lg"></i> Visit on Github Page &rarr;</a>