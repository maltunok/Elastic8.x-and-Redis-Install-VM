#INSTALL REDIS

sudo apt install net-tools
sudo apt update
sudo apt install redis-server
sudo nano /etc/redis/redis.conf
# ctrl+w supervised - write systemd
sudo systemctl restart redis.service
sudo systemctl status redis
redis-cli
ping
set test "devbase"
get test
exit
sudo systemctl restart redis
redis-cli
get test
sudo nano /etc/redis/redis.conf
# ctrl+w bind, remove #
sudo systemctl restart redis
sudo systemctl status redis
sudo netstat -lnp | grep redis

### Configuring a Redis Password
sudo nano /etc/redis/redis.conf
. . .
# requirepass foobared
. . .
# Uncomment it by removing the #, and change foobared to a secure password.

openssl rand 60 | openssl base64 -A

Output
RBOJ9cCNoGCKhlEBwQLHri1g+atWgn4Xn4HwNUbtzoVxAYxkiYBi7aufl4MILv1nxBqR4L6NNzI0X6cE

/etc/redis/redis.conf
requirepass RBOJ9cCNoGCKhlEBwQLHri1g+atWgn4Xn4HwNUbtzoVxAYxkiYBi7aufl4MILv1nxBqR4L6NNzI0X6cE

sudo systemctl restart redis
redis-cli

set key1 10
Output
(error) NOAUTH Authentication required.

auth your_redis_password

Output
OK

set key1 10

get key1
quit