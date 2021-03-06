# Уеб приложения със Sinatra

Това ръководство има за цел да ви даде някои насоки ако сте избрали уеб сайт за проект. Повечето от нещата тук са препоръки и със сигурност има повече от един правилен начин да се направят.

## Sinatra crash course

Sinatra е DSL, който ви предоставя минимален набор от инструменти за създаване на уеб приложения. За разлика от Rails и други подобни на него, Sinatra не ви дава готова и твърда структура, налагайки ви коя част от кода ви къде да стои. Бихте могли да сложите всичко в един файл (което, разбира се, не трябва да правите).

Най-добрият източник на информация за Sinatra е [документацията](http://www.sinatrarb.com/intro.html).

## Rack

Sinatra, както и повечето frameworks в света на Ruby, използва [Rack](http://rack.github.io/). Това е много полезно, защото Rack може да бъде разширяван чрез добавяне на посредници (middleware). Силно вероятно е ако ви липсва някаква функционалност, да намерите Rack посредник точно за това. Затова търсете и за `rack <feature>` в Google, вместо само `sinatra <feature>`. Например `Rack::Session::Cookie` добавя възможност за използване на сесия чрез бисквитки.

## База от данни

Щом ще правите уеб приложение е почти сигурно, че ще ви се наложи да използвате някаква база от данни. Конкретната база от данни, която ще изберете, за нас не е от значение.

### SQLite

SQLite е проста за използване SQL база от данни. Не се слави с голяма производителност, но е идеална докато разработвате проекта си, защото се "подкарва" лесно. Единственото, което ви е необходимо, е да добавите `gem 'sqlite3'` в `Gemfile`-а. _Забележка: Ако използвате windows може да се наложи да инсталирате допълнителни неща. [Този пост в github може да помогне.](https://github.com/sparklemotion/sqlite3-ruby/issues/82#issuecomment-18595074)_

Една база от данни при SQLite е просто един `something.sqlite` файл.

### ORM и ActiveRecord

ORM (Object-Relational Mapping) библиотеките като ActiveRecord, Sequel, DataMapper и други ви предоставят удобен интерфейс за работа с SQL бази от данни. Ключовите неща са, че (почти) не ви се налага да пишете SQL заявки и можете лесно да превключвате между различни бази от данни (SQLite, PostgreSQL, MySQL и т.н.).

За интегриране на ActiveRecord и Sinatra, може да използвате [Sinatra-ActiveRecord](https://github.com/janko-m/sinatra-activerecord). Добро ръководство за ActiveRecord има [тук](http://guides.rubyonrails.org/active_record_basics.html).

#### Миграции

Почти всеки ORM ви предоставя възможност да създавате и изпълнявате миграции. Една миграция представлява инструкция за промяна на структурата на база от данни. Например добавяне на таблица, добавяне или премахване на поле от таблица и т.н. Миграцията обикновено е просто Ruby код, който вика специални методи, които впоследствие, при "изпълнение" на миграцията, се транслират до съответните команди към SQL базата данни и така се извършва необходимата промяна.

Ръководство за създаване на ActiveRecord миграции има [тук](http://guides.rubyonrails.org/active_record_migrations.html). Единствената разлика при Sinatra-ActiveRecord е, че вместо `bin/rails generate migration <име>`, трябва да използвате `rake db:create_migration NAME=<име>`.

Изводът от тази секция е, че не трябва да се грижите ръчно за структурата на базата си данни. Това трябва да става само през миграции.

Миграциите могат да се ползват и за промяна на данни, не само на структура. Понякога възниква необходимост от това, като част от промяната на структурата.

## Файлова структура

Както вече споменахме, Sinatra не ви дава готова файлова структура с инструкции къде какъв код да пишете. Затова е и много лесно да се вземе не чак толкова добър подход.

Хубаво е да си разделите проекта на следните директории:

- `routes` - Вътре слагате ruby файлове, които използват DSL-a на Sinatra. Всеки файл в тази директория отговаря за няколко адреса, свързани логически по някакъв начин. Например в `routes/users.rb` може да се съдържат адресите за регистрация, вход, редакция на потребителски профил и други дейности, свързани с потребители. Ето примерен такъв файл, който съдържа един адрес (`users/all`), на който се показва списък от всички потребители на сайта:

    ```ruby
    get '/users/all' do
      @users = User.all
      erb :'users/all' # Това е пътят към темплейта в папката `views`. Обърнете внимание на това, че е символ.
    end
    ```

- `models` - Тук е мястото на моделите. Моделите са класове, чиято задача е да си комуникират с базата и да моделират някаква таблица от нея. Обикновено се създава по един клас за всяка такава. Ако използвате ORM, почти няма да ви се налага да добавяте методи към тези класове. Ако не използвате ORM, ще трябва да си създадете методи за нещата, които ще използвате. В горния пример сме използвали класов метод `all` на модела `User`. Примерна реализация за този модел може да е следният код:

    ```ruby
    class Users
      def self.all
        # Забележка - db е връзката към базата от данни
        db.execute('select * from users').to_a
      end
    end
    ```

  Горният пример не използва ORM - в ActiveRecord вече имате дефиниран такъв метод, който можете да използвате наготово.

- `views` - Това е мястото за HTML кода. Можете да използвате различни темплейт системи като ERB, HAML и т.н. Разликата е (почти) единствено в начина, по който се вгражда Ruby код в HTML. Целта на всеки template engine е след изпълнение, да се продуцира HTML код. ERB е част от стандартната библиотека и е по-праволинеен и лесен за схващане, затова ако се чудите кое да изберете - използвайте него. Ползва се доста масово. Всеки файл в директорията `views` съдържа шаблони на HTML страници в съответния формат на template engine-а. Например един `erb` файл (`views/users/all.erb`) за страницата `users/all` може да съдържа следното:

    ```erb
    <h1>Всички потребители</h1>
    <% if @users.any? %>
      <ul>
        <% @users.each do |user| %>
          <li><%= user.name %></li>
        <% end %>
      </ul>
    <% else %>
      <p>Все още няма регистрирани потребители.</p>
    <% end %>
    ```

  Хубаво е файловете да бъдат групирани по това за кои адреси и модели се отнасят - за всеки файл в `routes` тук ще има по една директория, с `erb` файлове за всеки адрес.

  Ако сложите файл с име `layout.erb`, той автоматично ще "опакова" всички останали темплейти.

    ```erb
    <html>
      <head>
        <title>Моят мега як сайт</title>
      </head>
      <body>
        <!-- На мястото на този yield ще се появава съдържанието на темплейта,
             който е зададен за показване. Например, `views/users/all.erb`
             ще бъде вмъкнат в този html и резултатът ще бъде пратен на браузъра. -->
        <%= yield %>
      </body>
    </html>
    ```

- `public` - Тук е мястото на всичко, което не е Ruby код - CSS, картинки, шрифтове, JavaScript код и подобни неща, разбира се добре разделени в подпапки. От тази папка (и нейните наследници), Sinatra директно изпраща файловете на браузъра. Например, ако имате файл `public/img/картинка.png`, то на адреса `http://<сайт>/img/картинка.png` ще се показва тази картинка.

### Външни зависимости

Най-вероятно ще ви се налага да ползвате външни зависимости, освен Sinatra. За целта е силно препоръчително да ползвате [Bundler](http://bundler.io). Инсталира се с `gem install bundler` и се ползва с `bundle` (без "r"). Документацията на сайта на Bundler е добра.

### Главен файл на приложението

Всички файлове от `routes` и `models` трябва да бъдат require-нати във файла, в който се зарежда Sinatra (`app.rb`, `server.rb`, `app.ru` в зависимост от това кой туториал за Sinatra следвате). Този файл може да наричаме "главен"/"стартов" файл на вашето уеб приложение. Той би могъл да изглежда така (ако приемем, че използвате Bundler):

```ruby
require 'rubygems'
require 'bundler/setup'
require 'sinatra'
require 'sinatra-activerecord'
# More require statements...

require_relative 'models/users'
require_relative 'models/products'
require_relative 'models/orders'
# More require statements...

require_relative 'routes/users'
require_relative 'routes/orders'
require_relative 'routes/products'
require_relative 'routes/pages'
# More require statements...
```

### Обобщение на архитектурата (MVC)

Файловете, които има в `routes` обикновено се наричат контролери - тяхната задача е да използват моделите и чрез изгледите (views) да показват неща на екрана (т.е. да генерират HTML страници, които да пращат на браузъра за визуализация).

Задачата на моделите (класовете в `models`) е да си комуникират с базата от данни, така че другите компоненти от системата да не знаят конкретно с каква база от данни се работи, как точно са структурирани данните там и да няма SQL код навсякъде.

Изгледите просто генерират HTML от подадените им данни.

Този подход за структуриране на код се нарича MVC (Model-View-Controller).

Ако ви трябва по-специална логика, която не пасва на някое от трите места, може да създадете папка lib, в която да се намират класове, които са необходими за работата на проекта. Тези класове може да се използват в контролерите или моделите.

## Тестване

За да тествате вашето уеб приложение, обикновено правите комбинация от две неща:

1. Тествате вашите модели и класове с unit-тестове. RSpec или MiniTest ще ви свършат много добра работа тук.
2. Тествате логиката в контролерите и изгледите (която би трябвало да е малко и само сглабяща) накуп, с интеграционни тестове. Тези интеграционни тестове симулират HTTP-заявки. За целта бихте могли да ползвате Capybara, в комбинация с RSpec или нещо сродно.

Може да намерите повече информация за това в [лекцията за тестване](http://2016.fmi.ruby.bg/lectures/09-testing#1).
