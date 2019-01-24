# Develop an Atlassian plugin

References:
- https://developer.atlassian.com/server/framework/atlassian-sdk/
- https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora

## Installation

SETUP: Windows 10 host + Cent OS 7 guest on Hyper-V

Create the Cent OS 7 VM in Hyper-V, then...
```bash
# Create a shared folder between host and guest
# - Windows: C:\Shared\altas-plugins
# - Linux: $HOME/atlas-plugins -> /mnt/atlas-plugins
sudo yum install cifs-utils -y
o="domain=hq,username=crozier,uid=1000,gid=1000,vers=2.1"
sudo mount.cifs -o $o //169.254.54.66/atlas-plugins /mnt/atlas-plugins
ln -s /mnt/atlas-plugins # From $HOME

# Install Oracle JAVA JDK 8 on Cent OS 7 + JAVA_HOME
# - First download jdk-8u191-linux-x64.rpm, then...
curl -OL -H "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" https://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.rpm
sudo yum install jdk-8u191-linux-x64.rpm -y
rm jdk-8u191-linux-x64.rpm
sudo sh -c "echo export JAVA_HOME=/usr/java/default/jre >> /etc/environment"
# Logout/Login then check JAVA version
java -version  # JRE (Check $PATH)
javac -version # JDK ($PATH includes $JAVA_HOME/bin)

# Install Atlassian SDK
cat <<EOF > /etc/yum.repos.d/artifactory.repo
[Artifactory]
name=Artifactory
baseurl=https://packages.atlassian.com/atlassian-sdk-rpm/
enabled=1
gpgcheck=0
EOF

sudo yum clean all
sudo yum updateinfo metadata
sudo yum install atlassian-plugin-sdk -y

atlas-version # Check java version ($PATH) and home ($JAVA_HOME) -> JDK 1.8.0

# Update the JAVA cacerts with firewall.hq.k.grp.cer (password = changeit)
sudo /usr/java/default/bin/keytool --import -alias firewall.hq.k.grp -keystore /usr/java/default/jre/lib/security/cacerts -file firewall.hq.k.grp.cer
```

## Setup

```bash
# Optional: Copy settings.xml into $HOME/.m2/
cfg=/usr/share/atlassian-plugin-sdk-6.3.12/apache-maven-3.2.1/conf/settings.xml
mkdir $HOME/.m2; cd $HOME/.m2; ln -s $cfg; cd

# Create the initial structure for JIRA
cd atlas-plugins/
atlas-create-jira-plugin
: com.nagra.dtv
: jira-sample-plugin

# Create an empty module with a plugin descriptor, then
# Launch JIRA in debug mode
cd jira-sample-plugin/
atlas-create-jira-plugin-module
atlas-debug
```

Open http://$centosip:2990/jira
```bash
# As root: Open PORTs
sudo firewall-cmd --add-port=1990/tcp --permanent
sudo firewall-cmd --add-port=2990/tcp --permanent
sudo firewall-cmd --add-port=5005/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
sudo iptables -S | grep -E "1990|2990|5005"
#iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 2990 -j ACCEPT
#iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 5005 -j ACCEPT
```
