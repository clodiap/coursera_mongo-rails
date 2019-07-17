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

## Import Dataset
Download json example : <http://media.mongodb.org/zips.json>  
Run the import command :  
`$ mongoimport --db test --collection zips --drop --file zips.json` (the file `zips.json` is in the same directory in which the shell is running)

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

#### Find
`db[:zips].find(:city => "BALTIMORE")`
`db[:zips].find.distinct(:state)` distinct data from database
`db[:zips].find(:city => "GERMANTOWN").count`
`pp db[:zips].find(:city => "GERMANTOWN", :state => "NY").first`
__print all__
`db[:zips].find().each { |r| puts r }`
__pretty printing__
`require 'pp'`
`db[:zips].find().each { |r| pp r }`

#### "_id" field
- `_id` is the primary key for every document
- default field for BSON object, indexed automatically
- can add a custom `id` (different from `_id`)
  
#### Projections
similar to SELECT in SQL
__true or 1__ = inclusive
__false or 0__ = exclusive
`db[:zips].find(:state => "MD").projection(state:true).first`
`db[:zips].find(:state => "MD").projection(state:true, _id:false).first`

#### Paging

##### Limit and skip
__skip(n)__ : skip n results  
`db[:zips].find.skip(3).limit(3).each { |r| pp r }` -> skips the first three documents and retrieves a list of three documents

__limit(n)__ : limit result length to n results  
`db[:zips].find.limit(3).each { |r| pp r }` -> retrieves a list of the first three documents

##### Sort
Specifies the order in which the query returns matching documents
- `{ field:value }`
__values of sort__  
- `1` for ascending : `db[:zips].find.limit(3).sort({ :city => 1 }).each { |r| pp r }`
- `-1` for descending : `db[:zips].find.limit(3).sort({ :city => -1 }).each { |r| pp r }`


#### Advanced find

- Evaluations `lt` & `gt` (less than & greater than)
  - `db[:zips].find(:city => { :$lt => 'D' }).limit(2).to_a.each { |r| pp r }`
  - `db[:zips].find(:city => { :$lt => 'D' }).limit(2).to_a.each { |r| pp r };nil` -> `;nil` at the end removes the trailing (what the shell returns)
  - `db[:zips].find(:city => { :$lt => 'P', :$gt => 'B' }).limit(3).to_a.each { |r| pp r }`
- Regex
  - `db[:zips].find(:city => { :$regex => 'X' }).limit(5).each { |r| pp r }` -> cities with an 'X' in their names
  - `db[:zips].find(:city => { :$regex => 'X$' }).limit(5).each { |r| pp r }` -> cities ending with an 'X'
  - `db[:zips].find(:city => { :$regex => '^X' }).limit(5).each { |r| pp r }` -> cities beginning with an 'X'
  - `db[:zips].find(:city => { :$regex => '^[A-E]' }).limit(5).each { |r| pp r }` -> cities beginning with 'A', 'B', 'C', 'D' and 'E'
- Exists (returns the documents that have the field in question)
  - `db[:zips].find(:city => { :$exists => true }).projection( { :_id => false } ).limit(3).to_a.each { |r| pp r }`
- Not (selects the documents that do not match the operator expression (lt & gt))
  - `db[:zips].find(:pop => {'$not' => {'$gt' => 9500}} ).projection( { :_id => false } ).limit(20).to_a.each { |r| pp r }`
- Type (selects the documents where the value of the field is an instance of the specified numeric BSON type)
  - [Mongo BSON types]<https://docs.mongodb.com/manual/reference/bson-types/>
  - `db[:zips].find({:state => {'$type' => 2}}).first` BSON type 2 = string


### Replace
__replace_one__ : replace a document in the collection

```ruby
db[:zips].insert_one( :_id => "101", :city => "citytemp", :loc => [-76.05922700000001, 39.564894], :pop => 4678, :state => "MD" )
db[:zips].find( :_id => "101" ).replace_one( :_id => "101", :city => "city02", :loc => [-78.22, 36.22], :pop => 2000, :state => "MD" ) 
db[:zips].find( :_id => "101" ).to_a
```

### Update
__update_one__ : change only a set of values  
`db[:zips].find( :_id => "101" ).update_one( :$set => { :city => "name2" } )`

__update_many__ : updates single or multiple documents
```ruby
db[:zips].find( :state => 'MD' ).count
db[:zips].find( :state => 'MD' ).update_many( :$set => { :state => 'XX' } )
db[:zips].find( :state => 'XX' ).count
```
#### inc
__$inc__ : to increment a value
`db[:zips].find( :city => "BALTIMORE" ).first.update_one( :$inc => { pop: 1000 } )` increments the value of pop by 1000

#### Upsert
if upsert is __true__ and __no__ document matches the query criteria, update() inserts a single document

`db[:zips].find( :city => "ODENVILLE1" ).count`
`db[:zips].find( :city => "ODENVILLE1" ).update_one({ :$set => { :city => "ODENVILLE2" }}, :upsert => true )`

### Delete
__delete_one__ : delete one document  
`db[:zips].find( :city => "name2" ).count`
`db[:zips].find( :city => "name2" ).delete_one()`

__delete_many__ : delete single or multiple documents  
`db[:zips].find( :state => 'MD' ).count`  
`db[:zips].find( :state => 'MD' ).delete_many()`

