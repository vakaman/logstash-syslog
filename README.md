```
# Fist we need to create a folder, to save all logs
mkdir /var/log/logstash

# Adjust permission
chmod -R 1000:0 /var/log/logstash

# Up logstash
docker-compose up -d 
```
