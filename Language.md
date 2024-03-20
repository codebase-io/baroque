# Baroque Language (yes, we could be whole programming language)

I don't know if this could be implemented, however if I would to 
do a programming language for Baroque this is how it would look like: 

```
> baroque(routine/0.1)

# Declarations 
# > var_name as type(value)
# > def var_name as value using type(width)
> a as int(64)  
> b as int(1024)

# Execution
return a + b
```

Each file is a routine which may be executed independently, and returns a single result.
Supported type are the same defined for the format.

### More examples

#### While 
```
> baroque(routine/0.1)

> def a as int(22) 
> def b as int(100)
> def number as 23846 using int(32)
> def email as "admin@localhost" using string(email)

# Instructions
while a > b 
	print email
	break
```

Will return 122;

#### MySQL database query, and shuffle through the results 
```
> baroque(routine/0.1)

# query context is inferred on the fly as we're using a 
# mysql connection, so it will call mysql.query(...)
> connect mysql server with user:baroque password:"admin" charset:'utf-8' as defaultConn
> result as query('SELECT * FROM users') using defaultConn 

while user in result 
	print firstName of user
```

#### Consuming http json endpoint,
```
> baroque(routine/0.1)

# Declarations
> max_points as int(0)
> endpoint as string('http://restdb.io/list.json')
> data as get(endpoint) using http

# Instructions
while record in data 
	print firstName of record
	
	if points of record > 10 
		set max_points to points
		
		
return max_points	

```

#### Http server example

Return statement is ommited for brevity

```
> baroque(routine/0.1)

> server(http/1.0)
> serve 'text/html'
> message as string('It works!')

<html>
	<body>${message}</body>
</html>
```

### Http server example executing another routine

The routine will be executed and result put in message

```
> baroque(routine/0.1)

> server(http/1.0)
> serve 'text/html'
> message as routine(baroque:./another/routine.brq) 

<html>
	<body>${message}</body>
</html>
```

### First class html types

```
> baroque(routine/0.1)

> server(http/2.0)
> header('Content-Type', 'text/html')
> header('Cache-Control', 'private, max-age:0')
> connect redis server with user:baroque password:"redis" as redisConn
> sessionId as get('user.sessionId') using redis

<html>
	<body>Your session id is ${sessionId}</body>
</html>
```