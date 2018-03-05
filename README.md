[![Build Status](https://travis-ci.org/SolaceLabs/pcf-car-demo.svg?branch=master)](https://travis-ci.org/SolaceLabs/pcf-car-demo)


# Tracking the Movement and Maintenance of Connected Cars
 **An IoT Microservice demo using Cloud Foundry and Solace Messaging**

This demo simulates a connected vehicle tracking scenario a lot like something we’ve helped one of our customers deploy. In this case, we’ll be tracking car sensor data to show different kinds of maintenance events and faults, and can give individual users the ability to be “tracked” and manually log maintenance faults to see them reflected on the monitoring side of things.

## **Monitoring Map**

The enterprise end of this equation is a map of London that displays the location and condition of any number of computer- and user-generated cars. The demo simulates the tracking of their location, because the demo is generally delivered to groups of people sitting in a room, which would make for an uninteresting, static and cluttered map.

![](http://dev.solace.com/wp-content/uploads/2017/04/pivotal-car-markers-mocked-up-768x434.jpg)

## **Car Condition App**

The demo also includes a mobile app that people can use to place themselves, in the form of a car marked with their name, on the map among the computer-generated cars. The app gives users the ability to cause and repair four different faults: front flat tire, rear flat tire, engine trouble and busted taillight.  

The point is to track the health of each car by sending failures, and fixes, to our backend system, which captures the events and displays them on the Google map. When a user causes a fault their map marker turns red and indicates which of the four faults they generated, along with a “Fix it” button with which they can easily clear the fault. Each of these interactions represent events that flow between the car and and/or roadside emergencies.

![](http://dev.solace.com/wp-content/uploads/2017/04/smartphone-car-flat-rear-768x382.png)

## **Behind the Scene**

This is the high-level scaling architecture for the connected car scenario. Solace runs native in all the clouds, in this case the IaaS/PaaS combination of Pivotal Cloud Foundry running in Google Cloud Platform. The cars connect, they start streaming their sensor data, and the elastic runtime does the work to scale up and down as necessary. All of these components support the open protocols we talked about before, and they can be auto configured for HA, DR and the kind of reliability you’d expect for enterprise-grade applications.

![](http://dev.solace.com/wp-content/uploads/2017/04/pcf-car-demo-architecture-768x407.png)


### **Get Started**

```
git clone https://github.com/SolaceLabs/pcf-car-demo.git
cd pcf-car-demo
./gradlew assemble
cf push
```


## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors

See the list of [contributors](https://github.com/SolaceLabs/pcf-car-demo/graphs/contributors) who participated in this project.

## License

This project is licensed under the Apache License, Version 2.0. - See the [LICENSE](LICENSE) file for details.

## References

Here are some interesting links if you're new to these concepts:

* [The Solace Developer Portal](http://dev.solace.com/)
* [Solace Labs](http://dev.solace.com/labs/)
* [Solace on Pivotal Cloud Foundry](http://dev.solace.com/clouds/pivotal-cloud-foundry/)
* [Martin Fowler on Microservices](http://martinfowler.com/articles/microservices.html)
* [A intro to Microservices Design Patterns](http://blog.arungupta.me/microservice-design-patterns/)
* [Horizontal Scaling of Aggregated Microservices Using Solace Messaging in Cloud Foundry](http://dev.solace.com/solution-demos/horizontal-scaling-aggregated-microservices/)
* [REST vs Messaging for Microservices](http://www.slideshare.net/ewolff/rest-vs-messaging-for-microservices)
* [Pivotal Network](https://network.pivotal.io/)

# How to run this demo outside of PCF

This demo is actually 3 spring boot applications, 2 out the 3 applications,  simply serve web applications.

* `cargen`: a spring boot application that keeps the car map up to date and reacts to events from the cardemo application
* `carmap`: serves the car map web application over port 8080
* `cardemo`: serves the car demo web application over port 9090

The flow of events is as follows:
```
cardemo -- MQTT --> solace -- SMF -> cargen 

cargent -- SMF --> solace -- solace web messaging --> car map
```

## Step 1: Configure and run cargen

*  Update the Solace Messaging coordinates in `cargen/src/main/resources/application.properties` file
```
solace.java.host=<Solace Messaging Host or IP>:<port>
solace.java.msgVpn=<Solace message VPN name>
solace.java.clientUsername=<Client UserName>
solace.java.clientPassword=<Client Password>
server.port=7070

```

For example (using Solace Cloud )
```
solace.java.host=mr-91b692du2x.messaging.solace.cloud:20256
solace.java.msgVpn=msgvpn-91b692imr1
solace.java.clientUsername=solace-cloud-client
solace.java.clientPassword=secret
```

* From a shell in the `cargen` directory
```
$ ../gradlew build

```
Then
```
$ ../gradlew bootRun

```
You should now have the cargen spring boot application running on port 7070

## Step 2: Configure and run carmap

This application is nothing but a web server serving the car map web application.

* To configure the web application you must update the Solace Messaging coordinates in `/carmap/src/main/resources/static/TransportProperties.js` with
```
TRANSPORT_PROPERTIES = {
    host : "<Solace Messaging Host or IP>:<port>",
    vpn  : "<Solace message VPN name>",
    user : "<Client UserName>",
    pw   : "<Client Password>",
}
```

For example (using Solace Cloud: Web Messaging Transport `ws` )
```
TRANSPORT_PROPERTIES = {
    host : "mr-91b692du2x.messaging.solace.cloud:20259",
    vpn  : "msgvpn-91b692imr1",
    user : "solace-cloud-client",
    pw   : "secret",
}
```

* From a shell in the `carmap` directory:

```
$ ../gradlew build
```

Then
```
$ ../gradlew bootRun

```
You should now have the carmap spring boot application running on port 8080 [car map](http://localhost:8080). Go to `http://localost:8080` to see the map


## Step 3: Configure and run cardemo

This application is nothing but a web server serving the car demo web application.

* To configure the web application you must update the Solace Messaging coordinates in `cardemo/src/main/resources/static/js/build.js` with
```
    var TRANSPORT_PROPERTIES = {
        host : "<Solace Messaging Host or IP>",
        port : <port>
        vpn  : "<Solace message VPN name>",
        user : "<Client UserName>",
        pw   : "<Client Password>",
    }
```


For example (using Solace Cloud: web socket MQTT `ws` )
```
    var TRANSPORT_PROPERTIES = {
        host : "mr-91b692du2x.messaging.solace.cloud",
        port:  20264,
        vpn  : "msgvpn-91b692imr1",
        user : "solace-cloud-client",
        pw   : "secret",
    }
```

* From a shell in the `carmap` directory
```
$ ../gradlew build

```
Then
```
$ ../gradlew bootRun

```
You should now have the carmap spring boot application running on port 9090 [car demo](http://localhost:9090). Go to `http://localost:9090
` to see the map



