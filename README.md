
## Edge Load Balancer
![](assets/edge-loadbalancer.png)

<br>
In this example, the client will monitor the data 'test-data' topic from an edge load balancer server. 
The load balancer will fetch the data from 3 sources (server1, server2, and server3) one at a time using a simple round robin method.   
It will then send the fetched data one at a time to the client or to every connected clients. 


If one of the load balancer 3 sources of data goes down due to some unexpected events, it will continue to provide the data from the other sources.    

#### Create a project directory for each edge endpoint and install *m2m*.
```js
$ npm install m2m
```
### Server 1
#### Save the code below as *app.js* in your server1 project directory.

```js
const m2m = require('m2m')

let edge = new m2m.Edge({name:'server1'})

// simulated data source
function dataSource(){
  return 20 + Math.floor(Math.random() * 10)
}

m2m.connect(() => {

  let port = 8144

  edge.createServer(port, '127.0.0.1', (server) => {

    server.on('error', (error) => { 
      console.log('error:', error)
    })

    server.dataSource('test-data', (tcp) => {
      tcp.send({server:1, topic:tcp.topic, value:dataSource()})         
    })
  })
})
```
### Server 2
#### Save the code below as *app.js* in your server2 project directory.

```js
const m2m = require('m2m')

let edge = new m2m.Edge({name:'server2'})

// simulated data source
function dataSource(){
  return 20 + Math.floor(Math.random() * 10)
}

m2m.connect(() => {

  let port = 8145

  edge.createServer(port, '127.0.0.1', (server) => {

    server.on('error', (error) => { 
      console.log('error:', error)
    })

    server.dataSource('test-data', (tcp) => {
      tcp.send({server:2, topic:tcp.topic, value:dataSource()})             
    })
  })
})
```
### Server 3

#### Save the code below as *app.js* in your server3 project directory.
```js
const m2m = require('m2m')

let edge = new m2m.Edge({name:'server3'})

// simulated data source
function dataSource(){
  return 20 + Math.floor(Math.random() * 10)
}

m2m.connect(() => {

  let port = 8146

  edge.createServer(port, '127.0.0.1', (server) => {

    server.on('error', (error) => { 
      console.log('error:', error)
    })

    server.dataSource('test-data', (tcp) => {
      tcp.send({server:3, topic:tcp.topic, value:dataSource()})       
    })
  })
})
```
### Load Balancer

#### Save the code below as *app.js* in your load balancer project directory.
```js
const m2m = require('m2m')

let edge = new m2m.Edge({name:'Load Balancer'})

m2m.connect(() => {

  let ec1 = new edge.client(8144, '127.0.0.1')
  let ec2 = new edge.client(8145, '127.0.0.1')
  let ec3 = new edge.client(8146, '127.0.0.1')

  let port = 8143, data = null, round = 1  

  edge.createServer(port, '127.0.0.1', (server) => {

    server.on('connection', (count) => { 
      console.log('connected client', count)
    })

    server.publish('test-data', async (tcp) => {
      if(round === 1){
        round = 2
        data = await ec1.read(tcp.topic)
      }
      else if(round === 2){
        round = 3
        data = await ec2.read(tcp.topic)
      }
      else if(round === 3){
        round = 1
        data = await ec3.read(tcp.topic)
      }
      tcp.send(data)
    })
  })
})
```
### Client

#### Save the code below as *app.js* in your client project directory.
```js
const m2m = require('m2m')

let edge = new m2m.Edge({name:'client'})

async function main (){

  await m2m.connect();

  let ec = new edge.client(8143, '127.0.0.1')

  ec.on('ready', (result) => { 
    console.log('ready:', result)
  })

  ec.on('error', (error) => { 
    console.log('error:', error)
  })

  ec.subscribe('test-data', (data) => {
    console.log(data)
  })
}
```

<br>

#### Start the application on each endpoint.
```js
$ node app.js
```

<br>

You should get a client output similar to the result as shown below.
```js
ready: true
{ server: 1, topic: 'test-data', value: 27 }
{ server: 2, topic: 'test-data', value: 25 }
{ server: 3, topic: 'test-data', value: 22 }
...


```


