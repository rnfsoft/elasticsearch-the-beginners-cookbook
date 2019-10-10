

index = database

type = table

document = row

#### All the indexes that have been created

	GET http://localhost:9200/_cat/indices

#### Set Header in Postman

	Content-Type as application/json
	
#### Create a new document with id

	PUT http://localhost:900/movies/movie/<id>

#### Delete a new document with id
	
    DEL http://localhost:900/movies/movie/<id>
	

#### Delete an index
	
    DEL http://localhost:9200/movies

#### Create an index
	
    PUT http://localhost:9200/movies
	
#### Update _mapping before inserting a document
	
    PUT http://localhost:9200/movies/_mapping 
		{
			"properties": {
				"genre": {
					"type": "keyword"
				},
				"metascore": {
					"type": "long"
				},
				"name": {
					"type": "text"
				},
				"rating": {
					"type": "float"
				},
				"summary": {
					"type": "text"
				},
				"votes": {
					"type": "long"
				},
				"yearofrelease": {
					"type": "long"
				}
			}

		}

#### Create a new document with id
	
    POST http://localhost:9200/movies/_doc/1
		{
			"name":"Justice League", 
			"genre":"Action",
			"summary":"Fueled by his restored faith in humanity and inspired by Superman's selfless act, Bruce Wayne enlists the help of his newfound ally, Diana Prince, to face an even greater enemy",
			"yearofrelease":201, 
			"metascore":45, 
			"votes":275122, 
			"rating":6.6
			
		}

#### Crete multiple documents using bulk

Each and last line separed with \n (Enter); {"index" :{}} defines indexing the document; define each record index, update, delete and document
	
	POST http://localhost:9200/movies/_doc/_bulk
		{ "index" : { } }
		{  "name":"Thor Ragnarok",  "genre":"Action", "summary":"Thor is imprisoned on the planet Sakaar, and must race  against time to return to Asgard and stop Ragnarök, the destruction of his world, at the hands of the powerful and ruthless villain Hela", "yearofrelease":2017, "metascore":74, "votes":374270, "rating":7.9 }
		{ "index" : {} }
		{  "name":"Infinity War",  "genre":"Sci-Fi",   "summary":"The Avengers and their allies must be willing to sacrifice all in an attempt to defeat the powerful Thanos before his blitz of devastation and ruin puts an end to the universe", "yearofrelease":2018, "metascore":68, "votes":450856, "rating":8.6 }
		{ "index" : {} }
		{  "name":"Christopher Robin",  "genre":"Comedy", "summary":"A working-class family man, Christopher Robin, encounters his childhood friend Winnie-the-Pooh, who helps him to  rediscover the joys of life", "yearofrelease":2018, "metascore":60, "votes":9648, "rating":7.9}

#### Search all documents
	
    GET http://localhost:9200/movies/_doc/_search
	
took: the time taken

_shards: number of shards searched, each index is split into multiple shards.

hits.total: total number of records matched

hits.maxscore: score of the most relevant

hits.hits: search resutls

#### Search contains the word

	GET http://localhost:9200/movies/_search?q=Action
	GET http://localhost:9200/movies/_search?q=genre:Action
	GET http://localhost:9200/movies/_search
		{
			"query": {
				"match" : {
					"genre" : "Action"
				}
			}
		}
	GET http://localhost:9200/movies/_search
		{
		 "query": {
			  "multi_match": {
				 "query": "defeated",
				 "fields": ["summary"]
			}
		  }
		}

#### Inverted Index

splitting the documents into tokens, listing all the unique words and then marking all the documents where the word appears.

    Term     | Document1 | Document2 |
    -----------------------------------
    I     	 |     X     |           |
    like   	 |     X     |           |
    You		 |     X     |     X     |
    are      |     X     |           |
    a        |           |     X     |
    good  	 |           |     X     |
    student  |           |     X     |
			
Document 1 is higher score because more matches

    Term     | Document1 | Document2 |
    -----------------------------------
    I     	 |     X     |           |
    like   	 |     X     |           |
    You		 |     X     |     X     |

#### Get settings

	GET http://localhost:9200/movies/_settings
	
number_of_shards: number of sub-indices

number_of_replicas: number of copies of index's shards, for data safe


#### Get mappings

the schema of the document, data-type of all fields

	GET http://localhost:9200/movies/_mappings
	
text: datatype, used for full-text search, converted into individual words before indexed

keyword: datatype, exact matches, not analyzed, stored as it is

fields.keyword: index the same field in different ways

#### How to use Analyzer

way data gets analyzed, standard analyzer: splits text into individual words then to lowercase
		
	POST http://localhost:9200/_analyze
		{
			"analyzer":"standard",
			"text":"How to create a contact"
		}
		
#### All together

	PUT http://localhost:9200/newmovies	
		{
			"settings":{
			  "analysis":{
				 "filter":{
					"stemmer":{
					   "type":"stemmer",
					   "language":"english"
					},
					"stopwords":{
					   "type":"stop",
					   "stopwords":[
						  "_english_"
					   ]
					}
				 },
				 "analyzer":{
					"custom_analyzer":{
					   "filter":[
						  "stopwords",
						  "lowercase",
						  "stemmer"
					   ],
					   "type":"custom",
					   "tokenizer":"standard"
					}
				 }
			  }
		   },
		   "mappings":{

				 "properties":{
					"genre":{
					   "type":"keyword"
					},
					"metascore":{
					   "type":"long"
					},
					"name":{
					   "type":"text",
					   "analyzer":"custom_analyzer",
					   "search_analyzer":"custom_analyzer"
					},
					"rating":{
					   "type":"float"
					},
					"summary":{
					   "type":"text",
					   "analyzer":"custom_analyzer",
					   "search_analyzer":"custom_analyzer"
					},
					"votes":{
					   "type":"long"
					},
					"yearofrelease":{
					   "type":"long"
					}
				 }
			  }
		}
		
	stemmer: root of the given word
	stopwords: remove all stop words

#### Custom Analyzer

	POST http://localhost:9200/newmovies/_analyze
		{ 
		  "analyzer":"custom_analyzer",
		  "text":"How to create a contact"
		}

#### Copy index

copy the documents from one index to another, doesn't copy setting and mapping	
	
	POST http://localhost:9200/_reindex
		{
		  "source": {
			"index": "movies"
		  },
		  "dest": {
			"index": "newmovies"
		  }
		}
		
#### Search using custom analyzer

	GET http://localhost:9200/newmovies/_search?q=defeated

#### References:

https://medium.com/@animeshblog/elasticsearch-the-beginners-cookbook-1cf30f98218
