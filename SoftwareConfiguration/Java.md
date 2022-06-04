# Java Enviroment

## Ubuntu 20.04

```shell
sudo apt install openjdk-8-jdk
# Checking Java versions installed on Ubuntu / Debian
sudo update-java-alternatives --list
# Set default Java version on Ubuntu / Debian
sudo update-alternatives --config java
# Set default Javac version on Ubuntu / Debian
sudo  update-alternatives --config javac
# Set JAVA_HOME
export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")
```








