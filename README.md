# RefBee

A little busy bee to display where the authors publications are listed, e.g. on Google Scholar, DBLP, ...
Depends on this [bee hive](https://github.com/kmdn/ResearcherPublicationMapping.git) as backend.

## Build and run using Docker
```
docker build -t refbee-v:latest .
docker run -d -p 80:80 refbee-v:latest
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

