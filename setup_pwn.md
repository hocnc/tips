# Normal
setup.sh
```
#!/bin/bash

apt-get install xinetd

user=bof
binary=bof
port=31335

useradd $user
mkdir /home/$user
cp $binary /home/$user/
chown -R root:root /home/$user
chown root:$user /home/$user/$binary
chmod 2755 /home/$user/$binary

echo "Flag{ok}" > /home/$user/flag
chown root:$user /home/$user/flag && chmod 440 /home/$user/flag;

cat <<EOF > /etc/xinetd.d/$user
service $user
{
    disable = no
    socket_type = stream
    protocol    = tcp
    wait        = no
    user        = $user
    bind        = 0.0.0.0
    server      = /home/$user/$binary
    type        = UNLISTED
    port        = $port
    flags = REUSE
    per_source = 5
    rlimit_cpu = 3
    nice = 18
}
EOF

service xinetd restart
```

# Docker
setup_docker.sh
```
#!/bin/bash


user=bof
binary=bof
port=1028
eof=EOF

# Create setup.sh

cat <<EOF > setup.sh
#!/bin/bash


cat <<EOF > /etc/xinetd.d/$user
service $user
{
    disable = no
    socket_type = stream
    protocol    = tcp
    wait        = no
    user        = $user
    bind        = 0.0.0.0
    server      = /home/$user/$binary
    type        = UNLISTED
    port        = $port
    flags = REUSE
    per_source = 5
    rlimit_cpu = 3
    nice = 18
}
$eof

service xinetd restart
tail -f /dev/null
EOF

#Create Dockerfile

cat <<EOF > Dockerfile
FROM i386/ubuntu:latest 

RUN apt-get update && apt-get install -y xinetd

RUN useradd -d /home/$user/ -m -p $user -s /bin/bash $user
RUN echo "$user:P@ssw0rd" | chpasswd

WORKDIR /home/$user

COPY setup.sh setup.sh
COPY $binary .

RUN chown -R root:root /home/$user
RUN chown root:$user /home/$user/$binary
RUN chmod 2755 /home/$user/$binary

RUN echo "flag{ok}" > /home/$user/flag.txt
RUN chown root:$user /home/$user/flag.txt && chmod 440 /home/$user/flag.txt

WORKDIR /home/$user
EXPOSE $port
RUN chmod +x setup.sh
CMD ["/home/$user/setup.sh"]
EOF

docker build -t $user .
docker run -d -p $port:$port --name $user --rm -it $user
```
rmi_docker.sh
```
#!/bin/bash

user=bof

docker stop $user
docker rmi $user
```

# Source
https://github.com/l4wio/CTF-challenges-by-me