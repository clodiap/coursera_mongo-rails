# Ruby on Rails Web Services and Integration with MongoDB

Following this course : <https://www.coursera.org/learn/ruby-on-rails-web-services-mongodb/home/welcome>


## CheatSheet

### start and stop MongoDB server
`$ sudo service mongodb start`  
`$ sudo service mongodb stop`

### MongoDB configuration file 
/etc/mongodb.conf

### installing gems on system in order to use mongo in irb
`$ gem install mongo`  
`$ gem install bson_ext`

## in IRB
`require 'mongo'`  
`db = Mongo::Client.new('mongodb://localhost:27017')`  
`db=db.use('databasename')` determine which database to use  
`db.database.name` shows the name of the database in use  
`db.database.collection_names` shows the collections  
`db[:zips].find.first` shows the first element of the `zips` collection  
`Mongo::Logger.logger.level = ::Logger::INFO` removes the DEBUG informations from the shell  

## CRUD

### Create
#### insert_one
`db[:zips].insert_one(:_id => "100", :city => "city01", :loc => [-76.05922700000001, 39.564894], :pop => 4678, :state => "MD")`  
`db[:zips].find(:city => "city01").count`  
`db[:zips].find(:city => "city01").to_a` display the element

#### insert_many
``` ruby
db[:zips].insert_many([
  { 
    :_id => "200", :city => "city02", 
    :loc => [-74.05922700000001, 37.564894], 
    :pop => 2000, :state => "CA"
  },
  {
    :_id => "201", :city => "city03", 
    :loc => [-75.05922700000001, 35.564894], 
    :pop => 3000, :state => "CA"
  }
  ])
```

### Read

## "_id" field
- `_id` is the primary key for every document
- default field for BSON object, indexed automatically
- can add a custom `id` (different from `_id`)

