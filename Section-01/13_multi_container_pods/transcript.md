# Transcript


In this video we're going to build our first multicontainer pod. In our demo, we'll have 2 containers running inside a single pod. 

In this scenario, the first container you define in your yaml file, is referred to as the main containers, and all the other containers are referred to as supporting containers. 


commands where you need to specify a container name, e.g. logs commands, if you omit the -c flag, then the command will default to using the main container's log. 