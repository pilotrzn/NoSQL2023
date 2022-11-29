docker network create mongoCluster



  docker run --name mongodb -d \
-v ~/docker/mongodb:/data/db \
-p 27017:27017 \
--replSet RS \
--network mongoCluster \
-e MONGO_INITDB_ROOT_USERNAME=alex \
-e MONGO_INITDB_ROOT_PASSWORD=alexpass \
mongo
      
docker run --name mongodb -d \
-v ~/docker/mongo2:/data/db \
-p 27018:27017 \
--replSet RS \
--network mongoCluster \
-e MONGO_INITDB_ROOT_USERNAME=alex \
-e MONGO_INITDB_ROOT_PASSWORD=alexpass \
mongo


https://www.mongodb.com/compatibility/deploying-a-mongodb-cluster-with-docker

