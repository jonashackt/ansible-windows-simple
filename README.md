# ansible-windows-talk
Talk materials, links, snippets and a simple ansible playbook as showcase

Link to the Slides: [Running Spring Boot Apps on Docker Windows Containers with Ansible.pdf](https://github.com/jonashackt/ansible-windows-talk/blob/master/Running%20Spring%20Boot%20Apps%20on%20Docker%20Windows%20Containers%20with%20Ansible.pdf)

### Repositories needed throughout the talk

https://github.com/jonashackt/ansible-windows-docker-springboot

https://github.com/jonashackt/cxf-spring-cloud-netflix-docker

### Links to dive in deeper

https://blog.codecentric.de/en/2017/01/ansible-windows-spring-boot/

https://blog.codecentric.de/en/2017/04/ansible-docker-windows-containers-spring-boot/

https://blog.codecentric.de/en/2017/05/ansible-docker-windows-containers-scaling-spring-cloud-netflix-docker-compose/

https://docs.microsoft.com/en-us/virtualization/windowscontainers/index

# Talk snippets

## Prerequisites (1. Windows box)

> This is a step that requires to download ~10GB and takes quite long to complete - if you want to follow the demo live @ the talk, you should prepare the hole step at home.

#### a) install Virtual Box, Vagrant and Packer

###### if you´re on a Mac, this can easily be accomplished via [brew](https://brew.sh/index_de.html)
* `brew cask install virtualbox` 
* `brew cask install vagrant`
* `brew install packer`

###### on Windows, take [chocolatey](https://chocolatey.org/)
* `choco install virtualbox`
* `choco install vagrant`
* `choco install packer` 

###### on Linux - just use your favourite package manager

#### b) Download ISO

https://www.microsoft.com/de-de/evalcenter/evaluate-windows-server-2016 (registration needed)

#### c) Build your Vagrant Box with Packer

Clone this GitHub repo [ansible-windows-docker-springboot](https://github.com/jonashackt/ansible-windows-docker-springboot), cd into it and the subfolder `step0-packer-windows-vagrantbox`. Then run:

```
packer build -var iso_url=14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_EN-US.ISO -var iso_checksum=70721288bbcdfe3239d8f8c0fae55f1f windows_server_2016_docker.json
```

#### d) Add the Vagrant box and run it
```
vagrant init windows_2016_docker_virtualbox.box 
```

Now fire up your Windows Server 2016 box:
```
vagrant up
```

> You can check if everything is ok as a last step if you cd into [ansible-windows-simple](https://github.com/jonashackt/ansible-windows-talk/tree/master/ansible-windows-simple) and run a `ansible windows-dev -i hostsfile -m win_ping` - which should give an `SUCCESS` 

Find more info here: https://github.com/jonashackt/ansible-windows-docker-springboot#build-your-windows-server-2016-vagrant-box


## 2. Ansible provisions Windows

cd into [ansible-windows-simple](https://github.com/jonashackt/ansible-windows-talk/tree/master/ansible-windows-simple) and test the connection first:

```
ansible windows-dev -i hostsfile -m win_ping
```

Then run the playbook:

```
ansible-playbook -i hostsfile windows-playbook.yml --extra-vars "host=windows-dev"
```


## 3. Prepare Docker on Windows

All the preparing playbooks can be found here: [step1-prepare-docker-windows](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step1-prepare-docker-windows/)

```
ansible-playbook -i hostsfile prepare-docker-windows.yml --extra-vars "host=ansible-windows-docker-springboot-dev"
```

```
docker run --name dotnetbot microsoft/dotnet-samples:dotnetapp-nanoserver
```

## 4. Run Spring Boot App on Docker Windows Container

Clone simple Spring Boot app [weatherbackend](https://github.com/jonashackt/spring-cloud-netflix-docker/tree/master/weatherbackend) & do a:
```
mvn clean package
```

Then cd into weatherbackend/target

```
java -jar weatherbackend-0.0.1-SNAPSHOT.jar
```

cd into [step2-single-spring-boot-app](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step2-single-spring-boot-app/) and run the playbook:

```
ansible-playbook -i hostsfile ansible-windows-docker-springboot.yml --extra-vars "host=ansible-windows-docker-springboot-dev app_name=weatherbackend jar_input_path=../../cxf-spring-cloud-netflix-docker/weatherbackend/target/weatherbackend-0.0.1-SNAPSHOT.jar"
```

## 5. Scale Spring Boot Apps

Example project [cxf-spring-cloud-netflix-docker](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker)

Spring Cloud docs: http://cloud.spring.io/spring-cloud-static/Dalston.RELEASE/

###### Zuul dynamic routing

zuul-edgeservice [application.yml](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker/blob/master/zuul-edgeservice/src/main/resources/application.yml)

###### Run playbook

cd into [step3-multiple-spring-boot-apps-docker-compose](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step3-multiple-spring-boot-apps-docker-compose/)

```
ansible-playbook -i hostsfile ansible-windows-docker-springboot.yml --extra-vars "host=ansible-windows-docker-springboot-dev"
```

###### Healthcheck

Spring Boot/Cloud [about well written clients](https://stackoverflow.com/a/42352258/4964553) 


###### Peer-awareness

Eureka [application.yml](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker/blob/master/eureka-serviceregistry/src/main/resources/application.yml)

###### Run final weatherclient test

cd into [weatherclient](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker/tree/master/weatherclient) and run

```
java -jar target/weatherclient-0.0.1-SNAPSHOT.jar
```




