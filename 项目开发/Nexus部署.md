```
version: '3'
services:
  nexus:
    image: sonatype/nexus3
    container_name: nexus
    restart: always
   
    ports:
      - 7081:8081
    volumes:
      - ./data:/nexus-data
```

