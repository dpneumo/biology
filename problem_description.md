For a jekyll based static site am attempting to use jekyll-picture-tag which requires mini_magick which in turn requires ImageMagick. Am developing on Windows 7. Running bundle exec jekyll build yields this error:

```
  Liquid Exception: No such file or directory - identify -ping C:/Users/loco/AppData/Local/Temp/mini_magick20171128-6736-1ea8fu7.jpg in index.html
```

Tracking this down with some 'puts debugging' shows that the error is generated in

# MiniMagick::Image.open:
```
module MiniMagick
  class Image
    # ---
    class << self
      # ---

      def open(file_or_url, ext = nil)
        file_or_url = file_or_url.to_s # Force String... Hell or high water
        if file_or_url.include?('://')
          require 'open-uri'
          ext ||= File.extname(URI.parse(file_or_url).path)
          Kernel.open(file_or_url) do |f|
            read(f, ext)
          end
        else # This path is taken
          ext ||= File.extname(file_or_url)
          puts file_or_url
          # -->> C:/Users/loco/Documents/GitHub/biology/assets/imgs/_fullsize/texas_grasslands.jpg
          File.open(file_or_url, 'rb') do |f|  # <<-- Error occurs here.
            read(f, ext)
          end
        end
      end

      # ---
    end
  end
end
```

Have given all users full control on C:/Users/loco/AppData/Local/Temp without effect.

A Google search does not turn up an explanation for the error generated. Review of the involved code does not suggest a explanation to me. So am hoping that other eyes will see what I have done to cause this error and suggest a route to fix it.

Setup:
  Windows 7 Pro, 64bit
  ruby 2.4.2p198 (2017-09-14 revision 59899) [x64-mingw32]
  jekyll 3.6.2
  jekyll-picture-tag 0.3.0
  mini_magick 3.8.1
  ImageMagick 6.9.9
  Visual C++ 2013 Redistributable Package (x64) - 12.0.40660

# Gemfile:
```
source "https://rubygems.org"
gem 'bootstrap', '~> 4.0.0.beta2'
gem "github-pages", group: :jekyll_plugins
group :jekyll_plugins do
  gem 'jekyll-picture-tag', '~> 0.3.0'
end
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
```

# _config.yml:
```
#Site settings
title: Biology
email: your-email@example.com
description: >- # this means to ignore newlines until "baseurl:"
  This site hopes to provide usefull and entertaining information about biology.
baseurl: "/biology"
url: ""

#Build settings
markdown: kramdown
plugins:
  - jekyll-picture-tag
permalink: pretty

repository: me/me.github.io
github: [metadata]

picture:
  source: "assets/imgs/_fullsize"
  output: "generated"
  markup: "picture"
  presets:
    default:
      ppi: [1, 1.5]
      source_medium:
        media: "(min-width: 40em)"
        width: "600"
        height: "300"
      source_default:
        width: "300"
```

# _layouts/default.html:
```
<!doctype html>
<html>
  {% include head.html %}
  <body>
    {{ content }}
    {% include bs_scripts.html %}
  </body>
</html>
```

# _includes/head.html:
```
<head>
  <title>Biology</title>

  <!-- Bootstrap required meta tags -->
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="chrome=1">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

  <!-- Bootstrap CSS -->
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.2/css/bootstrap.min.css" integrity="sha384-PsH8R72JQ3SOdhVi3uxftmaW6Vc51MKb0q5P2rRUpPvrszuE4W1povHYgTpBfshb" crossorigin="anonymous">

  <!-- Picturefill responsive image polyfill -->
  <script>
    // Picture element HTML5 shiv
    document.createElement( "picture" );
  </script>
  <script src="picturefill.js" async></script>
</head>
```

# _includes/bs_scripts.html
```
<!-- jQuery -->
<script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
<!-- Popper -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.3/umd/popper.min.js" integrity="sha384-vFJXuSJphROIrBnz7yo7oB41mKfc8JzQZiCq4NCceLEaO4IHwicKwpJf9c9IpFgh" crossorigin="anonymous"></script>
<!-- Bootstrap Core JavaScript -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.2/js/bootstrap.min.js" integrity="sha384-alpBpkh1PFOepccYVYDB4do5UnbKysX5WZXm3XxPqe5iKTfUKjNkCk9SaVuEZflJ" crossorigin="anonymous"></script>
```

# index.html:
```
---
layout: default
---
{% picture texas_grasslands.jpg %}
```
