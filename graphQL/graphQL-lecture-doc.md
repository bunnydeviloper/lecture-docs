# Lecture doc for graphQL

## Background
REST API  
/:roomId  
/:roomId/messages  
/:roomId/messages/:messageId  
/:roomId/moremessages/:start/:amount  

GraphQL  
/graphql  

**Advantage**: 
* Combine multiple URLs to 1 URL to get data
* Iterate more quickly without touch backend

## Basic

**SCHEMA**: typedef and resolver
```javascript
const schema = makeExecutableSchema(typeDefs, resolvers});
app.use('/graphiql', graphiqlExpress({ endpointURL: '/graphql' }));
app.use('/graphql', cors(), bodyParser.json(), graphqlExpress({ schema }));
```

**GraphIQL**: user interface

**TYPEDEF**
*Define structure of data*

```javascript
const typeDefs = `
scalar Long

type Query { 
  users(id: Int, name: String, is_active: Boolean, whereJoin: String): [User]
  getUsersInRoom(id: Int): [User]
  getUsersNotInRoom(id: Int): [User]
  getUserRooms(id: Int): [Room]
  getOtherRooms(id: Int): [Room]
}

type Room {
  id: Int
  name: String
  description: String
  is_editable: Boolean
  is_private: Boolean
}
`;
```

**RESOLVER**
*How to get data: send query to db and return data*  
**NOTE**: format of data request has to match with typedef above

```javascript
const resolvers = {
  Long: GraphQLLong,
  Query: { ... },
  getUserRooms: (obj, args) => {
    return new Promise((resolve, reject) => {
      db.query(
        `SELECT id, name, description, is_editable, is_private
	 FROM user_room
	 JOIN room
	 ON id=room_id
	 WHERE user_id = ?`,
	[args.id],
	(error, results, fields) => {
	  if (error) return reject(error);
	  resolve(results);
	}
      );
    });
  }
};
```

**CLIENT**
*Change function call we use to GET/UPDATE/DELETE data on front end*

```javascript
getUserRooms: function(userId, cb) {
  sendQuery(gql`
  		{
		  getUserRooms(id: ${userID}) {
		    room_id: id
		    name
		    description
		    is_editable
		    is_private
		  }
		}`)
    .then(response => {
    loader.store.dispatch({
      type: 'CHAT_ROOMS_LOADED',
      rooms: response.data.getUserRooms
    });
    cb();
    });
},
```

## Process
1. Typedef
2. Resolver
3. Client
4. Test
5. Remove old code


## Advance

Mutation

## Personal notes
- Make sure you're running both client side and server side.
- Make sure you change backend URLs to your current server url  
(src/db/chatdb.js -> BASE url and Domain url)
- Simple demonstration without touching database:
```javascript
typeDef: getSophia: String
resolver: return 'Hello' (no need to return Promise and all that)
```
- Whenever code is modified -> refresh GraphIQL
- Test on GraphIQL:
	- First: (after modified typedef and resolver) check the response, this is still RestAPI response.
	- Seconde: modified client side (chatdb.js), check the response again, this is GraphQL response. Make sure they match!
	- Third: check back your client side on browser, make sure nothing breaks.
	- Fourth: remove RestAPI routers code, check again, make sure nothing breaks.
