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
